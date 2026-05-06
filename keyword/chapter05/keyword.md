- 빌더패턴이란?
    
    > 빌더 패턴이란, 복잡한 객체 생성 과정을 단계적으로 진행할 수 있도록 돕는 생성 패턴이다.
    > 
    
    **[빌더 패턴 이전 문제점]**
    
    객체를 생성할 때, 필요한 매개변수를 모두 포함하는 생성자를 사용하게 되는데 이때 다양한 문제들이 발생할 수 있다. 
    
    1. 매개변수의 순서 혼동
    2. 일부 매개변수에 대해 null 값 또는 기본값 할당시 의미 파악 어려움
    3. 객체 변경 가능성
    
    **[빌더 패턴의 해결]**
    
    위와 같은 문제들을 해결하기 위해 별도의 Builder 클래스를 만들어 필수 값에 대해서는 생성자를 사용하고, 선택적인 값들에 대해서는 **메소드**를 통해 필요한 값들을 입력 받고 `build()` 매서드를 사용해 인스턴스를 리턴 받는다.
    
    **[빌더 패턴의 장단점]**
    
    - **장점**
        - 생성할 객체의 속성을 자유롭게 지정 가능
        - 선택적인 속성은 매서드 체이닝을 통해서 처리 가능
        - 코드의 가독성 유지
        - 재사용 가능
    - **단점**
        - 각 속성에 대한 setter 매서드가 필요하여, 전체적인 코드의 양이 많아질 수 있다
    
    **[구현 방법]**
    
    1. **빌더 클래스 직접 선언**
        1. 빌더 클래스를 선언하고, 생성할 객체의 속성에 대한 setter 매서드를 구현
        2. 이 매서드들은 빌더 객체 자신을 반환하므로 매서드 체이닝이 가능하다
    
    ```java
    public class User {
        private String name;
        private int age;
        
        public static class Builder {
            private String name;
            private int age;
            
            public Builder withName(String name) {
                this.name = name;
                return this;
            }
            
            public Builder withAge(int age) {
                this.age = age;
                return this;
            }
            
            public User build() {
                User user = new User();
                user.name = this.name;
                user.age = this.age;
                return user;
            }
        }
        
        public static void main(String[] args) {
            User user = new User.Builder().withName("Evan").withAge(26).build();
        }
    }
    ```
    
    1. **Lombok 라이브러리 사용**
        1. `@Builder` 어노테이션을 사용하여 빌더 패턴을 구현 가능
    
    ```java
    @Builder
    @Getter
    @Setter
    public class User {
        private String name;
        private int age;
    }
    
    public static void main(String[] args) {
        User user = User.builder().name("Evan").age(26).build();
    }
    ```
    
- record vs static class
    
    > Record란, 변경이 불가한 데이터 객체를 쉽게 만들 수 있게 해주는 자바 문법으로 Java16에서 정식 기능으로 포함되었다.
    > 
    
    **[Record]**
    
    `record` 는 데이터 그 자체를 표현하기 위해 도입되었다. 클래스 이름 뒤에 `()` 를 사용하여 필드를 선언하며, 컴파일러가 번거로운 **보일러플레이트 코**드를 대신 만들어준다.
    
    ```java
    // 한 줄로 데이터 객체 정의
    public record User(String name, int age) {}
    ```
    
    **[보일러플레이트 코드]**
    
    최소한의 변경(인자 혹은 결과 타입)으로 여러 곳에서 재사용 되면서 반복적으로 비슷한 형태를 가지고 있는 코드
    
    > `getter`, `setter`, `equals`, `hashCode`, `toString` 등
    > 
    
    **[Record 특징]**
    
    - 모든 필드가 `private final`로 선언된다.
    - 다른 클래스를 상속 받을 수 없지만, 인터페이스로는 구현이 가능하다.
    - `name()` 과 같은 형태의 Getter가 자동으로 생성된다.
    - 데이터의 값이 같으면 동일한 객체로 취급하도록 `equals`와 `hashCode`가 재정의된다.
    
    **[Static Class: 정적 내부 클래스]**
    
    Static Nested Class를 의미하며, 외부 클래스의 인스턴스 없이 생성할 수 있는 내부 클래스이다.
    
    ```java
    public class Outer {
        public static class Inner {
            private String name;
            private int age;
    
            public Inner(String name, int age) {
                this.name = name;
                this.age = age;
            }
            // Getter, Setter, equals, toString 등을 직접 구현해야 함
        }
    }
    ```
    
    **[Static Class 특징]**
    
    - 외부 클래스의 인스턴스 멤버에 접근할 수 없다.
    - 일반 클래스처럼 상태를 변경할 수 있고, 다른 클래스를 상속받을 수 있다.
    - 주로 특정 클래스 내부에서만 보조적으로 사용되는 구조를 정의할 때 쓴다.
    
    **[둘을 언제 사용해야 하는가]**
    
    1. **Record를 사용하는 경우**
        - DB 조회 결과나 API 응답 값을 담는 DTO를 만들 때
        - 값의 변경이 필요 없는 불변 객체가 필요할 때
        - `Map`의 키값으로 객체를 사용해야 해서 `equals/hashCode`가 중요할 때
        
    2. **Static Class를 사용하는 경우**
        - 객체의 상태가 계속 변해야(Setter 필요) 할 때.
        - 클래스 계층 구조가 필요하여 상속을 받아야 할 때.
        - 외부 클래스의 기능을 보조하는 논리적인 그룹화가 필요할 때.
    
