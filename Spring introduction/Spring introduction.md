# MVC
: Model, View, Controller  
  + Model: 생성된 데이터
  + View: Controller로 부터 받은 Model 데이터를 바탕으로 만들어진 화면, UI
  + Controller: Model과 View의 연결하는 제어 로직

### 정적 컨텐츠
  + 파일을 그대로 전달하는 방식
  + 관련 Controller 없음 
### MVC와 템플릿 엔진     
  + 템플릿 엔진을 model view controller 방식으로 쪼개서 view를 렌더링하여 렌더링이 된 html을 클라이언트에게 전달
  + view를 찾아서 템플릿 엔진을 통해 렌더링하여 html을 웹브라우저에 넘겨주는 방식  
#### Controller
  ```java
  @Controller
public class HelloController {

    @GetMapping("hello-mvc")
    public String helloMvc(@RequestParam("name") String name, Model model){
        model.addAttribute("name", name );
        return "hello-template";
    }
}
  ```
  + `@RequestParam`: 쿼리스트링으로 파라미터를 URL로 전송하고 controller에서 파라미터를 받을 때 사용

#### View
  ```html
  <!DOCTYPE html>
<html xmlns:th = "http://www.thymeleaf.org">
<body>
<p th:text="'hello ' + ${name}" >hello! empty</p>
</body>
</html>
   ```
  + helloController
    + return: hello-template, model(name:spring)
  + viewResolver
    + view를 찾아주고 template을 연결시켜줌
    + return의 string name(hello-template)과 똑같은 것을 찾아서 thymeleaf 템플릿 엔진에게 처리하라고 넘김   
    
### API
  JSON 형태({key:value})로 데이터를 그대로 전달하는 방식 (view가 없음)
  #### `@ResponseBody` 문자 반환
  ```java
  @Controller
public class HelloController { 
    @GetMapping("hello-string")
    @ResponseBody
    public String helloString(@RequestParam("name") String name){
        return "hello" + name;
    }
}
  ```
  + `@ResponseBody`: ViewResolver를 사용하지 않고 http body에 문자 내용을 직접 전달   
  #### `@ResponseBody` 객체 반환
  ```java
   @Controller
public class HelloController { 
    @GetMapping("hello-api")
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name){
        Hello hello = new Hello();
        hello.setName(name);
        return hello;
    }

    static class Hello {
        private String name;
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
    }
}
```
  + `helloController`
    + 단순 문자: 그대로 데이터를 넘김    
    + 객체: JSON 방식으로 데이터를 생성 
  + `HttpMessageConverter` (viewResolver 대신 동작)
    + 단순 문자: StringConverter
    + 객체: JsonConverter
----------------------------------------------

## 웹 애플리케이션 계층 구조
+ Controller: 웹 MVC의 컨트롤러 역할
+ Service: 핵심 비즈니스 로직 구현
+ Repository: DB에 접근, 도메인 객체를 DB에 저장하고 관리
+ Domain: 비즈니스 도메인 객체

#### Domain
+ DB에 저장되고 관리됨
```java
public class Member {
    private Long id;
    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
#### Repository
+ 도메인 객체를 DB에 저장하고 관리
```java
public interface MemberRepository {
    Member save(Member member);
    
    // Optional<T>: 'T'타입의 객체를 포장해 주는 래퍼 클래스(Wrapper class)로 null값으로 인해 발생하는 예외 처리 가능
    Optional<Member> findById(Long id);    
    Optional<Member> findByName(String name);
    List<Member> findAll();
}
```
-----------------------------------------
# Test
### 1) 회원 레포지토리 테스트 케이스
```java
public class MemoryMemberRepositoryTest {

    MemoryMemberRepository repository = new MemoryMemberRepository();
    
    // 하나의 test 실행 후마다 저장소를 비우는 메소드
    @AfterEach
    public void afterEach(){       // 하나의 test가 실행되고 끝날 때마다 저장소를 비우는 함수 (실행 순서와 관계없게 만들기 위해)
        repository.clearStore();
    }

