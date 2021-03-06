# 웹 애플리케이션 이해
## 📍 웹 시스템 구성 - WEB, WAS, DB
<img src = "https://user-images.githubusercontent.com/69106295/127811329-cc3a66bb-037c-4e83-97ee-b05e15190613.png" width=50% height=50%>

+ `웹 서버(Web Server)`: HTTP 기반으로 동작, **정적 리소스(파일) 제공**
+ `웹 애플리케이션 서버(WAS)`: HTTP 기반으로 동작, 웹 서버 기능 포함 + **애플리케이션 로직까지 수행**
+ 웹 시스템 구성의 장점 
  + WAS는 중요한 애플리케이션 로직만 전담 가능, 효율적인 리소스 관리 
  + WAS, DB 장애시 WEB 서버가 오류 화면 제공 가능 (Web Server는 잘 죽지X, WAS는 잘 죽음)
## 📍 서블릿
: 클라이언트의 요청을 처리하고, 그 결과를 반환하는 Servlet 클래스의 구현 규칙을 지킨 자바 웹 프로그래밍 기술

```java
@WebServlet(name = " helloServlet", urlPatterns = "/hello")  
public class HelloServlet extends HttpServlet {
   
   @Override
   protected void service(HttpServletRequest request, HttpServletResponse response){
     //애플리케이션 로직
   }
}
```
+ urlPatterns(/hello)의 URL이 호출되면 서블릿 코드가 실행
+ HTTP 요청 정보를 편리하게 사용할 수 있는 `HttpServletRequest`
+ HTTP 응답 정보를 편리하게 제공할 수 있는 `HttpServletResponse`

<img src = "https://user-images.githubusercontent.com/69106295/127980000-951b73d3-e0fa-4c21-b8fc-cc9bbf5216d2.png" width=70% height=70%>

+ HTTP 요청시
  + WAS: Request, Response 객체를 새로 만들어서 서블릿 객체 호출
  + 개발자: Request 객체에서 HTTP 요청 정보 꺼내서 사용, Response 객체에 HTTP 응답 정보 입력
  + WAS: Response 객체에 담겨있는 내용으로 HTTP 응답 정보 생성

### 서블릿 컨테이너
: 서블릿을 지원하는 WAS
+ 서블릿 객체를 자동 생성, 초기화, 호출, 종료하는 생명주기 관리
+ 서블릿 객체는 싱글톤으로 관리 (공유 변수 사용 주의)
+ request, response 객체는 항상 새로 생성
+ 동시 요청을 위한 멀티 쓰레드 처리 지원

## 📍 동시 요청 - 멀티 쓰레드
### 쓰레드
: 애플리케이션 코드를 하나하나 순차적으로 실행하는 것
  + 동시 처리가 필요하면 쓰레드 추가로 생성해야 함
> 요청마다 쓰레드 생성 장단점
+ 장점) 동시 요청 처리 가능
+ 단점)  → 쓰레드 풀로 해결
  + 생성 비용 비쌈 => 응답 속도 늦어짐
  + 컨텍스트 스위칭 비용 발생
  + 쓰레드 생성 제한 X  => 메모리 임계점을 넘어 서버가 죽을 가능성
### 쓰레드 풀
: 필요한 쓰레드를 쓰레드 풀에 보관하고 관리, 생성 가능한 쓰레드의 최대치를 관리
+ 사용법) 
  + 생성되어 있는 쓰레드를 쓰레드 풀에서 꺼내서 사용 후, 사용을 종료하면 쓰레드 풀에 반납
  + 기다리는 요청은 거절하거나 특정 숫자만큼만 대기하도록 설정 가능
+ 장점) 쓰레드생성, 종료 비용 절약되고 응답 시간이 빠름
> WAS의 주요 튜닝 포인트 = `최대 쓰레드 수`
+ 너무 낮게 설정한 경우: 동시 요청이 많으면, 서버 리소스는 여유롭지만, 클라이언트는 금방 응답 지연
+ 너무 높게 설정한 경우: 동시 요청이 많으면, CPU, 메모리 리소스 임계점 초과로 서버 다운
+ 적정 숫자: 애플리케이션 상황에 따라 모두 다름 => 성능테스트 필요   
**⭐ WAS가 멀티 쓰레드 지원을 함!!! (개발자는 멀티 쓰레드 관련 코드 신경 안써도 됨)**

## 📍 HTML,HTTP API,CSR,SSR
+ `정적 리소스`: 고정된 HTML 파일, CSS, JS, 이미지, 영상 등을 제공
+ `HTML 페이지`: 동적으로 필요한 HTML 파일을 생성해서 전달
+ `HTTP API`: HTML이 아니라 데이터를 전달 (주로 JSON 형식 사용)
<img src = "https://user-images.githubusercontent.com/69106295/127992872-5cd8cd19-5696-4e87-bc13-04a4d320f75f.png" width=70% height=70%>

### SSR (서버 사이드 렌더링)
: 서버에서 최종 HTML을 생성해서 클라이언트에 전달
+ 주로 정적인 화면에 사용
+ 관련기술: JSP, 타임리프
<img src = "https://user-images.githubusercontent.com/69106295/127994438-9a6ed78d-1182-43bc-b868-2a162b4007cb.png" width=70% height=70%>

