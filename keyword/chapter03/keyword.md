- **SOLID원칙이란?**
    
    > 객체지향 프로그래밍에서 코드의 유지보수성, 확장성, 유연성을 높이기 위해 준수해야할 5가지 핵심 원칙
    > 
    
    **[단일 책임 원칙 - Single Pesponsibility Principle]**
    
    - 하나의 클래스는 하나의 책임(기능)만 가져야 한다.
    - 클래스가 변경되는 이유는 하나여야 한다.
    
    **[개방-폐쇄 원칙 - Open/Closed Principle]**
    
    - 기존 코드를 변경하지 않고, 확장할 수 있어야 한다.
    - 새로운 기능 추가시 기존 코드를 수정하지 않고도 구현이 가능해야 한다.
    
    **[리스코프 치환 원칙 - Liskov Substitution Principle]**
    
    - 자식 클래스는 부모 클래스를 대체할 수 있어야 한다.
        - 부모 클래스로 자식 클래스를 받을 수 있어야 한다.
    - 하위 클래스가 상위 클래스의 기능을 변경하면 안 된다.
    
    **[인터페이스 분리 원칙 - Interface Segregation Principle]**
    
    - client가 사용하지 않는 인터페이스에 의존하지 않도록 해야한다.
    - 하나의 큰 인터페이스보다는 여러 개의 작은 인터페이스로 분리하는 게 좋다**.**
    
    **[의존성 역전 원칙 - Dependency Inversion Principle]**
    
    - 상위 모듈(추상화)이 하위 모듈(구현)에 의존하는 것이 아니라, 하위 모듈이 상위 모듈에 의존해야한다.
    - 의존성을 줄이기 위해 인터페이스를 활용한다.
    
    **[장, 단점]**
    
    - 장점: 확장성 향상, 유지보수 용이, 결합도 감소
    - 단점: 코드 복잡도 증가, 초기 개발 속도 저하
    

- **DI란?**
    
    > 의존성 주입(Dependency Injection)으로, 외부에서 객체를 생성해 넣어주는 설계 패턴
    > 
    
    **[의존성 - Dependency]**
    
    - 필요로 하는 것, 뭔가를 하기 위해 필요한 것
    
    **[주입 - Injection]**
    
    - 외부에서 누군가 넣어주는 것
    
    **[DI의 3가지 방식]**
    
    - **필드 주입** - 변수 위에 `@Autowired` 를 붙여 사용
        - 코드가 간단하지만, 필드가 `private` 라 순수 자바 단위 테스트가 불가하다.
    
    ```java
    @Service
    public class UserService {
        
        @Autowired
        private UserRepository userRepository;
    }
    ```
    
    - **setter 주입** - setter 매서드를 통해 주입
        - 런타임에 주입받아 중간에 의존성을 바꿀 수 있다.
    
    ```java
    @Service
    public class UserService {
        
        private UserRepository userRepository;
    
        @Autowired
        public void setUserRepository(UserRepository userRepository){
            this.userRepository = userRepository;
        }
    }
    ```
    
    - **생성자 주입**
        - `final`을 붙여 한 번 주입되면 바뀌지 않는다.
        - 의존성 누락시 생성자에서 컴파일 오류를 발생시켜 누락을 방지한다.
        - 롬복 라이브러리를 통해 `@RequiredArgsConstructor` 로 대체
            - `final`이 붙은 필드들이 인자로 들어가는 생성자 생성
    
    ```java
    @Service
    public class UserService {
       
        private final UserRepository userRepository;
    
        public UserService(UserRepository userRepository) {
            this.userRepository = userRepository;
        }
    }
    ```
    

- **IoC란?**
    
    > 제어의 역전(Inversion of Control)으로, 코드의 주도권이 개발자에서 프레임워크나 컨테이너에 위임하는 sw 설계 원칙
    > 
    
    > 개발자가 객체를 직접 `new` 로 생성하고 생명주기를 제어했지만, 이 역할을 프레임워크가 대신한다.
    > 
    
    **[요약된 흐름]**
    
    ```java
    public class Customer {
    	
    	private final Pizza pizza; // 피자를 받을 준비만
    	
    	// 생성자, 누군가 넣어줄 것을 기다림
    	public Customer(Pizza pizza){
    		this.pizza = pizza;
    	}
    	
    	public void eat(){
    		pizza.eat();
    	}
    }
    ```
    
    외부에서 넣어주는 피자를 그냥 먹기만 하면 된다.
    
    **[역전 - Inversion]**
    
    - 기존에는 app → 라이브러리 방향으로 호출
    - framework → app 방향으로 호출
    - 객체의 생명주기가 프레임워크로 넘어가, 흐름이 뒤집혔다고 하여 제어의 역전이라고 부른다.
    
    **[장, 단점]**
    
    - 코드 유연성이 높아진다.
        - `new` 로 강한 의존 관계에선 의존 관계 변경을 위해 service 코드에 수정이 필요하다.
        - 인터페이스를 통해 약한 의존 관계를 만들어, 의존 관계 변경시 service 코드 변경없이 수정 가능하다.
    - 테스트가 용이해진다.
        - Mock 객체를 주입하여 실제 동작 없이 테스트를 돌리기 용이하다.
    - 코드 복잡도가 증가한다.
        - 코드 흐름의 직관적인 파악이 어려워진다.
    

