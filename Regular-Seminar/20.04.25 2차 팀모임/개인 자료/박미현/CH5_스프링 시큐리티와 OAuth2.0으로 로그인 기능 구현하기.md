#5. 스프링 시큐리티와 OAuth2.0으로 로그인 기능 구현하기
###1. 스프링 시큐리티
#####1.1. 스프링 시큐리티란?

스프링 시큐리티는 막강한 인증과 인가(권한부여) 기능을 가진 프레임워크.     
스프링 기반의 애플리케이션에서 보안을 위한 표준.

#####1.2. 스프링부트2.0에서의 Oauth2
- 구글, 깃허브, 페이스북, 옥타(Okta)의 기본 설정값 모두 제공(스프링부트2.0에서 추가된CommonOAuth2Provider라는 enum) 
- 이외 다른 소셜 로그인(네이버, 카카오)은 직접 추가해 주어야한다.

***
###2. 구글 로그인 구현
#####2.1. 구글 서비스 등록하기

1. 구글 클라우드 플랫폼 주소(https://console.cloud.google.com)에서 새 프로젝트 생성
2. API 및 서비스 탭 → 사용자 인증 정보 → 사용자 인증 정보 만들기 → OAuth 클라이언트 ID
→ 동의 화면 구성
3. OAuth 동의 화면 탭에서 항목 작성 및 저장
    - 애플리케이션 이름 : 구글 로그인 시 사용자에게 노출될 애플리케이션 이름
    - 지원 이메일 : 사용자 동의 화면에서 노출될 이메일 주소(일반적으로 서비스의 help 이메일 주소)
    - Google API 범위 : 등록할 구글 서비스에서 사용할 범위 목록(기본값 email, profile, openid)
4. 프로젝트 이름 등록 → 애플리케이션 유형 선택 → URL주소 등록(승인된 리디렉션 URI) → 클라이언트ID, 클라이언트 보안 비밀
  - 승인된 리디렉션 URI
    - 서비스에서 파라미터로 인증 정보를 주었을 떄 인증이 성공하면 구글에서 리다이렉트할 URL
    - 시큐리티에서 Controller를 이미 구현해 두어 사용자의 별도 구현 불필요
    - 스프링부트2 버전의 시큐리티에서는 {도메인}/login/oauth2/code/{소셜서비스코드}로 기본적인 리다이렉트 URL지원
    
#####2.2. 스프링부트에 클라이언트ID, 클라이언트 보안비밀 코드 등록

- src/main/resources/application-oauth.properties
```java
spring.security.oauth2.client.registration.google.client-id=클라이언트 ID
spring.security.oauth2.client.registration.google.client-secret=클라이언트 보안 비밀
spring.security.oauth2.client.registration.google.scope=profile,email
```
    -scope의 기본값은 openid, profile, email
    -일반적으로 scope 별도지정하지 않음
    -openid가 scope에 포함되면 Open id Provider로 인식하여      
     Openid Provider인 서비스(구글)과 그렇지 않은 서비스(네이버, 카카오)를 나눠     
     각각 OAuth2Service를 만들어야 하는 번거로움
__*application-xxx.properties*__
:properties의 이름을 application-xxx.properties로 만들면
xxx라는 이름의 profile이 생성된다. profile=xxx로 호출하여 해당 
properties의 설정을 가져올 수 있다.

- src/main/resources/application.properties 코드 추가
```java
spring.profiles.include=oauth
```

-.gitignore등록
```java
application-oauth.properties
```
`클라이언트 ID와 클라이언트 보안 비밀은 보안이 중요한 정보로 공개되면 안된다. 
->언급된 파일 공개 방지하기 위해 .gitignore등록`

#####2.3. 구글 로그인 연동하기
- src/main/java/com/jojoldu/book/springboot/domain/user/User.java 
   - 사용자 정보 담당 도메인
```java
package com.jojoldu.book.springboot.domain.user;

import com.jojoldu.book.springboot.domain.BaseTimeEntity;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

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

    @Enumerated(EnumType.STRING)
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

    public String getRoleKey() {
        return this.role.getKey();
    }

}
```
__*@Enumerated(EnumType.STRING)*__
:JPA로 데이터베이스로 저장할 때 Enum 값을 어떤 형태로 저장할지 결정(기본값 int형). 
값이 의미하는 코드를 알기 위해 String으로 변경

- src/main/java/com/jojoldu/book/springboot/domain/user/Role.java 
   - 사용자의 권한 관리할 Enum클래스
```java
package com.jojoldu.book.springboot.domain.user;

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
`스프링 시큐리티에서는 권한 코드에 항상 ROLE_이 앞에 있어야한다.`

- src/main/java/com/jojoldu/book/springboot/domain/user/UserRepository.java
   - User의 CRUD에 쓰이는 UserRepository
```java
package com.jojoldu.book.springboot.domain.user;

import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByEmail(String email);

}
```
***
#####2.4. 스프링 시큐리티 설정
- build.gradle 의존성 추가
```java
    compile('org.springframework.boot:spring-boot-starter-oauth2-client') //dependencies에 추가
