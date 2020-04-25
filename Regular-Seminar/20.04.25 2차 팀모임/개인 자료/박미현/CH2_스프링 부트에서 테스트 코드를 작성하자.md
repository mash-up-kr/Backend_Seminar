#2. 스프링 부트에서 테스트코드 작성하기

####1.테스트코드란?
#####1.1. TDD(Test Driven Development)
'테스트가 주도하는 개발'로 3가지 절차를 반복한다.

    1. 실패: 항상 실패하는 테스트 먼저 작성
    2. 통과: 테스트가 통과하는 코드 작성
    3. 리팩토링: 테스트를 통과한 코드 리팩토링(구현한 코드 중 중복처리, 개선 등의 수정).
                리팩토링 후 테스트 통과 여부 확인.
   
#####1.2.단위테스트
-순수하게 기능 단위의 테스트 코드를 작성(TDD의 첫 단계)을 의미.

#####1.3. 테스트코드 작성 프레임워크
-xUnit(JUnit-Java /  DBUnit-DB / CppUnit-C++)

***

####2. 테스트코드 작성
#####2.1. 메인 클래스 생성
```java
package com.jojoldu.book.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

__*@SpringBootApplication*__
: 스프링부트 자동 설정, 스프링Bean읽기 및 생성 자동. 해당 파일 위치부터 읽어가기에 포함된 클래스는 항상 프로젝트 최상단에 위치.

__*SpringApplication.run*__
:내장 WAS 실행(늘 같은 환경에서 스프링 부트를 배포 할 수 있다는 장점으로 사용 권장)

#####2.2. 테스트용 Controller 생성

```java
package com.jojoldu.book.springboot.web;

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

__*@RestController*__
:JSON반환 컨트롤러
__*@GetMapping*__
:/hello로 요청오면 -> 문자열 "hello"반환(return값)

#####2.3. 작성한 컨트롤러 테스트 코드

```java
package com.jojoldu.book.springboot.web;


import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

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
}
```

__*@RunWith(SpringRunner.class)*__
:스프링부트 테스트와 JUnit 사이에 연결자 역할(테스트 시 JUnit에 내장된 실행자 외 다른 실행자 실행시킴)

__*@WebMvcTest*__
:Web(Spring MVC)에 집중할 수 있음. @Controller, @ControllerAdvice 사용가능(@Service, @Repository 등 불가)

__*@Autowired*__
:스프링이 관리하는 Bean을 주입받음. 객체의 의존성을 가지는 부분에 선언하여 의존성 주입받음.

***

###3. Lombok

#####3.1. Lombok이란?

Java기반에서 VO, DTO, Entity작업을 중복수행하지 않고 수월하도록 도와주는 라이브러리.
Getter, Setter, 기본생성자, toString 등을 어노테이션으로 자동 생성가능.

#####3.2. 의존성 등록(build.gradle)
gradle 5에서의 설정
```java
dependencies {
    annotationProcessor 'org.projectlombok:lombok'
    implementation 'org.projectlombok:lombok'
    testAnnotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.projectlombok:lombok'
}
configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}
```
*프로젝트마다 build.gradle에 라이브러리 추가, Enable annotation processing 체크 반드시 해 주어야한다.

#####3.3. Controller 코드 Lombok적용하기

- src/main/java/com/jojoldu/book/springboot/web/dto/HelloResponseDto.java 
```java
package com.jojoldu.book.springboot.web.dto;


import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public class HelloResponseDto {

    private final String name;
    private final int amount;

}
```
__*@Getter*__
:(롬복)생성된 모든 필드의 get메소드 생성
__*@RequiredArgsConstructor*__
:(롬복)선언된 모든 final필드가 포함된 생성자 생성. @Autowired없이 DI(Dependency Injection)주입 가능.

- src/test/java/com/jojoldu/book/springboot/web/dto/HelloResponseDtoTest.java 
```java
package com.jojoldu.book.springboot.web.dto;

import org.junit.Test;

import static org.assertj.core.api.Assertions.assertThat;

public class HelloResponseDtoTest {

    @Test
    public void 롬복_기능_테스트() {
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
```
__*assertThat*__
:assertj라는 테스트 검증 라이브러리의 검증 메소드로 검증하고 싶은 대상을 메소드 인자로 받는다.

__*isEqualTo*__
:assertj의 동등 비교 메소드.

- src/main/java/com/jojoldu/book/springboot/web/HelloController.java
```java
    @GetMapping("/hello/dto")
    public HelloResponseDto helloDto(@RequestParam("name") String name,
                                     @RequestParam("amount") int amount) {
        return new HelloResponseDto(name, amount);
    }
```
__*@RequestParam()*__
:@RequestParam("가져올  데이터 이름")[데이터 타입][변수]. 외부에서 API로 넘긴 parameter를 가져오는 어노테이션.
Model객체를 이용하여 뷰로 값을 넘겨준다.

-  src/test/java/com/jojoldu/book/springboot/web/HelloControllerTest.java 
```java
@Test
    public void helloDto가_리턴된다() throws Exception {
        String name = "hello";
        int amount = 1000;

        mvc.perform(
                get("/hello/dto")
                .param("name", name)
                .param("amount", String.valueOf(amount)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name", is(name)))
                .andExpect(jsonPath("$.amount", is(amount)));
    }
```
__*param*__
:API테스트할 때 사용될 요청 파라미터 설정. String만 사용가능(데이터 모두 문자열 형식으로 변경)

__*jsonPath*__
:JSON 응답값을 필드별로 검증가능. $기준으로 필드명 작성.
***
###4. 더 알아보기
#####4.1. MockMvc란?

-브라우저에서 요청과 응답을 의미하는 객체.

-Controller 단위 테스트를 위해 활용한다.

-perform()메소드 활용해 get, put, post, delete에 대한 요청 진행가능.

-andExpect():요청에 대한 예측결과 검증

-andDo(): 결과에 대해 특정 작업 시행. parameter로 ResultHandler전달하여 다양한 동작 가능

-andReturn(): 결과 자체를 전달받음. Request, Response 정보를 반환받음.

#####4.2. DTO(Data Transaction Object)

계층간 데이터 교환을 위한 자바Beans(객체).   
`계층: Controller, View, Business Layer, Persistent Layer`   
DTO는 로직을 갖지 않는 순수한 데이터 개체로 속성, Getter, Setter 메소드만 가진 class.

#####4.3. Controller

사용자의 요청이 진입하는 지점(entry point).   
요청에 따라 처리 방법 결정(->실질적 처리는 서비스에서 담당)   
사용자에게 View를 응답으로 보내준다.   
대규모 서비스로 갈수록 처리해야 할 서비스가 많아져 요청에 따른 Controller를 만들어 각각 필요한 로직처리 위한 서비스 호출 ->역할분담
`MVC패턴 사용하면 개발비용, 유지보수 비용 감소`
