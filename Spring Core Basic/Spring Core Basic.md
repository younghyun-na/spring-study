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
> 스프링 빈 등록
```java
public class AppConfig {

   @Bean
   public MemberService memberService(){
       return new MemberServiceImpl(memberRepository());
   }
```
+ 스프링 컨테이너는 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈 등록
+ 빈 이름은 메서드 이름을 사용하지만 직접 부여할 수 있음 `@Bean(name="member")`
> 스프링 빈 의존관계 설정
+ 설정 정보 참고해서 의존 관계를 주입

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
        
        // 좋지 않은 코드 (구현에 의존함)
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
 @DisplayName("부모 타입으로 조회 시, 자식이 둘이상 있으면, 빈 이름 지정하면 됌")
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
  +`@Bean`당 각각 하나씩 메타 정보가 생성
  + 스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성
+ xml, java 등 다양한 빈 설정 형식을 가능하게 지원