### CSR (클라이언트 사이드 렌더링)
: HTML 결과를 자바스크립트를 사용해 웹 브라우저에서 동적으로 생성해서 적용
+ 주로 동적인 화면에 사용, 웹 환경을 마치 앱처럼 필요한 부분부분 변경할 수 있음
+ 관련기술: React, Vue.js
<img src = "https://user-images.githubusercontent.com/69106295/127994594-6d7fdcab-3659-405a-9098-782c72bfa26d.png" width=70% height=70%>

# 서블릿 
## 📍 프로젝트 생성
스프링 부트가 내장 톰켓 서버를 띄워줌 (내부의 서블릿 컨테이너 → helloServlet 생성)
```java
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("HelloServlet.service");

        System.out.println("request = " + request);
        System.out.println("response = " + response);

        String username = request.getParameter("username");
        System.out.println("username = " + username);

        response.setContentType("text/plain");       //헤더 정보
        response.setCharacterEncoding("utf-8");      //헤더 정보
        response.getWriter().write("hello " + username);
    }
}
```
+ `name`: servlet 이름
+ `urlPatterns`: url 매핑

## 📍 HttpServletRequest
: http 프로토콜의 request 정보를 서블릿에게 전달하기 위한 목적으로 사용
> Http 요청 메시지
```
POST /save HTTP/1.1      // start line
Host: localhost:8080     // 헤더
Content-Type: application/x-www-form-urlencoded   // 헤더
username=kim&age=20      // 바디
```
+ `START LINE` : HTTP 메소드, URL, 쿼리 스트링, 스키마/프로토콜 쿼리 스트링
+ `헤더`: 헤더 조회
+ `바디`: form 파라미터 형식 조회, message body 데이터 직접 조회

✔ HttpServletRequest의 부가 기능 
1. 객체 임시 저장소 기능
+ 해당 HTTP 요청이 시작부터 끝날 때 까지 유지되는 임시 저장소 기능
  + 저장: `request.setAttribute(name, value)`
  + 조회: `request.getAttribute(name)`
  
2. 세션 관리 기능
+ `request.getSession(create: true)`

## 📍 HTTP 요청 데이터 (클라이언트 to 서버 데이터 전달 방법)
### 1. GET 쿼리 파라미터
: `request.getParameter()` 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
```java
/**
 * 1. 파라미터 전송 기능
 * http://localhost:8080/request-param?username=hello&age=20
 * <p>
 * 2. 동일한 파라미터 전송 가능
 * http://localhost:8080/request-param?username=hello&username=kim&age=20
 */
 
@WebServlet(name = "requestParamServlet", urlPatterns = "/request-param")
public class RequestParamServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
       // 1. 전체 파라미터 조회 
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName ->  System.out.println(paramName + "=" + request.getParameter(paramName)));
       /* 1-출력: username=hello
                age=20       */
                
                
       // 2. 단일 파라미터 조회
        String username = request.getParameter("username");
        String age = request.getParameter("age");
        System.out.println("request.getParameter(username) = " + username);
        System.out.println("request.getParameter(age) = " + age);
       /* 2-출력: request.getParameter(username) = hello
               request.getParameter(age) = 20        */   
               
               
       // 3. 이름이 같은 복수 파라미터 조회
        String[] usernames = request.getParameterValues("username");
        for (String name : usernames) {
            System.out.println("username=" + name); 
        }
       /* 3-출력: username=hello
               username=kim    */
    }
}
```
### 2. POST HTML Form
: 메시지 바디에 쿼리 파리미터 형식으로 데이터를 전달 
+ content-type: 메시지 바디의 데이터 형식을 지정하는 것으로, 꼭 필요함
  + `application/x-www-form-urlencoded` 형식
  + GET에서 본 쿼리 파라미터 형식과 같기에, `request.getParameter()`로  조회 가능 
+ message body: username=hello&age=20

### 3-1. API 메시지 바디 - 단순 텍스트
+ HTTP message body에 데이터를 직접 담아서 요청
+ 데이터 형식: 주로 JSON
+ `POST`, `PUT`, `PATCH`에서 이용
> 단순 텍스트 전송   
  + content-type: text/plain   
  + message body: hello
```java
@WebServlet(name = "requestBodyStringServlet", urlPatterns = "/request-body-string")
public class RequestBodyStringServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        ServletInputStream inputStream = request.getInputStream();    // byte 코드
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);    // byte 코드를 문자열로 반환 
        
        System.out.println("messageBody = " + messageBody);
        
        response.getWriter().write("ok");
    }
}
```
> 출력 결과
```
messageBody = hello
```

### 3-2. API 메시지 바디-JSON
> JSON 형식으로 파싱할 수 있게 객체 생성
```java
@Getter @Setter
public class HelloData {
 private String username;
 private int age;
}
```
> JSON 형식 전송   
  + content-type: application/json 
  + message body: {"username": "hello", "age": 20}
```java
@WebServlet(name = "requestBodyJsonServlet", urlPatterns = "/request-body-json")
public class RequestBodyJsonServlet extends HttpServlet {

    private ObjectMapper objectMapper = new ObjectMapper();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);

        System.out.println("messageBody = " + messageBody);
        System.out.println("helloData.username = " + helloData.getUsername());
        System.out.println("helloData.age = " + helloData.getAge());

        response.getWriter().write("ok");
    }
}
```
> 출력 결과
```
messageBody = {"username": "hello", "age": 20}
data.username = hello
data.age=20
```
+ `ObjectMapper`:  JSON 변환 라이브러리   

## 📍 HTTPServletResponse   
### 기본 사용법   
> HTTP 응답 메시지 생성
  + HTTP 응답 코드 지정
  + header 생성
  + message body 생성
  
