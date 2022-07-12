# 요청매핑

```java
@RestController
public class MappingController {
  private Logger log = LoggerFactory.getLogger(getClass());
  /**
  * 기본 요청
  * 둘다 허용 /hello-basic, /hello-basic/
  * HTTP 메서드 모두 허용 GET, HEAD, POST, PUT, PATCH, DELETE
  */
  @RequestMapping("/hello-basic")
  public String helloBasic() {
    log.info("helloBasic");
    return "ok";
  }
}
```
@Controller 는 반환 값이 String 이면 뷰 이름으로 인식된다. 그래서 뷰를 찾고 뷰가 랜더링 된다.

@RestController 는 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력한다. 따라서 실행 결과로 ok 메세지를 받을 수 있다.

@RequestMapping에 method 속성으로 HTTP 메서드를 지정하지 않으면 HTTP 메서드와 무관하게 호출된다. 즉 GET, HEAD, POST, PUT, PATCH, DELETE 모두 허용된다.


```java
/**
 * method 특정 HTTP 메서드 요청만 허용
 * GET, HEAD, POST, PUT, PATCH, DELETE
 */
@RequestMapping(value = "/mapping-get-v1", method = RequestMethod.GET)
public String mappingGetV1() {
  log.info("mappingGetV1");
  return "ok";
}
```
만약 여기에 POST 요청을 하면 스프링 MVC는 HTTP 405 상태코드(Method Not Allowed)를 반환.


```java
/**
 * 편리한 축약 애노테이션 (코드보기)
 * @GetMapping
 * @PostMapping
 * @PutMapping
 * @DeleteMapping
 * @PatchMapping
 */
@GetMapping(value = "/mapping-get-v2")
public String mappingGetV2() {
  log.info("mapping-get-v2");
  return "ok";
}
```
HTTP 메서드를 축약한 애노테이션을 사용하는 것이 더 직관적이다. 코드를 보면 내부에서 @RequestMapping과 method를 지정해서 사용하는 것을 확인 가능.


```java
/**
 * PathVariable 사용
 * 변수명이 같으면 생략 가능
 * @PathVariable("userId") String userId -> @PathVariable userId
 */
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data) {
  log.info("mappingPath userId={}", data);
  return "ok";
}

/**
 * PathVariable 사용 다중
 */
@GetMapping("/mapping/users/{userId}/orders/{orderId}")
public String mappingPath(@PathVariable String userId, @PathVariable Long 
orderId) {
  log.info("mappingPath userId={}, orderId={}", userId, orderId);
  return "ok";
}
```
@RequestMapping 은 URL 경로를 템플릿화 할 수 있는데, @PathVariable 을 사용하면 매칭 되는 부분을 편리하게 조회할 수 있다.

@PathVariable 의 이름과 파라미터 이름이 같으면 생략할 수 있다.


```java
/**
 * 파라미터로 추가 매핑
 * params="mode",
 * params="!mode"
 * params="mode=debug"
 * params="mode!=debug" (! = )
 * params = {"mode=debug","data=good"}
 */
@GetMapping(value = "/mapping-param", params = "mode=debug")
public String mappingParam() {
  log.info("mappingParam");
  return "ok";
}
```
```
http://localhost:8080/mapping-param?mode=debug
```
특정 파라미터가 있거나 없는 조건을 추가할 수 있다. 잘 사용하지는 않는다.


```java
/**
 * 특정 헤더로 추가 매핑
 * headers="mode",
 * headers="!mode"
 * headers="mode=debug"
 * headers="mode!=debug" (! = )
 */
@GetMapping(value = "/mapping-header", headers = "mode=debug")
public String mappingHeader() {
  log.info("mappingHeader");
  return "ok";
}
```
파라미터 매핑과 비슷하지만, HTTP 헤더를 사용한다. Postman과 같은 도구를 이용해 확인할 수 있다.


