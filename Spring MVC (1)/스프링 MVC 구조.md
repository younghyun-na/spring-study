# 스프링 MVC 구조 이해  
## 📍 Spring MVC 전체 구조   

### 직접 만든 프레임워크 → 스프링 MVC 비교
+ FrontController → `DispatcherServlet`
+ handlerMappingMap → `HandlerMapping`
+ MyHandlerAdapter → `HandlerAdapter`
+ ModelView → `ModelAndView`
+ viewResolver → `ViewResolver`
+ MyView → `View`

### DispatcherServlet 구조   
+ 부모 클래스에서 HttpServlet을 상속 받아서 사용하고, 서블릿으로 동작
  + DispatcherServlet → FrameworkServlet → HttpServletBean → HttpServlet
+ 스프링 부트는 `DispacherServlet`을 서블릿으로 자동으로 등록하면서 모든 경로( urlPatterns="/" )에 대해서 매핑    
+ `FrameworkServlet.service()` 를 시작으로 여러 메서드가 호출되면서 `DispacherServlet.doDispatch()` 가 호출된다

> doDispatch()   
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
  HttpServletRequest processedRequest = request;
  HandlerExecutionChain mappedHandler = null;
  ModelAndView mv = null;
  
  // 1. 핸들러 조회
  mappedHandler = getHandler(processedRequest);
  if (mappedHandler == null) {
    noHandlerFound(processedRequest, response);
    return;
  }
  
  // 2. 핸들러 어댑터 조회 - 핸들러를 처리할 수 있는 어댑터
  HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
  
  // 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환
  mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
  processDispatchResult(processedRequest, response, mappedHandler, mv,
  dispatchException);
}