```java
@WebServlet(name = "responseHeaderServlet", urlPatterns = "/response-header")
public class ResponseHeaderServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
       
       //[status-line]
        response.setStatus(HttpServletResponse.SC_OK);
        
        //[response-headers]
        response.setHeader("Content-Type", "text/plain;charset=utf-8");
        response.setHeader("Cache-Control", "no-cache, no-store, must- revalidate" );  // 캐시 완전 무효화
        response.setHeader("Pragma", "no-cache");
        response.setHeader("my-header","hello");

        //[Header 편의 메서드]
        content(response);
        cookie(response);
        redirect(response);

        //[message body]
        PrintWriter writer = response.getWriter();
        writer.print("ok");
    }
}
```  

> 편의 기능 제공
  + Content 편의 메서드
  ```java
  private void content(HttpServletResponse response) {
        //Content-Type: text/plain;charset=utf-8
        //Content-Length: 2
        //response.setHeader("Content-Type", "text/plain;charset=utf-8");
        response.setContentType("text/plain");
        response.setCharacterEncoding("utf-8");
        //response.setContentLength(2); //(생략시 자동 생성)
  }
  ```
  
  + 쿠키 편의 메서드
  ```java
  private void cookie(HttpServletResponse response) {
      //Set-Cookie: myCookie=good; Max-Age=600;
      //response.setHeader("Set-Cookie", "myCookie=good; Max-Age=600");
      Cookie cookie = new Cookie("myCookie", "good");
      cookie.setMaxAge(600);   //600초
      response.addCookie(cookie);
  }
  ```
  
  + redirect 편의 메서드
  ```java
  private void redirect(HttpServletResponse response) throws IOException {
  //Status Code 302
  //Location: /basic/hello-form.html
  //response.setStatus(HttpServletResponse.SC_FOUND); //302
  //response.setHeader("Location", "/basic/hello-form.html");
      response.sendRedirect("/basic/hello-form.html");
  ```
### HTTP 응답 데이터 (단순 텍스트, HTML)
1. 단순 텍스트 응답: `writer.println("ok");`
2. HTML 응답: content-type을 `text/html` 로 지정   

   ``` java
   @WebServlet(name = "responseHtmlServlet", urlPatterns = "/response-html")
   public class ResponseHtmlServlet extends HttpServlet {
       @Override
       protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
           //Content-Type: text/html;charset=utf-8
           response.setContentType("text/html");
           response.setCharacterEncoding("utf-8");
           PrintWriter writer = response.getWriter();
           writer.println("<html>");
           writer.println("<body>");
           writer.println(" <div>안녕?</div>");
           writer.println("</body>");
           writer.println("</html>");
       }
   }
   ```
3. HTTP API - MessageBody에 직접 JSON으로 응답
  + content-type을 `application/json` 로 지정 
  + `ObjectMapper.writeValueAsString()` 사용하여 객체를 JSON 문자로 변경
  
   ```java
   @WebServlet(name = "responseJsonServlet", urlPatterns = "/response-json")
   public class ResponseJsonServlet extends HttpServlet {
   
       private ObjectMapper objectMapper = new ObjectMapper();
       
       @Override
       protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
       
            response.setHeader("content-type", "application/json");
            response.setCharacterEncoding("utf-8");

            HelloData data = new HelloData();
            data.setUsername("kim");
            data.setAge(20);

            //{"username":"kim","age":20}
            String result = objectMapper.writeValueAsString(data);
            response.getWriter().write(result);
        }
   }    
   ``` 
# 서블릿, JSP, MVC 패턴
## 📍 서블릿으로 회원 관리  
+ **서블릿**: 웹 기반의 요청에 대한 동적인 처리가 가능한 서버 측에서 돌아가는 자바 프로그램 (Java코드 안에 HTML코드)
> MemberSaveServlet.java (회원 저장)
```java
@WebServlet(name = "memberSaveServlet", urlPatterns = "/servlet/members/save")
public class MemberSaveServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    
        // 1. 파라미터를 조회해서 Member 객체를 생성
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));
        Member member = new Member(username, age);
        
        // 2. Member 객체를 MemberRepository를 통해서 저장
        memberRepository.save(member);
        
        // 3. Member 객체를 사용해서 결과 화면용 HTML을 동적으로 만들어서 응답
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");
        PrintWriter w = response.getWriter();
        
        w.write("<html>\n" +
                " <head>\n" +
                " <meta charset=\"UTF-8\">\n" +
                "</head>\n" +
                "<body>\n" +
                "성공\n" +
                "<ul>\n" +
                " <li>id="+member.getId()+"</li>\n" +
                " <li>username="+member.getUsername()+"</li>\n" +
                " <li>age="+member.getAge()+"</li>\n" +
                "</ul>\n" +
                "<a href=\"/index.html\">메인</a>\n" +
                "</body>\n" +
                "</html>");
    }
}
```
✔ 한계점: 뷰(View)화면을 위한 HTML을 만드는 작업이 자바 코드에 섞여서 지저분하고 복잡   
+ 자바 코드로 HTML을 만들어 내는 것 보다 HTML 문서에 동적으로 변경해야 하는 부분만 자바 코드를 넣을 수 있다면 더 편리할 것   
 => `템플릿 엔진`:  HTML 문서에서 필요한 곳만 코드를 적용해서 동적으로 변경 가능 (ex. JSP, Thymeleaf 등)
 
