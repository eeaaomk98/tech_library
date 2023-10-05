# 스프링 MVC - 기본 기능

----------------------



## 로깅

운영 시스템에서는 `System.our.println()` 같은 시스템 콘솔을 사용해서 필요한 정보를 출력하지 않고, 별도의 로깅 라이브러리를 사용해서 로그를 출력한다.

<br>

### 로깅 라이브러리

스프링 부트 라이브러리를 사용하면 스프링 부트 로깅 라이브러리 (`spring-boot-starter-logging`)가 함께 포함된다.

스프링 부트 로깅 라이브러리는 기본으로 다음 로깅 라이브러리를 사용한다.

- SLF4J - http://www.slf4j.org
- Logback - http://logback.qos.ch

<br>

### 로그 선언

- `private Logger log = LoggerFactory.getLogger(getClass());`
- `private static final Logger log = LoggerFactory.getLogger(Xxx.class)`
- `Slf4j` : 롬복 사용 가능

<br>

### 로그 호출

- `log.info("hello")`

```java
@Slf4j
@RestController // 반환 값으로 뷰를 찾는 것이 아니라 HTTP 메시지 바디에 바로 입력.
public class LogTestController {
//    private final Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping("/log-test")
    public String logTest() {
        String name = "Spring";

        System.out.println("name = " + name);

        log.trace("trace log={}", name);
        log.debug("debug log={}", name);
        log.info ("info  log={}", name);
        log.warn ("warn  log={}", name);
        log.error("error log ={}", name);

        return "ok";
}
```

<img src="../../eeaaomk98.github.io/images/스프링 MVC - 기본 기능/image-20231005135552838.png" alt="image-20231005135552838" style="zoom:80%;" />

<br>

### 로그 레벨 설정

`application.properties`에서 로그 레벨을 설정할 수 있다.

```properties
# 전체 로그 레벨 설정(기본 info)
logging.level.root=info

#hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug
```

- LEVEL : `TRACE > DEBUG > INFO > WARN > ERROR`

<br>

----------



## 메서드 매핑

`@RequestMapping`에 `method` 속성으로 HTTP 메서드를 지정하지 않으면 HTTP 메서드와 무관하게 호출된다.

<br>

### HTTP 메서드 매핑

```java
@RequestMapping(value = "/mapping-v1", method = RequestMethod.GET)
public String mappingV1() {
  ...
}
```

```java
// 축약 애노테이션 - @GetMapping, @PostMapping 등등 모두 가능
@GetMapping(value = "/mapping-v2")
public String mappingV2() {
  ...
}
```

<br>

### PathVariable(경로 변수) 사용

```java
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data) {
  log.info("mappingPath userId={}", data);
  return "ok"
}
```

- `@PathVariable`의 이름과 파라미터 이름이 같으면 생략할 수 있다.

<br>

## HTTP 요청 - 기본, 헤더 조회

HTTP 헤더 정보를 조회하는 방법을 알아보자.

