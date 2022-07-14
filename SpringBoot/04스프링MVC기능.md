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
HttpServletRequest를 사용해서 직접 HTTP 메시지 바디에서 데이터를 읽어와서, 문자로 변환한다.
  
문자로 된 JSON 데이터를 Jackson 라이브러리인 objectMapper 를 사용해서 자바 객체로 변환.
  
```java
/**
 * @RequestBody
 * HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 *
 * @ResponseBody
 * - 모든 메서드에 @ResponseBody 적용
 * - 메시지 바디 정보 직접 반환(view 조회X)
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용
 */
@ResponseBody
@PostMapping("/request-body-json-v2")
public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {
  HelloData data = objectMapper.readValue(messageBody, HelloData.class);
  log.info("username={}, age={}", data.getUsername(), data.getAge());
  return "ok";
}
```
  
@RequestBody 를 사용해서 HTTP 메시지에서 데이터를 꺼내고 messageBody에 저장한다.
문자로 된 JSON 데이터인 messageBody 를 objectMapper 를 통해서 자바 객체로 변환한다.
  
  
```java
/**
 * @RequestBody 생략 불가능(@ModelAttribute 가 적용되어 버림)
 * HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter (contenttype: application/json)
 *
 */
@ResponseBody
@PostMapping("/request-body-json-v3")
public String requestBodyJsonV3(@RequestBody HelloData data) {
  log.info("username={}, age={}", data.getUsername(), data.getAge());
  return "ok";
}
```
@RequestBody 객체 파라미터 : @RequestBody 에 직접 만든 객체를 지정할 수 있다
  
HttpEntity , @RequestBody 를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
HTTP 메시지 컨버터는 문자 뿐만 아니라 JSON도 객체로 변환해주는데, 우리가 방금 V2에서 했던 작업을 대신 처리해준다.
  
@RequestBody는 생략 불가능.<BR>
스프링은 @ModelAttribute , @RequestParam 과 같은 해당 애노테이션을 생략시 다음과 같은 규칙을 적용한다.
- String , int , Integer 같은 단순 타입 = @RequestParam
- 나머지 = @ModelAttribute (argument resolver 로 지정해둔 타입 외)<br>
따라서 이 경우 HelloData에 @RequestBody 를 생략하면 @ModelAttribute 가 적용. 생략하면 HTTP 메시지 바디가 아니라 요청 파라미터를 처리하게 된다.
  
```java
@ResponseBody
@PostMapping("/request-body-json-v4")
public String requestBodyJsonV4(HttpEntity<HelloData> httpEntity) {
  HelloData data = httpEntity.getBody();
  log.info("username={}, age={}", data.getUsername(), data.getAge());
  return "ok";
}
```
HttpEntity를 사용 가능.
  
```java
/**
 * @RequestBody 생략 불가능(@ModelAttribute 가 적용되어 버림)
 * HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter (contenttype: application/json)
 *
 * @ResponseBody 적용
 * - 메시지 바디 정보 직접 반환(view 조회X)
 * - HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter 적용
(Accept: application/json)
 */
@ResponseBody
@PostMapping("/request-body-json-v5")
public HelloData requestBodyJsonV5(@RequestBody HelloData data) {
  log.info("username={}, age={}", data.getUsername(), data.getAge());
  return data;
}
```
응답의 경우에도 @ResponseBody 를 사용하면 해당 객체를 HTTP 메시지 바디에 직접 넣어줄 수 있다. 물론 이 경우에도 HttpEntity 를 사용해도 된다.
  
@RequestBody 요청 : JSON 요청 HTTP 메시지 컨버터 객체
@ResponseBody 응답 : 객체 HTTP 메시지 컨버터 JSON 응답
  

# HTTP 응답 - 정적 리소스, 뷰 템플릿
  
스프링(서버)에서 응답 데이터를 만드는 방법은 크게 3가지
  
1. 정적 리소스<br>
예) 웹 브라우저에 정적인 HTML, css, js를 제공할 때는, 정적 리소스를 사용한다.
  
2. 뷰 템플릿 사용<br>
예) 웹 브라우저에 동적인 HTML을 제공할 때는 뷰 템플릿을 사용한다.
  
3. HTTP 메시지 사용<br>
HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다.
  
  
### 정적 리소스
  
스프링 부트는 클래스패스의 다음 디렉토리에 있는 정적 리소스를 제공한다. : /static , /public , /resources , /META-INF/resources  
  