## 📍 JSP로 회원 관리 
+ HTML코드 안에 Java코드
> save.jsp (회원 저장)
```jsp
<%@ page import="hello.servlet.domain.member.MemberRepository" %>   // 자바의 import문
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    /* 회원 저장을 위한 비즈니스 로직 */
    // request, response 사용 가능
    MemberRepository memberRepository = MemberRepository.getInstance();
    
    System.out.println("save.jsp");
    
    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));
    
    Member member = new Member(username, age);
    System.out.println("member = " + member);
    memberRepository.save(member);
%>

/* 뷰를 렌더링하는 부분 */
<html>
<head>
    <meta charset="UTF-8">
</head>
<body>
성공
<ul>
    <li>id=<%=member.getId()%></li>
    <li>username=<%=member.getUsername()%></li>
    <li>age=<%=member.getAge()%></li>
</ul>
<a href="/index.html">메인</a>
</body>
</html>
```
+ `<% ~~ %>`: 자바 코드 입력하는 부분   
> ✔ 한계점: 다양한 코드가 모두 JSP에 노출, 너무 많은 역할을 함 (유지보수 힘듦)
+ 비즈니스 로직은 서블릿처럼 다른곳에서 처리하고, JSP는 목적에 맞게 HTML로 화면(View)을 그리는 일에 집중하도록 하면 편리할 것   
=> `MVC 패턴` 등장   

## 📍 MVC 패턴
### MVC 패턴의 개요
#### MVC 패턴이 필요한 이유 (서블릿과 JSP의 한계)
1. 서블릿과 JSP가 너무 많은 역할을 담당하면서 유지보수가 어려워짐
2. 비즈니스 로직과 UI가 "변경의 라이프 사이클"이 다름
3. JSP는 화면 렌더링에 최적화되어 있기 때문에 이 부분만 기능 특화시키는 것이 효과적
  
#### MVC 패턴  
: 서블릿이나 JSP로 처리하던 것을 컨트롤러(Controller)와 뷰(View)라는 영역으로 서로 역할을 나눈 것   
1) Controller   
+ HTTP 요청을 받아서 파라미터를 검증하고 비즈니스 로직을 실행   
+ 뷰에 전달할 결과 데이터를 조회해서 모델에 담음  
+ 비즈니스 로직이 있는 서비스를 호출하는 역할 (비즈니스 로직은 Service 계층을 만들어서 처리)
2) Model
+ 뷰에 출력할 데이터를 담아둠
+ 화면 렌더링에만 집중 가능 (뷰가 필요한 데이터를 모두 모델에 전달해주기 때문)   
3) View  
+ 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중 (HTML을 생성하는 부분)
<img src = "https://user-images.githubusercontent.com/69106295/128744487-ec7d18f3-6a3c-4437-a8e6-fd9030958ebf.png" width=70% height=70%>

### MVC 패턴의 적용
+ Controller: 서블릿
+ View: JSP
+ Model: HttpServletRequest 객체 (request 내부에 데이터 저장소 가지고 있음)
  + `request.setAttribute()`: 데이터 보관
  + `request.getAttribute()`: 데이터 조회

> 회원 등록 폼 - Controller
```java
@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);     // 서블릿에서 jsp 호출 가능
    }
}
```
+ `dispatcher.forward()`: 다른 서블릿이나 JSP로 이동할 수 있는 기능, 서버 내부에서 다시 호출이 발생   
+ `/WEB-INF`: 이 경로안에 JSP가 있으면 외부에서 직접 JSP를 호출할 수 없음 (컨트롤러를 통해서 JSP 호출해야 함)   

✔ 서블릿에서 특정 URL이나 페이지로 이동하게 하는 두 가지 방식 (Redirect vs Dispatcher)
  + Redirect방식 `sendRedirect()` (이동하기)
    + 실제 클라이언트에게 응답이 나갔다가 클라이언트가 redirect 경로로 다시 요청
    + 새로운 페이지로 완전히 이동해서 기존 데이터를 하나도 사용할 수 없음
  + Dispatcher방식 `forward()` (전달하기)
    + 클라이언트가 요청하면서 전송한 데이터를 그대로 유지 
    + 서버 내부에서 일어나는 호출이기 때문에 클라이언트가 전혀 인지하지 못함  
  
> 회원 등록 폼 - View
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<!-- 상대경로 사용, [현재 URL이 속한 계층 경로 + /save] -->
<form action="save" method="post">
    username: <input type="text" name="username" />
    age: <input type="text" name="age" />
    <button type="submit">전송</button>
</form>
</body>
</html>
```
+ 절대 경로가 아니라 상대경로인 것을 알 수 있음
  + 폼 전송시 현재 URL이 속한 계층 경로 + save가 호출   
   => 현재 경로 `/servlet-mvc/members/`에서 `/servlet-mvc/members/save`로 자동 변경)

> 회원 저장 - Controller
```java
@WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet/members/save")
public class MvcMemberSaveServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        //Model에 데이터를 보관한다.
        request.setAttribute("member", member);

        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```
+ Model: `HttpServletRequest` 
  + `setAttribute()`를 사용하여 request 객체에 데이터를 보관해서 뷰에 전달
+ View는 `request.getAttribute()`를 사용해서 데이터 꺼냄   

> 회원 저장 - View
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
 <meta charset="UTF-8">
</head>
<body>
성공
<ul>
 <li>id=${member.id}</li>
 <li>username=${member.username}</li>
 <li>age=${member.age}</li>
</ul>
<a href="/index.html">메인</a>
</body>
</html>
```
+ member 객체 조회: `<%= request.getAttribute("member")%>` → `${member}` (JSP에서 제공)   