private void processDispatchResult(HttpServletRequest request, HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {

  // 뷰 렌더링 호출
  render(mv, request, response);
}

protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
  View view;
  String viewName = mv.getViewName();
  
  // 6. 뷰 리졸버를 통해서 뷰 찾기, 7. View 반환
  view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
  
  // 8. 뷰 렌더링
  view.render(mv.getModelInternal(), request, response);
```   
### SpringMVC 구조
<img src = "https://user-images.githubusercontent.com/69106295/131795309-01e9cff1-3edf-4764-a0ed-b552d1e775d6.png" width=50% height=50%>   

#### 동작 순서 
1. `핸들러 조회`: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
2. `핸들러 어댑터 조회`: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. `핸들러 어댑터 실행`: 핸들러 어댑터를 실행한다.
4. `핸들러 실행`: 핸들러 어댑터가 실제 핸들러를 실행한다.
5. `ModelAndView 반환`: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서
반환한다.
6. `viewResolver 호출`: 뷰 리졸버를 찾고 실행한다.
  + JSP의 경우: InternalResourceViewResolver 가 자동 등록되고, 사용된다.
7. `View 반환`: 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를
반환한다.
  + JSP의 경우 InternalResourceView(JstlView) 를 반환하는데, 내부에 forward() 로직이 있다.
8. `뷰 렌더링`: 뷰를 통해서 뷰를 렌더링 한다.   

#### 주요 인터페이스 목록  
+ 핸들러 매핑: `org.springframework.web.servlet.HandlerMapping`
+ 핸들러 어댑터: `org.springframework.web.servlet.HandlerAdapter`
+ 뷰 리졸버: `org.springframework.web.servlet.ViewResolver`
+ 뷰: `org.springframework.web.servlet.View`   

## 📍 핸들러 매핑과 핸들러 어댑터 

✔ 컨트롤러 호출에 필요한 2가지
+ HandlerMapping(핸들러 매핑)
  + 핸들러 매핑에서 이 컨트롤러를 찾을 수 있어야 한다.
  + 예) 스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸들러 매핑이 필요하다.
+ HandlerAdapter(핸들러 어댑터)
  + 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.
  + 예) `Controller` 인터페이스를 실행할 수 있는 핸들러 어댑터를 찾고 실행해야 한다.   

### 핸들러 매핑   
> 스프링 부트가 자동 등록하는 핸들러 매핑
+ 0 = `RequestMappingHandlerMapping` : `@RequestMapping`이라는 애노테이션을 컨트롤러 클래스나 메서드에 선언하여 사용
+ 1 = `BeanNameUrlHandlerMapping` : 스프링 빈의 이름으로 핸들러를 찾는다.   

### 핸들러 어댑터   
> 스프링 부트가 자동 등록하는 핸들러 어댑터
+ 0 = `RequestMappingHandlerAdapter` : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
+ 1 = `HttpRequestHandlerAdapter` : HttpRequestHandler 처리
+ 2 = `SimpleControllerHandlerAdapter` : Controller 인터페이스(애노테이션X, 과거에 사용) 처리   

> OldController   
```java
@Component("/springmvc/old-controller")
public class OldController implements Controller {

    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("OldController.handleRequest");
        return null;
    }
}
```
> 동작 방식
1. 핸들러 매핑으로 핸들러 조회
  + `HandlerMapping` 을 순서대로 실행해서, 핸들러를 찾는다.
  + 이 경우 빈 이름으로 핸들러를 찾아야 하므로 이름 그대로 빈 이름으로 핸들러를 찾아주는 `BeanNameUrlHandlerMapping` 가 실행에 성공하고 핸들러인 `OldController` 를 반환한다.   
  
2. 핸들러 어댑터 조회
  + `HandlerAdapter` 의 supports() 를 순서대로 호출한다.
  + `SimpleControllerHandlerAdapter` 가 Controller 인터페이스를 지원하므로 대상이 된다.   
  
3. 핸들러 어댑터 실행
  + 디스패처 서블릿이 조회한 `SimpleControllerHandlerAdapter` 를 실행하면서 핸들러 정보도 함께 넘겨준다.
  + `SimpleControllerHandlerAdapter` 는 핸들러인 `OldController` 를 내부에서 실행하고, 그 결과를 반환한다.

정리 - OldController 핸들러매핑, 어댑터
OldController 를 실행하면서 사용된 객체
  + HandlerMapping = BeanNameUrlHandlerMapping
  + HandlerAdapter = SimpleControllerHandlerAdapter   

## 📍 뷰 리졸버    
> 스프링 부트가 자동 등록하는 뷰 리졸버
+ 1 = `BeanNameViewResolver` : 빈 이름으로 뷰를 찾아서 반환 (예: 엑셀 파일 생성 기능에 사용)
+ 2 = `InternalResourceViewResolver` : JSP를 처리할 수 있는 뷰를 반환

> 동작 방식
1. 핸들러 어댑터 호출
  + 핸들러 어댑터를 통해 new-form 이라는 논리 뷰 이름을 획득한다.
2. ViewResolver 호출
  + `new-form` 이라는 뷰 이름으로 `viewResolver`를 순서대로 호출한다.
  + `BeanNameViewResolver` 는 `new-form` 이라는 이름의 스프링 빈으로 등록된 뷰를 찾아야 하는데 없다.
  + `InternalResourceViewResolver` 가 호출된다.
3. InternalResourceViewResolver
  + 이 뷰 리졸버는 `InternalResourceView` 를 반환한다.
4. 뷰 - InternalResourceView
  + `InternalResourceView` 는 JSP처럼 포워드 forward() 를 호출해서 처리할 수 있는 경우에 사용한다.
5. view.render()
  + `view.render()` 가 호출되고 `InternalResourceView` 는 forward() 를 사용해서 JSP를 실행한다.

## 📍 스프링 MVC 시작   
### @RequestMapping   
: URL을 컨트롤러의 메서드와 매핑할 때 사용하는 스프링 프레임워크의 어노테이션  
+ 클래스나 메서드 선언부에 `@RequestMapping`과 함께 URL을 명시하여 사용   
+ `@RequestMapping` = 요청 URL을 어떤 메서드가 처리할지 여부를 결정하는 것   

> SpringMemberSaveControllerV1 - 회원 저장
```java
@Controller
public class SpringMemberSaveControllerV1 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members/save")

    public ModelAndView process(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        System.out.println("member = " + member);
        memberRepository.save(member);

        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member", member);     // ModelAndView 를 통해 Model 데이터를 추가할 때, addObject() => 데이터는 이후 뷰를 렌더링 할 때 사용
        return mv;
    }
}
```
+ `@Controller`: `@Component` + `@RequestMapping`
  + 스프링이 자동으로 스프링 빈으로 등록한다. (내부에 @Component으로 인해 컴포넌트 스캔의 대상이 됨)   
  + 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식함   
+ `@RequestMapping`: 요청 정보를 매핑, 해당 URL이 호출되면 이 메서드가 호출   
+ `ModelAndView` : 모델과 뷰 정보를 담아서 반환   
+ `RequestMappingHandlerMapping` 은 스프링 빈 중에서 `@RequestMapping` 또는 `@Controller`가 클래스 레벨에 붙어 있는 경우에 매핑 정보로 인식함   
   
## 📍 스프링 MVC - 실용적인 방식   
#### 컨트롤러 통합  
+ 클래스 레벨 `@RequestMapping("/springmvc/v2/members")`: 공통적인 url 경로를 상위 클래스 레벨로 가져옴
  + 메소드 레벨 `@RequestMapping("/new-form")` => `/springmvc/v2/members/new-form`
  + 메소드 레벨 `@RequestMapping("/save")` =>  `/springmvc/v2/members/save`
  + 메소드 레벨 `@RequestMapping` =>  `/springmvc/v2/members` 

> SpringMemberControllerV3 (컨트롤러 통합된 버전)
```java
@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @GetMapping("/new-form")
    //@RequestMapping(value = "/new-form", method = RequestMethod.GET)
    public String newForm() {
        return "new-form";     // 뷰 이름 직접 반환 
    }

    @PostMapping("/save")
    //@RequestMapping(value = "/save", method = RequestMethod.POST)
    public String save(
            @RequestParam("username") String username,   // 파라미터를 직접 받음
            @RequestParam("age") int age,
            Model model) {      // 파라미터로 넘어온 model
        Member member = new Member(username, age);
        memberRepository.save(member);

        model.addAttribute("member", member);
        return "save-result";
    }

    @GetMapping
    //@RequestMapping(method = RequestMethod.GET)
    public String members(Model model) {
        List<Member> members = memberRepository.findAll();
        model.addAttribute("members", members);
        return "members";
    }
}
```   
+ **Model 파라미터**:  모델을 생성하지 않고 파라미터로 받음
+ **ViewName 직접 반환**: 뷰의 논리 이름을 반환
+ **`@RequestParam` 사용**: 스프링은 HTTP 요청 파라미터를 @RequestParam 으로 받음
  + `@RequestParam("username")` = `request.getParameter("username")`
+ **`@RequestMapping` => `@GetMapping`, `@PostMapping`**: URL만 매칭하는 것이 아니라, HTTP Method도 구분 가능   
   + `@RequestMapping(value = "/new-form", method = RequestMethod.GET)` => `@GetMapping("/new-form")`
   + `@RequestMapping(value = "/save", method = RequestMethod.POST)` => `@PostMapping("/save")`
