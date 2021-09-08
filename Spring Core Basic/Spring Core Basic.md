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
  + 관심사를 분리함 (클라이언트 객체 = 실행, AppConfig = 구현 객체 생성과 연결)
+ `OCP(개방-폐쇄 원칙)`: 확장에는 열려 있으나 변경에는 닫혀 있어야 함
+ `LSP(리스코프 치환 원칙)`: 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 변경 가능해야 함
+ `ISP(인터페이스 분리 원칙)`: 특정 클라이언트를 위한 인터페이스 여러 개가 범용 인터페이스 하나보다 나음
+ `DIP(의존관계 역전 원칙)`: 추상화에 의존해야지, 구체화에 의존하면 안된다. 
  + 구현(구현체)이 아닌 역할(인터페이스)에 의존해야 유연하게 구현체 변경이 가능 
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
   => 객체를 생성하고 연결하는 역할(`AppConfig`)과 실행하는 역할을 명확히 분리

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
    public OrderService orderService() {
        return new OrderServiceImpl(new MemoryMemberRepository(), new FixDiscountPolicy());
    }
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

## AppConfig 리팩터링
+ 리팩터링 전
```java
public class AppConfig { 
    public MemberService memberService() {
        return new MemberServiceImpl(new MemoryMemberRepository());
    }
    public OrderService orderService() {
        return new OrderServiceImpl(new MemoryMemberRepository(), new FixDiscountPolicy());
    }
}
```
+ 리팩터링 후
```java
public class AppConfig {
    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }

    private MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public OrderService orderService(){
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    public DiscountPolicy discountPolicy(){
        return new FixDiscountPolicy();
    }
}
```   
+ 메소드 명과 리턴 타입을 보는 순간 역할이 보이고, 역할에 따른 구현이 한 눈에 보이도록 리팩터링함

## IoC 
+ 제어의 역전 IOC(Inversion of Control): 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것
  + AppConfig가 제어 흐름에 대한 권한을 가져감
+ 프레임 워크 vs 라이브러리
  + 프레임워크: 내가 작성한 코드를 제어하고, 대신 실행 (ex. JUnit)
  + 라이브러리: 내가 작성한 코드가 직접 제어의 흐름을 담당
  
## DI
+ 의존관계 주입 DI(Dependency Injection): 외부에서 객체들의 의존 관계를 주입하는 것
+ 의존관계: 정적인 클래스 의존 관계와, 실행 시점에 결정되는 동적인 객체 의존 관계 둘을 분리해서 생각
> 정적인 클래스 의존 관계
+ 실행하지 않아도 분석 가능 (클래스 다이어그램, import 코드만 보고 판단 가능)
+ 하지만 실제로 어떤 객체가 주입 될지는 알 수 없음
> 동적인 객체 인스턴스 의존 관계
+ 애플리케이션 실행 시점에 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계
  + 의존관계 주입: 실행 시점에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언트와 서버의 실제 의존관계가 연결 되는 것
    + 클라이언트 코드를 변경하지 않고, 클라이언트 호출 대상의 인스턴스 변경 가능
    + 정적인 클래스 의존관계를 변경하지 않고, 동적인 객체 인스턴스 의존관계 변경 가능   
    

## IoC 컨테이너, DI 컨테이너
+ DI(Ioc)컨테이너: AppConfig처럼 객체를 생성하고 관리 + 의존관계 연결해주는 것
+ 스프링 컨테이너(`ApplicationContext`)
  + `@Configuration` 이 붙은 AppConfig를 설정 정보로 사용
  + `@Bean`이라 적힌 메소드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록 => 스프링 빈
  + 스프링 빈 찾기: 스프링 컨테이너를 통해서 찾음 (`applicationContext.getBean()`메소드 이용) 
  
> AppConfig.java 중 일부
```java
@Configuration  // 어플리케이션의 설정 정보
public class AppConfig {

    @Bean  // 스프링 컨테이너에 등록됨
    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }
```
> MemberApp.java 중 일부
```java
public class MemberApp {

    public static void main(String[] args) {
        //AppConfig appConfig = new AppConfig();
        //MemberService memberService = appConfig.memberService();

        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        // AppConfig 안에 있는 것들을 스프링 컨테이너에 집어넣어서 관리함
        MemberService memberService = applicationContext.getBean("memberService", MemberService.class);
        // "memberService" 객체를 찾음
    }
}
```   

# 스프링 컨테이너와 스프링 빈
## 스프링 컨테이너 생성
> 스프링 컨테이너 생성
```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
```
+ `ApplicationContext`: 스프링 컨테이너, 인터페이스

> 스프링 빈 등록 (AppConfig.java)
```java
public class AppConfig {

   @Bean
   public MemberService memberService(){   // 빈 이름: memberService
       return new MemberServiceImpl(memberRepository());   // 빈 객체: MemberServiceImpl@x01
   }
```
+ 스프링 컨테이너는 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈 등록
+ 빈 이름은 메서드 이름을 사용하지만 직접 부여할 수 있음 `@Bean(name="member")`
+ 빈 이름은 항상 다른 이름을 부여해야 함

> 스프링 빈 의존관계 설정
+ 설정 정보 참고해서 의존 관계를 주입
+ 빈을 생성하고 의존관계를 주입하는 단계가 나누어져 있음 