### MVC 패턴의 한계
#### MVC 컨트롤러의 단점
+ 문제점: 공통 처리가 어렵다.   
  + forward 중복, ViewPath 중복 
+ 해결방법: `프론트 컨트롤러 패턴`(컨트롤러 호출 전에 먼저 공통 기능을 처리)   

# MVC 프레임워크 만들기
## 📍 FrontController란
<img src = "https://user-images.githubusercontent.com/69106295/129441286-f2e9ae85-04a5-4014-be12-b7c6e03e9372.png" width=50% height=50%>

+ 프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받음
+ 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출   
+ 프론트를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨

## 📍 프론트 컨트롤러 도입 - v1
+ 기존 코드를 최대한 유지하면서, 프론트 컨트롤러를 도입   

<img src = "https://user-images.githubusercontent.com/69106295/130256562-d3aa23a6-0077-4d88-b14d-f73a01978219.png" width=50% height=50%>   

> ControllerV1 (인터페이스)
```java
public interface ControllerV1 {
    void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```
+ 서블릿과 비슷한 모양의 컨트롤러의 인터페이스
+ 각 컨트롤러들은 이 인터페이스를 호출하여 로직의 일관성을 가져갈 수 있음

> MemberFormControllerV1 - 회원 등록 컨트롤러
```java
public class MemberFormControllerV1 implements ControllerV1 {
 @Override
 public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
 String viewPath = "/WEB-INF/views/new-form.jsp";
 RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
 dispatcher.forward(request, response);
 }
}
```
+ `ControllerV1` 인터페이스를 상속받음 

> FrontControllerServletV1 - 프론트 컨트롤러
```java
@WebServlet(name = "frontControllerServletV1", urlPatterns = "/front-controller/v1/*")
public class FrontControllerServletV1 extends HttpServlet {

    private Map<String, ControllerV1> controllerMap = new HashMap<>();

    public FrontControllerServletV1() {
    
        // 앞의 URI 호출 -> 뒤의 객체 인스턴스 반환
        controllerMap.put("/front-controller/v1/members/new-form", new MemberFormControllerV1());
        controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV1());
        controllerMap.put("/front-controller/v1/members/members", new MemberListControllerV1());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("FrontControllerServletV1.service");

        //front-controller/v1/members
        String requestURI = request.getRequestURI();
        
        /* 다형성: controller(부모) , 인스턴스 주소(자식) 부모가 자식에 담길 수 있음 */
        // requestURI를 꺼내면 controller가 찾아짐
        // ex) ControllerV1 controller = MemberListControllerV1()의 인스턴스 주소
        ControllerV1 controller = controllerMap.get(requestURI); 

        // 없는 경우
        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        // 잘 조회된 경우
        controller.process(request, response);
    }
}
```
+ `urlPatterns = "/front-controller/v1/*"` : /front-controller/v1 를 포함한 하위 모든 요청은
이 서블릿에서 받아들임
+ `controllerMap`
  + `key`: 매핑 URL
  + `value`: 호출될 컨트롤러
+ `service()`: requestURI 를 조회해서 실제 호출할 컨트롤러를 controllerMap 에서 찾고  
   + 없다면, `404(SC_NOT_FOUND)` 상태 코드를 반환
   + 있다면, `controller.process(request, response)` 을 호출해서 해당 컨트롤러를 실행   

## 📍 View 분리 - v2   
+ 뷰를 처리하는 객체인 `MyView` 생성 (why? 모든 컨트롤러에서 뷰로 이동하는 부분에 중복 존재, 이를 분리하기 위함)

<img src = "https://user-images.githubusercontent.com/69106295/130276060-01b81f17-496d-4d1c-a91f-f434727756d7.png" width=50% height=50%>   

> MyView: 뷰를 처리하는 객체
```java
public class MyView {
    private String viewPath;  // 4-2. viewPath = "/WEB-INF/views/new-form.jsp"

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    // 뷰를 만드는 행위 자체: render
    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);  // 4-3. new-form.jsp로 dispatcher forward가 됨
    }
}
```
+ `view.render()` 호출 결과: `forward`로직 수행으로 JSP 실행   

> ControllerV2 (인터페이스)
```java
public interface ControllerV2 {
    MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```
+ 컨트롤러의 호출 결과: `MyView`를 반환

> MemberFormControllerV2 - 회원 등록 폼
```java
public class MemberFormControllerV2 implements ControllerV2 {
    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        return new MyView("/WEB-INF/views/new-form.jsp");   // 3-2. MyView를 생성해서 넘겨줌 (모든 컨트롤러의 역할)
    }
}
```

> FrontControllerServletV2 - 프론트 컨트롤러
```java
// ex) 1. /front-controller/v2/members/new-form
@WebServlet(name = "frontControllerServletV2", urlPatterns = "/front-controller/v2/*")
public class FrontControllerServletV2 extends HttpServlet {

    private Map<String, ControllerV2> controllerMap = new HashMap<>();

    public FrontControllerServletV2() {
        controllerMap.put("/front-controller/v2/members/new-form", new MemberFormControllerV2());
        controllerMap.put("/front-controller/v1/members/save", new MemberSaveControllerV2());
        controllerMap.put("/front-controller/v2/members/members", new MemberListControllerV2());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String requestURI = request.getRequestURI();

        // 2. MemberFormControllerV2를 찾음
        ControllerV2 controller = controllerMap.get(requestURI);

        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        // 3-1. MemberFormControllerV2 호출 -> 3-3.반환 결과: new MyView("/WEB-INF/views/new-form.jsp")
        MyView view = controller.process(request, response);
        // 4-1. 이 view의 render를 호출 -> 4-4. JSP에서의 결과가 응답으로 웹브라우저에 나타남
        view.render(request, response);
    }
}
```
+ 프론트 컨트롤러의 도입 => `MyView` 객체의 `render()`를 호출하는 부분을 모두 일관되게 처리 가능해짐
+ 각각의 컨트롤러의 역할: `MyView` 객체를 생성만 해서 반환하는 것   

