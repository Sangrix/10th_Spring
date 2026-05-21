- Page와 Slice
    
    > Spring Data JPA에서 대량의 데이터를 나누어 조회하는 **페이징**처리를 할 때 사용하는 대표적인 반환 타입들
    > 
    
    **[특징]**
    
    **공통점**
    
    - 둘은 대량의 데이터를 청크 단위로 나눠 가져온다
    
    **차이점**
    
    - 전체 데이터의 개수를 알고 있는지
    - 동작하는 쿼리의 매커니즘
    
    **[Page]**
    
    - 페이징 처리를 위한 **가장 표준적인 반환 타입**
    - “현재 페이지의 데이터 + 전체 데이터의 총 개수” 까지 함께 조회
    - **제공하는 정보**: 총 페이지 개수(`getTotalPages()`), 전체 데이터 수(`getTotalElements()`), 현재 페이지 번호, 다음/이전 페이지 여부 등
    
    - **동작 매커니즘**: 데이터를 가져오는 `select` 쿼리 외에, 전체 레코드 수를 파악하기 위한 `SELECT COUNT(*)` 쿼리가 추가로 실행
    - **User Interface**: 게시판 하단의 페이징 네비게이션 바 “[1], [2], [3] … [다음]” 형태를 구현하는데 필수적
    - **주의점**: data가 수백만 건 이상으로 많아지면 `COUNT` 쿼리 자체가 DB에 큰 부하를 줘서 성능 저하의 원인이 될 수 있음
    
    **[Slice]**
    
    - **오직 다음 슬라이스(페이지)의 존재 여부만 확인**하는 반환 타입
    - “전체 데이터 개수를 조회하지 않는다.” → 총 페이지 수는 알 수 없음
    - **제공하는 정보**: 현재 페이지의 데이터, 다음 페이지의 존재 여부(`hashNext()`), 현재 슬라이스가 첫 번째/마지막인지 여부 등
    
    - **동작 매커니즘**: 사용자가 요청한 페이지 size가 N개라면, DB에 의도적으로 **N+1개의 데이터**를 조회
    
    이때, N+1번째 data가 존재한다면 다음 페이지가 있다고 판단하고, 반환 시에는 N개만 잘라서 반환하며 `COUNT` 쿼리가 나가지 않음
    - **User Interface**: 무한 스크롤이나 더보기 버튼 구현 시 적합
    - **장점**: `COUNT` 쿼리가 생략되므로 data 양이 많을 때 성능상 유리
    
    **[예시 코드]**
    
    ```java
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.PageRequest;
    import org.springframework.data.domain.Pageable;
    import org.springframework.data.domain.Slice;
    import org.springframework.data.domain.Sort;
    import org.springframework.stereotype.Service;
    import org.springframework.transaction.annotation.Transactional;
    
    import java.util.List;
    
    @Service
    @Transactional(readOnly = true)
    public class MemberService {
    
        private final MemberRepository memberRepository;
    
        public MemberService(MemberRepository memberRepository) {
            this.memberRepository = memberRepository;
        }
    
        public void runPaginationExample() {
            int age = 20;
            // 페이지 번호는 0부터 시작, 한 페이지에 10개씩, ID 내림차순 정렬
            Pageable pageable = PageRequest.of(0, 10, Sort.by(Sort.Direction.DESC, "id"));
    
            // ==================== [ Page 예시 ] ====================
            Page<Member> page = memberRepository.findPageByAge(age, pageable);
    
            List<Member> pageContent = page.getContent(); // 실제 조회된 데이터 
            long totalElements = page.getTotalElements(); // 전체 데이터 수
            int totalPages = page.getTotalPages();        // 전체 페이지 수
            int pageNumber = page.getNumber();            // 현재 페이지 번호
            boolean hasNextPage = page.hasNext();         // 다음 페이지가 있는지 여부
    
            // ==================== [ Slice 예시 ] ====================
            Slice<Member> slice = memberRepository.findSliceByAge(age, pageable);
    
            List<Member> sliceContent = slice.getContent(); // 실제 조회된 데이터
            int currentSize = slice.getSize();             // 요청한 페이지 사이즈
            int sliceNumber = slice.getNumber();           // 현재 슬라이스 번호
            boolean hasNextSlice = slice.hasNext();         // 다음 슬라이스가 있는지 여부
            
            // Slice는 전체 개수를 모르기 때문에 아래 메서드들을 지원하지 않는다
            // slice.getTotalElements(); // 컴파일 에러
            // slice.getTotalPages();    // 컴파일 에러
        }
    }
    ```
    
    **[실제 SQL 쿼리의 차이점]**
    
    `Page` 매서드 호출
    
    ```sql
    -- 1) 데이터 조회 쿼리 (LIMIT 10 OFFSET 0)
    select member0_.id, member0_.age, member0_.name 
    from member member0_ 
    where member0_.age=20 
    order by member0_.id desc 
    limit 10;
    
    -- 2) 총 개수 조회 쿼리 (추가 실행됨)
    select count(member0_.id) 
    from member member0_ 
    where member0_.age=20;
    ```
    
    `Slice` 매서드 호출
    
    ```sql
    -- 1) 데이터 조회 쿼리 (의도적으로 LIMIT를 N+1인 11로 설정)
    select member0_.id, member0_.age, member0_.name 
    from member member0_ 
    where member0_.age=20 
    order by member0_.id desc 
    limit 11; -- <--- 중요! 다음 페이지가 있는지 보려고 11개를 가져옵니다.
    
    -- (count 쿼리는 실행되지 않음)
    ```
    
