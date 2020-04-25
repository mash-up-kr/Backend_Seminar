# Chapter 5: 스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현하기

Created: Apr 20, 2020 8:14 PM
Tags: Annotation

## 5.4 어노테이션 기반으로 개선하기

- 같은 코드에 반복되면 유지 보수성이 떨어질 수 있다.
- IndexController에서 세션 값을 가져오는 부분이 반복될 수 있으므로, 이 부분을 **메소드 인자로 세션 값을 바로 받을 수 있도록** 변경한다.
- config.auth 패키지에 `@LoginUser` 어노테이션 생성

```java
// ~/config/auth/LoginUser

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {
}
```

- `@Target(ElementType.PARAMETER)`
    - 이 어노테이션이 생성될 수 있는 위치를 지정
    - PARAMETER로 지정했으므로 메소드의 파라미터로 선언된 객체에서만 사용 가능
- `@interface`
    - 이 파일을 어노테이션 클래스로 지정
    - LoginUser라는 이름의 어노테이션이 생성된 것과 같다.

```java
// ~config/auth/LoginUserArgumentResolver

import lombok.RequiredArgsConstructor;
import org.springframework.core.MethodParameter;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;

import javax.servlet.http.HttpSession;

@RequiredArgsConstructor
@Component
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {

    private final HttpSession httpSession;

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        boolean isLoginUserAnnotation = parameter.getParameterAnnotation(LoginUser.class) != null;
        boolean isUserClass = SessionUser.class.equals(parameter.getParameterType());
        return isLoginUserAnnotation && isUserClass;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        return httpSession.getAttribute("user");
    }

}
```

- `supportsParameter()`
    - 컨트롤러 메서드의 특정 파라미터를 지원하는지 판단
- `resolveArgument()`
    - 파라미터에 전달할 객체를 생성
    - 여기서는 세션에서 객체를 가져온다.

→ 스프링에서 인식될 수 있도록 WebMvcConfigurer에 추가한다. config 패키지에 WebConfig 클래스를 생성하여 설정을 추가한다.

- 설정을 끝낸 후 IndexController의 코드에서 반복되는 부분을 모두 `@LoginUser`로 개선

```java
@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;
    private final HttpSession httpSession;

    @GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user) {
        model.addAttribute("posts", postsService.findAllDesc());

        if(user != null) {
            model.addAttribute("userName", user.getName());
        }
        return "index";
    }

    @GetMapping("/posts/save")
    public String postsSave() {
        return "posts-save";
    }

    @GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model) {
        PostsResponseDto dto = postsService.findById(id);
        model.addAttribute("post", dto);

        return "posts-update";
    }

}
```

- `@LoginUser sessionUser user`
    - 기존에 `httpSession.getAttribute("user")`로 가져오던 세션 정보 값이 개선
    - 어느 컨트롤러든 `@LoginUser` 만 사용하면 세션 정보를 가져올 수 있다.

## 5.5 세션 저장소로 데이터베이스 사용하기

- 현재 서비스는 애플리케이션을 재실행하면 로그인이 풀린다.
    - 이는 세션이 **내장 톰캣 메모리에 저장**되기 때문이다. 내장 톰캣처럼 메모리에 저장되기 때문에 애플리케이션 실행 시 실행되는 구조에서는 항상 초기화된다.
    - 즉, 배포할 때마다 톰캣이 재시작된다.
- 2대 이상의 서버에서 서비스하고 있다면 **톰캣마다 세션 동기화** 설정이 필요하다. 일반적으로 세 가지 방식이 있다.
    - 톰캣 세션을 사용한다.
        - 별다른 설정을 하지 않을 때 기본적으로 선택되는 방식으로, 2대 이상의 WAS가 구동되는 환경에서는 톰캣들 간 세션 공유를 위한 추가 설정이 필요하다.
    - MySQL과 같은 데이터베이스를 세션 저장소로 사용한다. → 이 방식 사용!
        - 공용 세션을 사용할 수 있는 가장 쉬운 방법으로 로그인 요청마다 DB IO가 발생해 성능상 이슈가 발생할 수 있다. 로그인 요청이 많이 없는 용도에서 사용한다.
        - build.gradle에 `compile('org.springframework.session:spring-session-jdbc')` 추가
        - application.properties에 `spring.session.store-type=jdbc` 추가
        - JPA로 인해 세션 테이블이 자동 생성된다.
    - Redis, Memcached와 같은 메모리 DB를 세션 저장소로 사용한다.
        - B2C 서비스에서 가장 많이 사용하는 방식으로 실제 서비스로 사용하기 위해서는 Embedded Redis와 같은 방식이 아닌 외부 메모리 서버가 필요하다.