## 스프링 빈 조회
#### 컨테이너에 등록된 모든 빈 조회
```java
public class ApplicationContextInfoTest {

  AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
  
  @Test
  @DisplayName("모든 빈 출력하기")
  void findAllBean(){
      String[] beanDefinitionNames = ac.getBeanDefinitionNames();

      for (String beanDefinitionName : beanDefinitionNames) {
          Object bean = ac.getBean(beanDefinitionName);
          System.out.println("name = " + beanDefinitionName + "object=" + bean);

      }
  }

  @Test
  @DisplayName("애플리케이션 빈 출력하기")
  void findApplicationBean(){
      String[] beanDefinitionNames = ac.getBeanDefinitionNames();   
      
      for (String beanDefinitionName : beanDefinitionNames) {
          BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);
          
          if(beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION){
              Object bean = ac.getBean(beanDefinitionName);
              System.out.println("name = " + beanDefinitionName + "object=" + bean);

          }
      }
  }
} 
```
+ `getBeanDefinitionNames()`: 스프링에 등록된 모든 빈 이름 조회, `getBean()` : 빈 이름으로 빈 객체 조회
+ `ROLE_APPLICATION` : 일반적으로 사용자가 정의한 빈, `ROLE_INFRASTRUCTURE` : 스프링이 내부에서 사용하는 빈   

#### 기본 조회
```java
class ApplicationContextBasicFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("빈 이름으로 조회")
    void findBeanByName(){
        MemberService memberService = ac.getBean("memberService", MemberService.class);
        Assertions.assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("이름없이 타입으로 조회")
    void findBeanByType(){
        MemberService memberService = ac.getBean(MemberService.class);
        Assertions.assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("구체 타입으로 조회")
    void findBeanByName2(){
        
        // 좋은 코드는 아님(역할이 아닌 구현에 의존함)
        MemberService memberService = ac.getBean("memberService", MemberServiceImpl.class);
        Assertions.assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("빈 이름으로 조회 x")
    void findBeanByNameX(){
    
        // 예외가 발생해야 함
        assertThrows(NoSuchBeanDefinitionException.class,
                ()-> ac.getBean("xx",MemberService.class));
    }
}
```
+ 기본적인 조회 방법: `getBean(빈이름, 타입)`, `getBean(타입)`   

#### 동일한 타입이 둘 이상

```java
class ApplicationContextSameBeanFindTesst {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);

    @Test
    @DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면 중복 오류 발생")
    void findBeanByTypeDuplicate(){
        Assertions.assertThrows(NoUniqueBeanDefinitionException.class,
                () -> ac.getBean(MemberRepository.class));
    }

    @Test
    @DisplayName("같은 타입이 둘 이상 있으면, 빈 이름을 지정하면 됨")
    void findBeanByName(){
        MemberRepository memberRepository = ac.getBean("memberRepository1",MemberRepository.class);
        assertThat(memberRepository).isInstanceOf(MemberRepository.class);

    }

    @Test
    @DisplayName("특정 타입 모두 조회")
    void findAllBeanByType(){
        Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);
        for (String s : beansOfType.keySet()) {
            System.out.println("key = " + s + " value = " + beansOfType.get(s));
        }

        System.out.println("beansOfType = "+ beansOfType);
        assertThat(beansOfType.size()).isEqualTo(2); 
    }

    @Configuration
    static class SameBeanConfig{

        @Bean
        public MemberRepository memberRepository1(){
            return new MemoryMemberRepository();
        }

        @Bean
        public MemberRepository memberRepository2(){
            return new MemoryMemberRepository();
        }
    }
}
```   

#### 상속 관계
+ 부모 타입으로 조회하면, 자식 타입도 함께 조회됨
  + Object 타입(자바 객체의 최고 부모)으로 조회하면, 모든 스프링 빈이 조회됨
```java
public class ApplicationContextExtendsFindTest {

 AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

 @Test
 @DisplayName("부모 타입으로 조회 시, 자식이 둘이상 있으면 중복 오류 발생")
 void findBeanByTypeDuplicate(){
     DiscountPolicy bean = ac.getBean(DiscountPolicy.class);
     assertThrows(NoUniqueBeanDefinitionException.class,
             () -> ac.getBean(DiscountPolicy.class));
 }

 @Test
 @DisplayName("부모 타입으로 조회 시, 자식이 둘이상 있으면, 빈 이름 지정하면 됨")
 void findBeanByTypeBeanName(){
     DiscountPolicy rateDiscountPolicy = ac.getBean("rateDiscountPolicy", DiscountPolicy.class);
     assertThat(rateDiscountPolicy).isInstanceOf(RateDiscountPolicy.class);
 }

 @Test
 @DisplayName("특정 하위 타입으로 조회")  // 좋은 방법은 아님
 void findBeanBySubType(){
     DiscountPolicy bean = ac.getBean(RateDiscountPolicy.class);
     assertThat(bean).isInstanceOf(RateDiscountPolicy.class);
 }

 @Test
 @DisplayName("부모 타입으로 모두 조회")
 void findAllBeanByParentType(){
     Map<String, DiscountPolicy> beansOfType = ac.getBeansOfType(DiscountPolicy.class);
     assertThat(beansOfType.size()).isEqualTo(2);
 }

 @Test
 @DisplayName("부모 타입으로 모두 조회 - Object")
 void findAllBeanByObjectType(){
     Map<String, Object> beansOfType = ac.getBeansOfType(Object.class);
 }

 @Configuration
 static class TestConfig{
     @Bean
     public DiscountPolicy rateDiscountPolicy(){
         return new RateDiscountPolicy();
     }

     @Bean
     public DiscountPolicy fixDiscountPolicy(){
         return new FixDiscountPolicy();
     }
 }
}  
```

