# /24-07-22
## 프로젝트 생성 

# /24-07-23
## 로깅 간단히 알아보기
앞으로 로그를 사용할 것이기 때문에, 이번 시간에는 로그에 대해서 간단히 알아보자

운영 시스템에서는 System.out.println() 같은 시스템 콘솔을 사용해서 필요한 정보를 출력하지 않고, 별도의 로깅 라이브러리를 사용해서 로그를 출력한다.
참고로 로그 관련 라이브러리도 많고, 깊게 들어가면 끝이 없기 때문에, 여기서는 최소한의 사용 방법만 알아본다.

### 로깅 라이브러리
스프링 부트 라이브러리를 사용하면 스프링 부트 로깅 라이브러리 (spring-boot-starter-logging)가 함께 포함된다.
스프링 부트 로깅 라이브러리는 기본으로 다음 로깅 라이브러리를 사용한다.

- SLF4J
- Logback

로그 라이브러리는 Logback, Log4J, Log4J2 등등 수 많은 라이브러리가 있는데, 그것을 통합해서 인터페이스로 제공하는 것이 바로 SLF4J 라이브러리다.
쉽게 이야기해서 SLF4J는 인터페이스이고, 그 구현체로 Logback같은 로그 라이브러리를 선택하면 된다.
실무에서는 스프링 부트가 기본으로 제공하는 Logback을 대부분 사용한다.

### 로그 선언 
- private Logger log = LoggerFactory.getLogger(getClass());
- private static final Logger log = LoggerFactory.getLogger(Xxx.class)
- @Slf4j : 롬복 사용가능

### 로그 호출 
- log.info("hello")
시스템 콘솔로 직접 출력하는것 보다 로그를 사용하면 다음과 같은 장점이 있다. 실무에서는 항상 로그를 사용해야한다.

#### 매핑 정보
- @RestController
  - @Controller는 반환 값이 String이면 뷰 이름으로 인식된다. 그래서 뷰를 찾고 뷰가 랜더링 된다.
  - @RestController는 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력 한다.
    따라서 실행 결과로 ok 메시지를 받을 수 있다. @ResponseBody와 관련이 있는데 뒤에서 더 자세히 설명한다.

#### 테스트
- 로그가 출력되는 포맷확인
  - 시간, 로그 레벨, 프로세스 ID, 쓰레드 명, 클래스명, 로그메시지
- 로그 레벨 설정을 변경해서 출력 결과를 보자
  - LEVEL : TRACE > DEBUG > INFO > WARN > ERROR
  - 개발 서버는 debug 출력
  - 운영 서버는 info 출력
- @Slf4j로 변경

#### 로그 레벨 설정
application.properties
# 전체 로그 레벨 설정(기본 info)
logging.level.root=info
# hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug

#### 올바른 로그 사용법
- log.debug("data="+data)
  - 로그 출력 레벨을 info로 설정해도 해당 코드에 이쓴 "data="+data가 실제 실행이 되어 버린다. 결과적으로 문자 더하기 연산이 발생한다.
    연산이 발생되면 그에따른 메모리나 CPU를 사용하는데 쓸모없는 리소스낭비가 발생된다.