## 📍 Model 추가 - v3   

+ 뷰 이름 중복 제거
  + 컨트롤러가 뷰의 논리 이름을 반환하고, 실제 물리 위치의 이름은 프론트 컨트롤러에서 처리하도록 단순화   
    + `/WEB-INF/views/new-form.jsp` → new-form
+ `ModelView`: 서블릿의 종속성 제거 + View 이름 전달 하는 객체
  + 그동안 컨트롤러에서 서블릿에 종속적인 `HttpServletRequest` 사용
  + 그동안 Model은 `request.setAttribute()`을 통해 데이터를 뷰에 전달
 
<img src = "https://user-images.githubusercontent.com/69106295/130311317-4caabfe3-4e08-43a9-ab5d-0774d68dc4ac.png" width=50% height=50%>   

> ModelView: Model을 직접 만들고, 추가로 View 이름까지 전달하는 객체

```java
public class ModelView {

    private String viewName;     // 뷰의 이름
    private Map<String, Object> model =  new HashMap<>();    // 뷰를 렌더링할 때 필요한 model 객체

    public ModelView(String viewName) {
        this.viewName = viewName;
    }

    public String getViewName() {
        return viewName;
    }

    public void setViewName(String viewName) {
        this.viewName = viewName;
    }

    public Map<String, Object> getModel() {
        return model;
    }

    public void setModel(Map<String, Object> model) {
        this.model = model;
    }
}
```
+ `model`: map으로 되어있으므로 컨트롤러에서 뷰에 필요한 데이터를 (key, value)로 넣어주면 됨

> ControllerV3 (인터페이스)
```java
public interface ControllerV3 {

    // 프레임 워크에 종속적인 것이지, 서블릿에 종속적이지 않음
    ModelView process(Map<String, String> paramMap);
}
```
+ 서블릿 기술을 전혀 사용하지 않음 => 구현 단순, 테스트하기 쉬움
+ `HttpServletRequest`가 제공하는 파라미터는 프론트 컨트롤러가 paramMap에 담아서 호출
+ 응답 결과로 ModelView 객체를 반환 (뷰 이름과 뷰에 전달할 Model 데이터를 포함)   

> MemberFormControllerV3 - 회원 등록 폼
```java
public class MemberFormControllerV3 implements ControllerV3 {

    @Override
    public ModelView process(Map<String, String> paramMap) {
        return new ModelView("new-form");
    }
}
```
+ `ModelView` 생성 시, 뷰의 논리적 이름 `new-form`을 지정함 (물리적 이름은 프론트 컨트롤러가 처리)   

> MemberSaveControllerV3 - 회원 저장
```java
public class MemberSaveControllerV3 implements ControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public ModelView process(Map<String, String> paramMap) {
    
        /* (V2:) String username = request.getParameter("username"); */
        String username = paramMap.get("username");  // 파라미터 정보가 왔겠다고 생각하고 꺼내서 쓰면 됨
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        /* (V2:) request.setAttribute("member", member); */
        ModelView mv = new ModelView("save-result");    // 뷰의 논리적 이름 "save-result"
        mv.getModel().put("member", member);   // 모델에 뷰에서 필요한 member 객체를 담고 반환함
        
        /* (V2:) return new MyView("/WEB-INF/views/save-result.jsp"); */
        return mv;
    }
}
```
+ `paramMap`: 파라미터 정보는 map에 담겨있음. map에서 필요한 요청 파라미터를 조회하면 됨.   

> FrontControllerServletV3 - 프론트 컨트롤러   
```java
// ex) /front-controller/v2/members/new-form
@WebServlet(name = "frontControllerServletV3", urlPatterns = "/front-controller/v3/*")
public class FrontControllerServletV3 extends HttpServlet {

    private Map<String, ControllerV3> controllerMap = new HashMap<>();

    public FrontControllerServletV3() {
        controllerMap.put("/front-controller/v3/members/new-form", new MemberFormControllerV3());
        controllerMap.put("/front-controller/v3/members/save", new MemberSaveControllerV3());
        controllerMap.put("/front-controller/v3/members/members", new MemberListControllerV3());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String requestURI = request.getRequestURI();

        // 맵핑 정보 가져오기
        ControllerV3 controller = controllerMap.get(requestURI);

        if (controller == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        /* FrontControllerServletV2에서,
        MyView view = controller.process(request, response);
        view.render(request, response);
        */
        //paramMap(파라미터 정보) 넘겨주기
        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);

        String viewName = mv.getViewName();  // 논리이름 가져옴: new-form
        MyView view = viewResolver(viewName);  // new-form을 가지고 viewResolver를 호출 → MyView 반환
        view.render(mv.getModel(), request, response);  // 이걸 가지고 render 호출
    }

    //모든 파라미터 정보 가져와서 Map에 담는 함수
    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName, request.getParameter(paramName)));
        return paramMap;
    }

    // 컨트롤러가 반환한 논리 뷰 이름을 실제 물리 뷰 경로로 변경: /WEB-INF/views/new-form.jsp
    private MyView viewResolver(String viewName){
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}
```
+ `createParamMap()`: HttpServletRequest에서 파라미터 정보를 꺼내서 Map으로 변환 후 해당 Map(`paramMap`)을
컨트롤러에 전달하면서 호출