## 스프링 빈 설정 메타 정보 - BeanDefinition
+ BeanDefinition: 빈 설정 메타정보   
  + 이를 중심으로 추상화를 함
  + `@Bean`당 각각 하나씩 메타 정보가 생성
  + 스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성
+ xml, java 등 다양한 빈 설정 형식을 가능하게 지원

# 싱글톤 컨테이너
## 싱글톤 
+ DI 컨테이너: AppConfig에 요청이 올 때마다 객체를 생성함 → 메모리 낭비 심함 → 싱글톤 패턴의 필요성
> 순수한 DI 컨테이너 테스트 (SingletonTest.java)
```java
public class SingletonTest {
    @Test
    @DisplayName("스프링 없는 순수한 DI 컨테이너")
    void pureContainer() {
        AppConfig appConfig = new AppConfig();
        //1. 조회: 호출할 때 마다 객체를 생성
        MemberService memberService1 = appConfig.memberService();
        //2. 조회: 호출할 때 마다 객체를 생성
        MemberService memberService2 = appConfig.memberService();
        //참조값이 다른 것을 확인 -> 계속해서 객체 생성
        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);
        //memberService1 != memberService2
        assertThat(memberService1).isNotSameAs(memberService2);

    }
}
```
## 싱글톤 패턴
+ 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴
+ 이미 만들어진 객체를 공유해서 효율적으로 사용 가능
+ 문제점 → 스프링 프레임워크가 전부 해결 가능
  + 클라이언트가 구체 클래스에 의존 → DIP & OCP 위반 
  + 테스트가 어려움
  + 유연성이 떨어짐

> SingletonService.java
```java
public class SingletonService {

    //1. static 영역에 객체를 딱 1개만 생성해둔다. (SingletonService가 자기 자신을 생성해서 instance에 참조를 넣어놓는다.)
    private static final SingletonService instance = new SingletonService();


    //2. public으로 열어서 객체 인스터스가 필요하면 이 static 메서드를 통해서만 조회하도록 허용한다.
    public static SingletonService getInstance(){  
        return instance;
    }

    //3. 생성자를 private으로 선언해서 외부에서 new 키워드를 사용한 객체 생성을 못하게 막는다.
    private SingletonService(){
    }

    public void login(){
        System.out.println("싱글톤 객체 로직 호출");
    }
}
```
+ `객체 1개 생성`: static 영역에 하나의 객체만 생성해둠
+ `조회`: instance가 필요할 때 static 메서드(getInstance())만 이용해서 조회하도록 허용 → 같은 객체 반환 위함
+ `생성자를 private으로 선언`: 외부에서 임의로 new로 인한 객체 생성 가능성 차단

> 싱글톤 테스트 (SingletonTest.java)
```java
 @Test
    @DisplayName("싱글톤 패턴을 적용한 객체 사용")
    void singletonServiceTest(){
        //1. 조회: 호출할 때 마다 같은 객체를 반환
        SingletonService singletonService1 = SingletonService.getInstance();
        //2. 조회: 호출할 때 마다 같은 객체를 반환
        SingletonService singletonService2 = SingletonService.getInstance();

        //참조값이 같은 것을 확인
        System.out.println("singletonService1 = " + singletonService1);
        System.out.println("singletonService2 = " + singletonService2);

        // singletonService1 == singletonService2
        assertThat(singletonService1).isSameAs(singletonService2);
    }
```
  
## 싱글톤 컨테이너
+ 싱글톤 패턴의 문제점을 해결, 객체 인스턴스를 싱글톤(1개만 생성)으로 관리
+ `스프링 컨테이너`: 싱글톤 컨테이너의 역할을 함
> SingletonTest.java
```java
@Test
    @DisplayName("스프링 컨테이너와 싱글톤")
    void springContainer() {
        //AppConfig appConfig = new AppConfig(); 
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);  // 스프링으로 바꾼 코드
```
## 싱글톤 방식의 주의점
+ 하나의 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 무상태(stateless)로 설계해야 함
  + 클라이언트에 의존적이거나, 값을 변경할 수 있는 필드가 있으면 안됨
  + 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 함
> 상태 유지(stateful)하는 경우
```java
public class StatefulService {

    private int price;  // 상태를 유지하는 필드 (값 변경 가능 10000 -> 20000)

    public void order(String name, int price){
        System.out.println("name = " + name + " price = " + price);
        this.price = price;     // 여기가 문제!
    }
    
    public int getPrice(){
        return price;
    }
}
```
```java
class StatefulServiceTest {
    @Test
    void statefulServiceSingleton(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

        // StatefulService는 싱글톤으로 같은 객체 공유
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        //ThreadA: A사용자 10000원 주문
        statefulService1.order("userA", 10000);
        //ThreadB: B사용자 20000원 주문
        statefulService2.order("userB", 20000);

        //ThreadA: 사용자A 주문 금액 조회
        int price = statefulService1.getPrice();
        //ThreadA: 사용자A는 10000원을 기대했지만, 기대와 다르게 20000원 출력
        System.out.println("price = " + price);
    }
}
```
+ `StatefulService`와 `price` 필드는 공유되는 필드로, 값 변경이 가능함 → 문제 상황

