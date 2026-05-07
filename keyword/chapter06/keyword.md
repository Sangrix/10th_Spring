- JPA란?
    
    > 
    > 
    > 
    > JPA(Java Persistence API)란, 자바 진영의 **ORM(Object-Relational Mapping) 기술의 표준으로 사용되는 인터페이스의 모음**이다.
    > 
    
    **[ORM(Object-Relational Mapping)이란?]**
    
    JPA를 공부 하기 전 ORM에 대해 간단히 알아보고자 한다.
    
    - ORM이란, 객체지향  프로그래밍 언어의 객체와 RDB의 테이블 간의 데이터를 자동으로 연결(매핑)해주는 기술이다.
    - 이로 인해, 객체는 객체대로 설계하고, DB는 DB대로 설계할 수 있게 되었다.
    - 대중적인 언어에는 대부분 ORM이 존재한다.
    
    **[ORM과 SQL Mapper 비교]**
    
    대표적인 SQLMapper로는 Mybatis, Spring JdbcTemplate 등이 있다.
    
    **ORM**
    
    - DB 테이블을 Java 객체로 매핑, 객체 간의 관계를 바탕으로 SQL문을 자동 생성(객체를 통해 간접적으로 DB 데이터 조작)
    - Object 필드 → 매핑 → DB 데이터
    - SQL 쿼리가 아닌, 매서드로 데이터 조작
    - 객체간 관계를 바탕으로 SQL 자동 생성
    
    **SQL Mapper**
    
    - SQL을 직접 명시(SQL 문으로 DB 조작)
    - Object 필드 → 매핑 → SQL
    
    ORM은 RDB 관계를 객체에 반영하는 것이 목적, SQL Mapper는 단순히 필드를 매핑하는 것이 목적
    
    **[JPA (Java-Persistence API)란?]**
    
    Java 진영의 ORM 기술 표준, Java App에서 RDB를 사용하는 방식을 정의한 인터페이스의 모음
    
    - JPA가 직접 동작하는 것이 아닌, 인터페이스를 구현하여 사용해야 한다.
    - 표준 명세를 구현한 3가지 구현체
        - **Hibernate**
        - **EclipseLink**
        - **DataNucleus**
    - SQL 작성없이 **객체를 DB에 직접 저장**할 수 있게 도와주는 기술
    - App과 JDBC 사이에서 동작
    
    **[SQL을 직접 다룰 때의 문제점]**
    
    - **반복적인 코드 작성**
        - 모든 DB 작업에 대한 SQL 작성
        - 반복적인 CRUD SQL 작성과 객체를 SQL에 매핑하는 코드 작성에 시간 소요
    - **SQL 의존적 개발**
        - 테이블에 Column이 추가된다면 모든 DAO의 SQL 변경 필요
        - 오류 시, 논리 로직과 SQL문 둘 다 확인이 필요
    - **패러다임 불일치**
        - DB는 객체지향이 다루는 개념이 존재하지 않고, 지향하는 목적이 다름
        - 이로 인해, 개발자가 중간에서 문제 해결을 위한 코드를 직접 만들어야함
    
    **[JPA 장단점]**
    
    **장점**
    
    - **생산성**
        - JDBC API를 사용하는 반복 작업을 JPA가 대신해서 생산성 향상
        - DDL도 JPA가 자동 생성해주어 DB설계 중심을 객체 설계 중심으로 변경
    - **유지보수**
        - SQL을 직접 다룰 시, 필드의 작은 변경에도 SQL과 JDBC 코드를 전부 수정해야하는 일을 대신해줌
    - **패러다임 불일치 해결**
    - **성능**
        - App과 DB 사이에서 성능 최적화 기능 제공
        - 같은 트랜잭션 안에서는 같은 entity를 반환해 통신 횟수 줄임
    - **데이터 접근 추상화와 벤더 독립성**
        - DB는 같은 기능도 벤더마다 사용법이 달라, 처음 사용한 DB에 종속되고, 변경이 어려움
        - JPA는 App과 DB 사이에서 추상화된 데이터 접근을 제공하여 종속을 방지함
    
    **단점**
    
    - **학습 곡선이 높음**
        - JPA 핵심 개념 이해가 새로 필요
        - JPA의 핵심 개념인 영속성 컨텍스트에 대한 이해가 부족하면, SQL을 직접 사용하는 것 보다 못한 상황이 나올 수 있음
    - **속도 저하 가능성**
        - 프로젝트 규모가 크고 복잡하여 설계가 잘못되면, 속도 저하 및 일관성 무너짐
        - 복잡하고 무거운 Query는 속도를 위해 별도의 튜닝이 필요하고, 결국 SQL문을 사용해야함.
    