```
- src/main/java/com/jojoldu/book/springboot/config/auth/SecurityConfig.java
```java
package com.jojoldu.book.springboot.config.auth;


import com.jojoldu.book.springboot.domain.user.Role;
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
        http
                .csrf().disable()
                .headers().frameOptions().disable()
                .and()
                .authorizeRequests()
                .antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console/**").permitAll()
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
__*@EnableWebSecurity*__
:스프링 시큐리티 활성화

__*authorizeRequests*__
:시큐리티 처리에 HttpServletRequest 이용함 의미. URL별 권한 권리를 설정하는 옵션의 시작점

__*antMatchers()*__
: 권한 관리 대상 지정옵션(특정경로 지정). URL, HTTP메소드별로 관리 가능

__*permitAll()*__
:전체 열람 권한 옵션

__*hasRole()*__
:시스템 상 특정 권한 가진 사람만 접근 가능

__*anyRequest()*__
:설정된 값들 이외 나머지 URL

__*authenticated()*__
: 인증된 사용자들(로그인한 사용자)에게만 허용.

__*oauth2Login()*__
:OAuth2 로그인 기능 진입점

__*userInfoEndpoint*__
:OAuth2 로그인 성공 이후 사용자 정보 가져올 때의 설정 담당

- src/main/java/com/jojoldu/book/springboot/config/auth/CustomOAuth2UserService.java
   - 구글 로그인 이후 가져온 사용자의 정보들을 기반으로 가입 및 정보수정, 세션 저장 등의 기능 지원

```java
package com.jojoldu.book.springboot.config.auth;


import com.jojoldu.book.springboot.config.auth.dto.OAuthAttributes;
import com.jojoldu.book.springboot.config.auth.dto.SessionUser;
import com.jojoldu.book.springboot.domain.user.User;
import com.jojoldu.book.springboot.domain.user.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.oauth2.client.userinfo.DefaultOAuth2UserService;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserRequest;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserService;
import org.springframework.security.oauth2.core.OAuth2AuthenticationException;
import org.springframework.security.oauth2.core.user.DefaultOAuth2User;
import org.springframework.security.oauth2.core.user.OAuth2User;
import org.springframework.stereotype.Service;

import javax.servlet.http.HttpSession;
import java.util.Collections;

@RequiredArgsConstructor
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {

    private final UserRepository userRepository;
    private final HttpSession httpSession;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        String userNameAttributeName = userRequest.getClientRegistration().getProviderDetails().getUserInfoEndpoint().getUserNameAttributeName();

        OAuthAttributes attributes = OAuthAttributes.of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);
        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                attributes.getAttributes(),
                attributes.getNameAttributeKey());
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
__*OAuth2UserRequest.getClientRegistration().getRegistrationId()*__
:현재 로그인 진행 중인 서비스 구분 코드(예시> 구글인지, 네이버인지)

__*OAuth2UserRequest.getClientRegistration().getProviderDetails().getUserInfoEndpoint().getUserNameAttributeName()*__
:OAuth2 로그인 진행 시 키가 되는 필드값(PK와 같은 역할). 구글은 기본코드 제공("sub"), 네이버 카카오는 기본코드 미제공

__*OAuthAttributes*__
:OAuth2UserService 통해 가져온 OAuth2User의 attribute 담을 클래스

__*SessionUser*__
:세선에 사용자 정보를 저장하기 위한 Dto클래스

- src/main/java/com/jojoldu/book/springboot/config/auth/dto/OAuthAttributes.java
```java
package com.jojoldu.book.springboot.config.auth.dto;

import com.jojoldu.book.springboot.domain.user.Role;
import com.jojoldu.book.springboot.domain.user.User;
import lombok.Builder;
import lombok.Getter;

import java.util.Map;

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

    public static OAuthAttributes of(String registrationId,
                                     String userNameAttributeName,
                                     Map<String, Object> attributes) {
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
__*of()*__
:OAuth2User에서 반환하는 사용자 정보는 Map으로 값 하나하나 변환해야 한다.

__*toEntity()*__
:Entity생성(OAuthAttributes에서 엔티티 생성 시점은 처음 가입할 때)

- src/main/java/com/jojoldu/book/springboot/config/auth/dto/SessionUser.java
```java
package com.jojoldu.book.springboot.config.auth.dto;

import com.jojoldu.book.springboot.domain.user.User;
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
`SessionUser에는 인증된 사용자 정보만 필요`

#####2.5. 로그인 테스트 위한 코드 작성
-  src/main/resources/templates/index.mustache 
    - 로그인 버튼 추가 + 로그인 시 사용자 이름 보여주기
```html
<h1>스프링부트로 시작하는 웹 서비스</h1>
<div class="col-md-12">
    <!--로그인 기능 영역-->
    <div class="row">
        <div class="col-md-6">
            <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
            {{#userName}}
                Logged in as: <span id="user">{{userName}}</span>
                <a href="/logout" class="btn btn-info active" role="button">Logout</a>
            {{/userName}}
            {{^userName}}
                <a href="/oauth2/authorization/google"
                   class="btn btn-success active" role="button">Google Login</a>
            {{/userName}}
        </div>
    </div>
</div>
```
    
- src/main/java/com/jojoldu/book/springboot/web/IndexController.java
    - userName을 model에 추가하여 mustache에서 사용할 수 있도록
```java
    private final HttpSession httpSession;


    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("posts", postsService.findAllDesc());
        SessionUser user = (SessionUser) httpSession.getAttribute("user");

        if (user != null) {
            model.addAttribute("userName", user.getName());
        }
        return "index";
    }
```

***
###3. 어노테이션 기반으로 개선
같은 코드가 반복되는 부분을 개선해보자    
- 중복되는 부분→ IndexController에서 세션값 가져오는 부분
- 코드 → User user = httpSession.getAttribute("user");     
- 대안 → 해당 부분을 메소드 인자로 세션값을 바로 받도록 수정

#####3.1. @LoginUser 사용환경 구성
-  src/main/java/com/jojoldu/book/springboot/config/auth/LoginUser.java 
     - 어노테이션 생성
```java
package com.jojoldu.book.springboot.config.auth;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {
}
```
__*@Target()*__
: 이 어노테이션이 생성될 수 있는 위치 지정. 위의 코드에서는 메소드의 파라미터로 선언된 객체에서만 사용가능하게 지정.

__*@interface*__
: 이 파일을 어노테이션 클래스로 지정

- src/main/java/com/jojoldu/book/springboot/config/auth/LoginUserArgumentResolver.java
   - Login-UserArgumentResolver라는 HandlerMethodArgumentResolver 인터페이스 구현한 클래스
   
   `HandlerMethodArgumentResolver는 조건에 맞는 경우 메소드가 있다면 구현체가 지정한 값으로 해당 메소드의 파라미터로 넘길 수 있음.`
   
```java
package com.jojoldu.book.springboot.config.auth;

import com.jojoldu.book.springboot.config.auth.dto.SessionUser;
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
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        return httpSession.getAttribute("user");
    }

}
```
__*@Component*__
:사용된 클래스를 자동으로 빈에 등록

__*supportsParameter*__
:현재 parameter를 resolver이 지원하는지에 대한 boolean 리턴

__*resolveArgument()*__
:parameter에 전달할 객체 생성(위의 코드는 세션에서 객체 가져온다)

#####3.2. LoginUserResolver를 스프링이 인식하도록 WebMvcConfigurer에 추가
- src/main/java/com/jojoldu/book/springboot/config/WebConfig.java
```java
package com.jojoldu.book.springboot.config;


import com.jojoldu.book.springboot.config.auth.LoginUserArgumentResolver;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.util.List;

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
`HandlerMethodArgumentResolver는 항상 WebMvcConfigurer의 addArgumentResolvers() 통해 추가해야 한다`

#####3.3. 반복되는 부분 @LoginUser로 개선
-  src/main/java/com/jojoldu/book/springboot/web/IndexController.java
```java
public class IndexController {

    private final PostsService postsService;

    @GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user) {
        model.addAttribute("posts", postsService.findAllDesc());

```

***
###4. 세션 저장소로 데이터베이스 사용하기

- 세션이 내장 톰캣의 메모리에 저장된다 → 애플리케이션 재실행 시 로그인 풀림
- 2대 이상의 서버에서 서비스하고 있다면 톰캣마다 세션 동기화 설정 필요

\<세션 저장소 선택하기\>
   - 톰캣 세션 사용
     - 기본적인 방법. 톰캣(WAS)에 세션이 저장되기 때문에 2대 이상의 WAS가 구동되는 환경에서는 
     톰캣간의 세션 공유 위한 추가 설정 필요
   - MySQL과 같은 데이터 베이스를 세션 저장소로 사용
     - 여러 WAS 간의 공용 세션을 사용하는 가장 쉬운 방법
     - 많은 설정이 필요 없음
     - 로그인 요청마다 DB IO가 발생하여 성능상 이슈 가능성
     - 로그인 요청 적은 백오피스, 사내 시스템 용도에서 사용
   - Redis, Memcached 같은 메모리 DB를 세션 저장소로 사용
     - B2C에서 많이 사용
     - 실제 서비스로 사용하기 위해 외부 메모리 서버 필요
   
   ▶두 번째 방식 적용하기     
   (세번째 방식은 AWS에서 배포할 때 별도 사용료로 인해 부담스럽다)
 #####4.1. spring-session-jdbc 등록
 - build.gradle
 ```java
    compile('org.springframework.session:spring-session-jdbc') 
```
- src/main/resources/application.properties 
```java
spring.session.store-type=jdbc
```

***
###5. 네이버 로그인 구현
#####5.1. 네이버 API 등록
1. 네이버 오픈 API(https://developers.naver.com/apps/#/register?api=nvlogin)로 이동
2. 애플리케이션 이름, 네이버 아이디로 로그인의 권한 선택하기
3. 서비스환경 추가, 서비스 URL, Callback URL 등록
`네이버의 Callback URL은 구글의 리디렉션 URL과 같은 역할`

#####5.2. 스프링부트에 클라이언트ID, 클라이언트 보안비밀 코드 등록
`네이버는 스프링 시큐리티를 공식 지원하지 않아 전부 수동으로 입력`

-application-oauth.properties
```java
#registration
spring.security.oauth2.client.registration.naver.client-id=클라이언트 ID
spring.security.oauth2.client.registration.naver.client-secret=클라이언트 비밀
spring.security.oauth2.client.registration.naver.redirect_uri_template={baseUrl}/{action}/oauth2/code/{registrationId}
spring.security.oauth2.client.registration.naver.authorization_grant_type=authorization_code
spring.security.oauth2.client.registration.naver.scope=name,email,profile_image
spring.security.oauth2.client.registration.naver.client-name=Naver

#provider
spring.security.oauth2.client.provider.naver.authorization_uri=https://nid.naver.com/oauth2.0/authorize
spring.security.oauth2.client.provider.naver.token_uri=https://nid.naver.com/oauth2.0/token
spring.security.oauth2.client.provider.naver.user-info-uri=https://openapi.naver.com/v1/nid/me
spring.security.oauth2.client.provider.naver.user_name_attribute=response
```

#####5.3 스프링 시큐리티 설정 등록

`스프링 시큐리티에서는 하위 필드 명시 불가. 최상위 필드만 user_name으로 지정 가능`     
`네이버의 응답값 최상위 필드는 resultCode, message, response`

- src/main/java/com/jojoldu/book/springboot/config/auth/dto/OAuthAttributes.java
   - 기존의 OAuthAttributes에 네이버인지 판단하는 코드+네이버 생성자 추가
```java
    public static OAuthAttributes of(String registrationId,
                                     String userNameAttributeName,
                                     Map<String, Object> attributes) {
        if("naver".equals(registrationId)) {
            return ofNaver("id", attributes);
        }
        return ofGoogle(userNameAttributeName, attributes);
    }

    private static OAuthAttributes ofNaver(String userNameAttributeName,
                                           Map<String, Object> attributes) {
        Map<String, Object> response = (Map<String, Object>)attributes.get("response");

        return OAuthAttributes.builder()
                .name((String) response.get("name"))
                .email((String) response.get("email"))
                .picture((String) response.get("profileImage"))
                .attributes(response)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }
    private static OAuthAttributes ofGoogle(String userNameAttributeName,
                                            Map<String, Object> attributes) {
        return OAuthAttributes.builder()
```

- src/main/resources/templates/index.mustache 
   - 네이버 로그인 버튼 추가
```java
<a href="/oauth2/authorization/naver"
class="btn btn-secondary active" role="button">Naver Login</a>
```

###6. 기존 테스트에 시큐리티 적용하기
시큐리티 옵션이 활성화 되면 인증된 사용자만 API를 호출할 수 있다.
테스트 코드마다 인증한 사용자가 호출한 것처럼 작동하도록 수정 필요.

- build.gradle
  - 스프링 시큐리티 테스트 위한 여러 도구 지원
```java
    testCompile('org.springframework.security:spring-security-test')
```

- src/test/java/com/jojoldu/book/springboot/web/PostsApiControllerTest.java
```java
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

    @After
    public void tearDown() throws Exception {
        postsRepository.deleteAll();
    }

    @Test
    @WithMockUser(roles="USER")
    public void Posts_등록된다() throws Exception {
        //given
        String title = "title";
        String url = "http://localhost:" + port + "/api/v1/posts";

        //when
        mvc.perform(post(url)
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(new ObjectMapper().writeValueAsString(requestDto)))
                .andExpect(status().isOk());
        //then

        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);

    }

    @Test
    @WithMockUser(roles="USER")
    public void Posts_수정된다() throws Exception {
        //given
        Posts savedPosts = postsRepository.save(Posts.builder()
        HttpEntity<PostsUpdateRequestDto> requestEntity = new HttpEntity<>(requestDto);

        //when
        mvc.perform(put(url)
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(new ObjectMapper().writeValueAsString(requestDto)))
                .andExpect(status().isOk());

        //then

        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent()).isEqualTo(expectedContent);

```
__*@WithMockUser(roles="USER")*__
:인증된 가짜 사용자 만들어 사용. roles에 권한 추가 가능

    @WithMockUser가 MockMvc에서만 작동하기 때문에 
    @SpringBootTest인 환경에서 MockMvc사용해야한다
__*@Before*__
:테스트 시작하기 전 매번 MockMvc인스턴스 생성

__*mvc.perform*__
:생성된 MockMvc통해 API테스트


- src/test/java/com/jojoldu/book/springboot/web/HelloControllerTest.java 
```java
@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class,
        excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = SecurityConfig.class)
        }
)
public class HelloControllerTest {

    @Autowired
    private MockMvc mvc;

    @WithMockUser(roles = "USER")
    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";
                .andExpect(content().string(hello));
    }

    @WithMockUser(roles = "USER")
    @Test
    public void helloDto가_리턴된다() throws Exception {
        String name = "hello";
```
-@WebMvcTest는 CustomOAuth2UserService를 스캔하지 않는다    
->스캔대상에서 SecurityConfig제거 + @WithMockUser로 가짜 인증된 사용자 생성

- src/main/java/com/jojoldu/book/springboot/Application.java
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
- src/main/java/com/jojoldu/book/springboot/config/auth/JpaConfig.java 
```java
package com.jojoldu.book.springboot.config.auth;

import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;

@Configuration
@EnableJpaAuditing
public class JpaConfig {}

```
-@EnableJpaAuditing사용위해서는 최소 하나의 Entity 클래스가 있어야 하지만 @WebMvcTest는 없음     
->@EnableJpaAuditing과 @SpringBootApplication 분리

***
###. 더 알아보기
#####.1. 인증과 인가
- 인증(Authentication) : 아이디, 비밀번호로 로그인 하는 과정      
→ 보안 절차 거친 사용자만 해당 URL 접근 가능

- 인가(Authorization) : 어떤 대상이 특정 목적을 실행하도록 허용(Access) (=권한부여, 허가)     
→URL에 접근한 사용자가 특정한 자격이 있음



#####.2. URL vs URI
- URL
   - 웹 사이트 주소가 요청하는 파일이 아닌, 구분자로 보는 것
   - 웹 상에 서비스를 제공하는 각 서버들에 있는 파일 위치 표시하기 위함
   - 자원
- URI
   - 통합 지원 식별자
   - 인터넷에 있는 자원 나타내는 유일한 주소
   - URI의 존재는 인터넷에서 요구되는 기본 조건으로, 인터넷 프로토콜에 항상 붙어다님






