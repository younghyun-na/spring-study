# Spring MVC
## 웹 애플리케이션 이해
### 웹 시스템 구성 - WEB, WAS, DB
<img src = "https://user-images.githubusercontent.com/69106295/127811329-cc3a66bb-037c-4e83-97ee-b05e15190613.png" width=50% height=50%>

+ `웹 서버(Web Server)`: HTTP 기반으로 동작, **정적 리소스(파일) 제공**
+ `웹 애플리케이션 서버(WAS)`: HTTP 기반으로 동작, 웹 서버 기능 포함 + **애플리케이션 로직까지 수행**
+ 웹 시스템 구성의 장점 
  + WAS는 중요한 애플리케이션 로직만 전담 가능, 효율적인 리소스 관리 
  + WAS, DB 장애시 WEB 서버가 오류 화면 제공 가능 (Web Server는 잘 죽지X, WAS는 잘 죽음)
### 서블릿
