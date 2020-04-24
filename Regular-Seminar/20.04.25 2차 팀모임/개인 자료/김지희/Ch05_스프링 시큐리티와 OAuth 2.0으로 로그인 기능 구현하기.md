# 5. 스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현하기

## 진도 기록

|공부 날짜|공부한 페이지 수|
|---|---|
|2020년 04월|163p ~ 224p|

## 5.1 스프링 시큐리티와 스프링 시큐리티 Oauth2 클라이언트

**스프링 시큐리티**는 막강한 인증과 인가 기능을 가진 프레임워크이다.

스프링 시큐리티와 OAuth 2.0을 구현한 구글 로그인을 연동하여 로그인 기능을 만들어 보자.

**많은 서비스에서 소셜 로그인을 사용하는 이유?**

로그인을 직접 구현하면 다음을 전부 구현해야 한다.

* 로그인 시 보안
* 회원가입 시 이메일 혹은 전화번호 인증
* 비밀번호 찾기
* 비밀번호 변경
* 회원정보 변경

OAuth 로그인 구현 시 구글, 네이버 등에 맡기고 서비스 개발에 집중할 수 있다.

**스프링 부트 1.5방식과 2.0 방식**

스프링 부트 1.5 방식에서는 url 주소를 모두 명시해야 하지만, 2.0 방식에서는 client 인증 정보만 입력하면 된다.
**2.0버전이 되면서** 1.5버전에서 직접 입력했던 값들이 모두 `enum`으로 대체되었다.   
CommonOAuth2Provider 라는 enum이 새롭게 추가되어 구글, 깃허브, 페이스북, 옥타의 기본 설정값을 모두 제공한다.


## 5.2 구글 서비스 등록

구글 서비스에서 발급된 인증 정보(clientId와 clientSecret)를 통해서 로그인 기능과 소셜 서비스 기능을 사용할 수 있다. 

[구글 클라우드 플랫폼 주소](https://console.cloud.google.com) 이 링크로 들어가서 신규 서비스를 생성한다. 자세한 방법은 책을 참고하자.

구글 서비스 등록이 끝났다면 src/main/resources/ 디렉토리에 application-oauth.properties 파일을 생성 한다.

`application-oauth.properties`에 클라이언트ID와 클라이언트 보안 비밀 코드를 등록한다.

```
spring.security.oauth2.client.registration.google.client-id=클라이언트 ID
spring.security.oauth2.client.registration.google.client-secret=클라이언트 보안 비밀
spring.security.oauth2.client.registration.google.scope=profile,email
```

**.gitignore 등록**

보안이 중요한 정보들인 클라이언트ID와 클라이언트 보안 비밀이 깃허브에 올라가는 것을 방지하기 위해 `.gitignore`에 다음 코드 추가
```
application-oauth.properties
```

## 5.3 구글 로그인 연동하기

**순서**를 정리해보자면

1. `User` 클래스: 사용자 정보를 담당할 **도메인** 
   * domain 아래 user패키기 아래 User 클래스 생성

2. `Role` 클래스: 각 사용자의 권한을 관리할 **Enum 클래스**

3. `UserRepository` 클래스: **User의 CRUD**를 책임질 클래스

> User 엔티티 관련 코드 모두 작성 완료, 본격적으로 시큐리티 설정 진행

4. `SecurityConfig` 클래스: OAuth 라이브러리를 이용한 소셜 로그인 설정 코드 작성    
   * springboot 패키지 아래 config.auth 패키지 아래에 생성)

5. `CustomOAth2UserService` 클래스: 구글 로그인 이후 가져온 사용자의 정보(email, name, picture 등)들을 기반으로 **가입 및 정보수정, 세션 저장** 등의 기능을 지원

6. `OAuthAttributes` 클래스: OAuth의 **dto클래스**
   * config.auth.dto 패키지 생성 후 안에 클래스 생성

7. `SessionUser` 클래스: 인증된 사용자 정보만 필요
   * config.auth.dto 패키지에 생성


### 1. User 클래스