```java
/**
 * Content-Type 헤더 기반 추가 매핑 Media Type
 * consumes="application/json"
 * consumes="!application/json"
 * consumes="application/*"
 * consumes="*\/*"
 * MediaType.APPLICATION_JSON_VALUE
 */
@PostMapping(value = "/mapping-consume", consumes = "application/json")
public String mappingConsumes() {
  log.info("mappingConsumes");
  return "ok";
}
```
HTTP 요청의 Content-Type 헤더를 기반으로 미디어 타입으로 매핑한다. 만약 맞지 않으면 HTTP 415 상태코드(Unsupported Media Type)을 반환한다.


```java
/**
 * Accept 헤더 기반 Media Type
 * produces = "text/html"
 * produces = "!text/html"
 * produces = "text/*"
 * produces = "*\/*"
 */
@PostMapping(value = "/mapping-produce", produces = "text/html")
public String mappingProduces() {
  log.info("mappingProduces");
  return "ok";
}
```
HTTP 요청의 Accept 헤더를 기반으로 미디어 타입으로 매핑한다. 만약 맞지 않으면 HTTP 406 상태코드(Not Acceptable)을 반환한다.


```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpMethod;
import org.springframework.util.MultiValueMap;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Locale;

@Slf4j
@RestController
public class RequestHeaderController {
  @RequestMapping("/headers")
  public String headers(HttpServletRequest request, HttpServletResponse response, HttpMethod httpMethod, Locale locale,
    @RequestHeader MultiValueMap<String, String> headerMap, @RequestHeader("host") String host, @CookieValue(value = "myCookie", required = false) String cookie) {
    
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
애노테이션 기반의 스프링 컨트롤러는 다양한 파라미터를 지원한다. HTTP 헤더 정보를 조회가 가능하다.

HttpServletRequest

HttpServletResponse

HttpMethod : HTTP 메서드를 조회한다. org.springframework.http.HttpMethod

Locale : Locale 정보를 조회한다.

@RequestHeader MultiValueMap<String, String> headerMap : 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.

@RequestHeader("host") String host : 특정 HTTP 헤더를 조회한다.

@CookieValue(value = "myCookie", required = false) String cookie : 특정 쿠키를 조회한다.

MultiValueMap : MAP과 유사한데, 하나의 키에 여러 값을 받을 수 있다. HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다. 


# HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form

클라이언트에서 서버로 요청 데이터를 전달할 때는 주로 다음 3가지 방법을 사용.

1. GET - 쿼리 파라미터<br>
메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달<br>
예) 검색, 필터, 페이징등에서 많이 사용하는 방식

2. POST - HTML Form<br>
content-type: application/x-www-form-urlencoded<br>
메시지 바디에 쿼리 파리미터 형식으로 전달 username=hello&age=20<br>
예) 회원 가입, 상품 주문, HTML Form 사용

3. HTTP message body에 데이터를 직접 담아서 요청<br>
HTTP API에서 주로 사용, JSON, XML, TEXT<br>
데이터 형식은 주로 JSON 사용<br>
POST, PUT, PATCH

```java
@Slf4j
@Controller
public class RequestParamController {
  /**
  * 반환 타입이 없으면서 이렇게 응답에 값을 직접 집어넣으면, view 조회X
  */
  @RequestMapping("/request-param-v1")
  public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));
    log.info("username={}, age={}", username, age);
    response.getWriter().write("ok");
 }
}
```
request.getParameter(): HttpServletRequest가 제공하는 방식으로 요청 파라미터를 조회.

요청 파라미터에서 쿼리 파라미터, HTML Form의 경우 HttpServletRequest 의 request.getParameter()를 사용하면 다음 두가지 요청 파라미터를 조회할 수 있다.


```java
/**
 * @RequestParam 사용
 * - 파라미터 이름으로 바인딩
 * @ResponseBody 추가
 * - View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
 */
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2(@RequestParam("username") String memberName, @RequestParam("age") int memberAge) {
  log.info("username={}, age={}", memberName, memberAge);
  return "ok";
}

/**
 * @RequestParam 사용
 * HTTP 파라미터 이름이 변수 이름과 같으면 @RequestParam(name="xx") 생략 가능
 */
@ResponseBody
@RequestMapping("/request-param-v3")
public String requestParamV3(@RequestParam String username, @RequestParam int age) {
  log.info("username={}, age={}", username, age);
  return "ok";
}

