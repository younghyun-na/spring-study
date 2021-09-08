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
### HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form   
✔ 클라이언트에서 서버로 요청 데이터를 전달할 때 3가지 방법   
1. GET - 쿼리 파라미터
+ `/url?username=hello&age=20`
+ 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
+ ex) 검색, 필터, 페이징등에서 많이 사용하는 방식   

2. POST - HTML Form   
+ content-type: `application/x-www-form-urlencoded`   
+ 메시지 바디에 쿼리 파리미터 형식으로 전달 `username=hello&age=20`
+ ex) 회원 가입, 상품 주문, HTML Form 사용   

3. HTTP message body에 데이터를 직접 담아서 요청   
+ HTTP API에서 주로 사용, JSON, XML, TEXT (데이터 형식은 주로 JSON)
+ POST, PUT, PATCH   

=> GET 쿼리 파리미터 전송 & POST HTML Form 전송 둘다 **요청 파라미터(request parameter) 조회** 라고 한다.   

> requestParamController - requestParamV1
```java
@Slf4j
@Controller
public class RequestParamController {
    @RequestMapping("/request-param-v1")
    public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));
        log.info("username = {}, age={}", username, age);

        response.getWriter().write("ok");

    }
}
```
### HTTP 요청 파라미터 - `@RequestParam`
+ `@RequestParam`으로 요청 파라미터를 매우 편리하게 사용 가능   

> requestParamController - requestParamV2
```java
@Slf4j
@Controller
public class RequestParamController {
    @ResponseBody     // "ok"를 문자 그대로 HTTP 응답 메시지에 넣어서 반환함 (=@Restcontroller)
    @RequestMapping("/request-param-v2")
    public String requestParamV2(
            @RequestParam("username") String memberName,
            @RequestParam("age") int memberAge) {
        log.info("username={}, age={}", memberName, memberAge);
        return "ok";
    }
}
```
+ `@RequestParam` : 파라미터 이름으로 바인딩
  + `@RequestParam("username") String memberName` = `request.getParameter("username")`
  + 축약: HTTP 파라미터 이름이 변수 이름과 같으면 생략 가능   
     + ```java
       public String requestParamV3(
           @RequestParam String username,   // 변수명이 같아야 함
           @RequestParam int age)
       ```
  + 생략: String , int , Integer 등의 단순 타입이면 @RequestParam 생략 가능   
     + ```java
       public String requestParamV4(String username, int age)
       ```
 
+ `@ResponseBody` : View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
  + `@Restcontroller`와 비슷한 기능   

> 파라미터 필수 여부 - requestParamRequired   
```java
    @ResponseBody
    @RequestMapping("/request-param-required")
    public String requestParamRequired(
            @RequestParam(required = true) String username,
            @RequestParam(required = false) Integer age) {
        // Integer = null은 가능하지만 int = null은 불가능하므로 Integer로 설정

        log.info("username={}, age={}", username, age);
        return "ok";
    }
```
+ `@RequestParam.required`: 파라미터 필수 여부 (기본값이 파라미터 필수(true))   
  + `/request-param` -> username이 없으므로 예외
+ **주의! - 파라미터 이름만 사용**
  + `/request-param?username=` -> 파라미터 이름만 있고 값이 없어서 빈문자로 통과하여 에러 안남   
+ **주의! - 기본형(primitive)에 null 입력**
  + `@RequestParam(required = false) int age`   
    => null을 받을 수 있는 **Integer** 로 변경 or **defaultValue** 사용   
    + ✔ defaultValue 사용
      ```java
      public String requestParamDefault(
          @RequestParam(required = true, defaultValue = "guest") String username,
          @RequestParam(required = false, defaultValue = "-1") int age)   // null인 경우 -1이 적용되므로 int 사용 가능
      ```
> 파라미터를 Map으로 조회 - requestParamMap
```java
@ResponseBody
@RequestMapping("/request-param-map")
public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
    log.info("username={}, age={}", paramMap.get("username"),
    paramMap.get("age"));
    return "ok";
}
```
+ `@RequestParam Map`: Map(key=value)
+ `@RequestParam MultiValueMap`: MultiValueMap(key=[value1, value2, ...]   

### HTTP 요청 파라미터 - `@ModelAttribute`
+ `@ModelAttribute`: 요청 파라미터를 받아서 필요한 객체를 만들고 그 객체에 값을 넣는 과정을 자동화해줌   

> HelloData - 요청 파라미터를 바인딩 받을 객체 
```java
@Data
public class HelloData {
    private String username;
    private int age;

}
```
+ 롬복 `@Data`: `@Getter` , `@Setter` , `@ToString` , `@EqualsAndHashCode` , `@RequiredArgsConstructor`를 자동으로 적용   

> ModelAttributeV1
```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {  //  @ModelAttribute 생략 가능
    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
    return "ok";
}
```
+ `@ModelAttribute`: HelloData의 객체 생성, 요청 파라미터의 값 넣어줌
  + 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾은 후 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩) 한다. 
  + ex) 파라미터 이름이 username 이면 `setUsername()` 메서드를 찾아서 호출하면서 값을 입력  
