# HTTP
## 인터넷 네트워크
### IP (인터넷 프로토콜)
+ IP: 지정한 IP주소에 패킷이라는 통신 단위로 데이터 전달
  + ex) 100.100.100.1 (출발) -> 200.200.200.2(도착)
+ IP 프로토콜의 한계
  + 비연결성: 패킷을 받을 대상이 없거나 서비스 불능 상태여도 패킷 전송
  + 비신뢰성: 패킷이 사라지는 경우, 패킷 전달 순서에 이상이 생긴 경우 문제 발생
  + 프로그램 구분: 같은 IP를 사용하는 서버에서 통신하는 애플리케이션이 둘 이상일 때 구분 불가능
### TCP, UDP
+ `TCP`: 전송 제어 프로토콜로, 3 way handshake를 사용하여 데이터 전달을 보증하고 순서를 보장함
  + TCP segment: 출발지 PORT, 목적지 PORT, 전송 제어, 순서, 검증 정보 등
  + TCP/IP packet: 출발지 IP, 출발지 PORT, 목적지 IP, 목적지 PORT, 전송 데이터
> 3 way handshake
  + SYN: 접속 요청 
  + ACK: 요청 수락 (함께 데이터 전송 가능)
+ `UDP`: 사용자 데이터그램 프로토콜로, 데이터 전달 및 순서가 보장되진 않지만 단순하고 빠름
### PORT
+ `PORT`: 같은 IP 내에서 프로세스 구분
  + IP: 목적지 서버를 찾는 것 (아파트) 
  + PORT: 서버 안에서 돌아가는 애플리케이션을 구분 (동호수) 
### DNS
+ `DNS`: 도메인 네임 시스템으로, 도메인 명을 IP 주소로 변환 (IP의 기억하기 어려운 특성과 IP 변경 가능성 문제 해결)
  + ex) DNS 서버에 도메인 명 google.com을 IP 주소가 200.200.200.2라고 등록
        클라이언트 -> DNS 서버: google.com에 접근할래
        DNS 서버 -> 클라이언트: 200.200.200.2로 응답 (도메인 명에 해당하는 IP 주소로 응답)
        클라이언트 -> 서버: IP가 200.200.200.2인 곳에 접속
### URI
+ `URI` (Uniform Resource Identifier): 리소스 식별하는 통일된 방식, URI로 식별할 수 있는 모든 자원, 다른 항목과 구분하는데 필요한 정보
+ `URL` (Uniform Resource Locator): 리소스가 있는 위치를 지정
  + scheme://[userinfo@]host[:port][/path][?query][#fragment]
  + https://www.google.com:443/search?q=hello&hl=ko
    + scheme: 주로 프로토콜 사용 (https)
    + host: 도메인명 또는 IP 주소를 직접 사용가능 (www.google.com)
    + PORT: 접속 포트 (:443), http는 80, https는 443
    + path: 리소스 경로 (/search)
    + query: 웹서버에 제공하는 파라미터, 문자 형태로 ?로 시작하고 &로 기능을 추가함 (?q=hello&hl=ko)
    + fragment: html 내부 북마크 등에 사용 
+ `URN` (Uniform Resource Name): 리소스에 이름을 부여 (거의 쓰지 않음)
<img src="https://user-images.githubusercontent.com/69106295/127104886-8221ca23-a714-48b6-9258-83d0f265429e.png" width="60%" height="60%"/>

# HTTP
: **H**yper **T**ext **T**ransfer **P**rotocol   
HTTP 메시지에 모든 형태의 데이터 전송 가능   
단순하면서 확장 가능 특징이 있음 
## HTTP 특징
#### 1. 클라이언트 서버 구조
+ 클라이언트는 서버에 요청을 보내고, 응답을 대기
+ 서버가 요청에 대한 결과를 만들어서 응답
#### 2. 무상태 프로토콜 
> Stateful(상태 유지), Stateless(무상태) 차이
⭐ 최대한 Stateless를 사용해야 함
+ `Stateful`: 서버가 클라이언트의 상태를 유지. (중간에 다른 점원으로 바뀌면 안됨)
  + 상태 유지는 최소한만 사용 (꼭 필요한 경우: 로그인)
+ `Stateless`: 서버가 클라이언트의 상태를 보존하지 않음. (고객이 필요한 데이터를 그때그때 넘기므로 중간에 다른 점원으로 바뀌어도 됨)
  + 서버 확장성 높음 (응답 서버를 쉽게 바꿀 수 있기 때문에 무한한 서버 증설이 가능함)
  + 아무 서버나 호출해도 됨
  + 클라이언트가 추가 데이터 전송해야 함

#### 3. 비 연결성(connectionless)
+ Http는 기본이 연결을 유지하지 않는 모델 (최소한의 자원 사용)
+ 단점) 연결을 새로 맺는 과정에서 3 way handshake 시간 추가   
 => Http 지속 연결로 문제 해결 (연결, 종료시 발생하는 시간 낭비를 막기 위함)

#### 4. Http 메세지
> Http 메시지 구조   
   
 <img src="https://user-images.githubusercontent.com/69106295/127193142-022d20f3-5967-4f44-97b2-23b2dceb0784.png" width="30%" height="30%"/> <img src="https://user-images.githubusercontent.com/69106295/127194374-a5c4ecb5-141d-44ad-8a8f-9c8f4f212b37.png" width="50%" height="50%"/>

