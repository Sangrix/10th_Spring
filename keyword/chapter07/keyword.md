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
