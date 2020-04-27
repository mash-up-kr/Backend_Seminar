\* 본문은 <스프링부트와 aws로 혼자 구현하는 웹서비스>(2019, 이동욱, 프리렉)을 공부하고 정리한 내용입니다.

1\. Application 클래스, @SpringBootApplication, SpringApplication.run(Application.class, args)

\- **Application 클래스**가 프로젝트의 메인 클래스 역할을 한다. 이 위치를 기준으로 하향식으로 패키지를 탐색하며, 객체를 관리하는 컨테이너가 실행된다.

\- **@SpringBootApplication**이 있는 위치부터 설정을 읽어간다

\- **SpringApplication.run(Application.class,****args)**을 실행하면서 내장 was 실행

2\. 컨트롤러, web package

컨트롤러 관련한 것은 모두 web 패키지 안으로 들어간다. **컨트롤러**는 장고의 view와 비슷한 역할을 한다. 클라이언트의 요청은 먼저 컨트롤러로 진입하며, 요청에 따라 어떤 처리를 할지 결정한다. 사용자에게 뷰를 응답으로 보낸다 (항상 그런가?)

spring 및 springboot structure에 대해선 다음 글을 참고하면 된다.

[https://gmlwjd9405.github.io/2019/01/05/spring-directory-structure.html](https://gmlwjd9405.github.io/2019/01/05/spring-directory-structure.html)

컨트롤러를 비롯해 뷰, 모델에 대해 이해하려면 mvc 패턴에 대한 아래 글을 참고하면 된다.

[https://gmlwjd9405.github.io/2018/11/05/mvc-architecture.html](https://gmlwjd9405.github.io/2018/11/05/mvc-architecture.html)

3\. Hello Controller

\- **@RestController**: 컨트롤러를 제이슨을 반환하는 컨트롤러로 만들어줌.

-   JSON 변환시기본 생성자와 Getter 메소드가 없으면 JSON 컨버터는 어떻게 변환해야할지 알 수가 없다.
    
-   그래서 이 둘은 꼭 필요하니API 응답용 DTO에선 필수로추가해야 한다
    

\- **@GetMapping("/hello/dto"):** http method GET 요청을 받는 API를 만든다. 뷰의 요청 경로를 지정한다.

코드:

```
@RestController //JSON 을 반환하는 컨트롤러를 만들어준다.
public class HelloController {
// 컨트롤러 "hello" 리턴 시 해당하는 HTML 파일을 templates에서 찾아 자동으로 실행
    @GetMapping("/hello")//여기로 요청이 온다
    // /hello 로 요청이 오면 문자열 hello 를 반환
    public String hello() {
        return "hello"; //view의 역할을 한다
    }

    @GetMapping("/hello/dto")
    public HelloResponseDto helloDto(@RequestParam("name") String name, @RequestParam("amount") int amount) {
        return new HelloResponseDto(name, amount); 
    }
}
```

4\. Hello ControllerTest

\- **@RunWith(SpringRunner.class):** 테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킨다. 여기서는 SpringRunner란 스프링 실행자를 사용한다. -> 스프링부트 테스트와 JUnit사이에 연결자 역할을 한다

\* 실행자: 검색도 해봤는데 마땅한 결과를 못찾았고, 그냥 "이런 게 있구나" 하고 여길만한 개념인 것처럼 보인다

\- **@WebMvcTest(controllers \= HelloController.class)**: 스프링 테스트 어노테이션의 한 종류. Web(Spring MVC)에 집중할 수 있는 어노테이션. 더 자세한 정보는 생략 (책 62쪽 참고)

\*어노테이션: [https://zamezzz.tistory.com/171](https://zamezzz.tistory.com/171)

\- **@Autowired**: 스프링이 관리하는 빈을 주입받는다. 뒤에도 매우 많이 나오므로 꼭 숙지!

\* **빈**: 일반적으로 자바로 작성된 컴포넌트들 즉, 클래스(Class)를 말한다. JSP 프로그래밍에는 DTO(Data Transfer Object)나 DAO(Data Access Object)클래스의 객체를 JSP페이지에서 사용하기 위해 사용한다. 자바빈을 이용하여 프로그래밍을 하면 클래스의 객체 선언과 비즈니스 로직 등을 <% %> 스크립틀릿 영역에서 작성하지 않아서 가독성이 좋다.

아래 첫번째 링크가 위 설명의 출처며, 빈을 사용했을 때와 그렇지 않았을 때 코드의 차이를 볼 수 있다. 두번째 링크는 또 다른 참고 자료.

[https://developer-syubrofo.tistory.com/21](https://developer-syubrofo.tistory.com/21) 

[https://m.blog.naver.com/start3535/30190419527#](https://m.blog.naver.com/start3535/30190419527#)

\- **private MockMvc** **mvc**: org.springframework.test.web.servlet.MockMvc에 있던 클래스. 웹 API(http GET, POST)를 테스트할 떄 사용하며, 스프링 MVC 테스트의 시작점.

\-> mvc.perform(get("/hello")): MockMvc를 통해 /hello 주소로 http get 요청. 체이닝이 지원되어 여러 검증 기능을 이어서 선언할 수 있다.

코드:

```
@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {
    @Autowired
    private MockMvc mvc;

    @Test
    public void return_hello() throws Exception {
        String hello = "hello";
        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
```

5\. lombok 플러그인

목적: 기존 HelloController 코드를 롬복으로 전환해보고 문제가 생기는지 테스트 코드를 통해 알아본다.

web.dto(Data Trasfer ) 패키지: 모든 응답은 여기에 들어간다!

HelloResponseDto 코드로 먼저 실험해본다.

\- @Getter: 선언된 모든 필드의 get 메소드를 생성해준다. 코드 없이 어노테이션 하나로 이게 해결된단게 얼마나 경이로운가!

\- @RequiredArgsConstructor: final 필드가 있는 모든 필드에 대해 생성자를 만들어줌. 없는 필드는 안됨. 이 역시 어노테이션의 경이로운 역할!

코드: 

```
@Getter
@RequiredArgsConstructor
public class HelloResponseDto {
    private final String name;
    private final int amount;
}
```

\- param, jsonPath는 쉬우니까 책 66~7쪽 참고하자. 메모는 패스

\[번외\]

1\. 의존성이란?

내가 알고 있는 내용 그대로가 맞음. 내가 진짜 공부해야 하는건 이것의 개념이 아닌 것

개념은 아래 링크를 참고.

[https://tony-programming.tistory.com/entry/Dependency-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%9D%B4%EB%9E%80](https://tony-programming.tistory.com/entry/Dependency-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%9D%B4%EB%9E%80)

아래 글 역시 의존성에 대한 글이다. 프로젝트에 들어가는 모듈, API가 언제 어떻게 업데이트 됐는지 우리는 직접 확인하지 않으면 모르니까, 이걸 외부 의존성 관리 툴이 알려준다는 내용이다.

[https://medium.com/@miles3898/%EC%9D%98%EC%A1%B4%EC%84%B1-%EA%B4%80%EB%A6%AC-%EB%8F%84%EA%B5%AC-dependency-manager-612047ced556](https://medium.com/@miles3898/%EC%9D%98%EC%A1%B4%EC%84%B1-%EA%B4%80%EB%A6%AC-%EB%8F%84%EA%B5%AC-dependency-manager-612047ced556)

**프로젝트가 사용하고 있는 외부 라이브러리들을 남들이 알도록 하는 것**이 기본적인 목표라고 한다. 다음은 내가 열심히 읽었던 <백엔드면 이정도는 해야한다> 시리즈의 의존성 편 글이다. 이제 다 이해가 되는.

[https://velog.io/@city7310/%EB%B0%B1%EC%97%94%EB%93%9C%EA%B0%80-%EC%9D%B4%EC%A0%95%EB%8F%84%EB%8A%94-%ED%95%B4%EC%A4%98%EC%95%BC-%ED%95%A8-8.-%EC%9D%98%EC%A1%B4%EC%84%B1-%EA%B4%80%EB%A6%AC-%EB%8F%84%EA%B5%AC-%EA%B2%B0%EC%A0%95](https://velog.io/@city7310/%EB%B0%B1%EC%97%94%EB%93%9C%EA%B0%80-%EC%9D%B4%EC%A0%95%EB%8F%84%EB%8A%94-%ED%95%B4%EC%A4%98%EC%95%BC-%ED%95%A8-8.-%EC%9D%98%EC%A1%B4%EC%84%B1-%EA%B4%80%EB%A6%AC-%EB%8F%84%EA%B5%AC-%EA%B2%B0%EC%A0%95)

2\. 나는 설정 들어가서 컴파일러를 internal JRE로 바꿔야만 컴파일, 빌드가 됨. 이유를 모르겠음. 왜 내가 자체적으로 설치한 자바는 안되는지...

3\. gradle 다운그레이드 안했던 에러 내용

[https://github.com/jojoldu/freelec-springboot2-webservice/issues/2](https://github.com/jojoldu/freelec-springboot2-webservice/issues/2)