> 무상태(stateless)로 변경
```java
public class StatefulService {

    public int order(String name, int price){
        System.out.println("name = " + name + "price = " + price);
        return price;
    }
}
```
```java
class StatefulServiceTest {
    @Test
    void statefulServiceSingleton(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

        // StatefulService는 싱글톤으로 같은 객체 공유
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        //ThreadA: A사용자 10000원 주문
        int userAPrice = statefulService1.order("userA", 10000);
        //ThreadB: B사용자 20000원 주문
        int userBPrice = statefulService2.order("userB", 20000);

        //ThreadA: 10000원 출력
        System.out.println("price = " + userAprice);
    }
}
```
⭐ 공유 필드는 정말 조심해야 한다!   
⭐ 스프링 빈은 항상 무상태(stateless)로 설계하자.

## @Configuration과 바이트코드 조작 
> AppConfig에 호출 로그 남김
```java
@Configuration
public class AppConfig {

    @Bean 
    public MemberService memberService(){
        System.out.println("call AppConfig.memberService");
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        System.out.println("call AppConfig.orderService");
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService(){
        System.out.println("call AppConfig.memberRepository");
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy(){
        //return new FixDiscountPolicy();
        return new RateDiscountPolicy();
    }
}
```
+ 예상 결과 
```
call AppConfig.memberService
call AppConfig.memberRepository
call AppConfig.memberRepository
call AppConfig.orderService
call AppConfig.memberRepository
```
+ 실제 결과 
```
call AppConfig.memberService
call AppConfig.memberRepository
call AppConfig.orderService
```
+ 각각 2번 `new MemoryMemberRepository`를 호출해서 다른 인스턴스가 생성될 줄 알았는데?
  + @Bean memberService → new MemoryMemberRepository()
  + @Bean orderService → new MemoryMemberRepository()
+ 스프링이 클래스의 바이트코드를 조작하는 라이브러리를 사용했기 때문 => 싱글톤 보장
  + `CGLIB`(바이트코드 조작 라이브러리)로 `AppConfig@CGLIB`(AppConfig 클래스를 상속받은 임의의 클래스) 생성
  + `AppConfig@CGLIB`를 스프링 빈으로 등록

> AppConfig@CGLIB 예상 코드
```java
@Bean
public MemberRepository memberRepository() {
 
   if (memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면?) {  
       return 스프링 컨테이너에서 찾아서 반환;
   } 
   else { //스프링 컨테이너에 없으면
       기존 로직을 호출해서 MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록
       return 반환
   }
}
```
+ 스프링 빈이 존재하면 존재하는 빈을 반환, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 생성   
  => 싱글톤 보장
> `@Configuration`을 붙이지 않으면 싱글톤이 보장되지 않는다!   
  => 스프링 설정 정보는 항상 `@Configuration` 사용   
  
# 컴포넌트 스캔
## 컴포넌트 스캔과 의존관계 자동 주입

#### 1. 컴포넌트 스캔
: 설정 정보(AppConfig)가 없어도 자동으로 스프링 빈을 등록하는 기능
> AutoAppConfig.java
```java
@Configuration   // @Component가 붙은 클래스 모두 스프링 빈에 등록
@ComponentScan(
        excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)  // 스프링 빈으로 모두 등록해버리기 전에 뺄 것들 명시
)
public class AutoAppConfig {
}
```
+ `@ComponentScan`: `@Component` 붙은 클래스 모두 스프링 빈에 등록
  + 의존관계 자동 주입 필요 (설정정보를 안쓰기 때문에 원래 설정정보에 적었던 의존관계를 설정할 방법이 없음)

#### 2. 의존관계 자동 주입
: 스프링 컨테이너가 자동으로 타입에 맞는 스프링 빈을 찾아서 주입   

> MemberServiceImpl.java
```java
@Component  // 빈이 자동등록, 의존관계 설정할 방법이 없음 (수동 등록할 장소가 없음) → @AutoWired 사용
public class MemberServiceImpl implements MemberService{

    private final MemberRepository memberRepository;
    
    @Autowired // 자동 의존관계 주입 (MemberRepository의 타입에 맞는 스프링 빈을 주입)
               // ac.getBean(MemberRepository.class)
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```
+ 생성자에 `@Autowired`: 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입
  + 기본 조회 전략: 타입에 맞는 스프링 빈을 찾아서 주입
  + getBean(MemberRepository.class)와 동일하게 동작

## 탐색 위치와 기본 스캔 대상
#### 1. 탐색 위치
```java
package hello.core;

@Configuration
@ComponentScan( 
        basePackages = "hello.core.member",  // member패키지만 컴포넌트 스캔의 대상
        basePackageClasses = AutoAppConfig.class,  // AutoAppConfig 클래스의 패키지(hello.core)만 컴포넌트 대상
        excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes =
                Configuration.class)
)
public class AutoAppConfig {
}
```
+ `basePackage`: 탐색할 패키지의 시작 위치를 지정, 이 패키지를 포함해서 하위 패키지를 모두 탐색
+ `basePackageClasses`: 지정한 클래스의 패키지를 탐색 시작 위치로 지정
+ 지정하지 않으면, `@ComponentScan`이 붙은 설정 정보 클래스의 패키지(AutoAppConfig)가 시작 위치가 됨   
> ⭐ 권장 방법   
  : 패키지 위치를 지정하지 않고, 설정 정보 클래스(AppConfig)의 위치를 프로젝트 최상단(hello.core)에 두는 것   
#### 2. 컴포넌트 스캔 기본 대상
+ `@Component`: 컴포넌트 스캔에서 사용
+ `@Controller`: 스프링 MVC 컨트롤러로 인식
+ `@Repository`: 스프링 데이터 접근 계층으로 인식, 데이터 계층의 예외를 스프링 예외로 변환
+ `@Configuration`: 스프링 설정 정보로 인식, 스프링 빈이 싱글톤 유지하도록 추가 처리
+ `@Service`: 비즈니스 계층 인식에 도움