- N+1 문제란?
    
    > 연관관계가 설정된 Entity를 조회할 경우에 조회된 데이터 개수(N)만큼 연관관계의 조회 **쿼리가 추가로 발생**하는 현상이다.
    > 
    
    **[N+1 문제란?]**
    
    1개의 조회 쿼리만 발생하기를 원하는데, 연관관계에 따라 관련 N개의 동작쿼리가 발생하는 것이다.
    
    - 그래서 1+N 문제로 생각하면 이해가 더 쉬울 듯하다.
    - ex) 블로그 게시글과 댓글이 있는 경우, 게시글을 조회한 후 각 게시글마다 댓글을 조회하기 위한 추가 쿼리가 발생
    
    **[글로벌 패치 전략별 N+1 문제 상황]**
    
    JPA에서 제공하는 Repository의 `findAll()` 매서드를 보려고 한다.
    
    - 글로벌 패치 전략을 **즉시로딩**으로 설정한 경우
        - `findAll()`을 실행하면 **N+1 문제**가 발생
            - `findAll()`은 `<select u from User u>` 라는 JPQL 구문을 생성하여 실행하기 때문
            - JPQL은 글로벌 패치 전략을 고려하지 않고 query 실행
            - 모든 User를 조회하는 query 실행 후 → 즉시로딩 설정을 보고 연관관계에 있는 모든 entity를 조회하는 쿼리를 실행
    
    - 글로벌 패치 전략을 **지연 로딩**으로 설정한 경우
        - `findAll( )`을 실행하면 **N+1 문제**가 발생하지 않음
            - 연관관계에 있는 entity를 실제 객체 대신에 **프록시 객체로 생성**하여 주입하기 때문
            - 하지만, 프록시 객체를 사용할 경우, 실제 데이터가 필요하여 조회하는 query 발생
            - N+1 문제가 발생할 수 있음
    
    **[N+1 문제 해결법]**
    
    **fetch join**과 `@EntityGraph`를 사용하여 해결할 수 있다.
    
    **fetch join**
    
    연관관계에 있는 entity를 한 번에 즉시 로딩하는 구문이다.
    
    ```sql
    select distinct u
    from User u
    left join fetch u.posts
    ```
    
    **@EntityGraph**
    
    fetch join과 비슷한 효과를 낸다. 쿼리 매서드에 해당 어노테이션을 추가해 사용할 수 있다.
    
    ```java
    @EntityGraph(attributePaths = {"post"}, type = EntityGraphType.FETCH)
    List<User> findAll();
    ```
    
- 지연로딩과 즉시로딩의 차이는?
    
    > JPA 패치 전략으로, **연관된 entity를 조회할 때** 즉시로딩(EAGER)과 지연로딩(LAZY)로 방식을 지정하는 것이다.
    > 
    
    **즉시로딩 (EAGER)**
    
    연관된 entity가 **항상 같이 로딩**되어 가져온다.
    
    - ex) Member 엔티티와 Team 엔티티가 일대다 관계일때
    - Member 엔티티 조회 시 연관된 Team 엔티티도 함께 조회
    
    ```java
    @Entity
    public class Member{
    	@ID
    	@GeneratedValue
    	private Long id;
    	
    	private String name;
    	
    	@ManayToOne(fetch = FetchType.EAGER)
    	private Team team;
    	
    	// Getter, Setter
    }
    
    @Entity
    public class Team{
    	@Id
    	@Generated
    	private Lond id;
    	
    	private String name;
    	
    	// Getter, Setter
    }
    ```
    
    **지연로딩(LAZY)**
    
    지연 로딩은 연관관계에 있는 entity를 실제 객체 대신에 **프록시 객체로 생성**하여 주입하여 주고, 연관된 **entity가 실제 사용될 때** 로딩된다.
    
    - ex) Member 엔티티와 Order 엔티티가 일대다 관계 일 때, Member 엔티티를 조회할 때, 연관된 Order 엔티티는 필요할 때 로딩
    
    ```java
    @Entity
    public class Member{
    	@ID
    	@GeneratedValue
    	private Long id;
    	
    	private String name;
    	
    	@OneToMany(mappedBy = "member", fetch = FetchType.LAZY)
    	private List<Order> orders = new ArrayList<>();
    	
    	// Getter, Setter
    }
    
    @Entity
    public class Order{
    	@Id
    	@Generated
    	private Lond id;
    	
    	private String name;
    	
    	@ManayToOne
    	private Member member;
    	
    	// Getter, Setter
    }
    ```
    
