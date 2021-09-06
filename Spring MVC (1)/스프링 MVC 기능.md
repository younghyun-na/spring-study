# 스프링 MVC 기본 기능  
## 📍 로깅  
+ 로깅이란?   
 : 정보를 제공하는 일련의 기록인 로그(log)를 생성하도록 시스템을 작성하는 활동   
  + 운영 시스템에서는 System.out.println() 같은 시스템 콘솔을 사용해서 정보를 출력하지 않고, 별도의 로깅 라이브러리를 사용해서 로그를 출력  
+ 로깅 라이브러리
  + `SLF4J`: 수많은 로그 라이브러리들을 통합해서 인터페이스로 제공하는 것 (인터페이스)
  + `Logback`: 스프링 부트가 기본으로 제공하는 로그 라이브러리 (구현체)   

> LogTestController
```java
// @Controller => 뷰 이름을 반환
@RestController  // @RestController는 문자열 그대로 반환
public class LogTestController {
    private final Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping("/log-test")
    public String logTest(){
        String name = "Spring";

        // 밑으로 갈수록 레벨이 높아지는 것 순서

        // log.trace("trace my log =" + name); => 문자 더하기를 다 한뒤 trace를 사용하지 않는다는 결정이 나기 때문에 쓸모없는 리소스 사용 (비효율적)
        log.trace("trace log={}", name);   // 파라미터만 넘긴 뒤에 사용하지 않는 결정이 남 (효율적)
        log.debug("debug log={}", name);   // 개발 서버에서 debug로 배포
        log.info(" info log={}", name);    // 운영 서버에서 info로 배포
        log.warn(" warn log={}", name);
        log.error("error log={}", name);

        log.debug("String concat log=" + name);
        return "ok";    // 뷰 이름이 아닌, 문자열 그대로 반환
    }
}
```
+ `@RestController`   
  + `@Controller` 는 반환 값이 String 이면 뷰 이름으로 인식 => 뷰를 찾고 뷰가 랜더링 됨
  + `@RestController` 는 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력함 => 실행 결과로 ok 메세지를 받음      

### 로그 레벨 설정  
+ 레벨: TRACE > DEBUG > INFO > WARN > ERROR
  + 개발 서버는 debug 출력
  + 운영 서버는 info 출력
```text
#전체 로그 레벨 설정(기본 info)
    logging.level.root=info
    
#hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug
```
### 올바른 로그 사용법   
+ `log.debug("data={}", data)`
  + 로그 출력 레벨을 info로 설정시, 아무일도 발생하지 않음 => 의미없는 연산이 발생하지 않음
  + `log.debug("data="+data)`   
    : 로그 출력 레벨을 info로 설정해도, "data="+data가 실제 실행이 되어 버림 => 문자 더하기 연산이 발생   

### 로그 사용시 장점
+ 쓰레드 정보, 클래스 이름 같은 부가 정보도 볼 수 있음
+ 로그 레벨에 따라 로그를 서버의 종류에 맞게 조절 가능 (운영 서버, 개발 서버..)
+ 콘솔 뿐만 아니라, 파일이나 네트워크와 같은 별도의 위치에 남길 수 있음
+ 성능도 일반 System.out보다 좋음(내부 버퍼링, 멀티 쓰레드 등등)  => 실무에서는 꼭 로그를 사용!!!!   

## 📍 HTTP 요청 매핑   
> MappingController - 기본 
```java
@RestController
public class MappingController {
    private Logger log = LoggerFactory.getLogger(getClass());
  
    @RequestMapping("/hello-basic")
    public String helloBasic() {
        log.info("helloBasic");
        return "ok";
    }
  
}
```
#### 매핑 정보
+ `@RestController`: HTTP 메시지 바디에 바로 입력
+ `@RequestMapping("/hello-basic")`: `/hello-basic` URL 호출이 오면 이 메서드가 실행되도록 매핑   

> PathVariable(경로 변수) 사용   
```java
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data) {   // // @PathVariable로 값을 꺼내서 사용
    log.info("mappingPath userId={}", data);
    return "ok";
}
```
+ 최근 HTTP API는 다음과 같이 리소스 경로에 식별자를 넣는 스타일을 선호
+ `@PathVariable`: 매칭 되는 부분을 편리하게 조회 (이름과 파라미터 이름이 같으면 생략 가능)   

> 미디어 타입 조건 매핑 - HTTP 요청 Content-Type, consume
```java
@PostMapping(value = "/mapping-consume", consumes = "application/json")   // header의 content 타입이 application/json인 경우만 호출됨
public String mappingConsumes() {
    log.info("mappingConsumes");
    return "ok";
}
```
+ HTTP 요청의 Content-Type 헤더를 기반으로 미디어 타입으로 매핑
  + 맞지 않으면 HTTP 415 상태코드(Unsupported Media Type)을 반환  
  
> 미디어 타입 조건 매핑 - HTTP 요청 Accept, produce
```java
@PostMapping(value = "/mapping-produce", produces = "text/html")   // 컨트롤러가 생산해내는 content 타입
public String mappingProduces() {
    log.info("mappingProduces");
    return "ok";
}
```
+ HTTP 요청의 Accept 헤더를 기반으로 미디어 타입으로 매핑
  + 맞지 않으면 HTTP 406 상태코드(Not Acceptable)을 반환   

### 요청 매핑 - API 예시   
```java
@RestController
@RequestMapping("/mapping/users")
public class MappingClassController {

    /**
     * 회원 목록 조회: GET     /users
     * 회원 등록:     POST     /users
     * 회원 조회:     GET      /users/{userId}
     * 회원 수정:     PATCH    /users/{userId}
     * 회원 삭제:     DELETE   /users/{userId}
     */

    @GetMapping
    public String user(){
        return "get users";
    }

    @PostMapping
    public String addUser(){
        return "post user";
    }


    @GetMapping("/{userId}")
    public String findUser(@PathVariable String userId) {
        return "get userId=" + userId;
    }

    @PatchMapping("/{userId}")
    public String updateUser(@PathVariable String userId) {
        return "update userId=" + userId;
    }

    @DeleteMapping("/{userId}")
    public String deleteUser(@PathVariable String userId) {
        return "delete userId=" + userId;
    }
}
```
## 📍 HTTP 요청 데이터 조회
### HTTP 요청 - 기본, 헤더 조회   
```java
@Slf4j
@RestController
public class RequestHeaderController {

    @RequestMapping("/headers")
    public String headers(HttpServletRequest request,
                          HttpServletResponse response,
                          HttpMethod httpMethod,    // HTTP 메서드 조회
                          Locale locale,         //  Locale 정보 조회
                          @RequestHeader MultiValueMap<String, String> headerMap,  // 모든 HTTP 헤더를 MultiValueMap 형식으로 조회
                          @RequestHeader("host") String host,    // 특정 HTTP 헤더를 조회
                          @CookieValue(value = "myCookie", required = false) String cookie
                          ) {
        log.info("request={}", request);
        log.info("response={}", response);
        log.info("httpMethod={}", httpMethod);
        log.info("locale={}", locale);
        log.info("headerMap={}", headerMap);
        log.info("header host={}", host);
        log.info("myCookie={}", cookie);

        return "ok";
    }
}
```
