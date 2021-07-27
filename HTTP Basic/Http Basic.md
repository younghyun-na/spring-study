# HTTP
## 인터넷 네트워크
### IP (인터넷 프로토콜)
+ IP
  + 지정한 IP주소에 데이터 전달
  + 패킷이라는 통신 단위로 데이터 전달
  + 100.100.100.1 (출발) -> 200.200.200.2(도착)
+ IP 프로토콜의 한계
  + 비연결성: 패킷을 받을 대상이 없거나 서비스 불능 상태여도 패킷 전송
  + 비신뢰성: 패킷이 사라지는 경우, 패킷 전달 순서에 이상이 생긴 경우
  + 프로그램 구분: 같은 IP를 사용하는 서버에서 통신하는 애플리케이션이 둘 이상일 때 구분 불가능
### TCP, UDP
+ TCP: 전송 제어 프로토콜로, 3 way handshake를 사용하여 데이터 전달 및 보증하고 순서를 보장함
  + TCP segment: 출발지 PORT, 목적지 PORT, 전송 제어, 순서, 검증 정보, 전송 데이터 등
  + TCP/IP packet: 출발지 IP, PORT, 목적지 IP, PORT, 전송 데이터
> 3 way handshake
  + SYN: 접속 요청
  + ACK: 요청 수락 (함께 데이터 전송 가능)
+ UDP: 사용자 데이터그램 프로토콜로, 데이터 전달 및 순서가 보장되진 않지만 단순하고 빠름
### PORT
+ PORT: 같은 IP 내에서 프로세스 구분
### DNS
+ DNS: 도메인 네임 시스템으로, 도메인 명을 IP 주소로 변환
  + ex) 클라이언트 -> DNS 서버: 도메인 명 google.com
        DNS 서버 -> 클라이언트: 200.200.200.2 (도메인 명에 해당하는 IP 주소로 응답)
        클라이언트 -> 서버: IP가 200.200.200.2인 곳에 접속
### URI
+ `URI` (Uniform Resource Identifier): 리소스 식별하는 통일된 방식, URI로 식별할 수 있는 모든 자원, 다른 항목과 구분하는데 필요한 정보
+ `URL` (Uniform Resource Locator): 리소스가 있는 위치를 지정
  + scheme://[userinfo@]host[:port][/path][?query][#fragment]
  + https://www.google.com:443/search?q=hello&hl=ko
    + scheme: 주로 프로토콜 사용 (https)
    + host: 도메인명 또는 IP 주소를 직접 사용가능 (www.google.com)
    + PORT: 접속 포트 (:443)
    + path: 리소스 경로 (/search)
    + query: 웹서버에 제공하는 파라미터, 문자 형태로 ?로 시작하고 &로 기능을 추가함 (?q=hello&hl=ko)
    + fragment
+ `URN` (Uniform Resource Name): 리소스에 이름을 부여
<img src="https://user-images.githubusercontent.com/69106295/127104886-8221ca23-a714-48b6-9258-83d0f265429e.png" width="60%" height="60%"/>

# HTTP
: **H**yper **T**ext **T**ransfer **P**rotocol
HTTP 메시지에 모든 형태의 데이터 전송 가능
## HTTP 특징
#### 1. 클라이언트 서버 구조
+ 클라이언트는 서버에 요청을 보내고, 응답을 대기
+ 서버가 요청에 대한 결과를 만들어서 응답
#### 2. 무상태 프로토콜
> Stateful(상태 유지), Stateless(무상태) 차이
+ `Stateful`: 서버가 클라이언트의 상태를 유지. (중간에 다른 점원으로 바뀌면 안됨)
+ `Stateless`: 서버가 클라이언트의 상태를 보존하지 않음. (중간에 다른 점원으로 바뀌어도 됨)
  + 서버 확장성 높음 (응답 서버를 쉽게 바꿀 수 있기 때문에 무한한 서버 증설이 가능함)
  + 아무 서버나 호출해도 됨
  + 클라이언트가 추가 데이터 전송해야 함
#### 3. 비 연결성(connectionless)
+ Http는 기본이 연결을 유지하지 않는 모델
+ 일반적으로 초 단위의 이하의 빠른 속도로 응답
+ Http 지속 연결을 함 (연결, 종료 낭비를 막기 위함)
#### 4. Http 메세지
> Http 메시지 구조
<img src="https://user-images.githubusercontent.com/69106295/127193142-022d20f3-5967-4f44-97b2-23b2dceb0784.png" width="30%" height="30%"/>
<img src="https://user-images.githubusercontent.com/69106295/127194374-a5c4ecb5-141d-44ad-8a8f-9c8f4f212b37.png" width="30%" height="30%"/>

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
1. Start line: status-line = Http Version + status code + 이유 문구
  + `status code`: 요청 성공, 실패를 나타냄 (200: 성공, 400: 클라이언트 요청 오류, 500: 서버 오류)
  + `이유 문구`: 짧은 상태 코드 설명 글   
2. Header: HTTP 전송에 필요한 모든 부가정보
+ header-field = field-name: field-value   
3. Message body: 실제 전송할 데이터
<img src = "https://user-images.githubusercontent.com/69106295/127193728-dc393800-9602-4059-b389-3a74929d885a.png" width="30%" height="30%"/>

# Http 메서드
## API URI 설계 



