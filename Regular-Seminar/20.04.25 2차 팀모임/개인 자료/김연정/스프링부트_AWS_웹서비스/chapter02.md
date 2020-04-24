# 2. 스프링 부트에서 테스트 코드를 작성하자

## 진도 기록

|공부 날짜|공부한 페이지 수|
|---|---|
|2020.04.07|51p ~ 61p|
|2020.04.08|2.1 작성|
|2020.04.09|61p ~ 78p|
|2020.04.09|2.1 ~ 2.3 작성|
|2020.04.14|2.4 ~ 궁금한 점 작성|

## 2.1. 테스트 코드 소개

### # TDD(Test-Driven Development)란?

"**테스트가 주도하는 개발**", 테스트가 개발을 이끌어 나간다는 것을 이야기합니다. <br>
테스트 코드를 먼저 작성하는 것부터 시작합니다. <br>
다음 레드 그린 사이클의 과정을 반복하며 올바르게 동작하는지에 대한 **피드백**을 적극적으로 받는 것입니다.

<img src="../images/rgcycle.jpg" width="40%" height="40%" alt="레드그린사이클">

- 항상 실패하는 테스트를 먼저 작성 (Red)
  
- 테스트가 통과하는 프로덕션 코드를 작성 (Green)

- 테스트가 통과하면 프로덕션 코드를 리팩토링 (Refactor)
 
<br>

### # 단위 테스트(Unit Test)란?

TDD의 첫 번째 단계인 **기능 단위의 테스트 코드를 작성**하는 것을 이야기합니다. <br>
TDD와 달리 테스트 코드를 꼭 먼저 작성해야 하는 것도 아니고, 리팩토링도 포함되지 않습니다. <br>
순수하게 테스트 코드만 작성하는 것을 이야기합니다.

<br>

### # 단위 테스트 코드를 작성함으로써 얻는 이점 (feat. 위키피디아)

- 개발단계 초기에 문제를 발견하게 도와줍니다.
  
- 개발자가 나중에 코드를 리팩토링하거나 라이브러리 업그레이드 등에서 <br>
  기존 기능이 올바르게 작동하는지 확인할 수 있습니다. (예: 회귀 테스트)

- 기능에 대한 불확실성을 감소시킬 수 있습니다.

- 시스템에 대한 실제 문서를 제공합니다. 단위테스트 자체를 문서로 사용할 수 있습니다.

<br>

### # 테스트 코드가 해결해주는 문제

1. 코드를 수정할 때마다 톰캣을 내렸다가 다시 실행하는 일을 반복하는 문제 <br>
   → **빠른 피드백**. 테스트 코드는 눈과 손으로 직접 수정된 기능을 확인해야 하는 문제를 해결해줍니다.

2. System.out.println()을 통해 눈으로 검증해야 하는 문제 <br>
   → 테스트 코드를 작성하면 더는 사림이 눈으로 검증하지 않게 **자동검증**이 가능합니다.

3. 새로운 기능이 추가될 때, 기존 기능이 작동하지 않는 문제 <br>
   → 개발자가 만든 기능을 안전하게 보호해줍니다. 기본 기능을 비롯해 여러 경우를 모두 <br>
   테스트 코드로 구현해 놓음으로써 기존 기능이 잘 작동되는 것을 보장해줍니다.

<details>
  <summary> 테스트 코드가 없는 개발 방식 (Click👆🏻)</summary>

1. 코드를 작성하고

2. 프로그램(Tomcat)을 실행한 뒤

3. Postman과 같은 API 테스트 도구로 HTTP 요청하고

4. 요청 결과를 System.out.println()으로 눈으로 검증합니다.

5. 결과가 다르면 다시 프로그램(Tomcat)을 중지하고 코드를 수정합니다.

</details>

<br>

### # 테스트 코드 작성을 도와주는 프레임워크

가장 대중적인 테스트 프레임워크는 xUnit으로, <br>
개발환경(x)에 따라 Unit 테스트를 도와주는 도구라고 생각하면 됩니다. <br>
대표적인 xUnit 프레임워크들은 다음과 같습니다.

- **JUnit - Java**

- DBUnit - DB
  
