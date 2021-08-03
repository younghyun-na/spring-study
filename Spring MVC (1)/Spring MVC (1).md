# Spring MVC
## 웹 애플리케이션 이해
### 📍 웹 시스템 구성 - WEB, WAS, DB
<img src = "https://user-images.githubusercontent.com/69106295/127811329-cc3a66bb-037c-4e83-97ee-b05e15190613.png" width=50% height=50%>

+ `웹 서버(Web Server)`: HTTP 기반으로 동작, **정적 리소스(파일) 제공**
+ `웹 애플리케이션 서버(WAS)`: HTTP 기반으로 동작, 웹 서버 기능 포함 + **애플리케이션 로직까지 수행**
+ 웹 시스템 구성의 장점 
  + WAS는 중요한 애플리케이션 로직만 전담 가능, 효율적인 리소스 관리 
  + WAS, DB 장애시 WEB 서버가 오류 화면 제공 가능 (Web Server는 잘 죽지X, WAS는 잘 죽음)
### 📍 서블릿
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

#### 서블릿 컨테이너
: 서블릿을 지원하는 WAS
+ 서블릿 객체를 자동 생성, 초기화, 호출, 종료하는 생명주기 관리
+ 서블릿 객체는 싱글톤으로 관리 (공유 변수 사용 주의)
+ request, response 객체는 항상 새로 생성
+ 동시 요청을 위한 멀티 쓰레드 처리 지원

### 📍 동시 요청 - 멀티 쓰레드
#### 쓰레드
: 애플리케이션 코드를 하나하나 순차적으로 실행하는 것
  + 동시 처리가 필요하면 쓰레드 추가로 생성해야 함
> 요청마다 쓰레드 생성 장단점
+ 장점) 동시 요청 처리 가능
+ 단점)  → 쓰레드 풀로 해결
  + 생성 비용 비쌈 => 응답 속도 늦어짐
  + 컨텍스트 스위칭 비용 발생
  + 쓰레드 생성 제한 X  => 메모리 임계점을 넘어 서버가 죽을 가능성
#### 쓰레드 풀
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

### 📍 HTML,HTTP API,CSR,SSR
+ `정적 리소스`: 고정된 HTML 파일, CSS, JS, 이미지, 영상 등을 제공
+ `HTML 페이지`: 동적으로 필요한 HTML 파일을 생성해서 전달
+ `HTTP API`: HTML이 아니라 데이터를 전달 (주로 JSON 형식 사용)
<img src = "https://user-images.githubusercontent.com/69106295/127992872-5cd8cd19-5696-4e87-bc13-04a4d320f75f.png" width=70% height=70%>

#### SSR (서버 사이드 렌더링)
: 서버에서 최종 HTML을 생성해서 클라이언트에 전달
+ 주로 정적인 화면에 사용
+ 관련기술: JSP, 타임리프
<img src = "https://user-images.githubusercontent.com/69106295/127994438-9a6ed78d-1182-43bc-b868-2a162b4007cb.png" width=70% height=70%>

#### CSR (클라이언트 사이드 렌더링)
: HTML 결과를 자바스크립트를 사용해 웹 브라우저에서 동적으로 생성해서 적용
+ 주로 동적인 화면에 사용, 웹 환경을 마치 앱처럼 필요한 부분부분 변경할 수 있음
+ 관련기술: React, Vue.js
<img src = "https://user-images.githubusercontent.com/69106295/127994594-6d7fdcab-3659-405a-9098-782c72bfa26d.png" width=70% height=70%>

## 서블릿 
### 📍 프로젝트 생성