> 요청 메시지
1. Start line: `request-line` = Http 메서드 + 요청 대상 + Http Version   
  + `Http 메서드`: 서버가 수행해야 할 동작 지정 (GET: 리소스 조회, POST: 요청 내역 처리)   
  + `요청 대상`: "/"로 시작되는 절대 경로[?쿼리]  (ex. /search?q=hello&hl=ko)
  + `Http Version`: (ex. HTTP/1.1)
2. Header  
3. Message body: 실제 전송할 데이터
  + 요청 메세지가 body 본문을 가질 수도, 안 가질 수도 있음
<img src = "https://user-images.githubusercontent.com/69106295/127193512-35aad03f-e91f-4501-841c-184cabb0deac.png" width="30%" height="30%"/>

> 응답 메시지
1. Start line: `status-line` = Http Version + status code(상태 코드) + 이유 문구
  + `status code`: 요청 성공, 실패를 나타냄 (200: 성공, 400: 클라이언트 요청 오류, 500: 서버 오류)
  + `이유 문구`: 짧은 상태 코드 설명 글 
2. Header: HTTP 전송에 필요한 모든 부가정보
+ header-field = field-name: field-value   
3. Message body: 실제 전송할 데이터
<img src = "https://user-images.githubusercontent.com/69106295/127193728-dc393800-9602-4059-b389-3a74929d885a.png" width="30%" height="30%"/>

# HTTP 메서드
## API URI 설계
리소스와 행위을 분리해야 함! (리소스를 식별하는 것이 중요)
+ URI는 리소스만 식별, Http 메서드가 행위를 구분
  + ex) 회원 조회, 회원 등록, 회원 수정, 회원 삭제에서
    + `리소스`: 회원
    + `행위`: 조회, 등록, 삭제, 변경
## HTTP 메서드
#### 1. GET: 리소스 조회
+ 서버에 전달하고 싶은 데이터를 query(쿼리 파라미터, 쿼리 스트링)를 통해서 전달
#### 2. POST: 요청 데이터 처리, 등록
+ POST 메서드란: 대상 리소스가 리소스의 고유 한 의미 체계에 따라 요청에 포함된 표현을 처리하도록 요청하는 것
+ 메시지 바디를 통해 서버로 요청 데이터 전달하면 서버가 이를 처리함
  + 리소스 등록 (메시지 전달), 신규 리소스 식별자 생성, 응답 데이터
+ 요청 데이터 처리 방법: 리소스 URI에 POST 요청이 오면 요청 데이터를 어떻게 처리할지 리소스마다 따로 정해야 함 (정해진 것이 없음)
  + 새 리소스 생성 
  + 요청 데이터 처리
    + 단순히 데이터 생성 or 변경을 넘어 프로세스를 처리해야 하는 경우
  + 다른 메서드로 처리하기 어려운 경우 (애매하면 POST)

#### 3. PUT: 리소스를 완전 대체, 해당 리소스가 없으면 생성
⭐ 클라이언트가 리소스를 식별! (리소스를 알고있어야 함)
+ 클라이언트가 리소스 위치를 알고 URI 지정 (POST와의 차이점)
  + POST: `POST /members HTTP/1.1`
  + PUT: `PUT /members/100 HTTP/1.1`

#### 4. PATCH: 리소스 부분 변경

#### 5. DELETE: 리소스 제거

### HTTP 메서드의 속성
✔ **안전**: 호출해도 리소스를 변경하지 않음
+ 안전 O: `GET`, `HEAD`
+ 안전 X: `POST`, `PUT`, `PATCH`, `DELETE`  

✔ **멱등**: 몇 번 호출하던 결과가 같음 (f(f(x)) = f(x))
+ 멱등 O: `GET`, `PUT`, `DELETE`
+ 멱등 X: `POST` (두 번 호출하면 두 번 발생 가능)
+ 활용: 자동 복구 메커니즘 (서버에서 응답이 없을 때 클라이언트가 자동으로 재시도할 수 있도록)
+ 외부 요인으로 중간에 리소스가 변경되는 것까지는 고려하지 않음  

✔ **캐시 가능**: 응답 결과 리소스를 캐시해서 사용해도 되는가?
+ 캐시 가능: `GET`, `HEAD`(주로 사용) `POST`, `PATCH` (구현이 쉽지 않음)

## HTTP 메서드 활용
### 클라이언트에서 서버로 데이터 전송
> 쿼리 파라미터를 통한 데이터 전송
+ GET
+ 주로 정렬 필터
> 메시지 바디를 통한 데이터 전송
+ `POST`, `PUT`, `PATCH`
> 4가지 상황
#### 1. 정적 데이터 조회
+ 조회는 GET 사용
+ 일반적으로 쿼리 파라미터 없이 리소스 경로로 단순하게 조회 가능   

<img src = "https://user-images.githubusercontent.com/69106295/128603004-18225258-2354-4838-b4d5-09c895feee3b.png" width="70%" height="70%"/>

#### 2. 동적 데이터 조회
+ 주로 검색, 게시판 목록에서 정렬 필터(검색어)
+ 조회 조건을 줄여주는 필터, 조회 결과를 정렬하는 정렬 조건에 주로 사용
+ 조회는 GET 사용 (쿼리 파라미터를 사용해서 데이터를 전달)   

<img src = "https://user-images.githubusercontent.com/69106295/128602943-048b7d00-c60d-4b74-855c-2832084603d2.png" width="70%" height="70%"/>
<img src = "https://user-images.githubusercontent.com/69106295/128603038-a14df266-5a16-42b8-8ca4-7e6a93cb6359.png" width="70%" height="70%"/>