- Java stream API
    
    > Java 8에 도입된 기능으로 **배열이나 컬렉션(List, Set, Map 등)의 데이터**를 **함수형 프로그래밍 스타일**로 선언적이고 간결하게 처리할 수 있도록 해주는 API이다.
    > 
    
    > 기존의 `for` 문이나 `iterator` 를 사용하는 **외부 반복 방식**과 달리, **데이터 소스로부터 스트림을 생성**해 내부적으로 **loop를 돌며 데이터를 처리**한다.
    > 
    
    **[Stream의 파이프라인 구조]**
    
    Stream 연산은 여러 단계가 연결된 Pipeline 구조를 가진다 (마치, 컴퓨터구조에서 MIPS instruction pipeline 같이..)
    
    1. **스트림 생성**
        
        컬렉션, 배열, 파일 등의 데이터 소스로부터 스트림을 생성한다.
        
        ex) `list.stream()` , `Array.stream(array)`
        
    2. **중간 연산**
        
        데이터를 가공, 필터링, 변환한다.
        
        중간 연산은 즉시 실행되지 않고, 지연 연산(Lazy Evaluaion)된다.
        
        연산 결과로 다시 Stream을 반환하기 때문에, 여러 중간 연산을 체이닝할 수 있다.
        
        ex) 중간 연산 매서드들
        
        ```markdown
        `filter()`: 주어진 조건을 만족하는 요소만 골라내어 다음 스트림으로 넘김
        `map()`: 스트림의 요소들을 하나씩 특정 값이나, 다른 형태로 변환(Mapping)할 때 사용
        `sorted()`: 스트림의 요소들을 정렬, 기본값 오름차순
        `distinct()`: 스트림에서 중복 요소를 제거
        `limit()`: 앞부터 지정한 개수만큼 잘라서 다음 스트림으로 반환
        ```
        
    3. **최종 연산**
        
        파이프라인을 닫고 실제로 연산을 수행하여 결과를 도출한다.
        
        최종 연산이 호출되어야지만 앞에 묶여 있던 중간 연산들이 비로소 실행된다.
        
        **스트림**은 최종 연산 이후 **소모되어 재사용할 수 없다.** 
        
        ex) 최종 연산 매서드들
        
        ```markdown
        `collect()`: 새로운 컬렉션(List, Set, Map)이나 다른 형태로 수집할 때 사용
        `forEach()`: 스트림의 각 요소를 순회하며 특정 작업 수행할 때 사용, 반환값이 void
        `reduce()`: 스트림의 모든 요소를 하나의 값으로 집적, 축소할 때 사용
        `count()`: 스트림에 남아있는 요소의 총 개수를 반환, 반환 타입은 long
        `anyMatch()`: 스트림의 요소 중 하나라도 조건을 만족하는지 확인, 반환 타입은 bollean
        ```
        
    
    **[외부 반복 방식과 Stream API 방식 비교]**
    
    // 문자열 리스트에서 길이가 5 이상인 단어만 골라 대문자로 변환한 뒤, 정렬하여 리스트로 저장
    
    **기존 방식(for-each)**
    
    ```java
    List<String> words = Arrays.asList("apple", "banana", "cherry", "date");
    List<String> result = new ArrayList<>();
    
    for (String word : words) {
        if (word.length() >= 5) {
            result.add(word.toUpperCase());
        }
    }
    Collections.sort(result);List<String> words = Arrays.asList("apple", "banana", "cherry", "date");
    List<String> result = new ArrayList<>();
    
    for (String word : words) {
        if (word.length() >= 5) {
            result.add(word.toUpperCase());
        }
    }
    Collections.sort(result);
    ```
    
    **Stream API 방식**
    
    ```java
    List<String> words = Arrays.asList("apple", "banana", "cherry", "date");
    
    List<String> result = words.stream()                    // 1. 스트림 생성
        .filter(word -> word.length() >= 5)                 // 2. 중간 연산: 필터링
        .map(String::toUpperCase)                           // 2. 중간 연산: 대문자 변환
        .sorted()                                           // 2. 중간 연산: 정렬
        .collect(Collectors.toList());                      // 3. 최종 연산: 결과 수집
    ```
    
    **[Stream API의 주요 특징]**
    
    - **Read-Only**: 데이터 원본을 수정하지 않는다.
    - **일회용**: 스트림은 재사용되지 않는다.
    - **Lazy Evaluation**: 지연연산으로 최종 연산 수행되기 전까지 중간 연산은 실행되지 않는다.
    