src/main/resources 는 리소스를 보관하는 곳이고, 또 클래스패스의 시작 경로이다. 따라서 다음 디렉토리에 리소스를 넣어두면 스프링 부트가 정적 리소스로 서비스를 제공
  
정적 리소스는 해당 파일을 변경 없이 그대로 서비스하는 것
  
  
### 뷰 템플릿
  
뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달한다. 일반적으로 HTML을 동적으로 생성하는 용도로 사용하지만, 다른 것들도 가능하다. 뷰 템플릿이 만들 수 있는 것이라면 뭐든지 가능.

뷰 템플릿 경로 : src/main/resources/templates
  
```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
  
@Controller
public class ResponseViewController {
  
 @RequestMapping("/response-view-v1")
 public ModelAndView responseViewV1() {
  ModelAndView mav = new ModelAndView("response/hello").addObject("data", "hello!");
  return mav;
 }
  
 @RequestMapping("/response-view-v2")
 public String responseViewV2(Model model) {
  model.addAttribute("data", "hello!!");
  return "response/hello";
 }
  
 @RequestMapping("/response/hello")
 public void responseViewV3(Model model) {
  model.addAttribute("data", "hello!!");
 }
}
```
 
### String을 반환하는 경우 - View or HTTP 메시지
@ResponseBody 가 없으면 response/hello로 뷰 리졸버가 실행되어서 뷰를 찾고, 렌더링 한다.<br>
@ResponseBody 가 있으면 뷰 리졸버를 실행하지 않고, HTTP 메시지 바디에 직접 response/hello라는 문자가 입력된다. 
  
### Void를 반환하는 경우
@Controller를 사용하고, HttpServletResponse , OutputStream(Writer) 같은 HTTP 메시지 바디를 처리하는 파라미터가 없으면 요청 URL을 참고해서 논리 뷰 이름으로 사용<br>
이 방식은 명시성이 너무 떨어지고 이렇게 딱 맞는 경우도 많이 없어서, 권장하지 않는다.

HTTP 메시지 : @ResponseBody , HttpEntity 를 사용하면, 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 응답 데이터를 출력할 수 있다.
  
  
# HTTP 응답 - HTTP API, 메시지 바디에 직접 입력
  
HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다.
  
HTML이나 뷰 템플릿을 사용해도 HTTP 응답 메시지 바디에 HTML 데이터가 담겨서 전달된다. 여기서 설명하는 내용은 정적 리소스나 뷰 템플릿을 거치지 않고, 직접 HTTP 응답 메시지를 전달하는 경우를 말한다.
  
  
```java
import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
  
@Slf4j
@Controller
//@RestController
public class ResponseBodyController {
  
 @GetMapping("/response-body-string-v1")
 public void responseBodyV1(HttpServletResponse response) throws IOException {
  response.getWriter().write("ok");
 }
  
 /**
 * HttpEntity, ResponseEntity(Http Status 추가)
 * @return
 */
 @GetMapping("/response-body-string-v2")
 public ResponseEntity<String> responseBodyV2() {
  return new ResponseEntity<>("ok", HttpStatus.OK);
 }
  
 @ResponseBody
 @GetMapping("/response-body-string-v3")
 public String responseBodyV3() {
  return "ok";
 }
  
 @GetMapping("/response-body-json-v1")
 public ResponseEntity<HelloData> responseBodyJsonV1() {
  HelloData helloData = new HelloData();
  helloData.setUsername("userA");
  helloData.setAge(20);
  return new ResponseEntity<>(helloData, HttpStatus.OK);
 }
  
 @ResponseStatus(HttpStatus.OK)
 @ResponseBody
 @GetMapping("/response-body-json-v2")
 public HelloData responseBodyJsonV2() {
  HelloData helloData = new HelloData();
  helloData.setUsername("userA");
  helloData.setAge(20);
  return helloData;
 }
}
```
  
### responseBodyV1
서블릿을 직접 다룰 때 처럼 HttpServletResponse 객체를 통해서 HTTP 메시지 바디에 직접 ok 응답 메시지를 전달한다.

### responseBodyV2
ResponseEntity 엔티티는 HttpEntity 를 상속 받았는데, HttpEntity는 HTTP 메시지의 헤더, 바디 정보를 가지고 있다. ResponseEntity 는 여기에 더해서 HTTP 응답 코드를 설정할 수 있다.
HttpStatus.CREATED 로 변경하면 201 응답이 나가는 것을 확인할 수 있다.
  
