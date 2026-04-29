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