    @Test
    public void save() {
        Member member = new Member();
        member.setName("spring");

        repository.save(member);

        Member result = repository.findById(member.getId()).get();
        Assertions.assertThat(member).isEqualTo(result);

    }
}
```
+ `@AfterEach`: 각 test 실행 후마다 저장소를 비우는 메소드   
한 번에 여러 테스트를 실행하면 직전 테스트 결과가 남을 수 있어서 실행 순서에 따라 오류가 발생할 수 있어, 각 테스트가 종료할 때마다 메모리 DB에 저장된 데이터를 삭제하도록 함
+ 각 Test는 의존적이지 않은, 독립적인 실행이 이루어져야 한다.

### 2) 회원 서비스 테스트 케이스
```java
class MemberServiceTest {

    MemberService memberService;
    MemoryMemberRepository memberRepository;

    // 같은 MemoryMemberRepository 사용을 위해 각 test 실행 전에 수행하는 메소드
    @BeforeEach
    public void beforeEach(){
        memberRepository = new MemoryMemberRepository();
        memberService = new MemberService(memberRepository);
    }
}
```
+ `@BeforeEach`: 각 test 실행 전에 수행하는 메소드   
  직접 new를 하지 않고 외부에서 memberRepository를 Service에 넣어줌   
  ~~MemberService memberService = new MemberService(memberRepository)~~
  + 직접 new를 했다면, MemberService에서 사용하는 MemoryMemberRepository()랑 MemberServiceTest에서 사용하는 MemoryMemberRepository()가 서로 다른 인스턴스가 된다.
  
```java
@AfterEach
    public void afterEach(){
        memberRepository.clearStore();
    }

    @Test
    void 회원가입() {
        //given (상황이 주어짐)
        Member member = new Member();
        member.setName("hello");

        //when (이를 실행했을 때)
        Long saveId = memberService.join(member);

        //then (결과)
        Member findMember = memberService.findOne(saveId).get();
        assertThat(member.getName()).isEqualTo(findMember.getName());
    }

    @Test
    public void 중복_회원_예외(){
        //given
        Member member1 = new Member();
        member1.setName("spring");

        Member member2 = new Member();
        member2.setName("spring");

        //when
        memberService.join(member1);
        IllegalStateException e = assertThrows(IllegalStateException.class, () -> memberService.join(member2));

        //then
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
        /*
        try{
            memberService.join(member2);
            fail();
        } catch(IllegalStateException e){      // 예외 터짐
            assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
        }
        */
    }
```
---------------------------------------------
# 스프링 빈과 의존관계
## 컴포넌트 스캔과 자동 의존관계 설정
```java
@Controller
public class MemberController {
    private final MemberService memberService;

    // spring container에서 memberservice를 가져온다.
    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```
+ DI(의존성 종속): controller를 통해 외부요청을 받고 service에서 비즈니스 로직을 만들고 repository에서 데이터를 저장하는 것 => 정형화되어있는 패턴
+ 스프링 빈 등록 이미지
  + controller -> memberService -> memberRepository
+ `@Autowired`: 스프링이 연관된 객체를 스프링 컨테이너에서 찾아서 넣어줌 (의존관계 주입) 

  > DI의 세가지 방법
    + 생성자 주입: 생성자에 의존성 주입을 받고자 하는 field를 나열. 제일 권장하는 방법
      ```java
      @Controller  // 생성자를 통해서 memberService가 MemberController에 주입
      public class MemberController {
          private final MemberService memberService;
    
          @Autowired
          public MemberController(MemberService memberService) {
              this.memberService = memberService;
          }
      }
      ```
        + MemberController가 생성될 때, 스프링 빈에 등록되어 있는 Memberservice 객체를 가져다 넣어줌
        + MemberService를 스프링이 생성할 때 스프링 컨테이너에 등록하면서 생성자 호출
        + 스프링컨테이너에는 MemoryMemberRepository를 service에 주입   
           
    + field 주입: member field에 @Autowired 선언하여 주입받는 방법
      ```java
      @Controller 
      public class MemberController {
          @Autowired private MemberService memberService;
      }
      ```
    + setter 주입: setter 메소드에 @Autowired 선언하여 주입받는 방법
      ```java
      @Controller 
      public class MemberController {
          private MemberService memberService;
    