- 객체 그래프 탐색
    
    > 하나의 root 객체에서 시작하여, 연결된 참조 관계를 따라 다른 관련 객체들을 찾아가는 과정
    > 
    
    주문(Order), 회원(Member), 배송(Delivery) 객체들이 서로 연관 관계를 가정
    
    ```java
    public class Order {
        private Member member;
        private Delivery delivery;
    
        public Member getMember() { return member; }
        public Delivery getDelivery() { return delivery; }
    }
    ```
    
    이 상태에서 `Order` 객체를 가지고 있을 때, `.` 연산자나 `getter` 를 통해 연결된 다른 객체에 접근하는 행위가 그래프 탐색
    
    `// String memberName = order.getMember().getName();`
    
    `// String address = order.getDelivery().getAddress();`
    
    **[DB 환경에서 중요성]**
    
    객체는 그래프를 통해 탐색이 가능하지만, DB 테이블은 FK를 사용해서 `JOIN` 해야만 다른 테이블에 접근할 수 있는 **“객체와 DB의 패러다임 불일치”**가 생긴다.
    
    이를 해결하기 위해 JPA에선 여러 개념을 사용한다.
    
    - **지연로딩**과 **즉시로딩**
        - 하나의 엔티티 조회 시, 연관된 데이터를 어느 시점에 가져올 것인지
    - **객체 그래프 탐색의 한계**와 **NullPointException**, **ProxyException**
        - JPA 사용시 “어디까지 객체 그래프를 탐색할 수 있는가?”는 영속성 컨텍스트의 생명주기에 따라 결정된다.
        - 지연 로딩으로 설정, 트랜잭션이나 영속성 컨텍스트가 종료된 후에 객체 그래프 탐색 시도시 → **LazyInitializationException**이 발생
        - 연관관계가 맺어지지 않은 상태에서 그래프 탐색을 시도 → **NullPointException**이 발생
        