## 필터
+ `includeFilters` : 컴포넌트 스캔 대상을 추가로 지정
+ `excludeFilters` : 컴포넌트 스캔에서 제외할 대상을 지정
> BeanA.java
```java
package hello.core.scan.filter;

@MyIncludeComponent
public class BeanA {
}
```
> BeanB.java
```java
package hello.core.scan.filter;

@MyExcludeComponent
public class BeanB {
}
```
> ComponentFilterAppConfigTest.java
```java
public class ComponentFilterAppConfigTest {

    @Test
    void filterScan(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(ComponentFilterAppConfig.class);
        BeanA beanA = ac.getBean("beanA", BeanA.class);
        assertThat(beanA).isNotNull();    // 조회가 되어야 함. 값이 Null이면 안됨
        Assertions.assertThrows(
                NoSuchBeanDefinitionException.class, () -> ac.getBean("beanB", BeanB.class)
        );   // NoSuchBeanDefinitionException이 나와야 함
    }
    @Configuration
    @ComponentScan(
            includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
            excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
    )
    static class ComponentFilterAppConfig {
    }
}
```
+ includeFilters 에 `@MyIncludeComponent`을 추가해서 BeanA가 스프링 빈에 등록됨
+ excludeFilters 에 `@MyExcludeComponent`을 추가해서 BeanB는 스프링 빈에 등록되지 않음

## 중복 등록과 충돌
#### 1. 자동 빈 등록 vs 자동 빈 등록
: 컴포넌트 스캔에 의해 자동으로 스프링 빈이 등록될 때 그 이름이 같은 경우 오류 발생   
  => `ConflictingBeanDefinitionException` 예외 발생
#### 2. 수동 빈 등록 vs 자동 빈 등록
: 수동 빈 등록이 우선권을 가짐 (수동 빈이 자동 빈을 오버라이딩)
  + 최근 스프링 부트에서는 오류가 발생하도록 기본 값을 바꿈 (설정이 꼬이는 것을 방지)
  
# 의존관계 자동 주입
## 의존관계 주입 방법
#### 1. 생성자 주입   
+ 생성자 호출시점에 딱 1번만 호출되는 것이 보장됨
+ **불변**, **필수** 의존관계에 사용
  + 생성자를 통해서만 의존관계 주입, 외부에서 인스턴스 변경이 불가능  
> ⭐ 생성자가 1개일 때, `@Autowired` 생략 가능  

```java
@Component
public class OrderServiceImpl implements OrderService {

   private final MemberRepository memberRepository;  // final 값은 꼭 있어야 함
   private final DiscountPolicy discountPolicy;
   
   // @Autowired
   public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
       this.memberRepository = memberRepository;
       this.discountPolicy = discountPolicy;
   }
}
```
> 생성사 주입을 사용해야 한다!
  + 의존관계를 변경하지 못하게 하는 장점 (불변)
  + setter 주입에서는 의존관계 주입을 누락할 수 있다는 단점이 있음 (누락 방지)
  + `final` 키워드 사용 가능하다는 장점 (생성자에서 값이 설정되지 않는 오류 방지)
    + 생성자에서만 값을 세팅할 수 있음

#### 2. setter 주입
+ 필드의 값을 변경하는 수정자 메서드(setter)를 통해서 의존관계 주입
+ **선택**, **변경** 가능성이 있는 의존관계에 사용

```java
@Component
public class OrderServiceImpl implements OrderService {

    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;
    
    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
    
    @Autowired
    public void setDiscountPolicy(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}
```

#### 3. 필드 주입  
+ 필드에 바로 주입하는 방법
+ 외부에서 변경이 불가능해서 테스트 하기 힘듦
+ 테스트 코드나 @Configuration에서만 사용할 것, 그 외에는 사용 X    

```java
@Component
public class OrderServiceImpl implements OrderService {

    @Autowired private MemberRepository memberRepository;
    @Autowired private DiscountPolicy discountPolicy;
}
````  

#### 4. 일반 메서드 주입  
+ 한 번에 여러 필드를 주입 받을 수 있음
+ 잘 사용하지 않음
+ 
```java
@Component
public class OrderServiceImpl implements OrderService {
    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;

    @Autowired
    public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
       this.memberRepository = memberRepository;
       this.discountPolicy = discountPolicy;
    }
} 
```
#### 5. 옵션 처리
: 주입할 스프링 빈이 없을 때 자동 주입 대상을 옵션으로 처리하는 방법   

> AutoWiredTest.java
```java
static class TestBean{
        
        // 호출 안됨 (Member는 빈으로 등록되어 있지 않음)
        @Autowired(required = false)
        public void setNoBean1(Member noBean1){       
            System.out.println("noBean1 =" + noBean1);
        }    
        
        // null 호출 (@Nullable)
        public void setNoBean1(@Nullable Member noBean2){
            System.out.println("noBean2 =" + noBean2);
        }   
        