/**
 * @RequestParam 사용
 * String, int 등의 단순 타입이면 @RequestParam 도 생략 가능
 */
@ResponseBody
@RequestMapping("/request-param-v4")
public String requestParamV4(String username, int age) {
  log.info("username={}, age={}", username, age);
  return "ok";
}
```
@RequestParam : 파라미터 이름으로 바인딩

@RequestParam의 name(value) 속성이 파라미터 이름으로 사용

@RequestParam("username") String memberName -> request.getParameter("username")

@RequestParam 애노테이션을 생략하면 스프링 MVC는 내부에서 required=false 를 적용.

```java
/**
 * @RequestParam.required
 * /request-param-required -> username이 없으므로 예외
 *
 * 주의!
 * /request-param-required?username= -> 빈문자로 통과
 *
 * 주의!
 * /request-param-required
 * int age -> null을 int에 입력하는 것은 불가능, 따라서 Integer 변경해야 함(또는 다음에 나오는 defaultValue 사용)
 */
@ResponseBody
@RequestMapping("/request-param-required")
public String requestParamRequired(@RequestParam(required = true) String username, @RequestParam(required = false) Integer age) {
  log.info("username={}, age={}", username, age);
  return "ok";
}
```

@RequestParam.required는 파라미터 필수 여부로 기본값이 파라미터 필수(true).

/request-param 요청시 username 이 없으므로 400 예외가 발생.

파라미터 이름만 있고 값이 없는 경우 다음과 같이 요청을 하면 /request-param?username= 빈문자로 인식해 통과.

/request-param 요청 (@RequestParam(required = false) int age)의 경우 null 을 int 에 입력하는 것은 불가능(500 예외 발생)
따라서 null 을 받을 수 있는 Integer 로 변경하거나, 또는 defaultValue 사용.

```java
/**
 * @RequestParam
 * - defaultValue 사용
 *
 * 참고: defaultValue는 빈 문자의 경우에도 적용
 * /request-param-default?username=
 */
@ResponseBody
@RequestMapping("/request-param-default")
public String requestParamDefault(
  @RequestParam(required = true, defaultValue = "guest") String username,
  @RequestParam(required = false, defaultValue = "-1") int age) {
    log.info("username={}, age={}", username, age);
    return "ok";
}
```
파라미터에 값이 없는 경우 defaultValue 를 사용하면 기본 값을 적용할 수 있다.

이미 기본 값이 있기 때문에 required 는 의미가 없다.

defaultValue 는 빈 문자의 경우에도 설정한 기본 값이 적용된다. ex) /request-param-default?username=


```java
/**
 * @RequestParam Map, MultiValueMap
 * Map(key=value)
 * MultiValueMap(key=[value1, value2, ...]) ex) (key=userIds, value=[id1, id2])
 */
@ResponseBody
@RequestMapping("/request-param-map")
public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
  log.info("username={}, age={}", paramMap.get("username"),paramMap.get("age"));
  return "ok";
}
```
파라미터를 Map, MultiValueMap으로 조회할 수 있다.

파라미터의 값이 1개가 확실하다면 Map을 사용해도 되지만, 그렇지 않다면 MultiValueMap을 사용.


```java
/**
 * @ModelAttribute 사용
 * 참고: model.addAttribute(helloData) 코드도 함께 자동 적용됨, 뒤에 model을 설명할 때
자세히 설명
 */
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {
  log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
  return "ok";
}
```
HelloData 객체가 생성되고, 요청 파라미터의 값도 모두 들어간다.

스프링MVC는 @ModelAttribute 가 있으면 다음을 실행한다.
1. HelloData 객체를 생성한다.
2. 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩) 한다. 예) 파라미터 이름이 username 이면 setUsername() 메서드를 찾아서 호출하면서 값을 입력한다.

age=abc처럼 숫자가 들어가야 할 곳에 문자를 넣으면 BindException 이 발생한다. 

```java
/**
 * @ModelAttribute 생략 가능
 * String, int 같은 단순 타입 = @RequestParam
 * argument resolver 로 지정해둔 타입 외 = @ModelAttribute
 */