- @Valid vs @Validated
    
    > Spring Boot에서 유효성 검증(Validation)을 처리할 때 사용되는 어노테이션
    > 
    
    **[@Vaild]**
    
    - `@Valid`는 자바 표준 스펙인 “**Jakarta” Bean Vaildation**에 포함된 어노테이션
    - **특징**: 표준 스펙이기 때문에 기술 종속성이 낮고, 객체 내부의 다른 객체를 함께 검증하는 계층 구조 검증시 멤버 변수 위에 붙여서 사용
    
    - **동작 위치**: 주로 Controller 레이어에서 HTTP요청 바디(Request Body)등의 객체를 검증할 때 사용
    - **디스패터 서블릿 기능**: Spring MVC 엔진인 `DispatcherServlet` 이 컨트롤러의 매서드를 호출하는 과정에서 `ArgumentResolver` 에 의해 검증 수행
    - **예외발생**: 검증에 실패시 `MethodArgumentNotValidException`이 발생
    
    ```java
    @PostMapping("/users")
    public ResponseEntity<Void> createUser(@Valid @RequestBody UserDto userDto) {
        // userDto 내부의 @NotNull, @Size 등이 검증됨
        return ResponseEntity.ok().build();
    }
    ```
    
    **[@Validated]**
    
    - `@Validated`는 자바 표준인 `@Valid`를 기반으로 **Spring 프레임워크가 고유하게 확장하여 제공**하는 어노테이션
    - **특징**: `@Valid`에 없는 그룹화 기능을 지원
    
    - **동작 위치**: Controller뿐 아니라, Service, Repository 등 스프링 Bean으로 등록된 모든 레이어에서 사용 가능
    - **AOP 기능**: 스프링의 AOP를 기반으로 동작, 클래스 레벨에 `@Validated`
    를 붙이면 내부 메서드 호출 시 프록시 객체가 요청을 가로채서 유효성 검사
    - **예외 발생**: 검증에 실패하면 `ConstraintViolationException`이 발생
    
    ```java
    @Service
    @Validated // 클래스 레벨에 선언하여 AOP 기반 검증 활성화
    public class UserService {
        public void signUp(@NotNull String email) {
            // 서비스 레이어의 파라미터 직접 검증 가능
        }
    }
    ```
    
    **[차이점만 봐보자]**
    
    - **제약 조건의 그룹화**
    
    동일한 DTO 객체라도 **회원가입 시 검증해야 할 조건**과 **회원정보 수정 시 검증해야 할 조건**이 다를 수 있습니다. 
    
    `@Validated`는 `groups` 속성을 통해 특정 그룹을 지정하여 원하는 제약 조건만 선택적으로 검증할 수 있습니다.
    
    ```java
    // 1. 마커 인터페이스 선언
    public interface OnCreate {}
    public interface OnUpdate {}
    
    // 2. DTO에 그룹 지정
    public class UserDto {
        @NotNull(groups = OnCreate.class) // 가입 시에만 필수
        private String username;
        
        @NotNull(groups = {OnCreate.class, OnUpdate.class}) // 둘 다 필수
        private String email;
    }
    
    // 3. 컨트롤러에서 특정 그룹만 활성화
    @PostMapping("/signup")
    public void signUp(@Validated(OnCreate.class) @RequestBody UserDto dto) { ... }
    
    ```
    
    → 그치만 그룹 기능을 쓰면 DTO 코드가 복잡해져서, `UserCreateDto`, `UserUpdateDto` 처럼 DTO 자체를 분리하는 게 권장된다.
    
    - **중첩 객체 검증**
    
    DTO 내부에 또 다른 객체가 존재하고, 이 내부 객체의 필드 모두 유효성 검사를 하고 싶다면 **반드시 멤버 변수 위에 `@Valid`를 붙여야 한다.** `@Validated`는 중첩 검증을 지원하지 않는다.
    
    ```java
    public class UserDto {
        @Valid // 내부 객체 검증을 위해 자바 표준 @Valid 사용 필요
        private AddressDto address;
    }
    ```
  
