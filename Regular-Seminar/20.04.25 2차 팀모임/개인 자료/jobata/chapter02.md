# 2장 스프링 부트에서 테스트 코드를 작성하자

## Contents
- [테스트 코드 소개](#테스트-코드-소개)
- [Hello Controller 테스트 코드 작성하기](#Hello-Controller-테스트-코드-작성하기)
- [롬복 소개 및 설치하기](#롬복-소개-및-설치하기)
- [Hello Controller 코드를 롬복으로 전환하기](#Hello-Controller-코드를-롬복으로-전환하기)

---

### 테스트 코드 소개
- TDD
  - Test-driven Development
  - Red, Green, Refector
  - TDD와 Unit test는 다른 이야기다.
- 단위 테스트 코드를 작성하는 이유
  - 개발단계 초기에 문제를 발견하게 돕는다.
  - 코드를 리팩토링하거나 할 때 기존 기능이 올바르게 작동하는지 확인 가능.
  - 기능에 대한 불확실성 감소.
  - 시스템에 대한 실제 문서를 제공. 단위 테스트 자체를 문서로 사용 가능.

### Hello Controller 테스트 코드 작성하기
- `Application 클래스`
  ```java
  @SpringBootApplication
  public class Application {
      public static void main(String[] args) {
          SpringApplication.run(Application.class, args);
      }
  }
  ```
  - 프로젝트의 메인 클래스
    - @SpringBootApplication으로 인해 스프링 부트의 자동 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 설정된다.
      - @SpringBootApplication이 있는 위치부터 설정을 읽어가기 때문에 이 클래스는 항상 프로젝트의 최상단에 위치해야만 한다.
    - SprinApplication.run() 으로 내장 WAS를 실행한다.
      - 톰켓 설치 불필요, Jar 파일로 실행
      - 언제 어디선나 같은 환경에서 스프링 부트를 배포할 수 있게 한다.
  - @SpringBootAplication
    - @EnableAutoConfiguration
      - enable [Spring Boot’s auto-configuration mechanism](https://docs.spring.io/spring-boot/docs/2.1.14.BUILD-SNAPSHOT/reference/html/using-boot-auto-configuration.html)
    - @ComponentScan
      - enable @Component scan on the package where the application is located
    - @Configuration
      - allow to register extra beans in the context or import additional configuration classes
- `HelloController 클래스`
  ```java
  @RestController
  public class HelloController {
      @GetMapping("/hello")
      public String hello() {
          return "hello";
      }
  }
  ```
  - /hello로 요청이 오면 문자열 hello를 반환한다.
  - @RestController
    - 컨트롤러를 JSON을 반환하는 컨트롤러로 만들어 준다.
    - @ResponseBody를 각 메소드마다 선언했던 것을 한번에 사용할 수 있게 해준다.
  - @GetMapping
    - HTTP Method Get 요청을 받을 수 있는 API를 만들어 준다.
    - 이전엔 @RequestMapping(method = RequestMethod.GET)으로 사용되었었음.
- `HelloControllerTest 클래스`
  ```java
  @RunWith(SpringRunner.class)
  @WebMvcTest(controllers = HelloController.class)
  public class HelloControllerTest {
      @Autowired
      private MockMvc mvc;

      @Test
      public void hello가_리턴된다() throws Exception {
          String hello = "hello";

          mvc.perform(get("/hello"))
                  .andExpect(status().isOk())
                  .andExpect(content().string(hello));
      }
  }
  ```
  - @Runwith(SpringRunner.class)
    - 테스트를 진핼할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킨다.
      - 여기서는 SpringRunner 사용
      - 스프링 부트 테스트와 JUnit 사이에 연결자 역할
  - @WebMvcTest
    - 스트링 테스트 어노테이션 중 Web(Spring MVC)에 집중
    - @Controller, @ControllerAdvice 등을 사용할 수 있다.
      - @Sevice, @Component, @Repository 등은 사용할 수 없다.
  - @Autowired
    - 스프링이 관리하는 빈을 주입 받는다.
  - private MockMvc mvc
    - 웹 API를 테스트할 때 사용
    - 스프링 MVC 테스트의 시작점
    - HTTP Method에 대한 API 테스트를 할 수 있다.
  - mvc.perform(get("/hello"))
    - MockMvc를 통해 /hello 주소로 HTTP GET 요청을 한다.
    - 체이닝이 지원되어 여러 검증 기능을 이어서 선언 가능
  - .andExpect()
    - mvc.perform의 결과를 검증
  - status().isOk()
    - HTTP Header의 Status를 검증
      - 여기선 200, OK 인지 아닌지 검증
  - content().string(hello)
    - 응답 본문의 내용을 검증
      - Controller에서 hello를 리턴하기 때문에 이 값이 맞는지 검증

### 롬복 소개 및 설치하기
- Lombok
  - 자바 개발자들의 필수 라이브러리
  - Getter, Setter, 기본생성자, toString 등을 애노테이션으로 자동 생성해 준다.
- 롬복 추가하기
  - build.gradle에 등록
    ```gradle
    compile('org.projectlombok:lombok')
    ```
  - 프로젝트에서 Annotation Processing 적용 (IntelliJ IDEA)
    ```
    Settings > Build > Compiler > Annotation Processors > 'Enable annotation processing check'
    ```

### Hello Controller 코드를 롬복으로 전환하기
- 기존 코드를 롬복으로 전환할 수 있는 근거
  - 테스트 코드가 우리의 코드를 지켜주기 때문에 기능이 제대로 작동할지 예측할 수 있다.
- `HelloResponseDto 클래스`
  ```java
  @Getter
  @RequiredArgsConstructor
  public class HelloResponseDto {
      private final String name;
      private final int amount;
  }
  ```
  - @Getter
    - 선언된 모든 필드의 Get 메소드를 생성
  - @RequiredArgsConstructor
    - 선언된 모든 final 필드가 포함된 생성자를 생성
    - final이 없는 필드는 생성자에 포함되지 않는다.
- `HelloResponseDtoTest 클래스`
  ```java
  public class HelloResponseDtoTest {
      @Test
      public void 블록_기능_테스트() {
          // given
          String name = "test";
          int amount = 1000;
          
          //when
          HelloResponseDto dto = new HelloResponseDto(name, amount);
          
          //then
          assertThat(dto.getName()).isEqualTo(name);
          assertThat(dto.getAmount()).isEqualTo(amount);
      }
  }
  ```
  - assertThat
    - assertj라는 테스트 검증 라이브러리의 검증 메소드
      - [AssertJ가 JUnit의 assertThat 보다 편리한 이유
](https://www.youtube.com/watch?v=zLx_fI24UXM)
    - 검증하고 싶은 대상을 메소드 인자로 받는다.
    - 메소드 체이닝이 지원되어 메소드를 이어서 사용할 수 있다.
  - isEqualTo
    - assertj의 동등 비교 메소드
    - assertThat에 있는 값과 isEqualTo의 값을 비교해서 같을 때만 성공
- `HelloController 클래스`
  ```java
  @GetMapping("/hello/dto")
  public HelloResponseDto helloDto(@RequestParam("name") String name, @RequestParam("amount") int amount) {
      return new HelloResponseDto(name, amount);
  }
  ```
  - 새로만든 HelloResponseDto를 사용해보자.
  - @RequestParam
    - 외부에서 API로 넘긴 파라미터를 가져오는 어노테이션
      - 여기서는 외부에서 name (@RequestParam("name")) 이란 이름으로 넘긴 파라미터를 메소드 파리미터 name(String name)에 저장하게 된다.
- `HelloControllerTest 클래스`
  ```java
  @Test
  public void helloDto가_리턴된다() throws Exception {
      String name = "hello";
      int amount = 1000;

      mvc.perform(get("/hello/dto").param("name", name).param("amount", String.valueOf(amount)))
              .andExpect(status().isOk())
              .andExpect(jsonPath("$.name", is(name)))
              .andExpect(jsonPath("$.amount", is(amount)));
  }
  ```
  - 추가된 API도 역시 테스트해보자.
  - param
    - API 테스트할 때 사용될 요청 파라미터를 설정
      - 단, 값은 String만 허용하기 때문에 다른 타입의 데이터를 등록할 때 문자열로 변경해야만 한다.
  - jsonPath
    - JSON 응답값을 필드별로 검증할 수 있는 메소드
    - $를 기준으로 필드명을 명시