- JPQL란?
    
    > JPQL은 JPA에서 제공하는 객체지향 쿼리 언어로, SQL과 문법이 매우 유사하지만 테이블을 대상으로 하는 것이 아니라 **entity 객체를 대상**으로 쿼리
    > 
    
    **[JPQL 등장 배경]**
    
    RDBMS는 테이블 중심이고 자바는 객체 중심이다. 이로 인한 둘 사이 패러다임 불일치가 발생하고 이것의 해결을 위해 JPA가 등장했다. 하지만 복잡한 조건의 조회가 필요할 때, 특정 Dialect에 의존하는 SQL을 직접 쓰면 DB 변경시 유지보수가 어려워 이를 해결하기 위해 **Entity 객체를 대상으로 하는 쿼리 언어**가 필요해졌다.
    
    **[주요 특징]**
    
    - **객체 지향적:** 테이블 명이나 컬럼 명이 아닌, 엔티티 클래스 이름과 필드 이름을 사용한다.
    - **DB 방언 독립성:** 특정 데이터베이스(MySQL, Oracle 등)에 종속되지 않습니다. JPA가 사용 중인 DB 방언에 맞춰 적절한 SQL로 변환해 준다.
    - **대소문자 구분:** 엔티티와 필드는 대소문자를 엄격히 구분한다. 다만, JPQL 키워드인 `SELECT`, `FROM` 등은 구분하지 않는다.
    - **별칭(Alias) 필수:** JPQL에서는 엔티티에 대한 별칭(ex: `m`)을 반드시 지정해야 한다.
    
    **[사용법]**
    
    - `EntityManager`를 직접 사용
    
    ```java
    // 엔티티 객체(Member)를 대상으로 조회
    String jpql = "SELECT m FROM Member m WHERE m.username = :name";
    
    List<Member> result = em.createQuery(jpql, Member.class)
        .setParameter("name", "minsang")
        .getResultList();
    ```
    
    - Spring Data JPA의 `@Query` 어노테이션에서 사용
    
    ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
        @Query("select m from Member m where m.age > :age")
        List<Member> findMembersOlderThan(@Param("age") int age);
    }
    ```
    
    **[장단점]**
    
    **장점**
    
    - **객체 지향성**: 도메인 모델을 유지하며 쿼리 가능하다.
    - **유지보수**: DB 교체 시 쿼리 수정 최소화가 가능하다.
    - **강한 기능**: 조인, 페이징, 통계 함수 등 대부분의 SQL 기능을 지원한다.
    
    **단점**
    
    - **문자열 기반**: 문자열 기반이라 오타가 있어도 컴파일 타임에 에러를 발견할 수 없다.
    - **동적 쿼리의 어려움**: 조건에 따라 쿼리가 변하는 경우 구현이 복잡하다.
    - **학습 곡선**: 객체 그래프 탐색과 DB 성능 사이의 이해가 필요하다.
    
- Fetch Join란?
    
    > Fetch Join란, SQL `JOIN`의 종류가 아니라 JPQL에서 성능 최적화를 위해 제공하는 기능으로 연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회할 수 있게 해준다.
    > 
    
    <aside>
    💡
    
    **JPA 성능 최적화의 핵심**이자, N+1 문제를 해결하기 위한 필수적인 기술이다.
    
    </aside>
    
    **[주요 특징]**
    
    - **객체 그래프 유지:** 조회 시점에 연관된 객체들을 모두 채워두므로, 이후 객체 그래프를 탐색해도 추가 쿼리가 발생하지 않는다.
    - **런타임 최적화:** 글로벌 로딩 전략(EAGER/LAZY) 설정보다 우선하여 특정 상황에서만 즉시 로딩이 발생하도록 제어할 수 있다.
    - **SQL 변환:** 실제 SQL에서는 `INNER JOIN`이나 `LEFT OUTER JOIN`으로 변환되지만, `SELECT` 절에 연관된 엔티티의 모든 필드가 포함된다는 점이 일반 SQL 조인과 다르다.
    
    **[Fetch Join 사용법]**
    
    - JPQL의 `join fetch` 문법을 사용하거나
    
    ```sql
    -- Team 정보까지 한 번에 가져오기
    SELECT m FROM Member m JOIN FETCH m.team
    ```
    
    - Spring Data JPA의 `@EntityGraph`를 활용
    
    ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
        @Query("select m from Member m join fetch m.team")
        List<Member> findAllWithTeam();
    
        // 어노테이션 방식
        @EntityGraph(attributePaths = {"team"})
        List<Member> findAll();
    }
    ```
    
