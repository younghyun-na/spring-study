# Spring 핵심 원리
## Spring
+ 핵심 컨셉: 자바 언어 기반의 프레임워크 
+ 핵심: 좋은 객체 지향 애플리케이션을 개발할 수 있게 도와주는 프레임워크
  + 다음 기술로 다형성, OCP, DIP를 가능하게 지원함

## 스프링 프레임워크
+ 핵심 기술: 스프링 DI 컨테이너, AOP, 이벤트, ...

## 스프링 부트
+ 단독으로 실행할 수 있는 스프링 애플리케이션을 쉽게 생성
+ 기본적으로 스프링 프레임워크 사용

## 객체 지향 프로그래밍
+ 다형성: 역할(인터페이스)과 구현(인터페이스를 구현한 클래스, 구현 객체)으로 세상을 구분
  + 유연하고 변경에 용이: 클라이언트는 대상의 역할(인터페이스)만 알면되고 내부구조와 구현 대상의 변경에 영향을 받지 않는다.
  + 오버라이딩
  ```java
  public class MemberService {
    //private MemberRepository memberRepository = new MemoryMemberRepository();
      private MemberRepository memberRepository = new JdbcMemberRepository();
  }
  ```
  + 확장 가능한 설계: 클라이언트를 변경하지 않고 서버의 구현 기능을 유연하게 변경 가능
  + 인터페이스를 안정적으로 설계하는 것이 중요: 역할(인터페이스)가 변경되면 클라이언트, 서버 모두 변경해야 됨
  + 스프링이 다형성을 극대화: 제어의 역전(IoC), 의존관계 주입(DI)이 다형성을 활용
  
## SOLID 원칙
: 좋은 객체 지향 설계의 5가지 원칙
+ `SRP(단일 책임 원칙)`: 한 클래스는 하나의 책임만 가져야 함 (중요한 기준은 변경)
+ `OCP(개방-폐쇄 원칙)`: 확장에는 열려 있으나 변경에는 닫혀 있어야 함
+ `LSP(리스코프 치환 원칙)`: 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 변경 가능해야 함
+ `ISP(인터페이스 분리 원칙)`: 특정 클라이언트를 위한 인터페이스 여러 개가 범용 인터페이스 하나보다 나음
+ `DIP(의존관계 역전 원칙)`: 구현(구현체)이 아닌 역할(인터페이스)에 의존해야 유연하게 구현체 변경이 가능 
> 다형성 만으로는 OCP, DIP를 지킬 수 없다.
  ```java
  public class MemberService {
    //private MemberRepository memberRepository = new MemoryMemberRepository();
      private MemberRepository memberRepository = new JdbcMemberRepository();
  }
  ```
  + DIP 위반
    + 추상화(`memberRepository` 인터페이스)에 의존하고 있지만, 구체화(`MemoryMemberRepository` 클래스)에도 의존함   
    => `MemberService`는 `memberRepository`인터페이스에만 의존하도록 설계해야 한다.
  + OCP 위반
    + 구현 객체를 `MemoryMemberRepository`에서 `JdbcMemberRepository`로 변경하려면 코드 변경해야 함.   
    => `spring container`(객체를 생성하고 연관관계를 맺어주는 별도의 조립, 설정자)가 필요하다. 
  ```java
  public class OrderServiceImpl implements OrderService{

      private final MemberRepository memberRepository = new MemoryMemberRepository(); 
      //private final DiscountPolicy discountPolicy = new FixDiscountPolicy();  
      private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
  }
  ```
  + DIP 위반
    + 클라이언트인 `OrderServiceImpl`이 `DiscountPolicy`인터페이스 뿐만 아니라 `FixDiscountPolicy`인 구체클래스도 함께 의존하고 있다. 
  + OCP 위반
    + 기능을 확장해서 변경하는 경우, `FixDiscountPolicy` 를 `RateDiscountPolicy`로 변경하는 순간 `OrderServiceImpl`의 소스 코드도 함께 변경해야 한다
  => 이 문제를 해결하려면 누군가가 클라이언트인 `OrderServiceImpl`에 `DiscountPolicy`의 구현 객체를 대신 생성하고 주입해주어야 한다.
  
## 관심사의 분리
: 공연 기획자(`AppConfig`)를 만들고, 배우와 공연 기획자의 책임을 확실히 분리하자.   
  객체를 생성하고 연결하는 역할(`AppConfig`)과 실행하는 역할을 명확히 분리

#### 기존 소스코드
> MemberServiceImpl.java 중 일부
```java
public class MemberServiceImpl implements MemberService{

    private final MemberRepository memberRepository = new MemoryMemberRepository();
    // DIP 위반: MemberServiceImpl은 MemberRepository(추상화), MemoryMemberRepository(구체화)에 모두 의존
}
```
> OrderServiceImpl.java 중 일부
```java
public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository = new MemoryMemberRepository();  // 인터페이스에만 의존
    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();   // 인터페이스에만 의존
}
``` 
> OrderServiceTest.java 중 일부
```java
public class OrderServiceTest {
    MemberService memberService = new MemberServiceImpl();
    OrderService orderService = new OrderServiceImpl();
}
```

#### 변화된 소스코드
+ `AppConfig`: 애플리케이션의 실제 동작에 필요한 구현 객체를 생성하고, 생성한 객체 인스턴스 참조를 생성자를 통해서 연결하는 클래스  

> AppConfig.java 생성
```java
public class AppConfig {
    public MemberService memberService(){
        return new MemberServiceImpl(new MemoryMemberRepository());
}
```
> MemberServiceImpl.java 수정
```java
public class MemberServiceImpl implements MemberService{

    private final MemberRepository memberRepository;
    // MemberServiceImpl은 MemberRepository(추상화)에만 의존, 구체적인 것은 모름 (생성자 주입)
    
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```
+ `MemberServiceImpl` 입장에서 생성자를 통해 어떤 구현 객체가 들어올지는 알 수 없다.
+ `MemberServiceImpl`의 생성자를 통해서 어떤 구현 객체를 주입할지는 오직 외부(`AppConfig`)에서 결정된다.   
=> 의존관계에 대한 고민은 외부에 맡기고 실행에만 집중할 수 있게 됨

> OrderServiceImpl.java 수정
```java
public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository;  // 인터페이스에만 의존
    private final DiscountPolicy discountPolicy ;   // 인터페이스에만 의존

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;     // discountpolicy가 무엇이 들어올지 모름
    }
}
``` 
+ `OrderServiceImpl` 입장에서 생성자를 통해 어떤 구현 객체가 들어올지는 알 수 없다.
+ `OrderServiceImpl`의 생성자를 통해서 어떤 구현 객체을 주입할지는 오직 외부(`AppConfig`)에서 결정
한다.
+ `OrderServiceImpl`에는 `MemoryMemberRepository`, `FixDiscountPolicy` 객체의 의존관계가 주입
된다.

> OrderServiceTest.java 수정
```java
public class OrderServiceTest {

    MemberService memberService;
    OrderService orderService;

    @BeforeEach
    public void beforeEach(){
        AppConfig appConfig = new AppConfig();
        memberService = appConfig.memberService();
        orderService = appConfig.orderService();
    }
}
```