---
- Hibernate 2차 캐시
    
    > 애플리케이션 전체에서 공유할 수 있는 애플리케이션 범위의 캐시이다.
    > 
    
    **[1차 캐시와 2차 캐시 차이점]**
    
    **1차 캐시**
    
    - **범위**: 트랜잭션, 세션 단위
    - **동작 위치**: 영속성 컨텍스트 내부
    - 요청 처리 후 소멸하여 다른 사람과 공유가 불가능하다.
    - 끄기가 불가능하며 무조건 활성화되어 있다.
    
    **2차 캐시**
    
    ex) Redis와 같은 외부 캐시 솔루션
    
    - **범위**: 애플리케이션 단위
    - **동작 위치**: Hibernate와 DB 사이
    - 모든 사용자와 트랜잭션이 공유 가능하다.
    - 기본적으로 비활성화 되어있다.
    
    **[동작 매커니즘]**
    
    Entity 조회할 때 Hibernate는 다음의 순서로 캐시를 확인한다.
    
    - **1차 캐시 확인**: 영속성 컨텍스트에 찾는 엔티티가 있는지 먼저 확인
    - **2차 캐시 확인**: 1차 캐시에 없다면 2차 캐시 확인, 2차에 캐시에 데이터가 있으면 이를 가져와 1차 캐시에 적재한 후 반환
    - **DB 조회**: 2차 캐시에도 없다면 최종적으로 DB에 쿼리를 날려 데이터를 가져오고, 그 이후 1차, 2차 캐시 모두에 저장해둔다.
    
    **[2차 캐시 사용시 주의점(동시성)]** 
    
    2차 캐시는 여러 트랜잭션이 동시에 접근하기 때문에 데이터 일관성(Concurrency) 문제가 발생할 수 있다.
    
    → Hibernate는 이를 해결하기 위해 4가지 동시성 전략을 제공한다.
    
    - **READ_ONLY:** 변경되지 않는 데이터에 사용, 가장 빠르고 안전 (수정 시도 시 예외 발생)
    - **NONSTRICT_READ_WRITE:** 데이터가 거의 수정되지 않고, 아주 잠깐의 데이터 불일치(수정 중 이전 데이터가 보이는 현상)를 허용할 때 사용
    - **READ_WRITE:** 데이터가 수정될 수 있는 경우 사용, 데이터의 일관성을 유지하기 위해 **데이터 잠금(Lock) 메커니즘**을 사용
    - **TRANSACTIONAL:** JTA(Java Transaction API) 환경에서 완전한 트랜잭션 격리(Atomicity)를 보장해야 할 때 사용
    
    Hibernate 자체가 캐시 저장소를 직접 구현하기 보다는 외부 솔루션을 연동해 사용한다. → 최근 분산 환경에선 **Redis**나 **Hazelcast** 등의 **2차 캐시 프로바이더**로 많이 연동한다.