- 제네릭이란?
    
    > 제네릭(generics)은 데이터의 타입을 일반화하는 것을 의미한다
    > 
    
    **[제네릭 정의]**
    
    - C++의 템플릿 등 정적 언어(C, C++, Java)
    - 제네릭은 JDK 1.5부터 도입되었으며, 다양한 타입의 객체들을 다루는 메서드나 컬렉션 클래스에 컴파일 시점에 타입 체크를 해주는 기능
    - 객체의 타입을 고정하지 않고, 사용할 때 결정할 수 있도록 설계된 '타입의 일반화'
    
    **[제네릭 사용법]**
    
    1. **제네릭 클래스 및 인터페이스**
    
    클래스 이름 뒤에 `<T>`와 같은 타입 매개변수를 붙여 선언
    
    ```java
    public class Box<T> {
        private T item;
        public void set(T item) { this.item = item; }
        public T get() { return item; }
    }
    
    // 사용 예시
    Box<String> stringBox = new Box<>();
    stringBox.set("Hello");
    ```
    
    1. **제네릭 매서드**
    
    매서드 반환 타입 앞에 `<T>`를 선언하여 해당 매서드 내에서만 유효한 타입 정의
    
    ```java
    public <T> T printAndReturn(T t) {
        System.out.println(t);
        return t;
    }
    ```
    
    1. **와일드카드(`?`)**
    
    타입 매개변수의 범위를 제한하거나 유연하게 사용할 때 쓴다
    
    ```java
    <? extends T> // T와 그 자식 클래스만 가능 (상한 제한)
    
    <? super T> // T와 그 부모 클래스만 가능 (하한 제한)
    
    <?> // 모든 타입 가능
    ```
    
    **[제네릭 특징]**
    
    - **타입 안정성**: 의도하지 않은 타입의 객체가 저장되는 것을 막고, 꺼낼 때 타입 오류를 방지
    - **타입 소거**: 자바 컴파일러는 하위 호환성을 위해 컴파일 타임에만 제네릭 타입을 체크하고, 런타임에는 제네릭 정보를 제거하여 Object로 변환한다
    - **기본 타입 사용 불가**: `int`, `double` 같은 기본 타입을 제네릭에 사용할 수 없으며, `Integer` , `Double` 과 같은 Wrapper 클래스를 사용
    
    **[제네릭 장점]**
    
    1. **컴파일 타임 체크**: 런타임에 발생할 수 있는 `ClassCastException` 을 컴파일 시점에 미리 발견할 수 있다.
    2. **형변환(Casting) 생략**: 컬렉션에서 객체를 꺼낼때 수동으로 형변환을 할 필요가 없어 코드가 간결해진다.
    
    ```java
    String str = (String) list.get(0); // 일반
    
    String str = list.get(0); // 제네릭
    ```
    
    1. **코드 재사용성 향상**: 하나의 클래스나 매서드로 다양한 데이터 타입을 처리할 수 있어 중복 코드가 줄어든다.
    
