## 2. 스프링 부트에서 테스트 코드를 작성하자

| 공부 날짜  | 공부한 페이지 수 |
| ---------- | ---------------- |
| 2020.04.07 | p.50 ~ p.77      |

최근 테스트 코드 작성 능력에 대한 중요성이 커지고있다.

여러 기업들에서 테스트 코드에 대한 능력을 요구하기도 하지만, 테스트 코드 작성을 할 때의 이점이 분명 존재하기 때문에 우리는 더욱더 테스트 코드 작성법을 알아야할 것이다.

## 2.1. 테스트 코드 소개

그렇다면 테스트 코드는 왜 중요하고 도대체 무엇일까? 

`TDD`와 `단위 테스트` 개념을 비교하면서 알아가도록하자.

### TDD

테스트가 주도하는 개발을 의미한다. **테스트 코드를 먼저 작성한다는 것이 포인트!**

**TDD의 단계?**

1. 실패하는 테스트를 작성한다
2. 테스트가 통과하는 production 코드를 작성한다.
3. 테스트가 통과하면 production 코드를 리팩토링한다.

### 단위 테스트는?

`순수한 테스트 코드 작성`만을 의미한다.

리팩토링을 하거나, 테스트 코드를 꼭 먼저 작성해야하는 것도 아니다.



TDD와 단위 테스트를 간단히 비교해보았다. 

이번 장에서는 단위 테스트를 중심으로 배워볼 예정이다!



### 테스트 코드의 장점

- 단위테스트는 초기 문제 발견에 도움을 준다.
- 추후 리팩토링이나 lib 업그레이드에 있어 기존 기능이 올바르게 작동하는지 확인 가능
- 불확실성 감소
- 단위테스트를 문서처럼 사용가능

솔직히 잘 와닿지 않는다! 아직 공부를 안해서 그렇겠지만,, 

그러던 도중 이 책의 저자분의 경험담에서 이마를 쳤다.

바로 **빠른 피드백**이라는 장점이다.

개발을 하다보면 코드작성하고 서버 구동하고 postman으로 테스트하고 print해서 눈으로 보고.. 오류나면 다시 코드 고치고 또 포스트맨 키고.. 이런 과정에서 늘 분노를 느꼈는데, 테스트 코드는 순식간에 이를 해결해준다는 것이다.

- 서버를 올렸다 내렸다 할 필요도, 
- postman에서 수 많은 url을 뒤적이며 http 요청을 보낼 필요도,
- 새로운 기능이 추가됐을때 기존의 모든 기능을 수동으로 테스트할 필요도 없어지는 것이다!

### 테스트 프레임워크

보통 (개발환경) Unit의 이름으로 자주 사용된다.

Java의 경우는 JUnit을 사용한다.  

## 2.2. Hello Contoller 테스트 코드 작성하기

1. **패키지 생성하기 [New => Package]**
   - 일반적으로 패키지명은 **웹 사이트 주소**의 역순으로 한다.
   - `springboot.book.jojoldu.com` => `com.jojoldu.book.springboot`
2. **Application Class 생성**
   - 메인 클래스가 된다. 
   - SpringBootApplication이 있는 위치부터 설정을 읽어나가므로, **프로젝트 최상단에** 위치해야한다.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // 스프링 부트의 자동 설정, 스프링 bean읽기와 생성을 모두 자동으로 설정
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args); // 내장 WAS 실행.
    }
}
```

> 내장 WAS는 어디서든 같은 환경에서 스프링 부트를 배포할 수 있게 해준다.

3. **컨트롤러 만들기**
   - **web** 패키지 생성하기 - `컨트롤러와 관련된 클래스가 담긴다`
   - **HelloController** 클래스 생성

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController // JSON을 반환하는 컨트롤러로 만들어준다.
public class HelloController {
    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }
}
```

4. **테스트 코드 작성하기**
   - 대상클래스이름Test.class

```java
@RunWith(SpringRunner.class) // 내장 실행자 외에 다른 실행자.
@WebMvcTest(controllers = HelloController.class) // MVC에만 집중하는 어노테이션.
public class HelloControllerTest {
    @Autowired // 빈 주입
    private MockMvc mvc; // API 테스트시 사용.
    
    @Test
    public void hello_return() throws Exception {
        String hello = "hello";
        
        mvc.perform(get("/hello"))
            .andExpect(status().isOk())
            .andExpect(content().string(hello));
    }
}
```

테스트 함수를 실행하면 검증을 위해 선언했던 

```java
.andExpect(status().isOk())
.andExpect(content().string(hello));
```

코드의 테스트를 진행한다. 테스트를 통과하면 Tests passed라고 뜬다!

> Application의 main을 실행시켜 `localhost:8080/hello`로 직접 확인해보자. 잘 뜰 것이다.

## 2.3. 롬복 소개 및 설치

롬복은 **자바 개발을 위한 필수 라이브러리**이다.

Getter나 Setter, 기본생성자, toString 등을 어노테이션으로 자동 생성해준다. 

인텔리제이에서는 플러그인으로 쉽게 설정 및 설치가 가능하다.

1. bulid.gradle의 dependencies에 다음 코드를 추가한다.

`compile('org.projectlombok:lombok')` 

2. `Ctrl + Shift + A` 단축키로 플러그인 검색을 한다

`plugins`입력

3. `lombok`을 검색하여 설치한후 인텔리 제이를 재시작한다.

## 2.4. Hello Controller 코드 롬복으로 교체하기 

기존의 코드를 lombok으로 리팩토링 해보자. 

리팩토링을 하기 전에, 만약 큰 규모의 프로젝트였다면 어땠을까? 모든 코드를 롬복으로 전환하고 모든 기능이 정상적으로 동작하는지 테스트하기란 쉽지 않을 것이다. 이럴때 테스트 코드가 빛을 발한다. 테스트 코드를 쭉 돌려보고 실패한 코드만 찾아가면 되기 떄문이다.

테스트 코드의 중요성도 다시 상기해보았으니 롬복을 이용해 코드를 전환해보자.

1. **Dto 코드 작성하기**
   - web 밑에 dto 패키지를 생성한다.

HelloResponseDto

```java
@Getter
@RequiredArgsConstuctor
public class HelloResponseDto {
    private final String name;
    private final int amount
}
```

2. **HelloReponseDto Test 코드 작성**
   - test 폴더에 작성한다.
   - 위치는 HelloResponseDto와 동일!

```java
import org.junit.Test;
import static org.assertj.core.api.Assertions.assertThat;

public class HelloResponseDeoTest {
    @Test
    public void test_lombok() {
        String name = "test";
        int amount = 1000;
        
        HelloResponseDto dto = new HelloResponseDto(name, amount);
        
        assertThat(dto.getName()).isEqualTo(name);
        assertThat(dto.getAmount()).isEqualTo(amount);
    }
}
```

> 해당 테스트를 통해 롬복의 Getter와 RequiredArgsConstructor가 잘 동작했다는 것을 알 수 있다.

3. **HelloController 코드 추가하기**

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @GetMapping("/hello/dto")
    public HelloResponseDto helloDto(@RequestParam("name") String name, @RequestParam("amount") int amount) {
        return new HelloResponseDto(name, amount);
    }
}
```

## 회고

테스트 코드가 좀 막연한 느낌이 있었는데 여기서 쓰다보니 감이 잡히긴 하는 것 같다. 노드에서도 테스트 코드 작성해보려한다.