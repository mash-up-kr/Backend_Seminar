### 2.1 테스트 코드 소개

- TDD와 단위테스트는 다르다.
- TDD
    - 테스트주도개발
    - **RED** 항상 실패하는 테스트를 먼저 작성하라
    - **GREEN** 테스트가 통과하는 프로덕션 코드를 작성하고
    - **REFACTOR** 테스트가 통과하면 프로덕션 코드를 리팩토링한다.

- 단위테스트
    - 기능단위의 테스트 코드만을 작성하는 것
    - 이점
        - 단위테스트는 초기 문제 발견에 도움을 준다.
        - 코드를 리팩토링하거나 라이브러리 업그레이드 등에서 기존 기능이 올바르게 작동하는지 확인할 수 있음.
        - 기능에 대한 불확실성 감소
        - 단위테스트는 실제 문서를 제공함
        - 빠른 피드백이 가능하다.
        - 사람 눈으로 검증할 필요없이 자동검증이 가능하다.
        - 개발자가 만든 기능을 안전하게 보호해준다.
    - 대표적인 프레임워크
        - JUnit - Java
        - DBUnit - DB


### 2.2. Hello Contoller 테스트 코드 작성하기

- 패키지 추가 후 `Appication.java` 추가

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // 스프링부트의 자동 설정, 이 위치부터 설정을 읽어감-> 프로젝트의 최상단에 위치해야함
public class Application {
    public static void main(String[] args){
        SpringApplication.run(Application.class, args); //내장 WAS 실행됨
    }
}

```

- **내장 WAS를 사용하는 것이 권장됨** 
    - 톰캣을 설치할 필요없고, Jar로 실행하면됨 
    - **언제 어디서나 같은 환경에서 스프링부트를 배포**할 수 있기 때문
    - 내장 WAS 사용에 대한 성능상 이슈는 생각할 필요 적음

- 패키지 하단에 `web`패키지 만들고 `HelloController.java`추가
```java
@RestController //JSON을 반환하는 컨트롤러로 만들어줌
public class HelloController {

    @GetMapping("/hello") //Get 요청 받는 API
    public String hello(){
        return "hello";
    }
}
```

- 테스트(`/test/java/{패키지}/HelloControllerTest.java`)
```java
@RunWith(SpringRunner.class) //SpringRunner라는 스프링 실행자 사용해서 테스트 진행
@WebMvcTest(controllers = HelloController.class) //Web(Spring MVC)에 집중할 수 있는 어노테이션
public class HelloControllerTest {
    @Autowired//빈 주입
    private MockMvc mvc; //웹 API를 테스트할때 사용, 스프링 mvc 테스트의 시작점

    @Test
    public void returnHello() throws Exception{
        String hello = "hello";

        mvc.perform(get("/hello")) //get 요청
                .andExpect(status().isOk()) //status 검증
                .andExpect(content().string(hello)); //response content 검증
    }
}
```

- `returnHello()` Test code 실행 및 `Application.java`의 `main()` 실행으로 `localhost:8080/hello` 테스트하기  

### 2.3 롬복 소개 및 설치하기

- Getter, Setter, 기본생성자, toString()등을 어노테이션으로 자동 생성해줌
- `build.gradle`에 추가 -> sync
```gradle
compile('org.projectlombok:lombok')
```
- 롬복 플러그인 설치
- Ctrl + Shift + A -> Annotation Processors -> Enable Annotation Processing 체크

### 2.4 Hello Controller 코드 롬복으로 교체하기

- `java/{package}/web/dto/HelloResponseDto.java`
```java
@Getter //getter추가
@RequiredArgsConstructor //final 붙은 필드를 포함한 생성자 생성
public class HelloResponseDto {
    private final String name;
    private final int amount;
}
```
- 테스트(`/test/java/{패키지}/HelloResponseDtoTest.java`)

```java
import static org.assertj.core.api.Assertions.assertThat;

public class HelloResponseDtoTest {
    @Test
    public void testLombok() {
        //given
        String name = "test";
        int amount = 1000;
        //when
        HelloResponseDto dto = new HelloResponseDto(name, amount);
        //then
        assertThat(dto.getName()).isEqualTo(name); //assertj 테스트 검증 라이브러리의 검증메소드
        assertThat(dto.getAmount()).isEqualTo(amount);
    }
}
```
- 롬복의 @Getter 덕분에 `dto.getxxx()`가 가능
- 롬복의 @RequiredArgsConstructor 덕분에 `new HelloResponseDto(name, amount)`가 가능

- **assertj 가 Junit보다 좋은 이유**
    - Junit과 달리 assertj의 assertThat은() CoreMatchers 라이브러리가 추가적으로 필요없음
    - 자동완성이 확실히 지원됨

- `HelloController.java`에서 `GET /hello/dto`추가
```java
@RestController
public class HelloController {
    @GetMapping("/hello/dto")
    public HelloResponseDto helloDto(@RequestParam("name") String name, @RequestParam("amount") int amount) {
        return new HelloResponseDto(name, amount);
    }
}
```

- 테스트(`/test/java/{패키지}/HelloControllerTest.java`)

```java
import static org.hamcrest.Matchers.is; // 자동완성으로 못찾아서 직접 import 해줬음
@Test
    public void returnHelloDto() throws Exception{
        String name = "hello";
        int amount = 1000;

        mvc.perform(get("/hello/dto")
                .param("name",name)
                .param("amount",String.valueOf(amount))) //값은 String만 허용됨
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name",is(name))) //JSON 응답값을 $ 기준으로 필드별로 검증
                .andExpect(jsonPath("$.amount",is(amount)));
    }
```