          @Autowired
          public void setMemberService(MemberService memberService) {
              this.memberService = memberService;
          }
      }
      ```      
+ `@Component`: 개발자가 직접 작성한 Class 를 Bean 으로 만드는 것
  + `@Service`, `@Controller`, `@Repository`는 스프링 빈으로 자동 등록됨
  
## 자바 코드를 통한 스프링 빈 등록
```java
@Configuration // 스프링 빈에 등록하라는 뜻으로 인식
public class SpringConfig {
    
    @Bean // memberService를 스프링 빈에 등록함
    public MemberService memberService() {
        return new MemberService(memberRepository()); // 스프링 빈에 등록된 memberRepository를 넣어줌
    }
    @Bean
    public MemberRepository memberRepository() {  // memberRepository를 스프링 빈에 등록함
        return new MemoryMemberRepository();
    }
}
```
+ `@Bean`: 객체를 스프링 빈에 등록함
+ 변경이 필요할 때 Config 파일 하나만 변경하고, 나머지 코드는 건드리지 않아도 되는 장점이 있다
---------------------------------------------
# 웹 MVC 개발
> MemberController.java
```java
@GetMapping("/members/new")  // @GetMapping: 데이터를 조회할 때
public String createForm(){
    return "members/createMemberForm";
}

@PostMapping("/members/new")  // @PostMapping: 데이터를 폼에 넣어서 전달, 등록할 때
public String create(MemberForm form){
    Member member = new Member();
    member.setName(form.getName());

    memberService.join(member);

    return "redirect:/";
}
```
+ `@GetMapping()`: 데이터 조회할 때 사용
+ `@PostMapping()`: 데이터 전달, 등록할 때 사용
---------------------------------------------
# 스프링 DB 접근 기술
## 순수 JDBC
> sql/ddl.sql (테이블 관리 위함)
```sql
drop table if exists member CASCADE;
create table member
(
    id bigint generated by default as identity,
    name varchar(255),
    primary key (id)
)
```
> build.gradle 파일에 jdbc, h2 데이터베이스 관련 라이브러리 추가
```
implementation 'org.springframework.boot:spring-boot-starter-jdbc'
runtimeOnly 'com.h2database:h2'
```
> resources/application.properties (스프링 부트 데이터베이스 연결 설정 추가)
```
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
```
> JdbcMemberRepository.java 중 일부
```java
// 인터페이스 MenberRepository 구현
public class JdbcMemberRepository implements MemberRepository{

    // 데이터 소스 필요
    private final DataSource dataSource;

    // 스프링을 통해서 데이터 소스 주입받기
    public JdbcMemberRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public Member save(Member member) {
        String sql = "insert into member(name) values(?)";

        Connection conn = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;     // 결과를 받는 것
        try {
            conn = getConnection();
            pstmt = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
            pstmt.setString(1, member.getName());   // ?와 1이 매칭

            pstmt.executeUpdate();
            rs = pstmt.getGeneratedKeys();

            if (rs.next()) {
                member.setId(rs.getLong(1));
            } else {
                throw new SQLException("id 조회 실패");
            }
            return member;
        } catch (Exception e) {
            throw new IllegalStateException(e);
        } finally {
            close(conn, pstmt, rs);
        }
    }
```
> springConfig 수정 (스프링 설정 변경)
```java
@Configuration
public class springConfig {

    private DataSource dataSource;

    @Autowired
    public springConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository(){
        // return new MemoryMemberRepository();
        return new JdbcMemberRepository(dataSource);
    }
}
```
+ 개방-폐쇄 원칙(OCP) 준수
  + 확장에는 열려있고, 수정, 변경에는 닫혀있다.
+ 스프링의 DI을 사용하여 기존 코드를 전혀 손대지 않고, 설정만으로 구현 클래스를 변경
  + 기존에는 스프링 빈으로 등록되어있던 <memory> memberRepository를 빼고 <jdbc> memberRepository를 등록. 구현체만 이걸로 바뀌어서 돌아감.  

> 스프링 통합 테스트
```
@SpringBootTest
@Transactional   // test 실행 전에 먼저 실행하여 DB에 데이터를 다 넣은 뒤 test가 끝나면 롤백을 해주어 DB에 넣었던 데이터가 다 지워짐
public class MemberServiceIntegrationTest {
    @Autowired
    MemberService memberService;
    @Autowired
    MemberRepository memberRepository;