        //Optional.empty 호출
        @Autowired(required = false)
        public void setNoBean3(Optional<Member> member) {
            System.out.println("setNoBean3 = " + member);
        }
    }
}
```
+ `@Autowired(required = false)`: 자동 주입할 대상이 없으면 수정자 메서드 자체가 호출 안됨
+ `@Nullable` : 자동 주입할 대상이 없으면 null이 입력됨
+ `Optional<>` : 자동 주입할 대상이 없으면 'Optional.empty' 입력 

## 조회 빈이 2개 이상인 경우 문제점과 해결방안
+`@Autowired`: 타입으로 조회, ac.getBean(DiscountPolicy.class)와 유사하게 동작
> 타입으로 조회하면 선택된 빈이 2개 이상일 때 문제가 발생   
  Ex) `DiscountPolicy` 의 하위 타입인 `FixDiscountPolicy` , `RateDiscountPolicy` 둘 다 스프링 빈으로 선언하면 `NoUniqueBeanDefinitionException`오류 발생 (오류메세지: 하나의 빈을 기대했는데 아니었음)   
#### 해결방안
#### 1. @Autowired 필드 명 매칭
+ 타입 매칭의 결과가 2개 이상인 경우, 필드 명 또는 파라미터 명으로 빈 이름 매칭
```java
@Autowired
public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy rateDiscountPolicy) {
     this.memberRepository = memberRepository;
     this.discountPolicy = rateDiscountPolicy;
}  
```
#### 2. @Qualifier 사용
+ 추가 구분자를 붙여주는 방법으로, 빈 이름을 변경하는 것은 아님
  + 빈 등록시 `@Qualifier`를 붙여 준다
  + 주입시에 `@Qualifier`를 붙여주고 등록한 이름을 적어준다
  + `@Qualifier`끼리 매칭한다
    + 해당 `@Qualifier`가 없다면, 빈 이름을 매칭해본다
    + 같은 빈 이름도 없다면 `NoSuchBeanDefinitionException` 예외 발생
> 빈 등록
```java
@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {}
```
> 생성자 자동 주입
```java
@Autowired
public OrderServiceImpl(MemberRepository memberRepository, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
}
```
#### 3. @Primary 사용
+ 우선순위를 정하는 방법으로, 여러 빈이 매칭되면 `@Primary`가 우선권을 가짐
+ `@Primary`와 `@Qualifier` 활용
  + 메인 데이터베이스(:자주 사용)의 커넥션을 획득하는 스프링 빈은 `@Primary` 적용 => Simple
  + 서브 데이터베이스(:특별한 기능, 가끔 사용)의 커넥션 빈을 획득할 때는 `@Qualifier` 적용
```java
@Component
@Primary   // RateDiscountPolicy가 우선권을 가짐
public class RateDiscountPolicy implements DiscountPolicy {}
@Component
public class FixDiscountPolicy implements DiscountPolicy {}
```
#### 4. 직접 애노테이션 만들기
+ `@Qualifier("mainDiscountPolicy")` 이렇게 문자를 적으면 잘못 입력해도 컴파일시 타입 체크가 안되기에 사용하는 방법

> MainDiscountPolicy.java
```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Qualifier("mainDiscountPolicy")   
public @interface MainDiscountPolicy {
}
```
```java
@Component
@MainDiscountPolicy
public class RateDiscountPolicy implements DiscountPolicy {}
```
+ 사용하려는 빈 등록시 `@MainDiscountPolicy`를 붙여준다.
> 생성자 자동 주입
```java
@Autowired
public OrderServiceImpl(MemberRepository memberRepository, @MainDiscountPolicy DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
}
```
+ OrderServiceImpl 생성자 파라미터 앞에 `@MainDiscountPolicy`를 붙여준다.   

## 조회한 빈이 모두 필요한 경우 (List, Map)
> 클라이언트가 할인의 종류를 선택할 수 있어서 해당 타입의 스프링 빈이 모두 필요한 경우에 대한 Test 
```java
public class AllBeanTest {

    @Test
    void findAllBean(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);

        DiscountService discountService = ac.getBean(DiscountService.class);
        Member member = new Member(1L,"userA", Grade.VIP);
        int discountPrice = discountService.discount(member,10000,"fixDiscountPolicy");