@ResponseBody
@RequestMapping("/model-attribute-v2")
public String modelAttributeV2(HelloData helloData) {
  log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
  return "ok";
}
```
@ModelAttribute 는 생략할 수 있다.

String , int , Integer 같은 단순 타입 = @RequestParam

나머지 = @ModelAttribute (argument resolver 로 지정해둔 타입 외)

# HTTP 요청 메시지 - 단순 텍스트

HTTP message body에 데이터를 직접 담아서 요청

HTTP API에서 주로 사용, JSON, XML, TEXT 데이터 형식은 주로 JSON 사용

POST, PUT, PATCH


```java
@Slf4j
@Controller
public class RequestBodyStringController {

  @PostMapping("/request-body-string-v1")
  public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
    ServletInputStream inputStream = request.getInputStream();
    String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
    log.info("messageBody={}", messageBody);
    response.getWriter().write("ok");
  }
}
```
단순한 텍스트 메시지를 HTTP 메시지 바디에 담아서 전송한다. HTTP 메시지 바디의 데이터를 InputStream 을 사용해서 직접 읽을 수 있다.

```java
/**
 * InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
 * OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력
 */
@PostMapping("/request-body-string-v2")
public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
  String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
  log.info("messageBody={}", messageBody);
  responseWriter.write("ok");
}
```
InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력


```java
/**
 * HttpEntity: HTTP header, body 정보를 편리하게 조회
 * - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 *
 * 응답에서도 HttpEntity 사용 가능
 * - 메시지 바디 정보 직접 반환(view 조회X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 */
@PostMapping("/request-body-string-v3")
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
  String messageBody = httpEntity.getBody();
  log.info("messageBody={}", messageBody);
  return new HttpEntity<>("ok");
}
```
HttpEntity: HTTP header, body 정보를 편리하게 조회, 메시지 바디 정보를 직접 조회, 요청 파라미터를 조회하는 기능과 관계 없음 @RequestParam X, @ModelAttribute X

httpEntity는 응답에도 사용 가능 : 메시지 바디 정보 직접 반환, 헤더 정보 포함 가능, view 조회X

HttpEntity 를 상속받은 다음 객체들도 같은 기능을 제공한다.<BR>
1. RequestEntity<BR>
HttpMethod, url 정보가 추가, 요청에서 사용

2. ResponseEntity<BR>
HTTP 상태 코드 설정 가능, 응답에서 사용


```java
/**
 * @RequestBody
 * - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 *
 * @ResponseBody
 * - 메시지 바디 정보 직접 반환(view 조회X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 */
@ResponseBody
@PostMapping("/request-body-string-v4")
public String requestBodyStringV4(@RequestBody String messageBody) {
  log.info("messageBody={}", messageBody);
  return "ok";
}
```
@RequestBody 를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다. 응답결과를 HTTP 메시지 바디에 직접 담아서 전달할 수 있다. 이경우 view를 사용하지 않는다.
참고로 헤더 정보가 필요하다면 HttpEntity 를 사용하거나 @RequestHeader 를 사용하면 된다.
  
이렇게 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 @RequestParam, @ModelAttribute 와는 전혀 관계가 없다.
  
- 요청 파라미터를 조회하는 기능: @RequestParam , @ModelAttribute
- HTTP 메시지 바디를 직접 조회하는 기능: @RequestBody


# HTTP 요청 메시지 - JSON
  
```java
/**
 * {"username":"hello", "age":20}
 * content-type: application/json
 */
@Slf4j
@Controller
public class RequestBodyJsonController {
  private ObjectMapper objectMapper = new ObjectMapper();
  @PostMapping("/request-body-json-v1")
  public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
    ServletInputStream inputStream = request.getInputStream();
    String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
    log.info("messageBody={}", messageBody);
    HelloData data = objectMapper.readValue(messageBody, HelloData.class);
 log.info("username={}, age={}", data.getUsername(), data.getAge());
 response.getWriter().write("ok");
 }
}
```

  
```java

```
  
```java

```
  
```java

```