```
//import문 생략

@Getter
@NoArgsConstructor
@Entity
public class User extends BaseTimeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String email;

    @Column
    private String picture;

    @Enumerated(EnumType.STRING) // (1)
    @Column(nullable = false)
    private Role role;

    @Builder
    public User(String name, String email, String picture, Role role) {
        this.name = name;
        this.email = email;
        this.picture = picture;
        this.role = role;
    }

    public User update(String name, String picture) {
        this.name = name;
        this.picture = picture;

        return this;
    }

    public String getRoleKey(){
        return this.role.getKey();
    }
}
```
* `@Enumerated(EnumType.STRING)`
  * JPA로 데이터베이스로 저장할 때, Enum 값을 어떤 형태로 저장할지 결정
  * 기본적으로는 int로 된 숫자가 저장
  * 숫자로 저장되면 DB로 확인할 때 무슨 코드를 의미하는지 알 수가 없으므로 문자열(EnumType.STRING)로 저장될 수 있도록 선언

### 2. Role 클래스
```
import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum Role {

    GUEST("ROLE_GUEST", "손님"),
    USER("ROLE_USER", "일반 사용자");

    private final String key;
    private final String title;
}
```
* 스프링 시큐리티에서는 권한 코드에 항상 `ROLE_`이 앞에 있어야만 한다.

### 3. UserRepository 클래스
```
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}
```
* `findByEmail` : 소셜 로그인으로 반환되는 값 중 email을 통해, 처음 가입자인지 유무를 판단하기 위한 메소드

### 스프링 시큐리티 설정

`build.gradle`에 스프링 시큐리티 관련 의존성 추가
```
compile('org.springframework.boot:spring-boot-starter-oauth2-client')
// 클라이언트 입장에서 소셜 기능 구현 시 필요한 의존성
```

### 4. SecurityConfig 클래스

`config.auth` 패키지 : 앞으로 시큐리티 관련 클래스는 모두 이곳에
```
import lombok.RequiredArgsConstructor;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable().headers().frameOptions().disable()
                .and()
                    .authorizeRequests()
                    .antMatchers("/","/css/**","/images/**","/js/**","/h2-console/**").permitAll()
                    .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                    .anyRequest().authenticated()
                .and()
                    .logout()
                        .logoutSuccessUrl("/")
                .and()
                    .oauth2Login()
                        .userInfoEndpoint()
                            .userService(customOAuth2UserService);
    }
}
```
* `@EnableWebSecurity`
  * Spring Security 설정들을 활성화
* `.csrf().disable().headers().frameOptions().disable()`
  * h2-console 화면을 사용하기 위해 해당 옵션들을 disable
* `authorizeRequests`
  * URL별 권한 관리를 설정하는 옵션의 시작점
  * authorizeRequests가 선언되어야만 antMatchers 옵션을 사용 가능