- CppUnit - C++

- NUnit - .net

<br>

### 2.1. 궁금한점

1. TDD의 장단점

<br>

## 2.2. Hello Controller 테스트 코드 작성하기

### # Application 클래스 작성

1. `src/main/java` 경로에 `com.jojoldu.book.springboot` 패키지를 생성합니다.
   
   > 일반적으로 패키지명은 웹 사이트 주소의 역순으로 합니다. <br>
   > 여기서는 Group Id와 프로젝트명을 합쳐서 패키지명으로 지정하였습니다. <br>
   > 참고로 저는 `space.yjeong.springboot`으로 생성하였습니다.

2. 생성한 패키지 아래에 `Application` 클래스를 생성합니다.
   
3. `Application` 클래스의 코드를 다음과 같이 작성합니다.
   
   ```java
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   
   @SpringBootApplication
   public class Application {
     public static void main(String[] args) {
       SpringApplication.run(Application.class, args);
     }
   }
   ```

**Application 클래스**는 앞으로 만들 프로젝트의 **메인 클래스**가 됩니다.

- `@SpringBootApplication`
  
  스프링 부트의 자동 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 설정합니다. <br>
  특히 `@SpringBootApplication`이 있는 위치부터 설정을 읽어가기 때문에 <br>
  `Application` 클래스는 항상 프로젝트의 최상단에 위치해야만 합니다.

- `SpringApplication.run()`
  
  main에서 실행하는 `SpringApplication.run`으로 인해 내장 WAS를 실행합니다.

  <details>
    <summary> 내장 WAS란? (Click👆🏻)</summary>
    
    <br>

    별도로 외부에 WAS(Web Application Server)를 두지 않고 애플리케이션을 실행할 때 <br>
    내부에서 WAS를 실행하는 것을 이야기합니다. 이렇게 되면 항상 서버에 톰캣을 설치할 필요가 <br>
    없게 되고, 스프링 부트로 만들어진 Jar 파일(실행 가능한 Java 패키징 파일)로 실행하면 됩니다.

    스프링 부트에서는 내장 WAS를 사용하는 것을 권장하고 있습니다. <br>
    이유는 '언제 어디서나 같은 환경에서 스프링 부트를 배포'할 수 있기 때문입니다. <br>
    외장 WAS를 쓴다고 하면 모든 서버는 WAS의 종류와 버전, 설정을 일치시켜야만 합니다. <br>
    새로운 서버가 추가되면 모든 서버가 같은 WAS 환경을 구축해야만 합니다. <br>
    따라서 많은 회사에서도 내장 WAS를 사용하도록 전환하고 있습니다.
  
  </details>

<br>

### # HelloController 클래스 작성

1. `src/main/java` 경로에 생성한 패키지 하위에 `web` 패키지를 생성합니다.
   
   > 컨트롤러와 관련된 클래스들은 모두 이 패키지에 담도록 합니다.

2. 생성한 `web` 패키지 아래에 `HelloController` 클래스를 생성합니다.
   
3. `HelloController` 클래스의 코드를 다음과 같이 작성합니다.
   
   ```java
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   public class HelloController {

     @GetMapping("/hello")
     public String hello() {
       return "hello";
     }
   }
   ```

이 프로젝트는 /hello로 요청이 오면 문자열 hello를 반환하는 기능을 가지게 되었습니다.

- `@RestController`
  
  컨트롤러를 JSON을 반환하는 컨트롤러로 만들어줍니다. <br>
  예전엔 `@ResponseBody`를 각 메소드마다 선언했던 것을 한번에 사용할 수 있게 해줍니다.

- `@GetMapping`
  
  HTTP Method인 Get의 요청을 받을 수 있는 API를 만들어 줍니다. <br>
  예전엔 `@RequestMapping(method = RequestMethod.GET)`으로 사용되었습니다.

<br>

### # HelloControllerTest 클래스 작성

작성한 코드가 제대로 작동하는지 테스트를 하는데 <br>
WAS를 실행하지 않고, **테스트 코드로 검증**해 봅니다.