- log.debug("data={},data)
  - 로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다. 따라서 앞과 같은 의미없는 연산이 발생하지 않는다.

#### 로그 사용시 장점
- 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다.
- 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영서버에서는 출력하지 않는 등 로그를 상황에 맞게 조절할 수 있다.
- 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등, 로그를 별도의 위치에 남길 수 있다.
  특히 파일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다.
- 성능도 일반 System.out보다 좋다. (내부 버퍼링, 멀티 쓰레드 등등) 그래서 실무에서는 꼭 로그를 사용해야 한다.

# /24-07-24
## 요청 매핑

### 둘다 허용
다음 두가지 요청은 다른 URL이지만, 스프링은 다음 URL 요청들을 같은 요청으로 매핑한다.
- 매핑 : /hello-basic
- URL 요청 : /hello-basic, /hello-baisc/

### HTTP메서드
@RequestMapping에 method 속성으로 HTTP메서드를 지정하지 않으면 HTTP 메서드와 무관하게 호출된다.
모두 허용 GET, HEAD, POST, PUT, PATCH, DELETE

### PathVariable(경로 변수) 사용
최근 HTTP API는 다음과 같이 리소스 경로에 식별자를 넣는 스타일을 선호한다.
- /mapping/userA
- /users/1

- @RequestMapping은 URL 경로를 템플릿화 할 수 있는데, @PathVariable을 사용하면 매칭 되는 부분을 편리하게 조회할 수 있다.
- @PathVariable의 이름과 파라미터 이름이 같으면 생략할 수 있다.

### PathVariable 사용 - 다중

# /24-07-25
## 요청 매핑 - API예시
회원 관ㄴ리를 HTTP API로 만든다 생각하고 매핑을 어떻게 하는지 알아보자
(시렞 데이터가 넘어가는 부분은 생략하고 URL 매핑만)

### 회원관리 API
- 회원 목록 조회 : GET    /users
- 회원 등록 :     POST   /users
- 회원 조회 :     GET    /users/{userId}
- 회원 수정 :     PATCH  /users/{userId}
- 회원 삭제 :     DELETE /users/{userId}
  
# /24-07-26
## HTTP 요청 - 기본 헤더 조회
애노테이션 기반의 스프링 컨트롤러는 다양한 파라미터를 지원하다.
이번 시간에는 HTTP 헤더 정보를 조회하는 방법을 알아보자

- HttpServletRequest
- HttpServletResponse
- HttpMethod: HTTP 메서드를 조회한다. org.springframework.http.HttpMethod
- Locale : Locale 정보를 조회한다.
- @RequestHeader MultiValueMap<String, String> headerMap
  - 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.
- @RequestHeader("host") String host
  - 특정 HTTP 헤더를 조회한다.
  - 속성
    - 필수 값 여부 : required
    - 기본 값 속성 : defaultValue
- @CookieValue(value="myCookie", required = false) String cookie
  - 특정 쿠키를 조회한다.
  - 속성
    - 필수 값 여부 : required
    - 기본 값 : defaultValue

- MultiValueMap
- Map과 유사한데, 하나의 키에 여러 값을 받을 수 있다.
- HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다.
    - keyA=value1&keyA=value2

### @Slf4j

# /24-07-27
## HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form

### HTTP 요청 데이터 조회 - 개요
서블릿에서 학습했던 HTTP 요청 데이터를 조회하는 방법을 다시 떠올려보자. 
그리고 서블릿으로 학습했던 내용을 스프링이 얼마나 깔끔하고 효율적으로 바꾸어주는지 알아보자.

HTTP 요청 메시지를 통해 클라이언트에서 서버로 데이터를 전달하는 방법을 알아보자

#### 클라이언트에서 서버로 요청 데이터를 전달할 떄는 주로 다음 3가지 방법을 사용한다.
- GET - 쿼리 파라미터
  - url ?username=hello&age=20
  - 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
  - 예) 검색, 필터, 페이징등에서 많이 사용하는 방식

- POST - HTML Form
  - content-type: applicantion/x-www-form-urlencoded
  - 메시지 바디에 쿼리 파라미터 형식으로 전달 username=hello%age=20
  - 예) 회원가입, 상품 주문, HTML Form 사용

- HTTP message body에 데이터를 직접 담아서 요청
  - HTTP API에서 주로 사용, JSON, XML, TEXT
  - 데이터 형식은 주로 JSON 사용
  - POST, PUT, PATCH

### 요청 파라미터 - 쿼리 파라미터, HTML Form
HttpServletRequest의 request.getParameter()를 사용하면 다음 두가지 요청 파라미터를 조회할 수 있다.

#### GET, 쿼리 파라미터 전송
예시
http://localhost:8080/request-param?username=hello&age=20

#### POST, HTML Form 전송
예시
GET 쿼리 파라미터 전송 방식이든, POST HTML Form 전송 방식이든 둘다 형식이 같으므로 구분없이 조회할 수 있다.
이것을 간단히 요청 파라미터(request parameter)조회라 한다.

# /24-07-29
## HTTP 요청 파라미터 - @RequestParam
스프링이 제공하는 @RequestPara을 사용하면 요청 파라미터를 매우 편리하게 사용할 수 있다.
- @RequestParam : 파라미터 이름으로 바인딩
- @ResponseBody : View 조회를 무시하고, HTTP message body에 직접 해당 내용을 입력

### @RequestParam의 name(value) 속성이 파라미터 이름으로 사용
- @RequestParam("username") String memberName
- -> request.getParameter("username")

### 참고
이렇게 애노테이션을 완전히 생략해도 되는데, 너무 없는것도 약간 과하다는 주관적인 생각이 있다.
@RequestParam이 있으면 명확하게 요청 파라미터에서 데이터를 읽는 다는 것을 알 수 있다.