        Assertions.assertThat(discountService).isInstanceOf(DiscountService.class);
        Assertions.assertThat(discountPrice).isEqualTo(1000);
    }

    static class DiscountService{
        private final Map<String, DiscountPolicy> policyMap;
        private final List<DiscountPolicy> policies;

        @Autowired
        public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies) {
            this.policyMap = policyMap;
            this.policies = policies;
            System.out.println("policyMap = " + policyMap);
            System.out.println("polices = " + policies);
        }

        public final int discount(Member member, int price, String discountCode) {
            DiscountPolicy discountPolicy = policyMap.get(discountCode);
            return discountPolicy.Discount(member, price);
        }
    }
}
```
+ `DiscountService`: `Map`으로 모든 `DiscountPolicy`를 주입받음(`fixDiscountPolicy`, `rateDiscountPolicy` 주입됨)
+ `discount() 메서드`: discountCode로 스프링 빈(`fixDiscountPolicy` 또는 `rateDiscountPolicy`)을 찾아서 실행
+ `Map<>` : map의 키에 스프링 빈의 이름을 넣어주고, 그 값으로 해당 타입으로 조회한 모든 스프링 빈을 담아줌
+ `List<DiscountPolicy>` : DiscountPolicy 타입으로 조회한 모든 스프링 빈을 담아줌

### ✔️ 자동, 수동의 올바른 실무 운영 기준
> 편리한 자동 기능을 기본으로 사용하자
  + 최근 스프링 부트: 컴포넌트 스캔을 기본으로 사용, 스프링 빈들도 조건이 맞으면 자동으로 등록됨
> 수동 빈 등록을 사용하는 경우
  + 업무 로직 빈: 컨트롤러, 서비스, 로직을 처리하는 리포지토리등으로, 비즈니스 요구사항을 개발할 때 추가되거나 변경 
    => 자동 기능 사용 (문제 파악이 쉬움)
  + 기술 지원 빈: 업무 로직을 지원하기 위한 하부기술, 공통 관심사(AOP)를 처리할 때 주로 사용   
    => 수동 빈 등록을 사용 (광범위하게 영향을 미치고 문제 파악이 어려워 설정 정보에 명확하게 드러내야 함)
> 비즈니스 로직 중에서 다형성을 적극 활용할 때   
 EX) 의존관계 자동 주입으로 `Map<String, DiscountPolicy>`에 주입을 받는 상황   
 => 코드만 보고 한번에 빈 파악 불가능   
  + 수동 빈으로 등록 
  + 자동으로하면 특정 패키지에 같이 묶어두어야 함
  
# 빈 생명주기 콜백 
## 빈 생명주기 콜백 시작
+ 객체의 초기화와 종료 작업이 필요 (애플리케이션 시작 시점에 필요한 연결을 미리 해두고, 종료 시점에 연결을 모두 종료하는 작업을 진행하기 위함)

> 외부 네트워크에 미리 연결하는 객체 생성 (NetworkClient)
```java
public class NetworkClient {
    private String url; public NetworkClient() {
        System.out.println("생성자 호출, url = " + url);
        connect();     // 초기화 과정
        call("초기화 연결 메시지");
    }
}
```
#### ⏰ 스프링 빈의 이벤트 라이프 사이클   
`스프링 컨테이너 생성 → 스프링 빈 생성 → 의존관계 주입 → 초기화 콜백 → 사용 → 소멸전 콜백 → 스프링 종료`
+ 초기화 콜백: 빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출
+ 소멸전 콜백: 빈이 소멸되기 직전에 호출
⭐ 객체의 생성과 초기화를 분리하자

## 빈 생명주기 콜백 지원 방법
#### 1. 인터페이스(InitializingBean, DisposableBean)
```java
public class NetworkClient implements InitializingBean, DisposableBean {
  //의존관계 주입이 끝나면
  @Override
  public void afterPropertiesSet() throws Exception {
      connect();
      call("초기화 메시지");
  }

  //빈 소멸 전
  @Override
  public void destroy() throws Exception {
      disconnect();
  }
}
```
#### 2. 빈 등록 초기화, 소멸 메서드 지정
+ 설정 정보에 `@Bean(initMethod = "init", destroyMethod = "close")`처럼 초기화, 소멸 메서드를 지정할 수 있음
```java
public class NetworkClient{
 //의존관계 주입이 끝나면
    public void init() {
        System.out.println("NetworkClient.init");
        connect();
        call("초기화 메시지");
    }

    //빈 소멸 전
    public void close() {
        System.out.println("NetworkClient.close");
        disconnect();
    } 
}
```
+ 설정 정보 사용 특징: 코드를 고칠 수 없는 외부 라이브러리에도 초기화, 종료 메서드 적용 가능
+ 종료 메서드 추론
  + `destroyMethod`: 기본값이 '추론'으로 `close`, `shutdown`메서드를 추론해서 자동으로 호출
#### 3. 애노테이션 @PostConstruct, @PreDestory
```java
//의존관계 주입이 끝나면
@PostConstruct
public class NetworkClient{

    @PostConstruct
    public void init() throws Exception {
        System.out.println("NetworkClient.init");
        connect();
        call("초기화 메시지");
    }

    @PreDestroy
    public void close() throws Exception {
        System.out.println("NetworkClient.close");
        disconnect();
    }  
}
```
+ 스프링이 아닌 다른 컨테이너에서도 동작
+ 외부 라이브러리에는 적용 불가능 (외부 라이브러리를 초기화, 종료 해야 하면 `@Bean`의 기능 사용)

# 빈 스코프
## 빈 스코프란?
: 빈이 존재할 수 있는 범위
#### 스코프의 종류
+ `싱글톤`: 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프
+ `프로토타입`: 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프
+ `웹 관련 스코프`
  + `request`: 웹 요청이 들어오고 나갈때 까지 유지되는 스코프
  + `session`: 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프
  + `application`: 웹의 서블릿 컨텍스와 같은 범위로 유지되는 스코프   
## 프로토타입 스코프
+ 싱글톤 스코프: 스프링 컨테이너에 조회시, 항상 같은 인스턴스의 스프링 빈을 반환
+ 프로토타입 스코프: 스프링 컨테이너에 조회시, 항상 새로운 인스턴스를 생성해서 반환
+ 스프링 컨테이너는 프로토타입 빈을 생성하고, 의존관계 주입, 초기화까지만 처리함
  + 프로토타입 빈을 관리할 책임 → 프로토타입 빈을 받은 클라이언트
  + `@PreDestory` 같은 종료 메서드가 호출되지 않음
> 프로토타입 스코프 빈 테스트
```java
public class PrototypeTest {

    @Test
    void prototypeBeanFind(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);
        System.out.println("find prototypeBean1");
        PrototypeBean bean1 = ac.getBean(PrototypeBean.class);
        System.out.println("find prototypeBean2");
        PrototypeBean bean2 = ac.getBean(PrototypeBean.class);

        System.out.println("bean1 = " + bean1);
        System.out.println("bean2 = " + bean2);
        assertThat(bean1).isNotSameAs(bean2);
        