---
- 배치 사이즈
    
    > 연관된 데이터를 조회할 때 쿼리를 매번 하나씩 날리지 않고, 지정한 개수만큼 `IN` 절을 사용해 한 번에 묶어서 조회하는 기능이다.
    > 
    
    **[작동원리]**
    
    Team(1) 대 Member(N)의 관계를 예로 들면, 화면에 팀 10개를 보여주려고 할 때
    
    - **Batch Size가 없는 경우**
    
    반복문을 돌며 연관된 회원들을 조회하면, 각 팀의 ID를 조건으로 하는 쿼리가 10번 각각 실행된다.
    
    ```sql
    SELECT * FROM member WHERE team_id = 1;
    SELECT * FROM member WHERE team_id = 2;
    ...
    SELECT * FROM member WHERE team_id = 10;
    ```
    
    - **Batch Size를 10으로 설정했을 때**
    
    Hibernate는 반복문이 시작되는 순간 뒤이어 조회될 팀들의 ID까지 미리 파악하여 `IN` 절에 묶어 하나의 쿼리로 날린다.
    
    ```sql
    SELECT * FROM member WHERE team_id IN (1,2,3,4,5,6,7,8,9,10);
    // 단 1번 실행
    ```
    
    **[Batch Size 설정 방법]**
    
    - **글로벌 설정**
    
    애플리케이션 전체의 모든 지연 로딩에 배치 사이즈를 적용한다.
    
    ```yaml
    # application.yaml
    spring:
    	jpa:
    		properties:
    			hibernate:
    				default_batch_fetch_size: 100 # 보통 100 또는 500을 가장 많이 쓴다.
    ```
    
    - **개별 엔티티 설정**
    
    특정 연관 관계에만 다른 사이즈를 적용하고 싶을 때 사용한다.
    
    ```java
    @Entity
    public class Team {
        @Id @GeneratedValue
        private Long id;
    
        @BatchSize(size = 50) // 이 연관관계에만 50 적용
        @OneToMany(mappedBy = "team")
        private List<Member> members = new ArrayList<>();
    }
    ```
    
    **[Fetch Join과의 차이점]**
    
    기존에  배운 Fetch Join으로 풀지 않고 Batch Join을 고려하는 이유는 Fetch Join이 해결하지 못하는 단점을 해결해주기 때문이다.
    
    - **컬렉션 페이징(Paging) 문제**
    
    JPA에서 일대다(1:N) 관계의 컬렉션을 Fetch Join 하면서 페이징 처리를 시도하면, Hibernate는 모든 데이터를 **메모리로 전부 읽어와서 페이징을 수행한다는 단점**을 가진다. Out of Memory 문제
    
    → `Team` 만 페이징 쿼리로 DB에서 끊어 가져온 뒤, 연관된 `Member` 들은 Batch Size를 통해 `IN` 절로 묶어 가져오면 안전하게 페이징과 N+1을 모두 해결할 수 있다.
    
    - **둘 이상의 컬렉션 Fetch Join 가능**
    
    JPA는 2개 이상의 컬렉션을 동시에 Fetch Join하면 데이터가 기하급수적으로 늘어나는 `MultipleBagFetchException` 이 발생하여 애플케이션 구동이 안 된다. 이때 하나는 Fetch Join을 쓰고, 나머지는 Batch Size로 처리하면 해결된다.
    