### 파라미터 필수 여부 - reuqestParamRequired
- @RequestParam.required
  - 파라미터 필수 여부
  - 기본값이 파라미터 필수(true)이다
- /requred-param 요청
  - username이 없으므로 400 예외가 발생한다.

#### 주의
/request-param? username=
파라미터 이름만 있고 값이 없는 경우 -> 빈문자로 통과

#### 주의 - 기본형(primitive)에 null 입력
- /request-param 요청
- @RequestParam(required=false) int age

null을 int에 입력하는것은 불가능 (500예외 발생)
따라서 null을 받을 수 있는 Integer로 변경하거나, 또는 다음에 나오는 defaultValue 사용

### 기본 값 적용 - requestParamDefault
파라미터에 값이 없는 경우 defaultValue를 사용하면 기본 값을 적용할 수 있다.
이미 기본 값이 있기 때문에 required는 의미가 없다

defaultValue는 빈 문자의 경우에도 설정한 기본 값이 적용된다.
/reuqest-param?username=

### 파라미터를 Map으로 조회하기 - requestParamMap
파라미터를 Map, MultiValueMap으로 조회할 수 있다.

- @RequestParam Map,
  - Map(key=value)
- @RequestParam MultiValueMap
  - MultiValueMap(key=[value1, value2, ...] ex) (key=userIds, value=[id1,id2])

?userIds=id1&userIds=id2
파라미터의 값이 1개가 확실하다면 Map을 사용해도 되지만, 그렇지 않다면 MultiValueMap을 사용하자

# /24-07-29
## HTTP 요청 파라미터 - @ModelAttribute
실제 개발을 하면 요청 파라미터를 받아서 필요한 객체를 만들고 그 객체에 값을 넣어주어야 한다.
스프링은 이 과정을 완전히 자동화해주는 @ModelAttribute 기능을 제공한다.

- 롬복 @Data
  - @Getter, @Setter, @ToString, @EqualsAndHashCode, @RequiredArgsConstructor를 자동으로 적용해준다.

### @ModelAttribute 적용 - modelAttributeV1
스프링 MVC는 @ModelAttribute가 있으면 다음을 실행한다.
- HelloData 객체를 생성한다.
- 요청 파라미터의 이름으로 HelloData 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 setter를 호출해서 파라미터의 값을 입력(바인딩)한다.
- 예) 파라미터 이름이 username이면 setUsername() 메서드를 찾아서 호출함녀서 값을 입력한다.

### 프로퍼티
객체에 getUsername(), setUsername() 메서드가 있으면, 이 객체는 username이라는 프로퍼티를 가지고 있다.
username 프로퍼티의 값을 변경하면 setUsername()이 호출되고, 조회하면 getUsername()이 호출된다.

### 바인딩 오류 
age=abc처럼 숫자가 들어가야 할 곳에 문자를 넣으면 BindException이 발생한다.
이런 바인딩 오류를 처리하는 방법은 검증 부분에서 다룬다.

### @ModelAttribute 생략 - modelAttributeV2
@ModelAttribute는 생략할 수 있다.
그런데 @RequestParam도 생략할 수 있으니 혼란이 발생할 수 있다.

스프링은 해당 생략시 다음과 같은 규칙을 적용한다.
- String, int, Integer 같은 단순 타입 = @RequestParam
- 나머지 = @ModelAttribute(argument resolver로 지정해둔 타입 외)

# /24-07-30
## HTTP 요청 메시지 - 단순 텍스트

### Input, Output 스트림, Reader - requestBodyStringV2

#### 스프링 MVC는 다음 파라미터를 지원한다.
- InputStream(Reader) : HTTP 요청 메시지 바디의 내용을 직접 조회
- OutputStream(Writer) : HTTP 응답 메시지의 바디에 직접 결과 출력

### HttpEntity - requestBodyStringV3

#### 스프링 MVC는 다음 파라미터를 지원한다.
- HttpEntity : HTTPheader, body 정보를 편리하게 조회
  - 메시지 바디 정보를 직접 조회
  - 요청 파라미터를 조회하는 기능과 관계 없음 @RequestParam X, @ModelAttribute X
- HttpEntity는 응답에도 사용 가능
  - 메시지 바디 정보 직접 반환
  - 헤더 정보 포함 가능
  - view 조회 X

#### HttpEntity를 상속받은 다음 객체들도 같은 기능을 제공한다.
- RequestEntity
  - HttpMethod, url 정보가 추가, 요청에서 사용
- ResponseEntity
  - HTTP 상태 코드 설정 가능, 응답에서 사용
  - return new ResponseEntity<String>("Hello world", responseHeaders, HttpStatus.CREATED)

