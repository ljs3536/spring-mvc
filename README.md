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