- transform - groupBy
    
    > **변환(transform)**과 **그룹화(GroupBy)**는 데이터베이스에서 조회한 엔티티 목록을 client에 반환할 DTO나 API 스펙에 맞게 가공할 때 사용하는 패턴이다.
    > 
    
    > 주로 Java Stream API또는 QueryDSL의 Transform 기능을  사용하여 구현한다.
    > 
    
    **[Java Stream API 방법]**
    
    DB에서 엔티티 리스트를 다 가져온 뒤, 서버 메모리 상에서 Stream API를 이용해 그룹화와 변환을 처리하는 방식
    
    **<예시>**
    
    하나의 Order에 여러 개의 주문 상품(OrderItem)이 포함되어 있다. DB에서 OrderItem들을 조회한 후, OrderId별로 그룹화하면서 동시에 엔티티를 DTO로 변환하려고 한다.
    
    ```java
    // DB에서 조회해온 주문 상품 엔티티 리스트
    List<OrderItem> orderItems = orderItemRepository.findAll();
    
    // groupBy & transform 수행
    Map<Long, List<OrderItemDto>> orderMap = orderItems.stream()
        .collect(Collectors.groupingBy(
            OrderItem::getOrderId, // GroupBy 기준: 주문 ID (Key)
            Collectors.mapping(    // Transform: 엔티티를 DTO로 변환 (Value)
                item -> new OrderItemDto(item.getId(), item.getItemName(), item.getPrice()),
                Collectors.toList()
            )
        ));
    ```
    
    - `Collectors.groupingBy` : `orderId` 으로 그룹을 묶어 Map의 Key를 생성한다.
    - `Collectors.mapping` : Stream의 `map()`과 같은 역할으로, 그룹화하는 대상 엔티티들을 원하는 형태(DTO)로 변환(Transform)하여 List로 수집한다.
    
    **[QueryDSL의 `transform()` 방법]**
    
    DB에서 조인된 평면 데이터 구조를 중복없이 깔끔한 객체 그래프(Map 또는 DTO 구조)로 한 번에 변환할 수 있다.
    
    **<예시>**
    
    ```java
    import static com.querydsl.core.group.GroupBy.*;
    
    Map<Long, OrderResponseDto> result = queryFactory
        .from(order)
        .join(order.orderItems, orderItem)
        .transform(
            groupBy(order.id).as(new QOrderResponseDto(
                order.id,
                order.orderDate,
                // 그룹화하는 동시에 내부 컬렉션을 DTO로 변환(Transform)하여 리스트로 수집
                list(new QOrderItemDto(orderItem.id, orderItem.itemName, orderItem.price))
            ))
        );
    ```
    
    - `transform(groupBy(...).as(...))`: Querydsl 내부적으로 결과셋을 순회하며 `order.id`가 같은 행들을 하나로 묶어준다.
    - `list(...)`: 묶인 그룹 내에서 N에 해당하는 `orderItem` 데이터들을 `OrderItemDto`로 즉시 변환하여 리스트에 담아준다.
    
    - **장점**: 개발자가 직접 Stream으로 맵을 돌리며 가공할 필요 없이, 쿼리 실행 결과 자체를 완전히 구조화된 DTO 형태로 받아볼 수 있어 코드가 깔끔하다.
    
     **[Spring Boot 레이어 아키텍처]**
    
    - **Transform의 위치**: DB 엔티티가 영속성 컨텍스트를 벗어나 Controller나 클라이언트에게 그대로 노출되면 보안 문제가 생길 수 있어, **Service 레이어 뒷단이나 Querydsl 조회 시점에 DTO로 Transform하는 것이 정석**이다.
    
    - **GroupBy의 메모리 고려**: Java Stream을 이용한 `groupBy`는 DB 데이터를 WAS 메모리에 다 올려놓고 가공하는 방식이다. 만약 데이터가 수백만 건 단위로 너무 많다면 WAS 메모리에 부하가 가므로, DB 레벨에서 쿼리 튜닝을 하거나 페이징 처리를 먼저 수행한 후 소량의 데이터에만 적용해야 한다.
    
- order by null
    
    > MySQL 환경에서 과거에 성능 최적화를 사용했던 기법이다.
    > 
    
    **[MySQL에서 성능 최적화]**
    
    MySQL 5.7 이하에서는 `GROUP BY`  절을 사용하면 그룹화할 컬럼을 기준으로 내부적으로 자동으로 **오름차순(`ASC`) 정렬**을 수행했다.
    
    → 정렬 연산으로 인해 데이터가 많을 경우 성능 저하로 이어졌다.
    
    **<해결책>**
    
    정렬이 필요없고 단순히 그룹화만을 원할 때, 쿼리 끝에 `ORDER BY NULL`을 붙여 정렬을 방지한다.
    
    ```sql
    SELECT department_id, COUNT(*) 
    FROM employees 
    GROUP BY department_id
    ORDER BY NULL; -- 암묵적 정렬을 막아 정렬 연산을 방지한다.
    ```
    
    **[현대 MySQL에서의 변화]**
    
    MySQL 8.0 버전부터는 `GROUP BY` 의 암묵적 정렬 기능이 제거 되었다. 과거와 달리 그룹화만 수행하므로 최신 버전 MySQL 환경에서는 성능 최적화 목적의 `ORDER BY NULL`을 명시할 필요가 없다.
    