### 참고
스프링 MVC 내부에서 HTTP 메시지 바디를 읽어서 문자나 객체로 변환해서 전달해주는데, 이때 HTTP 메시지 컨버터(HttpMessageConverter)라는 기능을 사용한다.
이것은 조금 뒤에 HTTP 메시지 컨버터에서 자세히 설명한다.

### @RequestBody - requestBodyStringV4

#### @RequestBody
@RequestBody를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다. 
참고로 헤더 정보가 필요하다면 HttpEntity를 사용하거나 @RequestHeader를 사용하면 된다.
이렇게 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 @RequestParam, @ModelAttribute와는 전혀 관계가 없다.

#### 요청 파라미터 vs HTTP 메시지 바디
- 요청 파라미터를 조회하는 기능 : @RequestParam, @ModelAttribute
- HTTP 메시지 바디를 직접 조회하는 기능 : @RequestBody

#### @ResponseBody
@ResponseBody를 사용하면 응답 결과를 HTTP 메시지 바디에 직접 담아서 전달할 수 있다.
물론 이 경우에도 view를 사용하지 않는다.

# 24-07-31
## HTTP 요청 메시지 - JSON

### requestBodyJsonV3 - @RequestBody 객체 변환
#### @RequestBody 객체 파라미터 
- @RequestBody HelloData data
- @RequestBody에 직접 만든 객체를 지정할 수 있다.

HttpEntity, @RequestBody를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
HTTP 메시지 컨버터는 문자 뿐만 아니라 JSON도 객체로 변환해주는데, 우리가 방금 V2에서 했던 작업을 대신 처리해준다.
자세한 내용은 뒤에 HTTP 메시지 컨버터에서 다룬다.

@RequestBody는 생략 불가능
@ModelAttribute에서 학습한 내용을 떠올려보자.

스프링은 @ModelAttribute, @RequestParam 해당 생략 시 다음과 같은 규칙을 적용한다.
- String, int, Integer 같은 단순 타입 = @RequestParam
- 나머지 = @ModelAttribute (argument resolver로 지정해둔 타입 외)

따라서 이 경우 HelloData에 @RequestBody를 생략하면 @ModelAttribute가 적용되어 버린다.
HelloData data -> @ModelAttribute HelloData data
따라서 생략하면 HTTP 메시지 바디가 아니라 요청 파라미터를 처리하게 된다.

### 주의
HTTP 요청 시에 content-type이 application/json인지 꼭! 확인해야 한다. 
그래야 JSON을 처리할 수 있는 HTTP 메시지 컨버터가 실행된다.

#### @ResponseBOdy
응답의 경우에도 @ResponseBody를 사용하면 해당 객체를 HTTP 메시지 바디에 직접 넣어줄 수 있다.
물론 이 경우에도 HttpEntity를 사용해도 된다.

- @RequestBody 요청
  - JSON요청 -> HTTP 메시지 컨버터 -> 객체
- @ResponseBody 응답
  - 객체 -> HTTP 메시지 컨버터 -> JSON응답

# /24-08-01
## HTTP응답 - 정적 리소스, 뷰 템플릿 
응답 데이터는 이미 앞에서 일부 다룬 내용들이지만, 응답 부분에 초점을 맞추어서 정리해보자.
스프링(서버)에서 응답 데이터를 만드는 방법은 크게 3가지이다.

- 정적 리소스
  - 예) 웹 브라우저에 정적인 HTML, css, js를 제공할 때는, "정적 리소스"를 사용한다.
- 뷰 템플릿 사용
  - 예) 웹 브라우저에 동적인 HTML을 제공할 때는 뷰 템플릿을 사용한다.
- HTTP 메시지 사용
  - HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다.

### 정적 리소스
스프링 부트는 클래스패스의 다음 디렉토리에 있는 정적 리소스를 제공한다.
'/static', '/public', '/resouces', '/META-INF/resources'

'src/main/resources'는 리소스를 보관하는 곳이고, 또 클래스패스의 시작 경로이다.
따라서 다음 디렉토리에 리소스를 넣어두면 스프링 부트가 정적 리소스로 서비스를 제공한다.

#### 정적 리소스 경로
'src/main/resouces/static'

다음 경로에 파일이 들어있으면
'src/main/resources/static/basic/hello-form.html'

웹브라우저에서 다음과 같이 실행하면 된다.
'http://localhost:8080/basic/hello-form.html'