- @EntityGraph란?
    
    > `@EntityGraph`는 JPA 표준에서 정의한 엔티티 그래프 기능을 Spring Data JPA에서 **어노테이션 형태로 사용**할 수 있도록 제공하는 기능이다. 어떤 연관관계를 함께 조회할 지 어노테이션으로 명시하면, 내부적으로 **Fetch Join을 실행한다**.
    > 
    
    <aside>
    💡
    
    join fetch는 N+1 문제를 해결하는 좋은 방법이지만 이를 위해 동일한 쿼리를 반복 작성하거나 매서드 이름 기반 쿼리의 장점을 포기하고 `@Query`를 써야하는 불편함을 해소하기 위해 등장했다.
    
    </aside>
    
    **[주요 특징]**
    
    - **선언적 최적화:** JPQL을 직접 작성하지 않고도 특정 연관 관계를 즉시 로딩(EAGER)으로 가져오도록 설정할 수 있다.
    - **Query Method 지원:** Spring Data JPA의 메서드 이름 기반 쿼리에 바로 적용할 수 있다.
    - **기본 로딩 전략 무시:** 엔티티 모델에 설정된 `FetchType.LAZY`를 무시하고 해당 쿼리 실행 시점에만 `EAGER`로 동작하게 만든다.
    - **Left Outer Join 사용:** 기본적으로 SQL의 `LEFT OUTER JOIN`을 사용하여 데이터를 조회한다.
    
    **[사용 방법]**
    
    ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
    
        // 별도의 JPQL 없이 메서드 이름만으로 Team을 함께 조회
        @EntityGraph(attributePaths = {"team"})
        List<Member> findAll();
    
        // JPQL과 함께 사용도 가능 (fetch join 문구를 대체함)
        @EntityGraph(attributePaths = {"team"})
        @Query("select m from Member m where m.username = :username")
        Member findByUsername(@Param("username") String username);
    }
    ```
    
- commit과 flush 차이점은?
    
    JPA는 DB와 애플리케이션 사이에 영속성 컨텍스트라는 중간 계층을 두고, 이곳에는 1차 캐시와 쓰기 지연 SQL 저장소가 존재한다.
    
    - **Flush:** 영속성 컨텍스트의 변경 내용을 데이터베이스에 **반영**하는 과정이다.
    - **Commit:** 데이터베이스의 트랜잭션을 **완료**하여 변경 내용을 확정 짓는 작업이다.
    
    **[주요 특징과 동작 순서]**
    
    **Flush (반영)**
    
    1. **변경 감지(Dirty Checking):** 영속성 컨텍스트에 있는 엔티티와 스냅샷을 비교해 수정된 엔티티를 찾는다.
    2. **SQL 생성:** 수정된 엔티티에 대한 `INSERT`, `UPDATE`, `DELETE` 쿼리를 만들어 '쓰기 지연 SQL 저장소'에 등록한다.
    3. **전송:** 저장소의 쿼리를 데이터베이스에 전송한다.
        - **주의점:** DB에 쿼리는 날아가지만, 아직 확정(Commit)된 상태는 아닙니다. 문제가 생기면 롤백할 수 있다.
    
    **Commit (확정): 트랜잭션을 끝내는 단계**
    
    1. JPA에서 트랜잭션을 커밋하면 **내부적으로** `flush()`**가 먼저 호출된다.**.
    2. 데이터베이스에 변경 내용이 영구적으로 저장된다.
    3. 데이터베이스의 락(Lock)이 해제된다.
    
    **[발생 시점]**
    
    **Flush가 발생하는 경우**
    
    - **직접 호출:** `em.flush()`를 코드로 호출 (테스트나 특수 상황에서 사용한다.)
    - **트랜잭션 커밋 시:** 커밋 전 자동으로 호출된다.
    - **JPQL 쿼리 실행 시:** 쿼리 실행 전 자동으로 호출됨 (데이터 정합성을 위해)
    
    **Commit이 발생하는 경우**
    
    - **Spring 환경:** `@Transactional` 어노테이션이 붙은 메서드가 성공적으로 종료될 때
    - **기본 JPA:** `tx.commit()`을 호출할 때

---
- 히카리CP 작동원리
    
    서버가 DB에 접근하려면 매번 DB와 연결 통로가 필요하다.
    
    대강적인 흐름은 다음과 같다.
    
    ```
    클라이언트 요청
    → Spring Controller
    → Service
    → Repository / JPA / JDBC
    → DB 커넥션 획득
    → SQL 실행
    → 커넥션 반납
    → 응답 반환
    ```
    
    여기서 핵심은 **SQL을 실행하려면 DB 커넥션이 필요**하다는 점이다.
    
    ---
    
    **[커넥션이란?]**
    
    커넥션은 서버와 DB 사이에 만들어진 연결 통로이다.
    
    쉽게 말하면, 서버 ↔ MySQL/PostgreSQL 사이에 만들어진 연결이다.
    
    Spring에서는 보통 `java.sql.Connection` 객체로 다룬다.
    
    개념적으론 아래와 같은 느낌이다.
    
    ```java
    Connection conn = DriverManager.getConnection(url, username, password);
    PreparedStatement ps = conn.prepareStatement("SELECT * FROM user");
    ResultSet rs = ps.executeQuery();
    conn.close();
    ```
    
    이때 `getConnection()` 이 DB와 네트워크 연결을 만들고, 인증하고, 세션을 준비하는 등 가벼운 작업이 아니라는 것이 큰 문제다.
    
    요청마다 DB 연결을 새로 만들면 **연결 생성 비용**이 계속 반복된다.
    
    ---
    
    **[커넥션 풀이란?]**
    
    커넥션 풀(Connection Pool)은 DB 커넥션을 미리 여러 개 만들어두고, 필요할 때 비려쓰고 반납하는 구조이다.
    
    요청이 들어오면 새 커넥션을 만들지 않고 풀에서 하나를 빌린다.
    
    ```
    요청 A → Connection 1 빌림 → SQL 실행 → Connection 1 반납
    요청 B → Connection 2 빌림 → SQL 실행 → Connection 2 반납
    요청 C → Connection 3 빌림 → SQL 실행 → Connection 3 반납
    ```
    
    이때, 커넥션을 닫는 것이 아니라 풀에 돌려주는 것이다.
    
    Spring/JPA/JDBC 코드에서 `connection.close()` 또는 `트랜잭션 종료` 가 발생하면, 일반적으로 실제 TCP 연결을 끊는 것이 아니라 커넥션 풀에 반환된다.
    
    > 커넥션 풀은 성능 최적화 도구이면서 동시에 DB 보호 장치이다.
    > 
    
    ---
    
    **[HikariCP란?]**
    
    HikariCP는 Spring 환경에서 사용하는 JDBC **커넥션 풀 구현체**이다.
    
    SpringBoot에서는 `spring-boot-starter-jdbc` 또는 `spring-boot-starter-data-jpa` 를 사용하면 HikariCP 의존성이 자동으로 포함되며, Spring Boot는 사용가능한 경우 HikariCP를 우선 선택하는 알고리즘을 사용한다.
    
    실제 요청 흐름을 보면 다음과 같다.
    
    ```
    1. Controller가 요청을 받음
    2. Service에서 로그인 로직 수행
    3. Repository가 DB 조회 요청
    4. HikariCP에서 커넥션 하나를 빌림
    5. SELECT 쿼리 실행
    6. 트랜잭션 종료 또는 close 시점에 커넥션 반납
    7. 응답 반환
    ```
    
    개발자가 보통 직접 HikariCP를 호출하진 않는다.
    
    ```java
    userRepository.findByEmail(email);
    ```
    
    위와 같은 JPA 코드를 작성하면, 내부적으로 다음이 일어난다.
    
    ```
    Repository
    → EntityManager
    → Hibernate
    → DataSource
    → HikariCP
    → DB Connection
    ```
    
    ---
    
    **[HikariCP 주요 설정]**
    
    Spring Boot에서는 `application.yml` 에 설정한다.
    
    ```
    spring:
      datasource:
        url: jdbc:mysql://localhost:3306/mydb
        username: root
        password: password
        driver-class-name: com.mysql.cj.jdbc.Driver
        hikari:
          maximum-pool-size: 10
          minimum-idle: 10
          connection-timeout: 30000
          idle-timeout: 600000
          max-lifetime: 1800000
          leak-detection-threshold: 2000
    ```
    
    - `maximum-pool-size`: 풀에 존재할 수 있는 최대 커넥션 수이다.
    
    예를 들어 요청 30개가 동시에 들어왔는데 `maximum-pool-size`이 10이면,
    
    ```
    10개 요청 → 바로 DB 작업
    20개 요청 → 커넥션 반납될 때까지 대기
    ```
    
    - `minimum-idle`: 최소한으로 유지할 유휴 커넥션 수이다.
    - `connection-timeout`: 풀에서 커넥션을 빌릴 때 최대 대기 시간이다.
    
    만일 정해진 시간 안에 커넥션을 못 빌리면 예외가 발생한다.
    
    - `idle-timeout`: 사용되지 않는 커넥션을 얼마나 오래 유지할 지 정한다.
    - `max-lifetime`: 커넥션 하나의 최대 수명이다.
    - `leack-detection-threshold`: 커넥션 누수를 감지하기 위한 설정이다.
    
    커넥션을 빌려간 뒤 2초 이상 반납하지 않으면 로그로 경로를 남기는 시간을 정한다.
    
    이때, 너무 낮게 잡으면 정상적인 긴 쿼리도 누수처럼 보일 수 있다.
    
- SQM과 PartTree
    
    > SQM은 Hibernate 쪽 개념이고, PartTree는 Spring Data 쪽 개념이다.
    > 
    
    **[PartTree란?]**
    
    Spring Data가 Repository 매서드 이름을 분석해서 쿼리 조건 구조로 바꿔놓은 트리이다.
    
    예를 들어 이러한 매서드가 있다고 가정해본다.
    
    ```java
    List<User> findByEmailAndAgeGreaterThan(String email, int age);
    ```
    
    Spring Data는 이 매서드 이름(`findByEmailAndAgeGreaterThan`)을 그냥 문자열로 보지 않는다.
    
    ```
    find      → 조회 쿼리
    By        → 조건 시작
    Email     → User.email 필드 조건
    And       → AND 조건
    Age       → User.age 필드 조건
    GreaterThan → age > ?
    ```
    
    위처럼 해석하고,
    
    ```
    PartTree
    └── 조건부
        ├── Part: email = ?
        └── Part: age > ?
    ```
    
    이러한 구조를 만든다.
    
    쉽게 비유하면 Repository **매서드 이름 해석기**이다.
    
    개발자가 아래와 같은 매서드를 만들면,
    
    ```java
    findByUsernameAndStatusOrderByCreatedAtDesc(...)
    ```
    
    Spring Data는 이걸을 아래와 같이 해석한다.
    
    ```sql
    WHERE username = ?
    AND status = ?
    ORDER BY created_at DESC
    ```
    
    SQL을 직접 만드는 존재라기보다는, 매서드 이름을 **쿼리 조건 조각으로 분해하는** 중간 표현이다.
    
    -> sql을 직접 실행하는 게 아니라, 매서드 이름을 쿼리 구조로 바꾸는 역할까지만 담당한다.
    
    ---
    
    **[SQM이란?]**
    
    SQM은 Semantic Query Model의 약자이다.
    
    쉽게 표현하면 Hibernate가 JPQL/HQL/Criteria Query를 이해한 뒤 만드는 의미 기반 쿼리 모델이다.
    
    SQL을 만들기 전에 Hibernate가 먼저 아래와 같이 생각한다.
    
    ```
    이 쿼리는 User 엔티티를 조회하는구나.
    email 필드로 비교하는구나.
    age 필드가 특정 값보다 커야 하는구나.
    User는 users 테이블에 매핑되어 있구나.
    email은 varchar 컬럼이고, age는 integer 컬럼이구나.
    ```
    
    이와 같이 쿼리의 의미를 엔티티 메타데이터와 연결해서 이해한 구조가 SQM이다.
    
    JPQL과 SQL은 다르다.
    
    JPQL의 다음과 같은 예시가 있다면,
    
    ```java
    @Query("select u from User u where u.email = :email")
    User findByEmail(@Param("email") String email);
    ```
    
    `User`는 테이블 이름이 아니라 **엔티티 클래스 이름**이다.
    
    `u.email`도 컬럼명이 아니라 **엔티티 필드명**이다.
    
    Hibernate는 JPQL을 바로 SQL로 바꾸지 않고 의미를 먼저 해석한다.
    
    ```
    User 엔티티
    → users 테이블
    email 필드
    → email 컬럼
    ```
    
    그 이후 SQL로 변환한다.
    
    Hibernate 내부 흐름을 단순화하여 본다면,
    
    ```
    JPQL / HQL / Criteria Query
    → 파싱
    → SQM
    → SQL AST
    → SQL 문자열
    → JDBC 실행
    ```
    
    즉, SQM은 최종 SQL이 아니라 최종 SQL로 가기 전의 **Hibernate 내부 의미 모델**이다.
    
- EntityHolder
    
    > **Hibernate 영속성 컨텍스트 안에서** 특정 `EntityKey`에 해당하는 엔티티, 프록시, EntityEntry, 초기화 상태 등을 함께 들고 있는 **내부 보관 객체이다**.
    > 
    
    Hibernate 내부 구현 쪽 개념으로 `EntityHolder`는 보통 애플리케이션 코드에서 직접 쓰는 API라기 보다는, 영속성 컨텍스트가 엔티티를 추적할 때 쓰는 내부 보관 구조이다.
    
    대략적인 위치를 먼저 잡기 위해 Spring Data JPA 코드 예시를 보면,
    
    ```java
    Optional<Member> member = memberRepository.findById(1L);
    ```
    
    보이는 것은 Repository 매서드 하나지만 내부의 흐름을 보면,
    
    ```
    Repository 메서드 호출
    → Spring Data JPA
    → Hibernate
    → SQL 실행
    → ResultSet 읽기
    → 엔티티 객체 생성 또는 재사용
    → 영속성 컨텍스트에 저장
    ```
    
    이때, Hibernate는 같은 트랜잭션, 세션 안에서 같은 PK의 엔티티를 중복 생성하지 않기 위해 **1차 캐시**, 즉 `PersistenceContext`를 사용한다.
    
    `PersistenceContext` 는 Hibernate가 추적하는 것들의 상태를 나타낸다.
    
    그 안에서 **특정 엔티티 하나를 관리**하기 위한 보관 단위가 바로 **EntityHolder**이다.
    
    ---
    
    **[EntityHolder가 필요한 이유]**
    
    DB에 다음과 같은 ROW가 있고,
    
    ```sql
    member
    id = 1
    name = "Kim"
    ```
    
    같은 트랜잭션 안에서 아래 코드를 실행한다면,
    
    ```java
    Member m1 = entityManager.find(Member.class, 1L);
    Member m2 = entityManager.find(Member.class, 1L);
    
    System.out.println(m1 == m2); // true
    ```
    
    Hibernate가 영속성 컨텍스트 안에서 이미 `Member#1` 을 관리하기 때문에 **true**를 반환한다.
    
    개념적으로 아래와 같은 형태이다.
    
    ```
    PersistenceContext
    └── EntityKey(Member, 1)
        └── EntityHolder
            ├── entity: Member 객체
            ├── proxy: 프록시 객체가 있으면 보관
            ├── entityEntry: 엔티티 상태 정보
            ├── initialized 여부
            └── initializer 정보
    ```
    
    엔티티 객체 하나만 들고 있는 것이 아니라, 그 엔티티를 Hibernate가 어떻게 관리하고 있는지에 대한 정보를 묶어서 들고 있다.
    
    ---
    
    **[핵심 관계]**
    
    **EntityKey**
    
    **EntityKey**는 Hibernate Session 안에서 **특정 엔티티 인스턴스를 식별하는 키이다**.
    
    ```
    EntityKey = Member 엔티티 + id 1
    ```
    
    즉 DB 관점의 PK만 보는게 아니라, Hibernate 관점에서는
    
    ```
    (Member, 1)
    (Order, 1)
    ```
    
    다음과 같이 봐서, 둘 다 id가 1이어도 **타입이 달라 다른 EntityKey**이다.
    
    **EntityHolder**
    
    **EntityHolder**는 그 EntityKey에 매핑되는 보관 객체이다.
    
    Hibernate의 PersistenceContext에는 `getEntityHolder(EntityKey key)`와 `getEntityHoldersByKey()` 같은 매서드가 존재한다.
    
    즉, 영속성 컨텍스트 내부에서 EntityKey를 기준으로 EntityHolder를 조회할 수 있는 구조이다.
    
    ---
    
    **[EntityEntry와 EntityHolder의 차이]**
    
    **EntityHolder**: 특정 EntityKey에 대한 엔티티 보관 껍데기
    
    ```
    이 키에 해당하는 엔티티 객체가 있나?
    프록시가 있나?
    초기화되었나?
    누가 초기화하고 있나?
    EntityEntry는 무엇인가?
    ```
    
    위와 같은 걸 묶어서 들고 있다.
    
    **EntityEntry**: 엔티티의 영속성 상태 정보를 담는 객체이다.
    
    managed entity instance의 현재 상태와 persistent state에 대한 정보이다.
    
    ```
    이 엔티티가 MANAGED 상태인가?
    READ_ONLY인가?
    DELETED 상태인가?
    현재 id는 무엇인가?
    version은 무엇인가?
    loaded state 스냅샷은 무엇인가?
    dirty checking에 필요한 정보는 무엇인가?
    ```
    
    위와 같은 정보를 담당한다.
    
    정리하면 아래와 같은 차이이다.
    
    ```
    EntityHolder = 엔티티를 담는 상자
    EntityEntry = 그 엔티티의 상태 관리 명세서
    ```
    