- @RestControllerAdvice이란?
    
    > @RestControllerAdvice는 스프링 프레임워크에서 제공하는 전역 예외 처리 및 응답 공통 관리를 위한 기능
    > 
    
    **[배경]**
    
    만약 실제 운영하고 있는 시스템에서 에러가 발생해서 “Whitelable Error Page” 화면이 나오면 사용자들은 상당한 불편함을 겪을 것 입니다. 
    
    서버가 죽지 않게 할려면 try-catch 구문으로 예외처리를 일일히 작성해야 하지만, Spring에서는 `@ExceptionHandler` 어노테이션을 통해 매우 유연하고 간단하게 예외처리를 할 수 있게 해줍니다.
    
    **[@RestControllerAdvice란?]**
    
    `@ControllerAdvice`와 `@ResponseBody`를 합친 어노테이션
    
    - **@ControllerAdvice**: 여러 `@Controller`클래스에 분산된 `@ExceptionHandler`, `@InitBinder`, `@ModelAttribute`를 한곳에서 관리할 수 있게 해준다.
    - **@ResponseBody:** 메서드의 반환값을 HTTP 응답 바디(JSON 등)로 직접 렌더링한다.
    
    **[사용법]**
    
    일반적으로 공통 응답 형식을 정의한 뒤, 이를 반환하는 예외 처리기를 작성
    
    ```java
    @RestControllerAdvice
    public class GlobalExceptionHandler {
    
        // 특정 예외(CustomException)가 발생했을 때 실행
        @ExceptionHandler(CustomException.class)
        public ResponseEntity<ErrorResponse> handleCustomException(CustomException ex) {
            ErrorResponse response = new ErrorResponse("ERR_01", ex.getMessage());
            return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
        }
    
        // 그 외 모든 예외 처리
        @ExceptionHandler(Exception.class)
        public ResponseEntity<ErrorResponse> handleAllException(Exception ex) {
            ErrorResponse response = new ErrorResponse("ERR_500", "서버 내부 오류 발생");
            return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
    ```
    
    **[장단점]**
    
    **장점**
    
    - **보일러플레이트 코드 감소**: 각 컨트롤러마다 `try-catch` 를 작성할 필요가 없어져 코드가 깔끔해진다.
    - **일관된 응답구조**: 모든 에러 메시지 형식을 통일하여 통신 규약을 맞추기 쉽다.
    - **중앙 집중화**: 예외 처리 로직이 한데 모여있어 유지보수가 용이
    
    **단점**
    
    - **복잡성 증가**: 프로젝트 규모가 커지면 여러 Advice간 우선순위를 관리해야함
    - **과도한 추상화**: 모든 예외를 하나로 묶어서 처리하다 보면, 특정 컨트롤러에서만 필요한 특수 예외처리가 가려질 수 있다.
    
- Optional이란?
    
    > Java 8에서 도입된 기능으로, 값이 존재할 수도 있고 없을 수도 있는 상황을 객체로 감싸서 다루는 컨테이너 클래스
    > 
    
    **[Optional 이전]**
    
    `Optional` 이전에는 값이 없음을 나타내기 위해 `null` 을 직접적으로 사용하였으나 **NullPointerException**과 같은 런타임에 프로그램이 종료되는 오류 발생
    
    **[Optional이란?]**
    
    `Optional<T>`는 값이 존재할 수도 있고 아닐 수도 있는 단일 객체를 감싸 Wrapper 클래스입니다. 명시적으로 "이 변수는 값이 없을 수도 있음"을 사용자에게 알려주어, 잠재적인 `null` 관련 버그를 설계 단계에서 방지한다.
    
    **[사용법]**
    
    1. **객체 생성**
        - `Optional.of(value)`: 값이 확실히 있을 때 (null이면 예외 발생)
        - `Optional.ofNullable(value)`: 값이 null일 수도 있을 때
        - `Optional.empty()`: 비어있는 객체 생성
        
    2. **값 추출 및 처리**
    
    ```java
    Optional<String> opt = Optional.ofNullable(getName());
    
    // 값이 있을 때만 실행
    opt.ifPresent(name -> System.out.println("Hello, " + name));
    
    // 값이 없으면 기본값 반환
    String result = opt.orElse("Default Name");
    
    // 값이 없으면 예외 발생
    String result2 = opt.orElseThrow(() -> new NoSuchElementException());
    ```
    
    **[장단점]**
    
    **장점**
    
    - **명시적 의도 표현:** 반환 타입이 `Optional`이면 개발자는 반드시 null 가능성을 고려하게 된다.
    - **NPE 방지:** 직접적인 `null` 접근을 차단하여 런타임 안정성을 높인다.
    - **깔끔한 코드:** 복잡한 `if-null` 체크를 함수형 메서드로 대체하여 가독성을 높인다.
    
    **단점**
    
    - **오버헤드:** 객체를 한 번 더 감싸는 형태이므로 미세한 성능 저하와 추가 메모리 사용이 발생한다.
    - **남용 시 복잡성:** 모든 곳에 `Optional`을 사용하면 코드가 지나치게 복잡해지고, 직렬화 문제 등이 발생할 수 있다.