- 카테시안 곱
    
    > 카테시안 곱은 수학의 집합론에서 나온 개념, DB 환경에서 두 테이블의 **모든 행(row)**을 제한 조건 없이 **수평으로 전부 결합**하는 조인 방식
    > 
    
    > SQL 문법에서는 `CROSS JOIN`, **교차 조인으로** 조건문(ON, WHERE)을 걸지 않고 조인 수행하면 발생한다.
    > 
    
    **[연산 예시]**
    
    테이블 A의 행 개수가 M개이고, 테이블 B의 행 개수가 N개라면 카테시안 곱의 결과는 M*N이 된다.
    
    **[발생 상황]**
    
    - **실수로 조건문을 누락한 경우**
    
    `WHERE`절에 JOIN 조건을 누락하면 카테시안 곱이 발생한다.
    
    ```sql
    -- 위험한 쿼리 예시
    SELECT * 
    FROM orders, members; -- 조인 조건(ON 이나 WHERE)이 없다.
    ```
    
    → DB 서버의 CPU와 메모리가 감당할 수 없는만큼 데이터가 생성되어 서비스가 먹통이 될 수 있다.
    
    - **JPA/Hibernate에서의 카테시안 곱 (Fetch Join 중복)**
    
    하나의 엔티티가 **둘 이상의 일대다(1:N) 컬렉션 자식 관계**를 가지고 있을 때, 이를 모두 `Fetch Join`하려고 하면 Hibernate는 내부적으로 카테시안 곱 쿼리를 날린다.
    
    데이터가 무수히 뻥튀기되는 결과가 초래되며, JPA는 이를 감지하고 애플리케이션 구동 시점에 `MultipleBagFetchException`이라는 에러를 던지며 거부한다.
    
    - **의도적인 사용**
        - 테스트 데이터 벌크 생성: DB 부하 테스트를 위해 테이블 몇 개를 묶어 많은 더미 데이터를 한 번에 생성
        - 모든 가능한 조합 산출: 모든 조합을 생성하기 위한 사용
    
- MultipleBagFetchException
    
    > JPA/Hibernate 환경에서 둘 이상의 **일대다 컬렉션 관계**를 한 번에 **즉시 로딩(Fetch Join)**하려고 할 때 발생하는 대표적인 예외이다.
    > 
    
    **[원인]**
    
    JPA에서 **Bag**란 “중복을 허용하고 순서가 없는 컬렉션”인 **List 타입**을 의미하는데, 하나의 상위 엔티티가 2개 이상의 자식 List 컬렉션을 가지고 있고, 성능 최적화를 위해 둘 다 Fetch Join하려고 할 때 발생한다.
    
    **[해결 방법]**
    
    - **default_batch_fetch_size 설정**
    
    1개 혹은 둘 다 Fetch Join을 하지 않고 지연 로딩으로 둔 채 글로벌 배치 사이즈 설정을 주는 것이다.
    
    Fetch Join으로 묶어서 1번의 쿼리로 가져오고 그 후 컬렉션을 루프 돌며 사용할 때, **Batch Size** 기능 덕분에 `IN` 절로 묶여서 단 1번의 추가 쿼리로 조회된다.
    
    ```yaml
    spring:
      jpa:
        properties:
          hibernate:
            default_batch_fetch_size: 100
    ```
    
    - **컬렉션 타입을 Set으로 변경**
    
    중복을 허용하지 않는 자료구조를 사용하여 Hibernate가 카테시안 곱의 결과에서 중복을 걸러낸다.
    
    ```java
    @OneToMany(mappedBy = "post")
    private Set<Comment> comments = new LinkedHashSet<>();
    
    @OneToMany(mappedBy = "post")
    private Set<Tag> tags = new LinkedHashSet<>();
    ```
    
    에러는 막을 수 있지만, DB레벨에서 카테시안 곱이 발생하는 근본적인 비효율은 변하지 않는다.
    
    - **QueryDSL 등으로 쿼리 분리**
    
    조인 쿼리를 한 번에 다 묶지 않고 쿼리를 2~3번으로 쪼개서 가져온 뒤 메모리상에서 조합하는 방법
    
    **<예시>**
    
    1. Post 목록을 먼저 조회한다.
    2. 조회된 Post ID들을 뽑아서 Comment를 WEHRE ~ IN 절을 사용해서 따로 조회한다.
    3. 똑같이 Tag도 IN 절로 따로 조회한다.
    4. transform - groupBy나 Map을 활용해 메모리에서 조립한다.