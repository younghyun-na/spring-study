# MVC
: Model, View, Controller  
  + Model: 생성된 데이터
  + View: Controller로 부터 받은 Model 데이터를 바탕으로 만들어진 화면, UI
  + Controller: Model과 View의 연결하는 제어 로직

### 정적 컨텐츠
  + 파일을 그대로 전달하는 방식
  + 관련 Controller 없음 
### MVC와 템플릿 엔진     
  + // 템플릿 엔진을 model view controller 방식으로 쪼개서 view를 렌더링해서 렌더링이 된 html을 클라이언트에게 전달
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
  + @RequestParam: 쿼리스트링으로 파라미터를 URL로 전송하고 controller에서 파라미터를 받을 때 사용

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
  #### @ResponseBody 문자 반환
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
  + @ResponseBody: ViewResolver를 사용하지 않고 http body에 문자 내용을 직접 전달   
  #### @ResponseBody 객체 반환
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
  + helloController 
    + 단순 문자: 그대로 데이터를 넘김    
    + 객체: JSON 방식으로 데이터를 생성 
  + HttpMessageConverter (viewResolver 대신 동작)
    + 단순 문자: StringConverter
    + 객체: JsonConverter
    
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
+ <mark>@AfterEach</mark>: 각 test 실행 후마다 저장소를 비우는 메소드   
한 번에 여러 테스트를 실행하면 직전 테스트 결과가 남을 수 있어서 실행 순서에 따라 오류가 발생할 수 있어, 각 테스트가 종료할 때마다 메모리 DB에 저장된 데이터를 삭제하도록 함
+ 각 Test는 의존적이지 않은, 독립적인 실행이 이루어져야 한다.

  