- EntityEntry
    
    > EntityEntry는 Hibernate 영속성 컨텍스트 안에서 이 엔티티가 현재 어떤 상태이고, DB에서 읽어왔을 때 값이 무엇이었는지를 기록한다.
    > 
    
    **[대략적인 위치]**
    
    JPA/Hibernate에서 엔티티를 조회하면 단순히 객체만 만들어지는 것이 아니다.
    
    ```java
    Member member = entityManager.find(Member.class, 1L);
    ```
    
    내부를 보면 아래와 같다.
    
    ```
    PersistenceContext
    ├── EntityKey(Member, 1)
    │   └── EntityHolder
    │       ├── entity: Member 객체
    │       └── entityEntry: EntityEntry
    ```
    
    **EntityHolder**가 **엔티티를 담는 상자**면, **EntityEntry**는 그 안에 붙어있는 **상태 관리 카드**이다.
    
    ```
    EntityHolder = 엔티티 보관 상자
    EntityEntry = 엔티티 상태 기록표
    ```
    
    ---
    
    **[Dirty Checking의 흐름]**
    
    Dirty Checking은 영속 상태의 엔티티가 바뀌었는지 Hibernate가 자동으로 검사하는 기능이다.
    
    흐름은 아래와 같다.
    
    ```
    1. DB에서 엔티티 조회
    2. Entity 객체 생성
    3. EntityEntry 생성
    4. loadedState에 최초 상태 저장
    5. 개발자가 엔티티 필드 변경
    6. flush 시점 도달
    7. 현재 엔티티 값과 loadedState 비교
    8. 다르면 UPDATE SQL 생성
    9. UPDATE 후 loadedState를 최신 상태로 갱신
    ```
    
    예시를 보면
    
    ```java
    @Transactional
    public void updateMember(Long id) {
        Member member = entityManager.find(Member.class, id);
    
        member.setName("Lee");
    }
    ```
    
    명시적으로 `save()`를 호출하지 않아도 트랜잭션 커밋 시점에 update가 나갈 수 있다.
    
    Hibernate가 `EntityEntry.loadedState`와 현재 값을 비교하기 때문이다.
    
    ---
    
    **[EntityEntry가 flush에서 하는 역할]**
    
    flush는 영속성 컨텍스트의 변경 내용을 DB에 반영하는 과정이다.
    
    ```
    flush
    → 영속성 컨텍스트 안의 엔티티들을 검사
    → EntityEntry 확인
    → INSERT / UPDATE / DELETE 필요 여부 판단
    → SQL 실행
    ```
    
    개념적으론 아래와 같은 느낌이다.
    
    ```java
    for (EntityEntry entry : persistenceContext.entityEntries()) {
        if (entry.getStatus() == MANAGED) {
            // 현재 엔티티 값과 loadedState 비교
            // 다르면 UPDATE 예약
        }
    
        if (entry.getStatus() == DELETED) {
            // DELETE 예약
        }
    }
    ```
    
    ---
    
    **[정리]**
    
    **EntityHolder**는 "어떤 엔티티 객체를 들고 있느냐”에 가깝고,
    
    **EntityEntry**는 "그 엔티티가 어떤 상태이고 원래 값이 무엇이었느냐”에 가깝습니다.
    
    **EntityKey**는 누구인가? (엔티티를 식별하는 키)
    
    **EntityEntry**는 그 사람이 현재 어떤 상태인가? (엔티티 상태를 저장하는 기록)