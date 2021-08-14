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

## 📍 MVC 프레임워크 만들기
<img src = "https://user-images.githubusercontent.com/69106295/129441286-f2e9ae85-04a5-4014-be12-b7c6e03e9372.png" width=50% height=50%>