- **생성자 주입 vs 수정자, 필드 주입 차이는?**
    
    > 불변성과 순환참조, 테스트 용이 등의 장점으로 스프링에서 가장 추천하는 방식은 생성자 주입이다.
    > 
    
    **[필드 주입]**
    
    - 변수(필드) 바로 위에 `@Autowired` 를 붙여 사용한다.
    - 코드가 간단하다.
    - 외부에서의 변경이 어려워 단위 테스트 시, 가짜 객체를 넣기 힘들다.
    
    ```java
    @Service
    public class UserService {
        
        @Autowired
        private UserRepository userRepository;
    }
    ```
    
    **[Setter 주입]**
    
    - setter 매서드를 통해 의존성을 주입하는 방식이다.
    - 변경될 수 있는 의존 관계에 사용한다.
        - 매서드가 public이기에 수정이 가능하여 안정성이 떨어진다.
    
    ```java
    @Service
    public class UserService {
       
        private final UserRepository userRepository;
    
        public UserService(UserRepository userRepository) {
            this.userRepository = userRepository;
        }
    }
    ```
    
    **[생성자 주입]**
    
    - 생성자 호출 시 1번 초기화 하는 방식이다.
    - `final` 키워드를 사용하여, 불변성을 보장한다.
    - 의존성 누락시 생성자에서 컴파일 오류를 발생시켜 누락을 방지한다.
    - 롬복 라이브러리를 통해 `@RequiredArgsConstructor` 로 대체
        - `final`이 붙은 필드들이 인자로 들어가는 생성자 생성
    
    ```java
    @Service
    public class UserService {
       
        private final UserRepository userRepository;
    
        public UserService(UserRepository userRepository) {
            this.userRepository = userRepository;
        }
    }
    ```
    

- **AOP란?**
    
    > 관점 지향 프로그래밍(Aspect-Oriented Programming)으로, 핵심 모듈이 아닌 부가 기능을 분리하여 모듈화한다.
    > 
    
    > 즉, 핵심 비즈니스 로직에서 분리하여 재사용하겠다는 점
    > 
    
    **[AOP의 주요 개념]**
    
    - Aspect: 흩어진 관심사를 모듈화한 것(주로 부가기능)
    - Target: Aspect을 적용하는 곳
    - Advice: 실질적으로 어떤 일을 해야할 지에 대한 것, 실질적인 부가 기능을 담은 구현체
    - JointPoint: Advice가 적용될 위치, 끼어들 수 있는 지점
    - PointCut: JointPoint의 세부 스펙을 정의한 것
    
    **[스프링 AOP 특징]**
    
    - 프록시 패턴 기반의 AOP 구현체
    - 스프링 빈에만 적용 가능
    - 모든 AOP 기능을 제공하는 것이 아닌 스프링 IoC와 연동하여 “중복 코드, 프록시 클래스 작성의 번거로움, 객체 사이의 복잡도 증가”에 대한 해결책 지원
    

- **서블릿이란?**
    
    > 동적 웹페이지를 만들 때 사용되는 자바 기반의 웹 애플리케이션 프로그래밍 기술
    > 
    
    웹을 만들 때 다양한 Request와 Response가 있고, 이 요청과 응답에는 규칙이 존재한다.
    
    개발자가 이러한 요청과 응답을 일일이 처리하는 게 아니라, 비즈니스 로직에만 집중할 수 있도록 도와주는 기술이 서블릿이다.
    
    이러한 웹 요청과 응답의 흐름을 **간단한 매서드 호출**만으로 다룰 수 있게 해주는 기술이다.
    
    **[서블릿의 주요 특징]**
    
    - Client의 Request에 대해 동적으로 작동하는 웹 애플리케이션 컴포넌트다.
    - HTML을 사용하여 Response한다.
    - 자바의 스레드를 이용하여 동작한다.
    - MVC 패턴에서의 컨트롤러로 이용된다.
    - HTTP 프로토콜 서비스를 지원하는 HttpServlet 클래스를 상속받는다.
    - UDP보다 속도가 느리다.
    - HTML 변경시 Servlet을 다시 컴파일해야한다.
    
    **[동작 과정]**
    
    - `init()`
        - 서블릿이 처음으로 요청될 때 초기화를 하는 매서드
        - 초기화된 서블릿은 싱글톤으로 관리된다.
    - `service()`
        - 서블릿 컨테이너가 요청을 받고 응답을 내려줄 때 필요한 매서드
        - HttpServelt 클래스의 doGet, doPost 같은 매서드들이 호출된다.
    - `destroy()`
        - 더 이상 사용되지 않는 서블릿 클래스는 주기적으로 서블릿 컨테이너가 매서드를 호출하여 제거한다.
    
    **[서블릿 생명주기]**
    
    - 서블릿도 자바 클래스이므로 실행하면 초기화부터 서비스 수행 후 소멸하기까지의 과정을 거친다.


