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
