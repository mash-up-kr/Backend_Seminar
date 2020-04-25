
CHAPTER 2 : 스프링 부트에서 테스트 코드를 작성하자

TDD vs Unit Test(단위 테스트)

TDD란 ? 테스트가 주도하는 개발, 테스트 코드를 먼저 작성하는 것부터 시작
레드 그린 사이클 - 1. 항상 실패하는 테스트를 먼저 작성(Red)
               1. 테스트가 통과하는 프로덕션 코드를 작성(Green)
               2. 테스트가 통과하면 프로덕션 코드를 리팩토링(Refactor)

* 리팩토링 -  리팩토링은 외부동작을 바꾸지 않으면서 내부 구조를 개선하는 방법(자세한  따로)

단위테스트란? TDD의 첫 번째 단계인 기능 단위의 테스트 코드를 작성
위 사이클의 1번에 해당하는 부분

단위테스트의 이점
1. 단위 테스트는 개발단계 초기에 문제를 발견하게 도와준다. 세분화하여 테스트 하므로 문제 발견이 쉽다.
2. 단위 테스트는 개발자가 나중에 코드를 리팩토링하거나 라이브러리 업그레이드 등에서 기존 기능이 올바르게 자동하는지 확인할 수 있다.(ex, 회귀 테스트)
3. 단위 테스트는 기능에 대한 불확실성을 감소시킬 수 있다. 일일이 테스트 해보므로 기능이 돌아갈 가능성이 높다.
4. 단위 테스트는 시스템에 대한 실제 문서를 제공한다. 즉 단위 테스트 자체가 문서로 사용할 수 있다.

일일이 프로그램을 실행하고 요청결과를 확인하는 작업은 시간이 많이 소요된다. 그러므로 테스트 코드를 통해 실행 결과를 빠르게 확인하고 수정 가능하다.

테스크 코드 작성 도와주는 프로그램 - Java - JUnit

2.2 Hello Controller 테스트 코드 작성하기

main
  --java
    --org.example.springboot(package 파일)
        --Application(java 파일)


package org.example.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args){
        SpringApplication.run(Application.class, args);
    }
}

@SpringBootApllication으로 인해 스프링 부트의 자동 설정, 스프링 Bean 일긱와 생성 모두 자동으로 설정된다.
이 위치부터 읽기 때문에 항상 프로젝트의 최상단에 위치해야 한다.

main 메소드에서 시행하는 SpringApplication.run으로 인해 내장 WAS(웹 어플리케이션 서버)를 실행한다.
서버에 Tomcat 설치 없이 내부에서 WAS 실행 가능하다.

*Tomcat 이란? Java Servlet을 실행하고 JSP 가 포함된 웹 페이지를 렌더링하는 Apache Software Foundation의 응용 프로그램 서버다.
             톰캣(Tomcat)은 웹 서버와 연동하여 실행할 수 있는 자바 환경을 제공하여 자바 서버 페이지(JSP)와 자바 서블릿이 실행할 수 있는 환경을 제공하고 있다.
           
내장 WAS를 사용하면 여러 서버를 설치하는 상황에서 모두 동일한 환경에서 스프링 부트를 배포할 수 있다. 그러므로 오류가 더욱 적으므로 자주 사용한다.
         
main
  --java
    --org.example.springboot(package 파일)
        --web(package 파일)
            --HelloController(java 파일)
        --Application(java 파일)

src/main/java/org/example/springboot/web/HelloController
package org.example.springboot.web;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }
}

@RestController
    - 컨트롤러를 JSON을 반환하는 컨트롤러로 만들어 준다.
@GetMapping
    -HTTP Method인 Get의 요청을 받을 수 있는 API를 만들어 준다.

src/main/java/org/example/springboot/web/HelloControllerTest
package org.example.springboot.web;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;

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
        //given
        String hello = "hello";

        //when
        ResultActions perform = mvc.perform(get("/hello"));

        //then
        perform
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }
}

@RunWith(SpringRunnser.class)
    - 테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킨다.
    - 위에서는 SpringRunner라는 스프링 실행자를 사용한다.
    - 즉, 스프링 부트 테스트와 JUnit 사이에 연결자 역할을 한다.

