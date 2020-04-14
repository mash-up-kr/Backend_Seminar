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
  <summary> 테스트 코드가 없는 개발 방식</summary>

1. 코드를 작성하고

2. 프로그램(Tomcat)을 실행한 뒤

3. Postman과 같은 API 테스트 도구로 HTTP 요청하고

4. 요쳥 결과를 System.out.println()으로 눈으로 검증합니다.

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
    <summary> 내장 WAS란?</summary>
    
    별도로 외부에 WAS(Web Application Server)를 두지 않고 애플리케이션을 실행할 때 <br>
    내부에서 WAS를 실행하는 것을 이야기합니다. 이렇게 되면 항상 서버에 톰캣을 설치할 필요가 <br>
    없게 되고, 스프링 부트로 만들어진 Jar 파일(실행 가능한 Java 패키징 파일)로 실행하면 됩니다.

    스프링 부트에서는 내장 WAS를 사용하는 것을 권장하고 있습니다. <br>
    이유는 '언제 어디서나 같은 환경에서 스프링 부트를 배포'할 수 있기 때문입니다. <br>
    외장 wAS를 쓴다고 하면 모든 서버는 WAS의 종류와 버전, 설정을 일치시켜야만 합니다. <br>
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
   
   > 테스트 코드를 작성할 '테스크 클래스'는 **대상 클래스 이름에 Test**를 붙입니다.
   
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
   3-2 ) Settings > Build > Complier > Annotaion Processors > <br>
   Enable annotion processing을 체크합니다.

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
  <summary> 🛠 작성된 테스트 메소드를 실행하였을 때의 Error 해결 🛠</summary>
    
  저는 테스트 코드를 실행하였을 때, <br>
  정상적으로 수행되지 않고 다음과 같은 Error가 발생하였습니다.

  <details>
    <summary> Error Message 확인</summary>
    
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
  
  이를 해결하기 위해 build.gradle `dependencies` 블록 내 코드 사이에 다음의 코드를 추가합니다.

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

## 회고

읽어야 하는 내용은 짧았지만, 정리해야 할 내용은 길다... 