---
- 오케스트레이션 서비스와 쿠버네티스
    
    > 현대 IT 인프라에서 컨테이너 기술이 표준이 되면서, 이를 효율적으로 관리하기 위한 서비스와 대표적인 플랫폼이다.
    > 
    
    **[오케스트레이션 서비스 (Orchestration Service)]**
    
    과거에는 서비스마다 별도의 서버를 두거나 가상머신(VM)을 사용했다. 컨테이너(Docker 등) 기술이 등장하면서 가볍고 빠르게 배포가 가능해졌지만, 서비스가 커지면 관리해야할 컨테이너가 너무 많아져 개발자가 일일이 배포, 복구, 설정을 관리하는 것이 불가능해졌다. 이를 자동화해주는 기술이 “오케스트레이션”이다.
    
    **[오케스트레이션 특징]**
    
    - **배포 자동화:** 수많은 컨테이너를 여러 서버에 자동으로 배치한다.
    - **오토 힐링 (Auto-healing):** 장애가 발생한 컨테이너를 감지하고 자동으로 재시작하거나 교체한다.
    - **오토 스케일링 (Auto-Scaling):** 트래픽 증가 시 컨테이너 개수를 자동으로 늘리고 (Scale-out), 감소 시 줄인다 (Scale-in).
    
    **[쿠버네티스 ((Kubernetes, K8s)]**
    
    구글이 내부적으로 사용하던 컨테이너 관리 시스템 ‘Brog’를 기반으로 만든 오픈소스 컨테이너 오케스트레이션 플랫폼이다. 
    
    **[쿠버네티스 특징]**
    
    - **선언적 구성 (Declarative Configuration):** 사용자가 "컨테이너 3개를 유지해줘"라고 상태를 정의(YAML 파일 등)하면, 쿠버네티스가 현재 상태를 확인하고 목표 상태에 맞게 스스로 조정한다.
    - **서비스 디스커버리와 로드 밸런싱:** 컨테이너에 고유한 IP를 부여하고 트래픽을 골고루 분산시킨다.
    - **다양한 환경 지원:** 온프레미스, 퍼블릭 클라우드(AWS, GCP, Azure), 하이브리드 클라우드 어디서든 동일하게 동작한다.
    
    **[장, 단점]**
    
    **장점**
    
    - **고가용성:** 서비스 중단 없는 안정적 운영이 가능하다.
    - **비용 효율:** 자원 사용률 최적화가 가능하다.
    - **유연성:** 특정 클라우드 서비스 업체에 종속되지 않는다.
    
    **단점**
    
    - **높은 학습 곡선:** 설정과 개념이 복잡하다.
    - **운영 부담:** 직접 구축 시 인프라 관리가 까다롭다.
    - **오버헤드:** 소규모 서비스에는 오버엔지니어링일 수 있다.
    
    **[실제 사용 사례]**
    
    - **마이크로서비스 아키텍처 (MSA):** 각 기능을 독립된 컨테이너로 쪼개어 배포할 때 필요하다.
    - **CI/CD 자동화:** 코드 수정 후 빌드된 이미지를 자동으로 테스트 서버나 운영 서버에 배포하는 환경 구축에 사용된다.
    
- EDA
    
    > EDA란, 이벤트 기반 아키텍처(Event-Driven Architecture)로 현대 분산 시스템과 마이크로서비스 환경에서 중요한 위치를 차지하고 있는 설계 패턴이다.
    > 
    
    **[EDA 등장 배경]**
    
    과거 시스템은 주로 **요청-응답 방식**의 동기식 구조였다. 해당 구조는 **강한 결합도**, **확장성 저하**, **장애 전파** 등과 같은 한계를 보였고, 이러한 문제를 해결하기 위해 시스템간의 의존성을 낮추고, 독립성을 확보하려는 흐름에서 주목받게 되었다.
    
    **[EDA 정의]**
    
    EDA는 상태의 변화(이벤트)를 생성하고, 이를 감지하여 반응하는 방식으로 동작하는 소프트웨어 설계 패턴이다.
    
    - **이벤트(Event):** "주문 완료", "회원 가입" 등 시스템 내부에서 발생한 중요한 상태의 변화를 의미한다.
    - **동작 방식:** 특정 서비스가 이벤트를 발행(Publish)하면, 해당 이벤트에 관심 있는 다른 서비스들이 이를 수신(Subscribe)하여 각자의 로직을 처리한다.
    
    **[주요 특징]**
    
    - **비동기성 (Asynchrony):** 생산자는 이벤트를 던진 후 소비자의 처리가 끝날 때까지 기다리지 않고 다음 업무를 수행한다.
    - **느슨한 결합 (Loose Coupling):** 서비스를 제공하는 쪽과 사용하는 쪽이 서로의 존재를 몰라도 된다. 이벤트라는 데이터 인터페이스로만 소통한다.
    - **확장성 (Scalability):** 트래픽이 몰리는 특정 소비자(Consumer) 서버만 유연하게 늘려 대응하기 쉽다.
    
     
    
- 카프카
    
    > 카프카란, 고성능 분산 **이벤트 스트리밍 플랫폼**이다. 단순히 메시지를 주고받는 큐 역할을 넘어, 대규모 데이터를 실시간으로 수집, 저장, 처리할 수 있는 파이프라인 역할을 수행한다.
    > 
    
    **[카프카 등장 배경]**
    
    카프카란, 기존의 메시징 시스템(RabiitMQ 등)이 데이터의 순서 보장이나 대용량 처리에 한계가 있었고 서비스가 늘어날수록 시스템 간의 연결이 복잡해지는 문제가 있었다. 이를 해결하기 위해 등장한 중앙 집중형 메시지 스트리밍 플랫폼이다. 
    
    **[주요 특징]**
    
    - **분산 아키텍처:** 여러 대의 버(브로커)로 클러스터를 구성하여 데이터를 분산 저장하므로 성능 확장이 쉽다.
    - **고가용성 및 복제:** 데이터를 여러 브로커에 복제하여 저장하기 때문에 특정 서버가 고장 나도 데이터 유실 없이 서비스를 유지한다.
    - **디스크 저장 (Persistence):** 메시지를 메모리가 아닌 디스크에 저장합니다. 덕분에 컨슈머가 과거의 데이터를 다시 읽거나, 장애 복구 시 데이터를 보존하기 유리하다.
    - **높은 처리량 (Throughput):** 일괄 처리(Batching)와 제로 카피 기술을 사용하여 초당 수백만 건의 데이터를 처리할 수 있다.
    
    **[카프카 핵심 구성 요소]**
    
    - **Producer (생산자):** 이벤트를 생성하여 카프카의 특정 토픽(Topic)으로 보내는 애플리케이션이다.
    - **Topic (토픽):** 데이터가 저장되는 저장소로 데이터를 구분하는 기준이 된다.
    - **Consumer (소비자):** 토픽에 저장된 메시지를 가져와서 처리하는 애플리케이션이다.
    - **Broker (브로커):** 카프카가 설치된 서버 자체를 의미하며, 메시지를 저장하고 관리한다.
    
- 메시지큐와 이벤트 소싱
    
    > 메시지 큐와 이벤트 소싱은 비동기 시스템과 데이터 무결성을 설계할 때 핵심 개념들이다.
    > 
    
    **메시지 큐(Message Queue, MQ)**
    
    **[메시지 큐란?]**
    
    과거에는 시스템 간 데이터를 주고 받을 때, 상대방의 응답을 기다리는 **동기 방식**을 사용했다. 하지만 상대방 서버가 죽어있거나 응답이 늦어지면 시스템 전체가 마비되는 문제가 발생했고, 이를 해결하기 위해 중간에 메시지를 담아두는 역할을 하는 것이다.
    
    **[주요 특징]**
    
    - **비동기성(Asynchrony):** 생산자는 메시지를 큐에 넣고 즉시 자기 일을 계속한다.
    - **디커플링(Decoupling):** 송신자와 수신자가 서로의 존재나 상태를 몰라도 통신이 가능하다.
    - **버퍼링:** 트래픽이 몰려도 큐에 쌓아두었다가 수신자가 처리할 수 있는 속도로 가져간다.
    
    **[장단점]**
    
    - **장점:** 시스템의 확장성과 안정성이 높아지며, 부하 분산이 용이하다.
    - **단점:** 실시간 응답이 필요한 곳에는 부적합하며, 큐 관리라는 운영 비용이 발생한다.
    
    **[대표 도구]**
    
    RabbitMQ, ActiveMQ, Amazon SQS 등
    
    **이벤트 소싱 (Event Sourcing)**
    
    **[이벤트 소싱이란?]**
    
    일반적인 DB는 데이터의 **최종 상태**만 저장한다. 이벤트 소싱은 상태를 바꾸는 모든 사건(Event)을 순서대로 저장하여, 필요할 때 이 사건들을 처음부터 다시 실행(Replay)하여 현재 상태를 만들어내는 방식이다.
    
    **[주요 특징]**
    
    - **불변성(Immutability):** 한 번 발생한 이벤트는 수정되거나 삭제되지 않고 오직 추가만 된다.
    - **완벽한 감사 추적(Audit Trail):** 데이터가 왜 이렇게 변했는지 모든 히스토리를 추적할 수 있다.
    - **재현 가능성:** 과거 특정 시점의 데이터 상태를 언제든지 복원할 수 있다.
    
    **[장단점]**
    
    - **장점:** 데이터 유실 위험이 적고, 복잡한 비즈니스 로직의 히스토리를 완벽히 보존한다.
    - **단점:** 구현 난이도가 높고, 이벤트가 많아지면 현재 상태를 계산하는 속도가 느려져 스냅샷(Snapshot) 같은 보완 기법이 필요하다.
    
    **[대표 도구]**
    
    EventStoreDB, Apache Kafka(로그 저장소로 활용) 등
    
- 2PC 패턴과 SAGA 패턴
    
    > 분산 시스템에서 데이터 일관성을 유지하기 위한 핵심 전략들이다.
    > 
    
    **2PC (2-Phase Commit)**
    
    **[2PC란?]**
    
    여러 개의 데이터베이스나 서비스가 하나의 트랜잭션으로 묶여야 할 때, 데이터의 원자성을 보장하기 위해 등장했다. 중앙 제어자가 모든 참여자의 동의를 받아 트랜잭션을 확정하는 방식이다.
    
    **[주요 특징]**
    
    - **원자성(Atomicity) 보장:** '전부 아니면 전무(All or Nothing)'를 확실히 보장합니다.
    - **동기식 프로토콜:** 모든 참여자의 응답이 올 때까지 자원을 점유(Lock)하고 기다립니다.
    
    **[핵심 개념]**
    
    - **Prepare 단계:** 제어자가 모든 참여자에게 "준비됐니?"라고 묻습니다. 참여자들은 자원을 잠그고 "Yes" 또는 "No"로 응답합니다.
    - **Commit 단계:** 모두가 "Yes"라고 하면 제어자가 "확정해!"라고 명령합니다. 한 명이라도 "No"라면 "취소해!"라고 명령(Rollback)합니다.
    
    **[장단점]**
    
    - **장점:** 데이터의 강력한 일관성(Strong Consistency)을 보장한다.
    - **단점:** 성능 저하가 심합니다. 한 참여자가 응답하지 않으면 전체 시스템이 대기 상태에 빠지는 블로킹 문제가 발생하며, 장애 대응에 취약하다.
    
    **Saga 패턴 (Saga Pattern)**
    
    **[Saga 패턴이란?]**
    
    마이크로서비스 아키텍처(MSA)에서는 서비스마다 DB가 다르기 때문에 2PC를 쓰기 어렵고 느리다. 이를 해결하기 위해 긴 트랜잭션을 여러 개의 작은 로컬 트랜잭션으로 쪼개고 순차적으로 실행하는 방식이다.
    
    **[주요 특징]**
    
    - **최종 일관성(Eventual Consistency):** 실시간으로 모든 데이터가 일치하진 않지만, 결국에는 일치하게 된다
    - **보상 트랜잭션(Compensating Transaction):** 중간에 작업이 실패하면, 이미 성공한 앞선 단계들을 취소하는 **반대 작업**을 실행하여 논리적으로 롤백을 구현한다.
    
    **[핵심 개념]**
    
    - **코레오그래피(Choreography):** 중앙 제어자 없이 각 서비스가 이벤트를 주고받으며 다음 단계를 진행한다.
    - **오케스트레이션(Orchestration):** 별도의 관리자가 각 서비스에게 무엇을 할지 순서대로 지시한다.
    
    **[장단점]**
    
    - **장점:** 서비스 간 결합도가 낮고 성능이 뛰어납니다. 긴 시간이 걸리는 비즈니스 프로세스에 적합하다.
    - **단점:** 설계가 복잡합니다. 보상 트랜잭션을 일일이 작성해야 하며, 데이터가 잠시 불일치하는 상태를 고려한 프로그래밍이 필요하다.
    
- Spring Cloud Eureka
    
    > Spring Cloud Eureka는 서비스 디스커버리(Service Discovery) 서버로, 각 마이크로서비스의 위치(IP, 포트, 호스트명)를 등록하고 관리하며, 서비스가 서로를 이름(Service ID)으로 찾을 수 있게 해주는 역할을 한다.
    > 
    
    **[Spring Cloud Eureka** **등장 배경]**
    
    MSA 환경에서는 서비스들이 유동적으로 생성되고 소멸하며 오토스케일링에 의해 IP나 포트 본호가 수시로 변경된다. 특정 서비스가 다른 서비스를 호출하기 위해 설정 파일에 직접 기록해두는 방식은 관리가 어렵기 때문에, 이를 해결하기 위해 서비스의 위치를 자동으로 등록하고 찾아주는 시스템이 필요했다.
    
    **[주요 특징]**
    
    - **Service Registration:** 각 서비스가 구동될 때 자신의 정보를 Eureka 서버에 자동으로 등록한다.
    - **Service Discovery:** 클라이언트가 특정 서비스의 이름을 요청하면 Eureka가 사용 가능한 위치 목록을 반환한다.
    - **Health Check (Heartbeat):** 서비스가 일정 간격으로 Eureka에 "나 살아있어”라는 신호(Heartbeat)를 보냅니다. 신호가 끊기면 Eureka는 해당 서비스를 목록에서 제거한다.
    - **Self-Preservation (자기 보호 모드):** 일시적인 네트워크 장애로 인해 많은 서비스의 신호가 끊길 경우, 유효한 서비스까지 지워버리는 것을 방지하기 위해 등록 정보를 유지하는 기능이다.
    
    **[사용 방법]**
    
    Spring Cloud 환경에서 두 부분으로 나눠 설정한다.
    
    1. **Eureka Server:**
        - `@EnableEurekaServer` 어노테이션을 메인 클래스에 추가
        - `application.yml`에서 서버 포트와 자체 등록 방지(`register-with-eureka: false`) 등을 설정한다.
    2. **Eureka Client (각 마이크로서비스):**
        - `spring-cloud-starter-netflix-eureka-client` 의존성을 추가
        - `@EnableDiscoveryClient` 어노테이션을 사용한다.
        - `spring.application.name`을 지정하여 Eureka 서버에 등록될 이름을 결정