1. `src/test/java` 경로에 `com.jojoldu.book.springboot` 패키지를 생성합니다.
   
   > 마찬가지로 저는 `space.yjeong.springboot`으로 생성하였습니다.

2. `src/test/java` 경로에 생성한 패키지 하위에 `web` 패키지를 생성합니다.
   
3. 생성한 패키지 아래에 `HelloControllerTest` 테스트 클래스를 생성합니다.
   
   > 테스트 코드를 작성할 '테스트 클래스'는 **대상 클래스 이름에 Test**를 붙입니다.
   
4. `HelloControllerTest` 테스트 클래스의 코드를 다음과 같이 작성합니다.
   
   ```java
   import org.junit.Test;
   import org.junit.runner.RunWith;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
   import org.springframework.test.context.junit4.SpringRunner;
   import org.springframework.test.web.servlet.MockMvc;
   
   import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

   @RunWith(SpringRunner.class)
   @WebMvcTest
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

- `@RunWith(SpringRunner.class)`
  
  테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킵니다. <br>
  여기서는 SpringRunner라는 스프링 실행자를 사용합니다. <br>
  즉, 스프링 부트 테스트와 JUnit 사이에 연결자 역할을 합니다.

- `@WebMvcTest`
  
  여러 스프링 테스트 어노테이션 중, Web(Spring MVC)에 집중할 수 있는 어노테이션입니다. <br>
  선언할 경우 `@Controller`, `@ControllerAdvice` 등을 사용할 수 있습니다. <br>
  단, `@Service`, `@Component`, `@Repository` 등은 사용할 수 없습니다. <br>
  여기서는 컨트롤러만 사용하기 때문에 선언합니다.

- `@Autowired`
  
  스프링이 관리하는 빈(Bean)을 주입 받습니다.

- `private MockMvc mvc`
  
  웹 API를 테스트할 때 사용합니다. <br>
  스프링 MVC 테스트의 시작점입니다. <br>
  이 클래스를 통해 HTTP GET, POST 등에 대한 API 테스트를 할 수 있습니다.

- `mvc.perform(get("/hello"))`
  
  MockMvc를 통해 /hello 주소로 HTTP GET 요청을 합니다. <br>
  체이닝이 지원되어 아래와 같이 여러 검증 기능을 이어서 선언할 수 있습니다.

- `.andExpect(status().isOk())`
  
  mvc.perform의 결과를 검증합니다. <br>
  HTTP Header의 Status를 검증합니다. <br>
  우리가 흔히 알고 있는 200, 404, 500 등의 상태를 검증합니다. <br>
  여기선 OK 즉, 200인지 아닌지를 검증합니다.

- `.andExpect(content().string(hello))`
  
  mvc.perform의 결과를 검증합니다. <br>
  응답 본문의 내용을 검증합니다. <br>
  Controller에서 "hello"를 리턴하기 때문에 이 값이 맞는지 검증합니다.

5. 테스트 코드를 실행하여 테스트가 통과하는 것을 확인합니다.

📝 테스트 코드로 먼저 검증 후, 정말 못 믿겠다면 수동으로 검증(프로젝트 실행)한다는 점 명심!

<br>

### 2.2. 궁금한점

1. 스프링 테스트 어노테이션 종류
2. MockMvc란?

<br>

## 2.3. 롬복 소개 및 설치하기

자바 개발자들의 필수 라이브러리인 롬복(Lombok)은 자바 개발할 때 자주 사용하는 코드인 <br>
Getter, Setter, 기본 생성자, toString 등을 어노테이션으로 자동 생성해 줍니다.

### # 프로젝트에 롬복 추가

1. build.gradle `dependencies` 블록 내 코드 사이에 다음의 코드를 추가합니다.
   
   ```java
   compile('org.projectlombok:lombok')
   ```

2. 우측 Gradle 탭에 Refresh 버튼을 클릭하여 라이브러리(의존성)를 내려받습니다.

3. 롬복 플러그인을 설치합니다.
   
   1 ) 단축키 `Ctrl+Shift+A` → plugins 검색 후 해당 Action을 선택합니다. <br>
   2 ) Marketplace 탭 이동 → lombok 검색 및 설치 후 재시작합니다. <br>
   3 ) "롬복에 대한 설정이 필요하다"는 팝업이 등장합니다. <br>
   3-1 ) 팝업에 Enable 버튼(파란색 글씨)을 클릭하거나 <br>
   3-2 ) Settings > Build > Complier > Annotaion Processors > Enable annotion processing을 체크합니다.

롬복은 플러그인 설치는 한 번만 하면 되지만, build.gradle에 라이브러리를 추가하는 것과 <br>
Enable annotaion processing을 체크하는 것은 프로젝트마다 진행해야 합니다.

<br>

## 2.4. Hello Controller 코드를 롬복으로 전환하기

기존 코드를 롬복으로 리팩토링하는 과정에서 테스트 코드가 있음으로써 쉽게 전환할 수 있습니다.<br>
롬복으로 변경하고 어떤 문제가 생기는지는 테스트 코드만 둘러보면 알 수 있기 때문입니다.

### # HelloResponseDto 클래스 작성

1. `src/main/java` 경로에 있는 `web` 패키지 하위에 `dto` 패키지를 추가합니다.
   
   > 모든 응답 Dto 클래스들은 이 패키지에 담도록 합니다.

2. 생성한 패키지 아래에 `HelloResponseDto` 클래스를 생성합니다.
   
3. `HelloResponseDto` 클래스의 코드를 다음과 같이 작성합니다.
   
   ```java
   import lombok.Getter;
   import lombok.RequiredArgsConstructor;

   @Getter
   @RequiredArgsConstructor
   public class HelloResponseDto {
     private final String name;
     private final int amount; 
   }
   ```

- `@Getter`
  
  선언된 모든 필드의 get 메소드를 생성해 줍니다.

- `@RequiredArgsConstructor`
  
  선언된 모든 final 필드가 포함된 생성자를 생성해 줍니다. <br>
  final이 없는 필드는 생성자에 포함되지 않습니다.

<br>

### # HelloResponseDtoTest 클래스 작성

1. `src/test/java` 경로에 있는 `web` 패키지 하위에 `dto` 패키지를 추가합니다.

2. 생성한 패키지 아래에 `HelloResponseDtoTest` 클래스를 생성합니다.
   
3. `HelloResponseDtoTest` 클래스의 코드를 다음과 같이 작성합니다.
   
   ```java
   import org.junit.Test;
   import static org.assertj.core.api.Assertions.assertThat;
   
   public class HelloResponseDtoTest {
     
     @Test
     public void 롬복_기능_테스트() {
       // given
       String name = "test";
       int amount = 1000;
       
       // when
       HelloResponseDto dto = new HelloResponseDto(name, amount);
       
       // then
       assertThat(dto.getName()).isEqualTo(name);
       assertThat(dto.getAmount()).isEqualTo(amount);
     }
   }
   ```

- `assertThat`
  
  assertj라는 테스트 검증 라이브러리의 검증 메소드입니다. <br>
  검증하고 싶은 대상을 메소드 인자로 받습니다. <br>
  메소드 체이닝이 지원되어 `isEqualTo()`와 같이 메소드를 이어서 사용할 수 있습니다.

- `isEqualTo`
  
  assertj의 동등 비교 메소드입니다. <br>
  `assertThat`에 있는 값과 `isEqualTo`의 값을 비교해서 같을 때만 성공입니다.

여기서 Junit의 기본 `assertThat`이 아닌 assertj의 `assertThat`을 사용하였습니다.<br>
assertj 역시 Junit에서 자동으로 라이브러리 등록을 해줍니다.<br>
Junit과 비교하여 assertj의 장점은 다음과 같습니다.

- CoreMatchers와 달리 추가적으로 라이브러리가 필요하지 않습니다. <br> 
  : Junit의 assertThat을 쓰게 되면 is()와 같이 CoreMatchers 라이브러리가 필요합니다.

- 자동완성이 좀 더 확실하게 지원됩니다. <br>
  : IDE에서는 CoreMatchers와 같은 Matcher 라이브러리의 자동완성 지원이 약합니다.

<details>
  <summary> 🛠 작성된 테스트 메소드를 실행하였을 때의 Error 해결 🛠 (Click👆🏻)</summary>

  <br>

  저는 테스트 코드를 실행하였을 때, 정상적으로 수행되지 않고 다음과 같은 Error가 발생하였습니다.

  <details>
    <summary> Error Message 확인 (Click👆🏻)</summary>

    > Task :cleanTest
    > Task :compileJava FAILED
    C:\Users\YJeong\IdeaProjects\springboot_aws_webservice\src\main\java\space\yjeong\springboot\web\dto\HelloResponseDto.java:10: error: variable name not initialized in the default constructor
        private final String name;
                             ^
    C:\Users\YJeong\IdeaProjects\springboot_aws_webservice\src\main\java\space\yjeong\springboot\web\dto\HelloResponseDto.java:11: error: variable amount not initialized in the default constructor
        private final int amount;
                          ^
    2 errors
    FAILURE: Build failed with an exception.
    * What went wrong:
    Execution failed for task ':compileJava'.
    > Compilation failed; see the compiler error output for details.
    * Try:
    Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.
    * Get more help at https://help.gradle.org
    BUILD FAILED in 0s
    2 actionable tasks: 2 executed
  
  </details>
  
  이를 해결하기 위해 build.gradle `dependencies` 블록 내 코드 사이에 다음의 코드를 추가하였습니다.

  ```java
  compileOnly('org.projectlombok:lombok')
  annotationProcessor('org.projectlombok:lombok')
  ```

  ##### 참고 : [Dev.](https://blogdeveloperspot.blogspot.com/2019/03/springboot2-lombok-gradle-bootjar.html)

</details>

<br>

### # HelloController 클래스 코드 추가

1. `src/main/java` 경로의 `com.jojoldu.book.springboot.web` 패키지로 이동합니다.

2. `HelloController` 클래스의 코드를 다음과 같이 수정합니다.
   
   ```java
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestParam;
   import org.springframework.web.bind.annotation.RestController;
   import space.yjeong.springboot.web.dto.HelloResponseDto;

   @RestController
   public class HelloController {
     
     @GetMapping("/hello")
     public String hello() {
       return "hello";
     }
     
     /* 추가된 코드 */
     @GetMapping("/hello/dto")
     public HelloResponseDto helloDto(@RequestParam("name") String name,
                                      @RequestParam("amount") int amount) {
       return new HelloResponseDto(name, amount);
     }
   }
   ```

name과 amount는 API를 호출하는 곳에서 넘겨준 값들입니다.

- `@RequestParam`
  
  외부에서 API로 넘긴 파라미터를 가져오는 어노테이션입니다. <br>
  여기서는 외부에서 name (`@RequestParam("name")`) 이란 이름으로 넘긴 <br>
  파라미터를 메소드 파라미터 name`(String name)`에 저장하게 됩니다.

<br>

### # HelloControllerTest 클래스 코드 추가

추가된 API를 테스트하는 코드를 추가합니다.

1. `src/test/java` 경로의 `com.jojoldu.book.springboot.web` 패키지로 이동합니다.

2. `HelloControllerTest` 클래스의 코드를 다음과 같이 수정합니다.
   
   ```java
   import org.junit.Test;
   import org.junit.runner.RunWith;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
   import org.springframework.test.context.junit4.SpringRunner;
   import org.springframework.test.web.servlet.MockMvc;

   import static org.hamcrest.Matchers.is;
   import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

   @RunWith(SpringRunner.class)
   @WebMvcTest
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

     /* 추가된 코드 */
     @Test
     public void helloDto가_리턴된다() throws Exception {
       String name = "hello";
       int amount = 1000;
       
       mvc.perform(get("/hello/dto")
                .param("name", name)
                .param("amount", String.valueOf(amount)))
                .andExpect(status().isOk()).andExpect(jsonPath("$.name", is(name)))
                .andExpect(jsonPath("$.amount", is(amount)));
     }
   }
   ```

- `param`
  
  API를 테스트할 때 사용될 요청 파라미터를 설정합니다. <br>
  단, 값은 String만 허용됩니다. <br>
  그래서 숫자/날짜 등의 데이터도 등록할 때는 문자열로 변경해야만 가능합니다.

- `jsonpPath`
  
  JSON 응답값을 필드별로 검증할 수 있는 메소드입니다. <br>
  `$`를 기준으로 필드명을 명시합니다. <br>
  여기서는 name과 amount를 검증하니 `$.name`, `$.amount`로 검증합니다.

<br>

### 2.4. 궁금한점

1. CoreMatchers 라이브러리

<br>

## 궁금한 점 스스로 찾아보기

### 1. TDD의 장단점

빠른 피드백, 코드 복잡도가 떨어짐, 디버깅 시간 감소, 리팩토링이 편함 등은 <br>
보통 TDD의 장점이라고 하지만 이것은 테스트 코드를 작성했을 때의 장점이라고도 할 수 있습니다. <br>
따라서, TDD만의 장점을 알아보았습니다.

> 켄트 백이 테스트 주도 개발을 설명하면서 강조하는 진짜 가치는 <br>
> **'어려운 일을 작은 단계로 쪼갤 수 있는 능력'** 입니다. 

#### # TDD의 장점

1. 오버 엔지니어링을 방지합니다.
   
   잘 정리된 요구사항의 역할도 하므로 딱 필요한 만큼만 코딩하도록 유도하여 <br>
   불필요하게 복잡해지거나 오버 엔지니어링을 줄여줍니다.

2. 테스트 커버리지가 높아집니다.
      
   테스트 코드를 먼저 작성하니까 당연한 이야기입니다. <br>
   테스트 코드를 나중에 작성해도 테스트 커버리지는 높아진다 할 수 있지만, <br>
   나중에 작성하면 잊어버리거나, 기능이 잘 되니까 미루는 경우를 방지합니다.

3. 아이디어를 생각해내는 데도 도움이 되고 한 번에 하나씩만 집중할 수 있습니다.
   
   모듈, 클래스, 메소드를 구체적으로 정의하도록 강제하여 일을 점진적으로 진행할 수 있습니다. <br>
   하나의 클래스, 메소드에 집중하지 못하고 관련된, 혹은 다른 여러 개를 동시에 <br>
   작업하면서 흐름과 방향을 잃는 경우를 방지합니다.

4. 설계에 대한 피드백이 빠릅니다.
   
   객체지향 설계에서는 객체 간의 의존도를 낮추는 것이 중요한데, <br>
   대상 코드가 잘못된 설계와 과도한 복잡도를 갖고 있으면 새로운 테스트 코드를 작성하기가 점점 어려워집니다. <br>
   이 때, 설계에 대한 피드백을 빠르게 받을 수 있습니다.
  
무엇보다 가장 큰 장점은 높은 퀄리티의 소프트웨어를 보장한다는 것입니다.

<details>
  <summary> 높은 퀄리티의 소프트웨어란? (Click👆🏻)</summary>

1. 절대 에러나 버그가 없을 것
   
2. 추가적인 요구사항이 있을 때 손쉽게 그 요구사항을 반영할 수 있을 것
   
3. 유지보수에 용이할 것

</details>

#### # TDD의 단점
  
1. 개발 생산성이 낮아집니다.
   
   테스트 코드를 추가적으로 개발해야 하므로 개발 시간이 더 소요됩니다. (대략 10~30%)

2. 진입 장벽이 높습니다.
   
   기존의 자신이 개발하던 방식을 많이 바꾸어야 하기 때문에, <br>
   익숙하지 않은 개발 방식에 많은 어려움을 느낄 수 있습니다. <br>
   또한, '반드시 툴을 써서 어떻게 해야된다.'라는 이미지나 틀, 규칙에 얽매여 <br>
   똑같은 테스트를 copy&paste하는 경우가 있습니다. <br>
   너무 도구, 규칙에 집착하는 경우 더 어렵게 느끼게 됩니다.

3. 코드 생산 비용이 높아집니다.
   
   기존의 개발 프로세스에 테스트케이스 설계까지 추가되므로 그만큼의 생산 비용이 추가됩니다. <br>
   이러한 이유로 TDD를 도입하는 것에 대해 걸림돌이 되기도 합니다.

##### 출처 : [dpudpu](https://dublin-java.tistory.com/54), [Heee's Development Blog](https://gmlwjd9405.github.io/2018/06/03/agile-tdd.html)

<br>

### 2. 스프링 테스트 어노테이션 종류

|어노테이션|설명|Bean|
|---|---|---|
|@SpringBootTest|통합 테스트, 전체|Bean 전체|
|@WebMvcTest|단위 테스트, Mvc 테스트|MVC 관련된 Bean|
|@DataJpaTest|단위 테스트, Jpa 테스트|JPA 관련 Bean|
|@RestClientTest|단위 테스트, Rest API 테스트|일부 Bean|
|@JsonTest|단위 테스트, Json 테스트|일부 Bean|

#### # @SpringBootTest

**통합 테스트**를 제공하는 기본적인 스프링 부트 어노테이션입니다. <br>
애플리케이션이 실행될 때의 설정을 임의로 바꾸어 테스트를 진행할 수 있으며 **여러 단위 테스트를 하나의 통합된 테스트로 수행할 때 적합합니다.**

`@SpringBootTest`는 만능입니다. <br>
실제 구동되는 Application과 똑같이 Application Context를 로드하여 테스트하기 때문에 하고 싶은 테스트를 모두 수행할 수 있습니다.

그렇지만 Application의 설정을 모두 로드하기 때문에 Application 규모가 클수록 느려집니다. <br>
이렇게 되면 단위 테스트라는 의미가 희석됩니다.

`@RunWith` 어노테이션을 사용하면 JUnit에 내장된 러너를 사용하는 대신 어노테이션에 정의된 러너 클래스를 사용합니다.

`@SpringBootTest` 어노테이션을 사용하려면 JUnit의 SpringJUnit4ClassRunner 클래스를 상속 받는 `@RunWith(SpringRunner.class)`를 꼭 붙여서 사용해야 정상동작합니다.

기존 Spring Framework에서 사용하던 spring-test 이외에도 여러 유용한 라이브러리를 포함하고 있습니다. <br>
Test Scope Dependencies 아래의 의존성을 자동으로 갖습니다.
- JUnit: The de-facto standard for unit testing Java applications.

- Spring Test & Spring Boot Test: Utilities and integration test support for Spring Boot applications.

- AssertJ: A fluent assertion library.

- Hamcrest: A library of matcher objects (also known as constraints or predicates).

- Mockito: A Java mocking framework.

- JSONassert: An assertion library for JSON.

- JsonPath: XPath for JSON.

#### # @WebMvcTest

MVC를 위한 테스트 어노테이션입니다. 웹에서 테스트하기 힘든 Controller를 테스트하는데 적합합니다.<br>
웹상에서 요청과 응답에 대한 테스트를 진행합니다.

Security 혹은 Filter까지 자동으로 테스트하여 수동으로 추가/삭제가 가능합니다.

`@WebMvcTest` 어노테이션을 사용하면 MVC 관련된 설정인 `@Controller`, `@ControllerAdvice`, `@JsonComponent`와 Filter, WebMvcConfiguer, HandlerMethodArgumentResolver만 로드되기 때문에 @SpringBootTest 어노테이션보다 가볍게 테스트할 수 있습니다.

#### # @DataJPaTest

JPA 관련된 설정만 로드하는 어노테이션입니다.

데이터 소스의 설정이 정상적인지, JPA를 사용해서 데이터를 제대로 생성/수정/삭제하는지 등의 테스트가 가능합니다.

기본적으로 인메모리 데이터베이스(in-memory embedded database)를 이용합니다.

`@Entity` 클래스를 스캔하여 스프링 데이터 JPA 저장소를 구성합니다.

#### # @RestClientTest

REST 관련 테스트를 도와주는 어노테이션입니다.

REST 통신의 JSON 형식이 예상대로 응답을 반환하는지 등을 테스트합니다.

#### # @JsonTest

JSON의 직렬화, 역직렬화를 수행하는 라이브러리인 Gson과 Jackson의 테스트를 제공합니다.

##### 출처 : [Yun Blog](https://cheese10yun.github.io/spring-boot-test/), [TOAST Meetup!](https://meetup.toast.com/posts/124)

<br>

### 3. MockMvc란?

실제 객체와 비슷하지만 테스트에 필요한 기능만 가지는 가짜 객체를 만들어서 웹 애플리케이션을 애플리케이션 서버에 배포하지 않고도 스프링 MVC 동작을 재현할 수 있는 클래스입니다.

#### # MockMvc가 왜 필요한가?

스프링 MVC 컨트롤러의 테스트를 할 때, 정작 컨트롤러 자체에는 단위 테스트가 필요할 만한 비즈니스 로직이 존재하지 않습니다.

따라서 스프링 MVC의 프레임워크 기능까지 통합된 상태인 **통합 테스트의 관점**으로 봐야합니다.

<details>
  <summary> Controller의 주요 역할 (Click👆🏻)</summary>

- 요청 경로

- 처리내용의 매핑

- 입력값 검사

- 요청한 데이터의 취득

- 비즈니스 로직 호출

- 다음 이동 화면의 제어

</details>

통합한 상태에서 컨트롤러 테스트는 E2E(End to End)로 테스트를 합니다.

<details>
  <summary> E2E Test란? (Click👆🏻)</summary>

E2E(End-to-End) 테스트는 종단(EndPoint)간 테스트로 사용자의 입장에서 테스트 하는 것입니다. <br>
전체 시스템이 제대로 작동하는지 확인 하기 위한 테스트로 <br>
시나리오 테스트, 기능 테스트, 통합 테스트, GUI 테스트를 하는데 사용합니다. <br>
사용자에게 직접 노출되는 부분을 점검한다고 생각하면 됩니다.

</details>

이때, 뷰가 생성한 응답 데이터(HTML)의 유효성을 검증할 수 있다는 장점이 있습니다.

하지만 다음의 단점들이 있습니다.
- 애플리케이션이나 데이터베이스를 반드시 기동시켜야 합니다.
- 트랜잭션이 커밋되기 떄문에 테스트를 실시하기 이전의 상태로 되돌릴 수 없습니다.
- 회귀 테스트를 실행하기 위해 Selenium등을 활용해서 테스트 케이스를 구현해야합니다.

스프링 테스트는 E2E의 단점을 해소하면서 통합한 상태의 컨트롤러 테스트를 위해 'MockMvc' 제공합니다.

#### # MockMvc의 흐름

1. Test Case의 메서드는 DispathcherServlet에 요청할 데이터(요청경로나 요청파라미터)를 설정합니다.
   
2. MockMvc는 DispathcherServlet에 요청을 보냅니다. <br>
   DispathcherServlet은 테스트용으로 확장된 TestDispathcherServlet 입니다.

3. DispathcherServlet은 요청을 받아 매핑정보를 보고 그에 맞는 핸들러(컨트롤러) 메서드를 호출합니다.

4. 테스트 케이스 메서드는 MovkMvc가 반환하는 실행결과를 받아 실행 결과가 맞는지 검증합니다.

#### # MockMvc의 모드

사용자 정의 DI 컨테이너 모드와 단독 모드가 있습니다.

- 사용자 정의 DI 컨테이너 모드 <br>
  : 스프링 MVC의 설정을 적용한 DI 컨테이너를 만들어 이 DI 컨테이너를 사용해 스프링 MVC 동작을 재현합니다. <br>
  애플리케이션 서버에 배포한 것과 같은 것 처럼 테스트가 가능합니다.

- 단독 모드 <br>
  : 스프링 MVC 설정을 '스프링 테스트' 측에서 처리합니다. <br>
  스프링 테스트가 생성된 DI 컨테이너를 사용해 스프링 MVC 동작을 재현합니다. <br>
  스프링 테스트의 각종 설정은 테스트 케이스 측에서 커스텀 가능합니다. <br>
  스프링 MVC 기능을 이용하면서도 단위 테스트 관점에서 컨트롤러 테스트를 합니다.

##### 출처 : [IT모아](https://itmore.tistory.com/entry/MockMvc-상세설명)

<br>

## 회고

읽어야 하는 내용은 짧았지만, 정리해야 할 내용은 많고도 많았다... <br>
어노테이션 종류가 너무 많아서 다 외울 수 있을까 한다.