✔️ 뷰 리졸버
+ `MyView view = viewResolver(viewName)` 
  + 컨트롤러가 반환한 논리 뷰 이름을 실제 물리 뷰 경로로 변경
  + 실제 물리 경로가 있는 MyView 객체를 반환
+ `view.render(mv.getModel(), request, response)`
  + 뷰 객체를 통해서 HTML 화면을 렌더링
  + 뷰 객체의 `render()`는 모델 정보도 함께 받음
  + JSP는 `request.getAttribute()`로 데이터를 조회하므로, 모델의 데이터를 꺼내서 `request.setAttribute()`로 담아둠   
  
> MyView - 뷰 리졸버   
```java
public class MyView {

    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }

    public void render(Map<String, Object> model, HttpServletRequest request,  HttpServletResponse response) throws ServletException, IOException {
        modelToRequestAttribute(model, request);
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }

    private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
        model.forEach((key, value) -> request.setAttribute(key, value));
    }
}
```

## 📍 단순하고 실용적인 컨트롤러 - v4   
<img src = "https://user-images.githubusercontent.com/69106295/130589656-a76f73ad-b580-40ed-8273-5713af884407.png" width=50% height=50%>   

+ 컨트롤러가 `ModelView`를 반환하지 않고, `ViewName`만 반환   
+ frontcontroller가 Model을 만들어서 넘겨줌 
+ frontcontroller에서 controller 호출할 때 model 넘겨주는 것, view 반환하는 것의 차이   

> ControllerV4 (인터페이스)
```java
public interface ControllerV4 {
    /**
     *
     * @param paramMap
     * @param model
     * @return viewName
     */
    String process(Map<String, String> paramMap, Map<String, Object> model);
}
``` 
+ model 객체는 파라미터로 전달되기 때문에 그냥 사용하면 되고, 결과로 뷰의 이름만 반환해주면 된다.   

> FrontControllerServletV4 - 프론트 컨트롤러
```java
@WebServlet(name = "frontControllerServletV4", urlPatterns = "/front-controller/v4/*")
public class FrontControllerServletV4 extends HttpServlet {
    private Map<String, ControllerV4> controllerMap = new HashMap<>();
    ...
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ...        
        // 모델 객체를 프론트 컨트롤러에서 생성해서 전달
        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>();    // 추가 

        // 컨트롤러가 뷰의 논리 이름을 직접 반환하므로 이 값을 이용해서 실제 물리 뷰 찾기 
        String viewName = controller.process(paramMap, model);  
        MyView view = viewResolver(viewName);

        view.render(model, request, response);  // 프론트 컨트롤러가 직접 모델을 제공함 (모델뷰에서 모델 꺼낼 필요 없음)
    }
    ...
}
```

> MemberSaveControllerV4 - 회원 저장
```java
public class MemberSaveControllerV4 implements ControllerV4 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {

        String username = paramMap.get("username");  // 파라미터 정보가 왔겠다고 생각하고 꺼내서 쓰면 됨
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        /* (V3:)  
        ModelView mv = new ModelView("save-result");    
        mv.getModel().put("member", member); 
        return mv;
        */
        model.put("member", member);   // 모델이 파라미터로 전달되기 때문에, 모델을 직접 생성하지 않아도 됨
        return "save-result";      // 뷰의 논리 이름만 있으면 됨 (model view 필요 없음)           
    }
}
```   
## 📍 유연한 컨트롤러 - v5   
+ 어댑터를 도입해서 프론트 컨트롤러가 다양한 방식의 컨트롤러를 처리할 수 있도록 함
  + 핸들러 어댑터: 핸들러를 추가해주는 어댑터
  + 핸들러: 컨트롤러의 넓은 범위   

<img src = "https://user-images.githubusercontent.com/69106295/131458701-c3cf9087-8d74-4357-950c-af92641af5c7.png" width=50% height=50%>   

✔️ 핸들러 어댑터
+ 핸들러 매핑을 통해 찾은 컨트롤러를 직접 실행하는 기능을 수행   
+ 어댑터용 인터페이스를 구현해서 생성   

> MyHandlerAdapter (어댑터용 인터페이스)     

```java
public interface MyHandlerAdapter {

    boolean supports(Object handler);  // handler = 컨트롤러

    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;       // handler 호출해줌, 반환할 때 ModelView에 맞춰서 반환
}
```

+ `boolean supports(Object handler)`: 어댑터가 해당 컨트롤러를 처리할 수 있는지 판단하는 메서드   
+ `ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler)`: 프론트 컨트롤러가 아닌 어댑터가 실제 컨트롤러를 호출, 그 결과로 ModelView를 반환   