### responseBodyV3
@ResponseBody 를 사용하면 view를 사용하지 않고, HTTP 메시지 컨버터를 통해서 HTTP 메시지를 직접 입력할 수 있다. ResponseEntity 도 동일한 방식으로 동작한다.
  
### responseBodyJsonV1
ResponseEntity 를 반환한다. HTTP 메시지 컨버터를 통해서 JSON 형식으로 변환되어서 반환된다.

### responseBodyJsonV2
ResponseEntity 는 HTTP 응답 코드를 설정할 수 있는데, @ResponseBody 를 사용하면 이런 것을 설정하기 까다롭다.
@ResponseStatus(HttpStatus.OK) 애노테이션을 사용하면 응답 코드도 설정할 수 있다. 물론 애노테이션이기 때문에 응답 코드를 동적으로 변경할 수는 없다. 프로그램 조건에 따라서 동적으로
변경하려면 ResponseEntity 를 사용하면 된다.
  
### @RestController
@Controller 대신에 @RestController 애노테이션을 사용하면, 해당 컨트롤러에 모두 @ResponseBody가 적용되는 효과가 있다. 따라서 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에
직접 데이터를 입력한다. 이름 그대로 Rest API(HTTP API)를 만들 때 사용하는 컨트롤러이다.
참고로 @ResponseBody는 클래스 레벨에 두면 전체 메서드에 적용되는데, @RestController 에노테이션 안에 @ResponseBody가 적용되어 있다.
  

# HTTP 메시지 컨버터
  
뷰 템플릿으로 HTML을 생성해서 응답하는 것이 아니라, HTTP API처럼 JSON 데이터를 HTTP 메시지 바디에서 직접 읽거나 쓰는 경우 HTTP 메시지 컨버터를 사용하면 편리.
  
![image](https://user-images.githubusercontent.com/65898555/178420576-7c413f24-549d-4bf2-849b-f092c3ed1bee.png)

@ResponseBody 를 사용
- HTTP의 BODY에 문자 내용을 직접 반환
- viewResolver 대신에 HttpMessageConverter 가 동작
- 기본 문자처리: StringHttpMessageConverter
- 기본 객체처리: MappingJackson2HttpMessageConverter
- byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음
  
응답의 경우 클라이언트의 HTTP Accept 해더와 서버의 컨트롤러 반환 타입 정보 둘을 조합해서 HttpMessageConverter가 선택
  
스프링 MVC는 다음의 경우에 HTTP 메시지 컨버터를 적용
- HTTP 요청: @RequestBody , HttpEntity(RequestEntity)
- HTTP 응답: @ResponseBody , HttpEntity(ResponseEntity)
  
HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용된다.
- canRead() , canWrite() : 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크
- read() , write() : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능
  
### HTTP 요청 데이터 읽기
1. HTTP 요청이 오고, 컨트롤러에서 @RequestBody , HttpEntity 파라미터를 사용한다.<br>
2. 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 canRead() 를 호출한다.<br>
2-1. 대상 클래스 타입을 지원하는가. 예) @RequestBody 의 대상 클래스 ( byte[] , String , HelloData )<br>
2-2. HTTP 요청의 Content-Type 미디어 타입을 지원하는가. 예) text/plain , application/json , */*<br>
3. canRead() 조건을 만족하면 read() 를 호출해서 객체 생성하고, 반환한다<br>
  
### HTTP 응답 데이터 생성
1. 컨트롤러에서 @ResponseBody , HttpEntity 로 값이 반환된다. <br>
2. 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 canWrite() 를 호출한다.<br>
2-1. 대상 클래스 타입을 지원하는가. 예) return의 대상 클래스 ( byte[] , String , HelloData )<br>
2-2. HTTP 요청의 Accept 미디어 타입을 지원하는가.(더 정확히는 @RequestMapping 의 produces ) 예) text/plain , application/json , */*<br>
3. canWrite() 조건을 만족하면 write() 를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다. <br>
  