## 5.6 네이버 로그인

- `application-oauth.properties`에 클라이언트 ID와 클라이언트 비밀, Common-OAuth2Provider에서 설정해 주던 값들을 입력

```java
# registration
spring.security.oauth2.client.registration.naver.redirect-uri={baseUrl}/{action}/oauth2/code/{registrationId}
spring.security.oauth2.client.registration.naver.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.naver.scope=name,email,profile_image
spring.security.oauth2.client.registration.naver.client-name=Naver

# provider
spring.security.oauth2.client.provider.naver.authorization-uri=https://nid.naver.com/oauth2.0/authorize
spring.security.oauth2.client.provider.naver.token-uri=https://nid.naver.com/oauth2/token
spring.security.oauth2.client.provider.naver.user-info-uri=https://openapi.naver.com/v1/nid/me
spring.security.oauth2.client.provider.naver.user-name-attribute=response
```

- `user_name_attribute=response`
    - 기준이 되는 user_name의 이름을 네이버에서는 response로 해야 한다.
    - 네이버의 회원 조회 시 반한되는 JSON 형태 때문
- 스프링 시큐리티에서는 하위 필드를 명시할 수 없고 최상위 필드들만 user_name으로 지정 가능하다.
    - 하지만 네이버의 응답값 최상의 필드는 resultcode, message, response이다.
    - 이런 이유로 스프링 시큐리티에서 인식 가능한 필드는 세 개중 하나로 골라야 한다.

### 스프링 시큐리티 설정 등록

- OAuthAttributes에 **네이버인지 판단하는 코드와 네이버 생성자** 추가

```java
@Getter
public class OAuthAttributes {
		...
    public static OAuthAttributes of(String registrationId,
                                     String userNameAttributeName,
                                     Map<String, Object> attributes) {
        if("naver".equals(registrationId)) {
            return ofNaver("id", attributes);
        }
        return ofGoogle(userNameAttributeName, attributes);
    }

	  ...

    private static OAuthAttributes ofNaver(String userNameAttributeName, Map<String, Object> attributes) {
        Map<String, Object> response = (Map<String, Object>) attributes.get("response");

        return OAuthAttributes.builder()
                .name((String) response.get("name"))
                .email((String) response.get("email"))
                .picture((String) response.get("profile_image"))
                .attributes(response)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    ...
}
```

## 5.7 기존 테스트에 시큐리티 적용하기

- 기존 API 테스트 코드들이 인증에 대한 권한을 받지 못해 API를 인증된 사용자만 호출할 수 있도록 시큐리티 옵션이 활성화된 상태에서는 수정이 필요하다. → 테스트 코드마다 인증한 사용자가 호출한 것처럼 작동하도록