```java
@Slf4j  // 로그 선언
@RestController  // 반환 값을 HTTP 메시지 바디에 바로 입력.
public class RequestHeaderController {

    @RequestMapping("/headers")
    public String headers(HttpServletRequest request,
                          HttpServletResponse response,
                          HttpMethod httpMethod, // HTTP 메서드 조회
                          Locale locale,  // Locale 정보
                          @RequestHeader MultiValueMap<String, String> headerMap, // 모든 헤더를 MultiValueMap 형식으로 조회. 하나의 키에 여러 값을 받음
                          @RequestHeader ("host") String host, // 특정 HTTP 헤더 조회
                          @CookieValue(value = "myCookie", required = false) String cookie){ // 특정 쿠키 조회 

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

<img src="../../eeaaomk98.github.io/images/스프링 MVC - 기본 기능/image-20231005141840622.png" alt="image-20231005141840622" style="zoom: 67%;" />

<br>

## HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form

클라이언트에서 서버로 요청 데이터를 전달할 때는 주로 3가지 방법을 사용한다.

- **GET - 쿼리 파라미터**, **POST - HTML Form**

    - **request.getParameter()**

        - ```java
            @RequestMapping("/request-param-v1")
            public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
              
              String username = request.getParameter("username");
              int age = Integer.parseInt(request.getParameter("age"));
              
              response.getWriter().write("ok");
            }
            ```

            

    - **@RequestParam** - 파라미터 이름으로 바인딩. 

        - ```java
            @ResponseBody
            @RequestMapping("/request-param-v2")
            public String requestParamV2 (
              			// HTTP 파라미터 이름과 변수 이름이 같으면 ("username"), ("age") 부분 생략 가능
            				@RequestParam("username") String memberName,
            				@RequestParam("age") int memberAge) {
            				
              return "ok";				
            }
            ```

        - ```java
            // (required = true) 이면 파라미터가 필수로 들어가야 함. 기본값이 true
            // defaultValue를 사용하면 파라미터에 값이 없는 경우 기본 값을 적용할 수 있다.
            @RequestParam(required = true, defaultValue = "guest") String username,
            @RequestParam(required = false, defaultValue = "-1") Integer age) 
            ```

        - ```java
            // Map으로 파라미터 조회하기
            @ResponseBody
            @RequestMapping("/request-param-map")
            public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
              log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));
              return "ok";
            }
            ```

        <br>

    - **@ModelAttribute**

        - ```java
            // HelloData 클래스를 만들었다고 가정
            @ResponseBody
            @RequestMapping("/model-attribute-v1")
            public String modelAttributeV1(@ModelAttribute HelloData helloData) {
              log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
              return "ok";
            }
            ```

            - `@ModelAttribute`는 생략할 수 있다.
            - 생략 시 `String, int, Integer`같은 단순 타입 = `@RequestParam`, 나머지 = `@ModelAttribute`

        <br>

- **HTTP message body**에 데이터를 직접 담아 요청

    - **단순 텍스트** - InputStream을 사용해 읽기

        - ```java
            // HTTP 메시지 바디의 데이터를 InputStream을 사용해 직접 읽을 수 있음. 
            @PostMapping("/request-body-string-v1")
            public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
                      ServletInputStream inputStream = request.getInputStream();
                      String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
            
            	log.info("messageBody={}", messageBody);
            
            	response.getWriter().write("ok");
            }
            ```

    - `@RequestBody` - HTTP 메시지 바디 정보를 편리하게 조회할 수 있음.

        - ```java
            @ResponseBody  // 응답 결과를 HTTP 메시지 바디에 직접 담아 전달
            @PostMapping("/request-body-string-v4")
            public String requestBodyStringV4(@RequestBody String messageBody) {
            	log.info("messageBody={}", messageBody);
            	return "ok";
            }
            ```

        - 헤더 정보가 필요하다면 `HttpEntity`혹은 `@RequestHeader`를 사용하면 된다.

        

    - **JSON 데이터 형식 조회**

        - ```java
            // HttpServletRequest를 사용해서 바디에서 데이터를 가져와서 문자로 변환한다. 
            // 문자로 된 JSON 데이터를 objectMapper를 사용해서 자바 객체로 변환
            @Slf4j
            @Controller
            public class RequestBodyJsonController {
              
            	private ObjectMapper objectMapper = new ObjectMapper();
            
              @PostMapping("/request-body-json-v1")
            
              public void requestBodyJsonV1(HttpServletRequest request,
            HttpServletResponse response) throws IOException {
            
                ServletInputStream inputStream = request.getInputStream();
            
                String messageBody = StreamUtils.copyToString(inputStream,
            StandardCharsets.UTF_8);
            
                log.info("messageBody={}", messageBody);
                HelloData data = objectMapper.readValue(messageBody, HelloData.class);
                log.info("username={}, age={}", data.getUsername(), data.getAge());
            
                response.getWriter().write("ok");
            }
            ```

            

        - ```java
            // HttpEntity, @RequestBody를 사용하면 HTTP 메시지 바디의 내용을 문자나 객체 등으로 변환해줌.
            @ResponseBody
            @PostMapping("/request-body-json-v3")
            public String requestBodyJsonV3(@RequestBody HelloData data) {
            	log.info("username={}, age={}", data.getUsername(), data.getAge());
            	return "ok";
            }
            ```

    <br>

## HTTP 응답 - 정적 리소스, 뷰 템플릿

스프링(서버)에서 응답 데이터를 만드는 방법은 크게 3가지이다.

- **정적 리소스** -웹 브라우저에 정적인 HTML, css, js를 제공할 때
    - `src/main/resources`는 리소스를 보관하는 곳이고, 클래스패스의 시작 경로이다. 
    - 정적 리소스 경로 - `src/main/resources/static`
    - 정적 리소스는 해당 파일을 변경 없이 그대로 서비스하는 것.



- **뷰 템플릿 사용** - 웹 브라우저에 동적인 HTML을 제공할 때
    - 뷰 템플릿을 거쳐 HTML이 생성되고, 뷰가 응답을 만들어 전달한다. 일반적으로  HTML을 동적으로 생성하는 용도로 사용
    - 뷰 템플릿 경로 - `src/main/resources/templates`
    - **String을 반환하는 경우** - View or HTTP 메시지
        - `@ResponseBody`가 없으면 뷰 리졸버가 실행되어 뷰를 찾고, 렌더링
        - `@ResponseBody`가 있으면 HTTP 메시지 바디에 직접 문자 입력



- **HTTP 메시지 사용**
    - HTTP API를 제공하는 경우에는 HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다.
    - `@ResponseBody`, `@HttpEntity`를 사용하면, HTTP 메시지 바디에 직접 응답 데이터를 출력한다.

<br>



