![image](https://user-images.githubusercontent.com/65898555/178421464-3efd6c72-eb78-4850-bdfb-314aa5186810.png)
![image](https://user-images.githubusercontent.com/65898555/178421497-9732eab9-a83f-4964-b7e4-4c22c937b6de.png)
  
RequestMappingHandlerAdapter 동작 방식
  
애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할 수 있었다.
  
HttpServletRequest , Model 은 물론이고, @RequestParam , @ModelAttribute 같은 애노테이션 그리고 @RequestBody , HttpEntity 같은 HTTP 메시지를 처리하는 부분까지 매우 큰 유연함을보여주었다. 이렇게 파라미터를 유연하게 처리할 수 있는 이유가 바로 ArgumentResolver 덕분이다.
  
애노테이션 기반 컨트롤러를 처리하는 RequestMappingHandlerAdapter는 바로 이ArgumentResolver를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성.
  
그리고 이렇게 파리미터의 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨준다.
  
스프링은 30개가 넘는 ArgumentResolver 를 기본으로 제공한다.

ArgumentResolver 의 supportsParameter()를 호출해서 해당 파라미터를 지원하는지 체크하고, 지원하면 resolveArgument() 를 호출해서 실제 객체를 생성한다. 그리고 이렇게 생성된 객체가 컨트롤러 호출시 넘어가는 것이다.

HandlerMethodReturnValueHandler 를 줄여서 ReturnValueHandler라 부른다.
ArgumentResolver 와 비슷한데, 이것은 응답 값을 변환하고 처리한다.
컨트롤러에서 String으로 뷰 이름을 반환해도, 동작하는 이유가 바로 ReturnValueHandler 덕분이다.
  
스프링은 10여개가 넘는 ReturnValueHandler 를 지원한다. 예) ModelAndView , @ResponseBody , HttpEntity , String

![image](https://user-images.githubusercontent.com/65898555/178422043-870ed43e-72d7-47c5-a819-9b41bf4e44c3.png)

HTTP 메시지 컨버터를 사용하는 @RequestBody 도 컨트롤러가 필요로 하는 파라미터의 값에 사용된다.
  
@ResponseBody의 경우도 컨트롤러의 반환 값을 이용한다. 

요청의 경우 @RequestBody 를 처리하는 ArgumentResolver 가 있고, HttpEntity 를 처리하는ArgumentResolver가 있다. 
이 ArgumentResolver들이 HTTP 메시지 컨버터를 사용해서 필요한객체를 생성하는 것이다.
  
응답의 경우 @ResponseBody 와 HttpEntity 를 처리하는 ReturnValueHandler가 있다. 그리고 여기에서 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.  
  
스프링 MVC는 @RequestBody @ResponseBody가 있으면RequestResponseBodyMethodProcessor (ArgumentResolver) / HttpEntity가 있으면 HttpEntityMethodProcessor (ArgumentResolver)를 사용. 
  
  
# ModelAttribute 추가 기능
  
```java
/**
 * @ModelAttribute("item") Item item
 * model.addAttribute("item", item); 자동 추가
 */
@PostMapping("/add")
public String addItemV2(@ModelAttribute("item") Item item, Model model) {
  itemRepository.save(item);
  //model.addAttribute("item", item); //자동 추가, 생략 가능
  return "basic/item";
}
```
@ModelAttribute - Model 추가
  
@ModelAttribute 는 중요한 한가지 기능이 더 있는데, 바로 모델(Model)에 @ModelAttribute로 지정한 객체를 자동으로 넣어준다. 
지금 코드를 보면 model.addAttribute("item", item) 가 주석처리되어 있어도 잘 동작한다.
  
모델에 데이터를 담을 때는 이름이 필요하다. 이름은 @ModelAttribute 에 지정한 name(value) 속성을 사용한다. 만약 다음과 같이 @ModelAttribute 의 이름을 다르게 지정하면 다른 이름으로 모델에 포함된다.
  
- @ModelAttribute("hello") Item item 이름을 hello 로 지정
- model.addAttribute("hello", item); 모델에 hello 이름으로 저장
  
```java
/**
 * @ModelAttribute name 생략 가능
 * model.addAttribute(item); 자동 추가, 생략 가능
 * 생략시 model에 저장되는 name은 클래스명 첫글자만 소문자로 등록 Item -> item
 */
@PostMapping("/add")
public String addItemV3(@ModelAttribute Item item) {
  itemRepository.save(item);
  return "basic/item";
}
```
@ModelAttribute 의 이름을 생략하면 모델에 저장될 때 클래스명을 사용한다. 
이때 클래스의 첫글자만 소문자로 변경해서 등록한다.
  
예) @ModelAttribute 클래스명 모델에 자동 추가되는 이름 Item -> item / HelloWorld -> helloWorld

```java
/**
 * @ModelAttribute 자체 생략 가능
 * model.addAttribute(item) 자동 추가
 */
@PostMapping("/add")
public String addItemV4(Item item) {
  itemRepository.save(item);
  return "basic/item";
}
```
@ModelAttribute 자체도 생략가능하다. 대상 객체는 모델에 자동 등록된다. 나머지 사항은 기존과 동일하다.

  
# 리다이렉트
  
스프링은 redirect:/... 으로 편리하게 리다이렉트를 지원한다.
  
컨트롤러에 매핑된 @PathVariable 의 값은 redirect 에도 사용 할 수 있다. ex) redirect:/basic/items/{itemId}
{itemId}는 @PathVariable Long itemId 의 값을 그대로 사용한다.
  
  
HTML Form 전송은 PUT, PATCH를 지원하지 않는다. GET, POST만 사용할 수 있다. > PUT, PATCH는 HTTP API 전송시에 사용
  