정적 리소스는 해당 파일을 변경 없이 그대로 서비스하는 것이다.

### 뷰 템플릿
뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달한다.
일반적으로 HTML을 동적으로 생성하는 용도로 사용하지만, 다른 것들도 가능하다.
뷰 템플릿이 만들 수 있는 것이라면 뭐든지 가능하다.

스프링 부트는 기본 뷰 템플릿 경로를 제공한다.

#### 뷰 템플릿 경로
'src/main/resources/templates'

#### String을 반환하는 경우 - View or HTTP 메시지
@ResponseBody가 없으면 response/hello 로 뷰 리졸버가 실행되어서 뷰를 찾고, 렌더링 한다.
@ResponseBody가 있으면 뷰 리졸버를 실행하지 않고, HTTP 메시지 바디에 직접 response/hello 라는 문자가 입력된다.

여기서는 뷰의 논리 이름인 response/hello 를 반환하면 다음 경로의 뷰 템플릿이 렌더링 되는 것을 확인할 수 있다.
- 실행 : templates/response/hello.html

#### void를 반환하는 경우
- @Controller를 사용하고, HttpServletResponse, OutputStream(Writer) 같은 HTTP 메시지 바디를 처리하는 파라미터가 없으면 요청 URL을 참고해서 논리 뷰 이름으로 사용
  - 요청 URL : /response/hello
  - 실행 : templates/response/hello.html
- 참고로 이 방식은 명시성이 너무 떨어지고 이렇게 딱 맞는 경우도 많이 없어서, 권장하지 않는다.

#### HTTP 메시지
@ResponseBody, HttpEntity를 사용하면, 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 응답 데이터를 출력할 수 있다.

### Thymeleaf 스프링 부트 설정
Thymeleaf 라이브러리를 추가하면
스프링 부트가 자동으로 ThymeleafViewResolver와 필요한 스프링 빈들을 등록한다.
그리고 기본 값 설정도 해준다.
application.properties
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html

# /24-08-02
## HTTP 응답 - HTTP APi, 메시지 바디에 직접 입력
HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에 JSON같은 형식으로 데이터를 실어 보낸다.
HTTP 요청에서 응답까지대부분 다루었으므로 이번시간에는 정리를 해보자.

### 참고
HTML이나 뷰 템플릿을 사용해도 HTTP 응답 메시지 바디에 HTML 데이터가 담겨서 전달된다.
여기서 설명하는 내용은 정적 리소스나 뷰 템플릿을 거치지 않고, 직접 HTTP 응답 메시지를 전달하는 경우를 말한다.

#### responseBodyV1
서블릿을 직접 다룰 때 처럼
HttpServletResponse 객체를 통해서 HTTP 메시지 바디에 직접 ok 응답 메시지를 전달한다.
response.getWriter().write("ok")

#### responseBodyV2
ResponseEntity 엔티티는 HttpEntity를 상속 받았는데, HttpEntity는 HTTP 메시지의 헤더, 바디 정보를 가지고 있다.
ResponseEntity는 여기에 더해서 HTTP 응답 코드를 설정할 수 있다.

#### responseBodyV3
@ResponseBody를 사용하면 view를 사용하지 않고, HTTP 메시지 컨버터를 통해서 HTTP 메시지를 직접 입력할 수 있다.
ResponseEntity도 동일한 방식으로 동작한다.

#### responseBodyJsonV1
ResponseEntity를 반환한다. HTTP 메시지 컨버터를 통해서 JSON 형식으로 반환되어서 반환된다.

#### responseBodyJsonV2
ResponseEntity는 HTTP 응답 코드를 설정할 수 있는데, @ResponseBOdy를 사용하면 이런 것을 설정하기 까다롭다.
@ResponseStatus(HttpStatus.OK) 애노테이션을 사용하면 응답 코드도 설정할 수 있다.

물론 애노테이션이기 때문에 응답 코드를 동적으로 변경할 수 없다.
프로그램 조건에 따라서 동적으로 변경하려면 ResponseEntity를 사용하면 된다.

#### @RestController
@Controller 대신에 @RestController 애노테이션을 사용하며느, 해당 컨트롤러에 모두 @ResponseBody가 적용되는 효과가 있다.
따라서 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 데이터를 입력한다.
이름 그대로 RestAPI(HTTP API)를 만들 때 사용하는 컨트롤러이다.

참고로 @ResponseBody는 클래스 레벨에 두면 전체 메서드에 적용되는데, @RestController 애노테이션 안에 @ResponseBody가 적용되어 있다.