        ac.close();
    }

    @Scope("prototype")
    static class PrototypeBean{

        @PostConstruct
        public void init(){
            System.out.println("PrototypeBean.init");
        }

        @PreDestroy
        public void destroy(){
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```
#### 싱글톤 빈과 함께 사용시 문제점
+ 스프링은 일반적으로 싱글톤 빈을 사용하므로, 싱글톤 빈이 프로토타입 빈을 사용하게 됨
+ 싱글톤 빈은 생성 시점에만 의존관계 주입을 받기 때문에, 프로토타입 빈이 새로 생성되기는 하지만, 싱글톤 빈과 함께 계속 유지된다는 문제 발생
> 해결 방법
#### 1. 스프링 컨테이너에 요청
```java
@Autowired
private ApplicationContext ac;
public int logic() {
    PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);
    prototypeBean.addCount();
    int count = prototypeBean.getCount();
    return count;
}
```
+ `ac.getBean()`을 통해서 항상 새로운 프로토타입 빈이 생성
+ DL(의존관계 조회): 직접 필요한 의존관계를 찾는 것
#### 2. ObjectFactory, ObjectProvider
```java
@Autowired
private ObjectProvider<PrototypeBean> prototypeBeanProvider;
public int logic() {
    PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
    prototypeBean.addCount();
    int count = prototypeBean.getCount();
    return count;
}
```
+ `prototypeBeanProvider.getObject()`을 통해서 항상 새로운 프로토타입 빈이 생성
+ DL(의존관계 조회): ObjectProvider의 `getObject()`를 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반
환
+ 스프링에 의존함
#### 3.JSR-330 Provider
```java
//implementation 'javax.inject:javax.inject:1' gradle 추가 필수
@Autowired
private Provider<PrototypeBean> provider;
public int logic() {
    PrototypeBean prototypeBean = provider.get();
    prototypeBean.addCount();
    int count = prototypeBean.getCount();
    return count;
}
```
+ `provider.get()`을 통해서 항상 새로운 프로토타입 빈이 생성
+ DL(의존관계 조회): provider의 `get()`을 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환
## 웹 스코프 
+ 웹 환경에서만 동작
+ 스프링이 해당 스코프의 종료시점까지 관리하므로 종료 메서드가 호출됨
#### 웹 스코프 종류 
+ `request`: HTTP 요청 하나가 들어오고 나갈 때까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고 관리
+ `session`: HTTP Session과 동일 생명주기를 가지는 스코프
+ `application`: 서블릿 컨텍스트와 동일한 생명주기를 가지는 스코프
+ `websocket`: 웹 소켓과 동일한 생명주기를 가지는 스코프
#### Request 스코프
+ 동시에 여러 HTTP 요청이 와서 정확히 어떤 요청이 남긴 로그인지 구분하기 어려울 때 사용하기 좋은 스코프 
> 로그를 출력하기 위한 클래스 (MyLogger)
```java
@Component
@Scope(value = "request")
public class MyLogger {
 private String uuid;
 private String requestURL;
 public void setRequestURL(String requestURL) {
 this.requestURL = requestURL;
 }
 public void log(String message) {
 System.out.println("[" + uuid + "]" + "[" + requestURL + "] " +
message);
 }
 @PostConstruct
 public void init() { uuid = UUID.randomUUID().toString();
 System.out.println("[" + uuid + "] request scope bean create:" + this);
 }
 @PreDestroy
 public void close() {
 System.out.println("[" + uuid + "] request scope bean close:" + this);
 }
}
```
+ requestURL 은 이 빈이 생성되는 시점에는 알 수 없으므로, 외부에서 setter로 입력
+ 빈이 생성되는 시점에 자동으로 `@PostConstruct` 초기화 메서드를 사용해서 uuid를 생성해서 저장

> 로거가 잘 작동하는지 확인하는 테스트용 컨트롤러 (LogDemoController)
```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {
 private final LogDemoService logDemoService; private final MyLogger myLogger;
 @RequestMapping("log-demo")
 @ResponseBody
 public String logDemo(HttpServletRequest request) {
 String requestURL = request.getRequestURL().toString();
 myLogger.setRequestURL(requestURL);
 myLogger.log("controller test");
 logDemoService.logic("testId");
 return "OK";
 }
}
```
+ HttpServletRequest를 통해서 요청 URL을 받고, 받은 requestURL 값을 myLogger에 저장
+ myLogger를 통해 다른 http 요청과 구분 가능
> 비즈니스 로직이 있는 서비스 계층에서의 로그 출력
```java
@Service
@RequiredArgsConstructorpublic class LogDemoService {
 private final MyLogger myLogger;
 public void logic(String id) {
 myLogger.log("service id = " + id);
 }
}
```
+ MyLogger 덕분에 이런 부분을 파라미터로 넘기지 않고, MyLogger의 멤버변수에 저장해서 코드와 계층을 깔끔하게 유지
⭐ 웹과 관련된 부분은 컨트롤러까지만 사용해야 한다!
## 스코프와 프록시
```java
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
@Component
public class MyLogger {
}
```
+ 적용 대상이 인터페이스가 아닌 클래스일 경우: TARGET_CLASS 
+ 적용 대상이 인터페이스인 경우: INTERFACES 
+ MyLogger의 가짜 프록시 클래스를 만들어두고 HTTP request와 상관 없이 가짜 프록시 클래스를 다른 빈에 미리 주입 가능   
⭐ 애노테이션 설정 변경만으로 원본 객체를 프록시 객체로 대체할 수 있다 (다형성과 DI 컨테이너가 가진 큰 강점)