@WebMvcTest
    - 여러 스프링 테스트 어노테이션 중 Web(Spring MVC)에 집중할 수 있는 어노테이션이다.
    - 선언할 경우 @Controller, @ControllerAdvice 등을 사용할 수 있다.
    - 단, @Service, @Component, @Repository 등은 사용할 수 없다.
    - 여기서는 컨트롤러만 사용하기 때문에 선언

@Autowired
    - 스프링이 관리하는 빈(Bean)을 주입 받는다.

@private MockMvc mvc
    - 웹 API를 테스트할 때 사용한다.
    - 스프링 MVC 테스트의 시작점
    - 이 클래스를 통해 HTTP, GET,POST 등에 대한 API 테스트를 할 수 있다.

@mvc.perform(get("/hello"))
    -MockMvc를 통해 /hello 주소로 HTTP GET 요청을 한다.

@.andExpect(status().isOk())
    -mvc.perform의 결과를 검증한다.
    -HTTP Header의 Status를 검증
    - 200, 404, 500 등의 상태 검증
    - 여기선 OK 즉, 200인지 아닌지 검증

@.andExpect(content().string(hello))
    -mvc.perform의 결과를 검증
    -응답 본문의 내용을 검증
    -Controller에서 "hello"를 리턴하기 때문에 이 값이 맞는지 검증 

*Bean란? 미리 만들어 놓은 객체를 말한다.
<빈의 사용 용도>
All properties private (use getters/setters)
A public no-argument constructor
Implements Serializable.

Controller가 있는데 를 들면, LoginController, MemberAddConroller, MemberDeleteController 등등
이런 것들을 초반에 미리 생성해 놓는 것이 Bean 인거 같습니다.
그리고 필요할 때( 예를 들면, 사용자가 login.do를 요청하거나, add.do를 요청하거나 등등)
가져다 쓸수 있는 그런 인스턴스들을 말하는 것

내부 서버를 킬 때에는 Application에서 실행해서 서버를 킬 수 있다.

2.3 롬복 소개 및 설치하기
롬복은 라이브러리로 Getter, Setter, 기본생성자, toString 등을 어노테이션으로 자동 생성해 준다.

build.gradle에 
dependencies 안에 compile('org.projectlombok:lombok')을 입력하여 프로젝트에 추가한다.
그 다음 plugins에서 lombok을 설치한다.

2.4 Hello Controller 코드를 롬복으로 전환하기
web 패키지에 dto 패키지 추가하여 앞으로 모든 응답 Dto는 이 Dto 패키지에 추가한다.

src/main/java/org/example/springboot/web/dto/HelloResponseDto
package org.example.springboot.web.dto;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public class HelloResponseDto {

    private final String name;
    private final int amount;

}

@Getter
    - 선언된 모든 필드의 get 메소드를 생성해 준다.

@RequiredArgsConstructor
    - 선언된 모든 final 필드가 포함된 생성자를 생성해 준다.
    - final이 없는 필드는 생성자에 포함되지 않는다.

test
    --java
        --org.example.springboot.web
            --dto
                --HelloResponseDtoTest

src/test/java/org/example/springboot/web/dto/HelloResponseDtoTest
package org.example.springboot.web.dto;

import org.junit.Test;

import static org.assertj.core.api.Assertions.assertThat;

public class HelloResponseDtoTest {

    @Test
    public void 롬복_기능_테스트(){
        //given
        String name = "test";
        int amount = 1000;

        //when
        HelloResponseDto dto = new HelloResponseDto(name, amount);

        //then
        assertThat(dto.getName()).isEqualTo(name);
        assertThat(dto.getAmount()).isEqualTo(amount);
    }
}

assertThat
    - assertj라는 테스트 검증 라이브러리의 검증 메소드
    - 검증하고 싶은 대상을 메소드 인자로 받는다.
    - 메소드 체이닝이 지원되어 isEqualTo와 같이 메소드를 이어서 사용가능

isEqualTo
    - assertj의 동등 비교 메소드
    - assertThat에 있는 값과 isEqualTo의 값을 비교해서 같을때만 성공

@RequestParam
    - 외부에서 API로 넘긴 파라미터를 가져오는 어노테이션
    
param
    - API 테스트 할 때 사용될 요청 파라미터를 설정
    - 단, 값은 String만 허용
    - 숫자/날짜 등의 데이터도 등록할 때 문자열로 변경해야 가능

jsonPath
    - JSON 응답값을 필드별로 검증할 수 있는 메소드
    - $를 기준으로 필드명 명시
