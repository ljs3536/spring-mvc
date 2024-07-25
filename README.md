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
  

