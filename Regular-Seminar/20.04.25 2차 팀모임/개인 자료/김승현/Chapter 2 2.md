# Chapter 2: 스프링 부트에서 테스트 코드 작성하기

Created: Mar 20, 2020 7:07 PM
Tags: dto, lombok

## 2.3 롬복?

- 자바 개발할 때 자주 사용하는 코드 Getter, Setter, 기본 생성자, toString 등을 어노테이션으로 자동 생성해 준다.

## 2.4 Hello Controller 코드를 롬복으로 전환하기

- web 패키지에 dto 패키지를 추가한 후 **모든 응답 Dto는 dto 패키지에 추가**하도록 한다.
- Dto?
    - Data transfer object
    - 데이터 전송에 사용하는 객체로 View 단에 사용하거나 다른 비즈니스 로직에 전송할 때 사용한다.

```java
package example.org.practice.web.dto;

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
    - 선언된 모든 필드의 get 메소드를 생성해 준다.
- `RequiredArgsConstructor`
    - 선언된 모든 final 필드가 포함된 생성자를 생성해 준다. final이 없는 필드는 생성자에 포함되지 않는다.

```java
package example.org.practice.web.dto;

import org.junit.Test;
import static org.assertj.core.api.Assertions.assertThat;

// HelloResponseDto의 테스트 클래스
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

- `assertThat`
    - assertj라는 테스트 검증 라이브러리의 검증 메소드이다. 검증하고 싶은 대상을 메소드 인자로 받는다.
    - 메소드 체이닝이 지원되어 isEqualTo와 같이 메소드를 이어서 사용할 수 있다.
- `isEqualTo`
    - assertj의 동등 비교 메소드이다.
    - asserThat에 있는 값과 isEqualTo의 값을 비교해서 같을 때만 성공이다.
- assertj vs. Junit
    - 추가적으로 라이브러리가 필요하지 않다.
    - 자동 완성이 좀 더 확실하게 지원된다.

- HelloController에서 ResponseDto를 사용하도록 한다.

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

    @GetMapping("/hello/dto")
    public HelloResponseDto helloDto(@RequestParam("name") String name,
                                     @RequestParam("amount") int amount) {

        return new HelloResponseDto(name, amount);
    }

}
```

- `@RequestParam`
    - 외부에서 API로 넘긴 파라미터를 가져온다.
    - 여기서는 외부에서 `name (@RequestParam("name"))`이란 이름으로 넘긴 파라미터를 메소드 파라미터 name(String name)에 저장하게 된다.

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

    @WithMockUser(roles = "USER")
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
}
```

- `param`
    - API를 테스트할 때 사용될 요청 파라미터를 설정한다. 단, 값은 String만 허영된다.
    - 따라서 숫자나 날짜 등의 데이터도 등록할 때는 문자열로 변경해야만 가능하다.
- `jsonPath`
    - JSON 응답값을 필드별로 검증할 수 있는 메소드이다. $를 기준으로 필드명을 명시한다.
    - 여기서는 name과 amount를 검증하므로 $.name, $amount로 검증한다.