스프링에서 HTTP POST로 Form 요청할 때 히든 필드를 통해서 PUT, PATCH 매핑을 사용하는 방법이 있지만, HTTP 요청상 POST 요청이다.
  
![image](https://user-images.githubusercontent.com/65898555/178660974-47b7d8f6-fe0e-4dad-9089-a08c49d2fc2c.png)

브라우저의 새로 고침은 마지막에 서버에 전송한 데이터를 다시 전송한다. 상품 등록 폼에서 데이터를 입력하고 저장을 선택하면 POST /add + 상품 데이터를 서버로 전송한다.
이 상태에서 새로 고침을 또 선택하면 마지막에 전송한 POST /add + 상품 데이터를 서버로 다시 전송하게 된다. 그래서 내용은 같고, ID만 다른 상품 데이터가 계속 쌓이게 된다.
  
![image](https://user-images.githubusercontent.com/65898555/178661072-dea25ede-52a6-44ff-99e8-69e92b22e93d.png)

웹 브라우저의 새로 고침은 마지막에 서버에 전송한 데이터를 다시 전송한다. 새로 고침 문제를 해결하려면 상품 저장 후에 뷰 템플릿으로 이동하는 것이 아니라, 상품 상세 화면으로
리다이렉트를 호출해주면 된다. 웹 브라우저는 리다이렉트의 영향으로 상품 저장 후에 실제 상품 상세 화면으로 다시 이동한다. 따라서 마지막에 호출한 내용이 상품 상세 화면인 GET /items/{id} 가 되는 것이다. 이후 새로고침을 해도 상품 상세 화면으로 이동하게 되므로 새로 고침 문제를 해결할 수 있다.
  
이런 문제 해결 방식을 PRG Post/Redirect/Get라 한다.
  
"redirect:/basic/items/" + item.getId() redirect에서 +item.getId() 처럼 URL에 변수를 더해서 사용하는 것은 URL 인코딩이 안되기 때문에 위험하다.  
RedirectAttributes를 사용
  
# RedirectAttributes
  
```java
/**
 * RedirectAttributes
 */
@PostMapping("/add")
public String addItemV6(Item item, RedirectAttributes redirectAttributes) {
  Item savedItem = itemRepository.save(item);
  redirectAttributes.addAttribute("itemId", savedItem.getId());
  redirectAttributes.addAttribute("status", true);
  return "redirect:/basic/items/{itemId}";
}
```
```html
<div class="container">
 <div class="py-5 text-center">
 <h2>상품 상세</h2>
 </div>
 <!-- 추가 -->
 <h2 th:if="${param.status}" th:text="'저장 완료!'"></h2>
```
리다이렉트 할 때 간단히 status=true 를 추가한다. 그리고 뷰 템플릿에서 이 값이 있으면, 저장되었습니다. 라는 메시지를 출력한다.
실행해보면 다음과 같은 리다이렉트 결과가 나온다. http://localhost:8080/basic/items/3?status=true

RedirectAttributes 를 사용하면 URL 인코딩도 해주고, pathVarible, 쿼리 파라미터까지 처리해준다.

${param.status} : 타임리프에서 쿼리 파라미터를 편리하게 조회하는 기능으로 원래는 컨트롤러에서 모델에 직접 담고 값을 꺼내야 한다. 그런데 쿼리 파라미터는 자주 사용해서 타임리프에서 직접 지원한다.

# @ModelAttribute의 특별한 사용법
  
```java
@ModelAttribute("regions")
public Map<String, String> regions() {
 Map<String, String> regions = new LinkedHashMap<>();
 regions.put("SEOUL", "서울");
 regions.put("BUSAN", "부산");
 regions.put("JEJU", "제주");
 return regions;
}
```
@ModelAttribute는 컨트롤러에 있는 별도의 메서드에 적용할 수 있다. 이렇게하면 해당 컨트롤러를 요청할 때 regions 에서 반환한 값이 자동으로 모델( model )에 담기게 된다.
