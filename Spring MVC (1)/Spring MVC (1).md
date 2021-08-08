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
## 📍 서블릿