* `antMatchers`
  * 권한 관리 대상을 지정하는 옵션
  * ”/” 등 지정된 URL 들은 permitAll() 옵션을 통해 전체 열람 권한을 주었다.
  * “/api/v1/**” 주소를 가진 API는 USER 권한을 가진 사람만 가능하도록 설정
* `anyRequest`
  * 설정된 값들 이외 나머지 URL들
  * 여기서는 authenticated()을 추가하여 나머지 URL들은 모두 인증된 사용자들에게만 허용하도록
  * 인증된 사용자(로그인한 사용자) 의미
* `logout().logoutSuccessUrl("/")`
  * 로그아웃 기능에 대한 여러 설정의 진입점
  * 로그아웃 성공 시 / 주소로 이동
* `oauth2Login()`
  * OAuth 2 로그인 기능에 대한 여러 설정의 진입점
* `userInfoEndpoint()`
  * OAuth 2 로그인 성공 이후 사용자 정보를 가져올 때의 설정들을 담당
* `userService`
  * 소셜 로그인 성공 시 후속 조치를 진행할 UserService 인터페이스의 구현체를 등록

### 5. CustomOAth2UserService 클래스
```
//import문 생략

@RequiredArgsConstructor
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {
    private final UserRepository userRepository;
    private final HttpSession httpSession;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        String registrationId = userRequest
                .getClientRegistration()
                .getRegistrationId();
        String userNameAttributeName = userRequest
                .getClientRegistration()
                .getProviderDetails()
                .getUserInfoEndpoint()
                .getUserNameAttributeName();

        OAuthAttributes attributes = OAuthAttributes
                .of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);
        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(
                Collections.singleton(
                        new SimpleGrantedAuthority(user.getRoleKey())
                ),
                attributes.getAttributes(),
                attributes.getNameAttributeKey()
        );
    }
    private User saveOrUpdate(OAuthAttributes attributes) {
        User user = userRepository.findByEmail(attributes.getEmail())
                .map(entity -> entity.update(attributes.getName(),
                        attributes.getPicture()))
                .orElse(attributes.toEntity());
        return userRepository.save(user);
    }
}
```
* `registrationId`
  * 현재 로그인 진행 중인 서비스를 구분하는 코드
  * 지금은 구글만 사용하는 불필요한 값, 이후 네이버 로그인 연동 시 네이버/구글 로그인 구분하기 위해 사용
* `getUserNameAttributeName`
  * OAuth2 로그인 진행 시 키가 되는 필드값 (Primary Key)
  * 이후 네이버 로그인과 구글 로그인을 동시 지원시 사용
* `OAuthAttributes`
  * OAuth2userService를 통해 가져온 OAuth2User의 attribute를 담을 클래스
  * 이후 네이버 등 다른 소셜 로그인도 이 클래스를 사용
  * 바로 아래에서 이 클래스의 코드가 나오니 차례로 생성
* `SessionUser`
  * 세션에 사용자 정보를 저장하기 위한 Dto 클래스
  * 왜 User 클래스를 쓰지 않고 새로 만들어서 쓰는지 의문...?

### 6. OAuthAttributes 클래스
```
// import문 생략

@Getter
public class OAuthAttributes {
    private Map<String, Object> attributes;
    private String nameAttributeKey;
    private String name;
    private String email;
    private String picture;

    @Builder
    public OAuthAttributes(Map<String, Object> attributes,
                           String nameAttributeKey, String name,
                           String email, String picture) {
        this.attributes = attributes;
        this.nameAttributeKey = nameAttributeKey;
        this.name = name;
        this.email = email;
        this.picture = picture;
    }

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes) {
            return ofGoogle(userNameAttributeName, attributes);
        }

    private static OAuthAttributes ofGoogle(String userNameAttributeName,
                                            Map<String, Object> attributes) {
        return OAuthAttributes.builder()
                .name((String) attributes.get("name"))
                .email((String) attributes.get("email"))
                .picture((String) attributes.get("picture"))
                .attributes(attributes)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    public User toEntity() {
        return User.builder()
                .name(name)
                .email(email)
                .picture(picture)
                .role(Role.GUEST)
                .build();
    }
}
```
* `of()`
  * OAuth2User에서 반환하는 사용자 정보는 Map이기 때문에 값 하나하나를 변환해야만 한다.
* `toEntity()`
  * User 엔티티를 생성
  * OAuthAttribute에서 엔티티를 생성하는 시점은 처음 가입할 때
  * 가입할 때의 기본 권한을 GUEST로 주기 위해서 role 빌더값에는 Role.GUEST를 사용

### 7. SessionUser 클래스

```
import lombok.Getter;
import java.io.Serializable;

@Getter
public class SessionUser implements Serializable {
    private String name;
    private String email;
    private String picture;

    public SessionUser(User user) {
        this.name = user.getName();
        this.email = user.getEmail();
        this.picture = user.getPicture();
    }
}
```

시큐리티 설정 모두 완료 !

### 로그인 테스트

`idex.mustache`

스프링 시큐리티가 잘 적용되었는지 확인 위해 화면에 로그인 버튼 + 로그인 성공 시 사용자 이름 보여주도록 추가
```
<h1>스프링 부트로 시작하는 웹 서비스</h1>
    <div class="col-md-12">
        <!-- 로그인 기능 영역 -->
        <div calss="row">
            <div class="col-md-6">
                <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
                {{#userName}}
                    Logged in as: <span id="user">{{userName}}</span>
                    <a href="/logout" class="btn btn-info active" role="button">Logout</a>
                {{/userName}}
                {{^userName}} 
                    <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">Google Login</a> 
                {{/userName}}
            </div>
        </div>
    </div>
    <br>
    <!--목록 출력 영역-->
```
index.mustache에서 userName을 사용할 수 있게 하기 위해서 `IndexController`에서 userName을 model에 저장하는 코드를 추가해준다. 
```
public class IndexController {
    ...
    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("posts", postsService.findAllDesc());

        SessionUser user = (SessionUser) httpSession.getAttribute("user");
        if(user != null) { 
            model.addAttribute("userName", user.getName());
        }
        return "index";
    }
}
```
기본적인 구글 로그인, 로그아웃, 회원가입, 권한관리 기능 모두 구현 완료 !

## 5.4 어노테이션 기반으로 개선하기

**같은 코드가 반복**되는 부분은 유지보수성이 떨어지며, 수정할 때 문제가 발생할 확률이 높다. 

* **IndexController에서 세션값을 가져오는 부분**
  * index 메소드 외에 다른 곳에서 세션값이 필요할 때마다 직접 세션에서 값을 가져와야 한다.
  * 같은 코드의 반복이므로 불필요
  * 메소드 인자로 세션값을 바로 받을 수 있도록 변경

`@LoginUser` 어노테이션 생성 (config.auth패키지에)
```
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {
}
```

* `@Target(ElementType.PARAMETER)`
  * 이 어노테이션이 생성될 수 있는 위치를 지정
  * PARAMETER로 지정했으니 메소드의 파라미터로 선언된 객체에서만 사용 가능
* `@interface`
  * 이 파일을 어노테이션 클래스로 지정
  * LoginUser라는 이름을 가진 어노테이션이 생성되었다고 생각하기

`LoginUserArgumentResolver` 생성 (같은 위치에)
* HandlerMethodArgumentResolver 인터페이스를 구현한 클래스
* HandlerMethodArgumentResolver는 조건에 맞는 경우 메소드가 있다면 HandlerMethodArgumentResolver의 구현체가 지정한 값으로 해당 메소드의 파라미터로 넘길 수 있다.
```
@RequiredArgsConstructor
@Component
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {
    
    private final HttpSession httpSession;

    @Override
    public boolean supportsParameter(MethodParameter parameter){
        boolean isLoginUserAnnotation = parameter.getParameterAnnotation(LoginUser.class) != null;
        boolean isUserClass = SessionUser.class.equals(parameter.getParameterType());
        return isLoginUserAnnotation && isUserClass;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        return httpSession.getAttribute("user");
    }
}
```
* `supportsParameter()`
  * 컨트롤러 메서드의 특정 파라미터를 지원하는지 판단
  * 여기서는 파라미터에 @LoginUser 어노테이션이 붙어 있고, 파라미터 클래스 타입이 Sessionuser.class인 경우 true를 반환합니다.
* `resolveArgument()`
  * 파라미터에 전달할 객체를 생성
  * 여기서는 세션에서 객체를 가져온다.

생성된 LoginUserArgumentResolver를 스프링에서 인식될 수 있도록 WebMvcConfigurer에 추가한다.

`WebConfig` 클래스 생성 (config 패키지에)
```
@RequiredArgsConstructor
@Configuration 
public class WebConfig implements WebMvcConfigurer { 
    private final LoginUserArgumentResolver loginUserArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
    argumentResolvers.add(loginUserArgumentResolver);
    }
}
```

설정 완료했으므로 IndexController의 코드에서 반복되는 부분 모두 @LoginUser로 변경하기 !

## 5.5 세션 저장소로 데이터베이스 사용하기

현재 우리의 서비스는 애플리케이션을 재실행하면 로그인이 풀린다.

세션이 내장 톰캣의 메모리에 저장되는데, 배포할 때마다 톰캣은 초기화되어 재시작 하기 때문이다.

2대 이상의 서버에서 서비스하고 있다면 톰캣마다 세션 동기화 설정을 해야 하는 문제도 있다.

**세션 저장소**
* **톰캣 세션**을 사용
* MySQL과 같은 **데이터베이스**를 세션 저장소로 사용
  * 여러 WAS 간의 공용 세션을 사용할 수 있는 가장 쉬운 방법
  * 로그인 요청마다 DB IO가 발생하여 성능상 이슈 발생 가능
  * 백오피스, 사내 시스템 등 로그인 요청이 적은 용도로 주로 사용 
* Redis, Memcached와 같은 **메모리 DB**를 세션 저장소로 사용

#### 두 번째 방식 적용하여 spring-session-jdbc 등록

* build.gradle 에 의존성 등록
```
compile('org.springframework.session:spring-session-jdbc')
```
* application.properties에 세션 저장소를 jdbc로 선택하도록 코드 추가
```
spring.session.store-type=jdbc
```
> 세션 저장소를 데이터베이스로 교체했다. 아직은 스프링을 재시작하면 세션이 풀리지만, 후에 AWS로 배포하게 되면 세션이 풀리지 않을 것이다.

## 5.6 네이버 로그인

네이버 로그인을 추가하기 위해 [네이버 오픈 API](https://developers.naver.com/apps/#/register?api=nvlogin)로 이동하여 책을 참고해 서비스를 등록한다.

* 네이버에서는 스프링 시큐리티를 공식 지원하지 않기 때문에 그동안 CommonOAuth2Provider에서 해주던 값들을 모두 수동으로 입력해야 한다.
* 스프링 시큐리티에서는 **하위 필드를 명시할 수 없다**. 최상위 필드들만 user_name으로 지정 가능하다. 네이버의 응답값 최상위 필드는 resultCode, message, response 이다.
* 3개 중 본문에서 담고 있는 response를 user_name으로 지정하고 이후 자바 코드로 id 변경

스프링 시큐리티 설정 등록

* OAuthAttributes에 네이버인지 판단하는 코드와 네이버 생성자 추가
* index.mustache에 네이버 로그인 버튼 추가

네이버 로그인까지 성공 !

## 5.7 기존 테스트에 시큐리티 적용하기

기존 테스트에 시큐리티 적용으로 문제가 되는 부분들을 해결해보자. 

시큐리티 옵션이 활성화되면서 인증된 사용자만 API를 호출할 수 있는데, 기존의 API 테스트 코드들은 모두 인증에 대한 권한을 받지 못했으므로 문제 발생

### 문제 1. CustomOAuth2UserService을 찾을 수 없음

* CustomOAuth2UserService를 생성시 필요한 소셜 로그인 관련 설정값들이 없기 때문에 발생
* src/main 과 src/test의 환경 차이 때문에 application-oauth.properties의 설정값을 가져오지 못한다.
* test에 파일이 없을 때 main에서 자동으로 가져오는 범위는 application.properties 파일까지
* 문제 해결을 위해 application.properties를 생성하여 **가짜 설정 값을 등록**

### 문제 2. 302 Status Code

* 스프링 시큐리티 설정이 인증되지 않은 사용자의 요청은 이동시키기 때문에 발생
* **임의로 인증된 사용자를 추가**하여 API만 테스트해 볼 수 있도록 한다.     
* build.gradle에 spring-security-test 의존성 추가
* PostsApiControllerTest의 2개 테스트 메소드에 **임의 사용자 인증 추가**
```
@Test
@WithMockUser(roles="USER")
public void Posts_등록된다() throws Exception {

@Test
@WithMockUser(roles="USER")
public void Posts_수정된다() throws Exception {
```
* `@WithMockUser(roles="USER")`
* 인증된 모의(가짜) 사용자를 만들어서 사용
* roles에 권한 추가 가능
* 이 어노테이션으로 ROLE_USER 권한을 가진 사용자가 API를 요청하는 것과 동일한 효과

아직까지는 @WithMockUser가 MockMvc에서만 작동하므로, 코드를 변경하여 @SpringBootTest에서 MockMvc를 사용하도록 해야 한다.

### 문제 3. @WebMvcTest에서 CustomOAuth2UserService을 찾을 수 없음

* HelloControllerTest는 @WebMvcTest를 사용
* @WebMvcTest는 **CustomOAuth2UserService를 스캔하지 않기 때문에** 문제 발생
* @WebMvcTest에게 @Repository, @Service, @Component는 스캔 대상이 아니다.
* 따라서 SecurityConfig는 읽었지만, SecurityConfig를 생성하기 위해 필요한 CustomOAuth2UserService는 읽을수가 없어 앞에서와 같은 에러가 발생한 것
* **스캔대상에서 SecurityConfig를 제거**해서 해결 가능
```
@WebMvcTest(controllers = HelloController.class,
        excludeFilters = {@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,
                classes = SecurityConfig.class)
                }
)
```
* @WithMockUser를 사용해서 가짜로 인증된 사용자를 생성
* @EnableJpaAuditing로 인해 추가 에러 발생
  * @EnableJpaAuditing를 사용하기 위해서는 최소 하나의 @Entity 클래스가 필요한데 @WebMvcTest는 없으므로
* @EnableJpaAuditing이 @SpringBootApplication과 함께 있어서 @WebMvcTest에서도 스캔하게 된 것
* 이 둘을 분리하여 해결 가능
  * Application.java에서 @EnableJpaAuditing 제거
  * config 패키지에 JpaConfig를 생성하여 @EnableJpaAuditing 추가 
```
@Configuration
@EnableJpaAuditing // JPA Auditing 활성화
public class JpaConfig {}
```
드디어 모든 테스트 통과 !

> 지금까지 인텔리제이로 스프링 부트 통합 개발환경을 만들고, 테스트와 JPA로 데이터를 처리하고, 머스테치로 화면을 구성했으며, 시큐리티와 OAuth로 인증과 권한을 배워보며 간단한 게시판을 완성했다 !


## 회고
드디어 5장이 끝났다...! 로그인 기능은 분량도 많고 아직 어렵게 느껴진다. 여러번 반복해야 할 것 같다.