    @Test
    public void 회원가입() throws Exception {
        //given
        Member member = new Member();
        member.setName("spring");
        //when
        Long saveId = memberService.join(member);
        //then
        Member findMember = memberRepository.findById(saveId).get();
        assertEquals(member.getName(), findMember.getName());
    }
}
```
+ `@SpringBootTest` : 스프링 컨테이너와 테스트를 함께 실행한다.
+ `@Transactional` : 테스트 시작 전에 트랜잭션을 시작하고, 테스트 완료 후에 항상 롤백한다. 이렇게 하면 DB에 데이터가 남지 않으므로 다음 테스트에 영향을 주지 않는다.
  
## JDBC Template

> JdbcTemplateMemberRepository.java 중 일부
```java
public class JdbcTemplateMemberRepository implements MemberRepository {
    private final JdbcTemplate jdbcTemplate;
    
    @Autowired // 생성자 하나일 때는 생략 가능
    public JdbcTemplateMemberRepository(DataSource dataSource) {
        jdbcTemplate = new JdbcTemplate(dataSource);
    }
    
    @Override
    public Member save(Member member) {
        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");
        
        Map<String, Object> parameters = new HashMap<>();
        parameters.put("name", member.getName());
        Number key = jdbcInsert.executeAndReturnKey(new MapSqlParameterSource(parameters));
        member.setId(key.longValue());
        
        return member;
    }
    
    @Override
    public Optional<Member> findById(Long id) {
        List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);
        return result.stream().findAny();
    }
```
+ JDBC API의 반복코드 제거
+ SQL은 직접 작성해야 함
+ 생성자 하나일 때, @Autowired 생략 가능

## JPA
+ 반복 코드 제거, 기본적인 SQL도 JPA가 직접 만들어서 실행
+ SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임을 전환

> JpaMemberRepository.java 
```java
public class JpaMemberRepository implements MemberRepository {

    private final EntityManager em;      // jpa는 em(entity manager)로 동작
    
    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }
    
    public Member save(Member member) { 
        em.persist(member);
        return member;
    }
    
    // pk기반
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);      // (조회할 타입, 식별자)로 찾기
        return Optional.ofNullable(member);
    }
    
    // pk기반 아님
    public List<Member> findAll() {      // 객체 자체를 select
        return em.createQuery("select m from Member m", Member.class).getResultList();
    }
    
    // pk기반 아님
    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m where m.name = :name", 
        Member.class).setParameter("name", name)
                     .getResultList();
        return result.stream().findAny();
    }
}
```
## 스프링 데이터 JPA
+ 리포지토리에 구현 클래스 없이 인터페이스 만으로 개발 가능
+ 인터페이스를 통한 기본적인 CRUD 제공

```java
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {

    //JPQL select m from Member m where m.name = ? 
    @Override
    Optional<Member> findByName(String name);    // 인터페이스 이름만으로 개발이 끝남
}
```
+ SpringDataJpa가 구현체를 자동으로 만들어서 스프링 빈에 자동으로 등록함
---------------------------------------------
# AOP
: Aspect Oriented Programming (공통 관심 사항과 핵심 관심 사항 분리)
+ 공통 관심 사항: 시간을 측정하는 로직
+ 핵심 관심 사항: 비즈니스 로직과 같은 기능
```java
@Component
@Aspect
public class TimeTraceAop {

    @Around("execution(* hello.hellospring..*(..))")
    
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        System.out.println("START: " + joinPoint.toString());
        try {
            return joinPoint.proceed();   
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END: " + joinPoint.toString()+ " " + timeMs + "ms");
        }
    }
}
```
+ AOP 적용 후: 프록시(가짜 memberService)가 만들어져서 실제 memberService 대신 호출됨
