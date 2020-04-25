# Chapter 2: 스프링 부트에서 테스트 코드 작성하기

Created: Mar 17, 2020 10:11 PM
Tags: UnitTest

## 2.1 테스트 코드 소개

- TDD는 **테스트가 주도하는 개발**을 의미한다. 이는 테느스 코드를 먼저 작성하는 것부터 시작한다.
    - 항상 실패하는 테스트를 먼저 작성한다. (Red)
    - 테스트가 통과하는 프로덕션 코드를 작성한다. (Green)
    - 테스트가 통과하면 프로덕션 코드를 리팩토링한다. (Refactor)
- 단위 테스트(Unit test)는 TDD의 첫 번째 단계인 **기능 단위의 테스트 코드를 작성**하는 것을 이야기한다.

- 테스트 코드를 작성할 때의 이점은 다음과 같다.
    - 단위 테스트는 개발 단계 초기에 문제를 발견하게 도와준다.
    - 단위 테스트는 개발자가 나중에 코드를 리팩토링하거나 라이브러리 업그레이드 등에서 기존 기능이 올바르게 작동하는지 확인할 수 있다.
    - 단위 테스트는 기능에 대한 불확실성을 감소시킨다.
    - 단위 테스트는 시스템에 대한 실제 문저를 제공한다. 즉, **단위 테스트 자체가 문서로 사용**될 수 있다.
    - 빠른 피드백이 가능하다.
    - 사람의 눈으로 검증하는 게 아닌, **자동 검증**이 가능하다.
    - **개발자가 만든 기능을 안전하게 보호**해 준다. → 새로운 기능이 추가될 때 기존 기능이 잘 작동되는 것을 보장해 준다.

- 테스트 코드 작성을 도와주는 프레임워크들이 있는데, 가장 대중적인 테스트 프레임워크로 **xUnit**이 있다.
    - 자바에서는 자바용인 JUnit을 사용할 수 있다.

## 2.2 Hello Controller 테스트 코드 작성하기

- 일반적으로 패키지명은 웹 사이트 주소의 역순으로 한다!
- `Application` 클래스 → 프로젝트의 **메인 클래스**
    - `@SpringBootApplication`으로 인해 스프링 부트의 자동 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 설정한다.
    - `@SpringBootApplication`이 있는 위치부터 설정을 읽어 가기 때문에, 이 클래스는 항상 **프로젝트 최상단에 위치해야 한다.**
        - 스프링 컨테이너: 주입을 이용하여 객체를 관리하는 컨테이너이다. 컨테이너의 사전적 의미는 무언가를 담는 용기, 즉 그릇을 의미한다. 컨테이너는 객체 관리를 주로 수행하는 그릇 정도로 이해할 수 있다. 빈의 생성과 관계, 사용, 생명 주기 등을 관장한다. 컨테이너를 통해 시스템 전반에서 언제든 사용 가능하다. → 객체를 사용하기 위해서는 new 생성자를 이용하거나 getter/setter 기능을 써야만 하는데, 한 애플리케이션에는 이러한 객체가 무수히 많이 존재하고 서로 참조하고 있다. 정도가 심할 수록 의존성이 높다고 할 수 있는데, 낮은 결합도와 높은 캡슐화로 대변되는 객체 지향 프로그래밍에서 높은 의존성은 지양된다. 그래서 의존성 제어, 즉 객체 간의 의존성을 낮추기 위해 바로 스프링 컨테이너가 사용된다.
        - 스프링 bean: 스프링 컨테이너에 의해 생성된 자바 객체를 의미한다.
- main 메소드에서 실행하는 `SpringApplication.run`으로 인해 내장 WAS(Web Application Server)를 실행한다. → 서버에 톰캣을 설치할 필요가 없게 되고, 스프링 부트로 만들어진 Jar 파일로 실행하면 된다.
    - 내정 WAS를 사용하면 **언제 어디서나 같은 환경에서 스프링 부트를 배포**할 수 있다.

- Application 패키지 하위에 web 패키지를 생성 후, 컨트롤러와 관련된 클래스들은 모두 이 패키지에 담는다.

```java
package example.org.practice.web;

import example.org.practice.web.dto.HelloResponseDto;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }

}
```

- `@RestController`
    - 컨트롤러를 **JSON을 반환**하는 컨트롤러로 만들어 준다.
- `@GetMapping`
    - HTTP method인 Get의 요청을 받을 수 있는 API를 만들어 준다.

- 위의 테스트가 제대로 작동하는지 테스트를 하기 위해서 WAS를 실행하지 않고, `src/test/java` 디렉토리에 생성했던 패키지들을 그대로 다시 생성한다.
    - 일반적으로 테스트 클래스는 **대상 클래스 이름에 Test를 붙인다**.

```java
package example.org.practice.web;

import example.org.practice.config.auth.SecurityConfig;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.FilterType;
import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;

import static org.hamcrest.Matchers.is;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class,
            excludeFilters = {
            @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = SecurityConfig.class)
            })
public class HelloControllerTest {

    @Autowired
    private MockMvc mvc;

    @WithMockUser(roles="USER")
    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }
}
```

- `@RunWith`
    - 테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킨다.
    - 여기서는 `SpringRunner`라는 스프링 실행자를 사용한다.
    - 즉, 스프링 부트 테스트와 JUnit 사이에 연결자 역할을 한다.
- `@WebMvcTest`
    - 여러 스프링 테스트 어노테이션 중, Web(Spring MVC)에 집중할 수 있는 어노테이션이다.
    - 선언할 경우 `@Controller`, `@ControllerAdvice` 등을 사용할 수 있으나 `@Service`, `@Component`, `@Repository` 등은 사용할 수 없다.
- `@Autowired`
    - 스프링이 관리하는 빈을 주입받는다.
- `private MockMvc mvc`
    - 웹 API를 테스트할 때 사용하며 스프링 MVC 테스트의 시작점이다. 이 클래스를 통해 HTTP, GET, POST 등에 대한 API 테스트를 할 수 있다.
- `mvc.perform(get("/hello"))`
    - MockMvc를 통해 "/hello" 주소로 HTTP GET 요청을 한다. 체이닝이 지원되어 아래와 같이 여러 검증 기능을 이어서 선언할 수 있다.
- `.andExpect(status().isOk())`
    - `mvc.perform`의 결과를 검증한다.
    - HTTP header의 status를 검증한다. 여기서는 200인지를 검증한다.
- `.andExpect(content().string(hello))`
    - `mvc.perform`의 결과를 검증한다. 응답 본문의 내용을 검증한다. 즉, controller에서 "hello"를 리턴하기 때문에 이 값이 맞는지 검증한다.