- 문제 1. `CustomOAuth2UserService`를 찾을 수 없음
    - CustomOAuth2UserService를 생성하는 데에 필요한 **소셜 로그인 관련 설정 값들이 없기 때문에** 발생
        - main 환경과 test 환경의 차이 때문 → 테스트 환경을 위한 [application.properties](http://application.properties) 생성

```java
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.h2.console.enabled=true
spring.session.store-type=jdbc

# Test OAuth

spring.security.oauth2.client.registration.google.client-id=test
spring.security.oauth2.client.registration.google.client-secret=test
spring.security.oauth2.client.registration.google.scope=profile,email 
```

- 문제 2. 302 Status Code
    - 스프링 시큐리티 설정 때문에 **인증되지 않은 사용자의 요청은 이동**시키기 때문 → **임의로 인증된 사용자를 추가**
    - 스프링 시큐리티 테스트를 위한 도구를 지원하는 spring-security-test를 build.gradle에 추가
    - `testCompile("org.springframework.security:spring-security-test")`
    - PostsApiControllerTest의 테스트 메소드에 **임의 사용자 인증** 추가

    ```java
    @Test
    @WithMockUser(roles="USER")
    public void Posts_등록된다() throws Exception {
    	...
    }
    ```

    - `@WithMockUser(roles="USER")`: 인증된 모의 사용자를 만들어서 사용한다. roles에 권한을 추가할 수 있다. → 이 어노테이션으로 인해 ROLE_USER 권한을 가진 사용자가 API를 요청하는 것과 동일한 효과
    - `@WithMockUser`는 MockMvc에서만 작동하므로 `@SpringBootTest`에서 MockMvc 사용할 수 있도록 한다.

    ```java
    import org.springframework.http.MediaType;
    import org.springframework.security.test.context.support.WithMockUser;
    import org.springframework.test.web.servlet.MockMvc;
    import org.springframework.test.web.servlet.setup.MockMvcBuilders;
    import org.springframework.web.context.WebApplicationContext;
    import static org.springframework.security.test.web.servlet.setup.SecurityMockMvcConfigurers.springSecurity;
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.put;
    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

    @RunWith(SpringRunner.class)
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    public class PostsApiControllerTest {

        ...

        @Autowired
        private WebApplicationContext context;

        private MockMvc mvc;

        @Before
        public void setup() {
            mvc = MockMvcBuilders
                    .webAppContextSetup(context)
                    .apply(springSecurity())
                    .build();
        }

        ...

        @Test
        @WithMockUser(roles="USER")
        public void Posts_등록된다() throws Exception {
    				...
            //when
            mvc.perform(post(url)
                    .contentType(MediaType.APPLICATION_JSON_UTF8)
                    .content(new ObjectMapper().writeValueAsString(requestDto)))
                    .andExpect(status().isOk());

            //then
            List<Posts> all = postsRepository.findAll();
            assertThat(all.get(0).getTitle()).isEqualTo(title);
            assertThat(all.get(0).getContent()).
                                            isEqualTo(content);

        }

        @Test
        @WithMockUser(roles = "USER")
        public void Posts_수정된다() throws Exception {
    				...
            //when
            mvc.perform(put(url)
                    .contentType(MediaType.APPLICATION_JSON_UTF8)
                    .content(new ObjectMapper().writeValueAsString(requestDto)))
                    .andExpect(status().isOk());

            //then
            List<Posts> all = postsRepository.findAll();
            assertThat(all.get(0).getTitle())
                                    .isEqualTo(expectedTitle);
            assertThat(all.get(0).getContent())
                                    .isEqualTo(expectedContent);
        }

    }
    ```

    - `@Before`: 매번 테스트가 시작되기 전에 MockMvc 인스턴스 생성
    - `mvc.perform`: 생성된 MockMvc 통해 API를 테스트한다. 본문 영역은 문자열로 표현하기 위해 ObjectMapper를 통해 문자열 JSON으로 변환
- 문제 3. `@WebMvcTest`에서 CustomOAuth2UserService를 찾을 수 없음
    - HelloControllerTest에서 사용하는 `@WebMvcTest`는 CustomOAuth2UserService를 스캔하지 않기 때문에 발생하는 문제이다.
    - `@WebMvcTest`는 WebSecurityConfigurerAdapter, WebMvcConfigurer를 비롯한 `@ControllerAdvice`, `@Controller`를 읽는다. `@Repository`, `@Service`, `@Component`는 스캔 대상이 아니다.
    - SecurityConfig를 읽었지만 이를 생성하기 위해 필요한 CustomOAuth2UserService는 읽을 수 없어 에러 발생 → 스캔 대상에서 SecurityConfig 제거

    ```java
    @WebMvcTest(controllers = HelloController.class,
                excludeFilters = {
                @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = SecurityConfig.class)
                })
    ```

    - 마찬가지로 `@WebMockUser` 사용해서 가짜로 인증된 사용자 생성
    - 이후 발생하는 `java.lang.IllegalArgumentException: At least one JPA metamodel must be present!` 에러는 `@EnableJpaAuditing`으로 인해 발생한다. ← 사용하기 위해서는 최소 하나의 `@Entity` 클래스가 필요하지만 `@WebMvcTest`이므로 당연히 없기 때문이다.
    - `@EnableJpaAuditing`이 `@SpringBootApplication`과 함께 있어 `@WebMvcTest`에서 스캔하므로 둘을 분리한다. → Application.java에서 `@EnableJpaAuditing` 제거
    - config 패키지에 JpaConfig 생성 후 `@EnableJpaAuditing` 추가