> ControllerV3HandlerAdapter - V3 핸들러 어댑터
```java
public class ControllerV3HandlerAdapter implements MyHandlerAdapter {

    @Override
    // ControllerV3를 처리할 수 있는 어댑터인가
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV3);
    }

    @Override
    // handler를 컨트롤러 V3로 변환한 다음에 V3 형식에 맞도록 호출함
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        ControllerV3 controller = (ControllerV3) handler;      // 캐스팅

        Map<String, String> paramMap = createParamMap(request);
        ModelView mv = controller.process(paramMap);      // 어댑터에서 실제 컨트롤러 호출

        return mv;      // ModelView 반환
    }

    private Map<String, String> createParamMap(HttpServletRequest request) {
        Map<String, String> paramMap = new HashMap<>();
        request.getParameterNames().asIterator()
                .forEachRemaining(paramName -> paramMap.put(paramName,
                        request.getParameter(paramName)));
        return paramMap;
    }
}
```    
> ControllerV4HandlerAdapter 중 일부 
```
@Override
public class ControllerV4HandlerAdapter implements MyHandlerAdapter{
    ...
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) {
    
        // 실행 로직: handler를 ControllerV4로 케스팅 하고, paramMap, model을 만들어서 해당 컨트롤러를 호출, viewName을 반환
        ControllerV4 controller = (ControllerV4) handler;

        Map<String, String> paramMap = createParamMap(request);
        Map<String, Object> model = new HashMap<>();

        String viewName = controller.process(paramMap, model);

        // 어댑터 변환 (**중요!!**)
        ModelView mv = new ModelView(viewName);
        mv.setModel(model);

        return mv;
    }
    ...
}
```
⭐ ControllerV4가 뷰의 이름을 반환 => 어댑터는 이것을 ModelView로 만들어서 형식을 맞추어 반환해야 함!!

> FrontControllerServletV5 - 프론트 컨트롤러
```java
@WebServlet(name = "frontControllerServletV5", urlPatterns = "/front-controller/v5/*")
public class FrontControllerServletV5 extends HttpServlet {

    /*  (V4:) private Map<String, ControllerV4> controllerMap = new HashMap<>(); */
    private final Map<String, Object> handlerMappingMap = new HashMap<>();     // 핸들러 매핑 정보 
    private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();   // 핸들러 어댑터 리스트

    public FrontControllerServletV5() {
        initHandlerMappingMap();  //핸들러 매핑 초기화
        initHandlerAdapters();    //어댑터 초기화
    }

    private void initHandlerMappingMap() {
        handlerMappingMap.put("/front-controller/v5/v3/members/new-form", new MemberFormControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members/save", new MemberSaveControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members", new MemberListControllerV3());

        handlerMappingMap.put("/front-controller/v5/v4/members/new-form", new MemberFormControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members/save", new MemberSaveControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members", new MemberListControllerV4());
    }

    private void initHandlerAdapters() {
        handlerAdapters.add(new ControllerV3HandlerAdapter());
        handlerAdapters.add(new ControllerV4HandlerAdapter());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        Object handler = getHandler(request);   // 1. 핸들러 매핑: 요청 정보를 가지고 핸들러를 찾아옴

        if (handler == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyHandlerAdapter adapter = getHandlerAdapter(handler);   // 2. 핸들러를 처리할 수 있는 어댑터 조회
        ModelView mv = adapter.handle(request, response, handler);   // 3. 핸들러 어댑터: 실제 컨트롤러 호출, ModelView 반환

        MyView view = viewResolver(mv.getViewName());   // 4. ViewResolver 호출, MyView 반환
        view.render(mv.getModel(), request, response);  // 5. render(model) 호출하면서 model 넘겨주기
    }

    // 핸들러 매핑
    private Object getHandler(HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        return handlerMappingMap.get(requestURI);
    }

    // 핸들러를 처리할 수 있는 어댑터 조회
    private MyHandlerAdapter getHandlerAdapter(Object handler) {
        for (MyHandlerAdapter adapter : handlerAdapters) {    // 핸들러를 다 뒤져봄
            if (adapter.supports(handler)) {       // 어댑터가 핸들러를 지원하면(핸들러가 ControllerV3 인터페이스를 구현했다면) 어댑터 반환
                return adapter;
            }
        }
        throw new IllegalArgumentException("handler adapter를 찾을 수 없습니다. handler=" + handler);  // 예외 처리
    }
    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}
```
+ **핸들러 매핑** `getHandler(request)`: handlerMappingMap에서 URL에 매핑된 핸들러 객체를 찾아서 반환   
+ **핸들러를 처리할 수 있는 어댑터 조회** `getHandlerAdapter(handler)`
  + handler 를 처리할 수 있는 어댑터를 `adapter.supports(handler)`를 통해서 찾음
  + if (handler가 ControllerV3 인터페이스를 구현했다면) => `ControllerV3HandlerAdapter` 객체가 반환
+ **어댑터 호출** `adapter.handle(request, response, handler)`   
  + 어댑터의 `handle(request, response, handler)` 메서드를 통해 실제 어댑터가 호출
  + 어댑터는 handler를 호출 => 결과를 어댑터에 맞추어 반환    

# 스프링 MVC 구조 이해  
## Spring MVC 전체 구조   

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
### 📍 SpringMVC 구조
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
+ 0 = `RequestMappingHandlerMapping` : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용   
+ 1 = `BeanNameUrlHandlerMapping` : 스프링 빈의 이름으로 핸들러를 찾는다.   

### 핸들러 어댑터   
> 스프링 부트가 자동 등록하는 핸들러 어댑터
+ 0 = `RequestMappingHandlerAdapter` : 애노테이션 기반의 컨트롤러인 @RequestMapping에서
사용
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

### 뷰 리졸버    
> 스프링 부트가 자동 등록하는 뷰 리졸버
+ 1 = `BeanNameViewResolver` : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성 기능에 사용)
+ 2 = `InternalResourceViewResolver` : JSP를 처리할 수 있는 뷰를 반환한다.

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
