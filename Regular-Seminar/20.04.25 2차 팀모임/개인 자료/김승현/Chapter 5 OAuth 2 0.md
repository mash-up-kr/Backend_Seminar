# Chapter 5: 스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현하기

Created: Apr 06, 2020 8:10 PM
Tags: OAuth2, Spring Security

## 5.1 스프링 시큐리티와 스프링 시큐리티 Oauth2 클라이언트

- OAuth: 인터넷 사용자들이 비밀번호를 사용하지 않고 다른 웹 사이트 상의 자신들의 정보에 대해 웹 사이트와 애플리케이션의 접근 권한을 부여할 수 있는 공통적인 수단으로서 사용되는, 접근 위임을 위한 개방형 표준이다.
- 스프링 부트 2 방식: Spring Security Oauth2 Client 라이브러리
    - 스프링 부트용 라이브러리(starter) 출시
    - 확장 포인트를 고려해서 설계
    - **client** 인증 정보만 입력하면 된다.
    - `CommonOAuth2Provider` 라는 enum이 새롭게 추가되어 구글, 깃허브, 페이스북, 옥타의 기본 설정값은 모두 제공한다.
    - 이외에 다른 소셜 로그인(네이버, 카카오 등)을 추가한다면 직접 추가해 주어야 한다.

## 5.2 구글 서비스 등록

- 구글 클라우드 플랫폼 접속
    - 새 프로젝트 생성
    - API 및 서비스 카테고리에서 사용자 인증 정보 만들기
    - OAuth 클라이언트 ID 클릭, 클라이언트 ID 생성 전 동의 화면 구성
    - URL 주소 등록 → 승인된 리디렉션 URI 항목에 `[http://localhost:8080/oauth2/code/google](http://localhost:8080/oauth2/code/google)` 등록
        - 서비스에서 파라미터로 인증 정보를 주었을 때 인증이 성공하면 구글에서 리다이렉트할 URL이다.
        - 스프링 부트 2 버전의 시큐리티에서는 기본적으로 `{도메인}/login/oauth2/code/{소셜서비스코드}`로 리다이렉트 URL 지원한다.
        - 사용자가 별도로 리다이렉트 URL을 지원하는 Controller를 만들 필요가 없다.

- application-oauth 등록
- `src/main/resources/` 디렉토리에 `[application-oauth.properties](http://application-oauth.properties)` 파일 생성
- 해당 파일에 클라이언트 ID와 클라이언트 보안 비밀 코드를 등록한다.

```java
spring.security.oauth2.client.registration.google.client-id=클라이언트 ID
spring.security.oauth2.client.registration.google.client-secret=클라이언트 보안 비밀
spring.security.oauth2.client.registration.google.scope=profile,email
```

- scope의 기본값은 openid, profile, email이다.
    - openid라는 scope가 있으면 Open Id Provider로 인식하기 때문에 OpenId Provider인 서비스(구글)와 그렇지 않은 서비스(네이버, 카카오 등)로 나눠서 각각 OAuth2Service를 만들어야 한다.

- 스프링 부트에서는 properties의 이름을 application-xxx.properties로 만들면 xxx라는 이름의 **profile**이 생성되어 이를 통해 관리할 수 있다.
    - 즉, profile=xxx라는 식으로 호출하면 해당 properties의 설정들을 가져올 수 있다.
- `application.properties`에 `spring.profiles.include=oauth`를 추가한다.

- 구글 로그인을 위한 클라이언트 ID와 클라이언트 보안 비밀은 보안이 중요한 정보들이므로, 깃허브에 `[application-oauth.properties](http://application-oauth.properties)` 파일이 올라가는 것을 방지하도록 한다.
- `.gitignore`에 `application-oauth.properties`를 추가한다.

## 5.3 구글 로그인 연동하기

- 사용자 정보를 담당할 도메인인 User 클래스 생성
- domain 아래에 user 패키지 생성

```java
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.EnumType;
import javax.persistence.Enumerated;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

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

- `@Enumerated(EnumType.STRING)`
    - JPA로 데이터베이스로 저장할 때 Enum 값을 어떤 형태로 저장할지를 결정한다. 기본적으로는 int로 된 숫자가 저장된다.
    - 숫자로 저장되면 데이터베이스로 확인할 때 그 값이 무슨 코드를 의미하는지 알 수 없으므로 문자열로 저장될 수 있도록 한다.

- 사용자의 권한을 관리할 Enum 클래스 Role을 생성한다.

```java
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

- 스프링 시큐리티에서는 권한 코드에 항상 **ROLE_이 앞에 있어야만** 한다.
    - 따라서 코드별 키 값을 ROLE_GUEST, ROLE_USER 등으로 지정한다.

- User의 CRUD를 담당할 UserRepository를 생성

```java
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}
```

- `findByEmail`
    - 소셜 로그인으로 반환되는 값 중 email을 통해 이미 생성된 사용자인지, 처음 가입하는 사용자인지 판단하기 위한 메소드이다.

### 스프링 시큐리티 설정

- build.gradle에 스프링 시큐리티 관련 의존성 추가
    - `compile('org.springframework.boot:spring-boot-starter-oauth2-client')`
- OAuth 라이브러리를 이용한 소셜 로그인 설정 코드를 작성 → config.auth 패키지 생성 후 **시큐리티 관련 클래스는 모두 이곳에**

```java
// SecurityConfig 클래스

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
                    .antMatchers("/", "/css/**", "/images/**",
                            "/js/**", "/h2-console/**").permitAll()
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

- `@EnableWebSecurity`
    - Spring Security 설정들을 활성화시킨다.
- `csrf().disable().headers().frameOptions().disable()`
    - h2-console 화면을 사용하기 위해 해당 옵션 disable
- `authorizeRequests`
    - URL별 권한 관리를 설정하는 **옵션의 시작점**
    - 이게 선언되어야 antMatchers 옵션 사용 가능
- `antMatchers`
    - 권한 관리 대상을 지정하는 옵션
    - URL, HTTP 메소드별로 관리 가능 → 메소드별로는 어떻게?
    - "/" 등 지정된 URL들은 `permitAll()` 옵션을 통해 전체 열람 권한
    - "api/v1/**" 주소를 가진 API는 USER 권한을 가진 사람만 가능
- `anyRequest`
    - 설정된 값을 이외 나머지 URL들을 나타낸다.
    - `authenticated()`을 추가하여 나머지 URL들은 모두 인증된 사용자들에게만 허용하게 한다. (= 로그인한 사용자)
- `logout().logoutSuccessUrl("/")`
    - 로그아웃 기능에 대한 여러 설정의 진입점
    - 로그아웃 성공 시 / 주소로 이동
- `oauth2Login`
    - OAuth 2 로그인 기능에 대한 여러 설정의 진입점
- `userInfoEndpoint`
    - OAuth 2 로그인 성공 이후 사용자 정보를 가져올 때의 설정 담당
- `userService`
    - 소셜 로그인 성공 시 후속 조치를 진행할 UserService 인터페이스의 구현체를 등록한다.
    - 리소스 서버(즉, 소셜 서비스들)에서 사용자 정보를 가져온 상태에서 추가로 진행하고자 하는 기능을 명시할 수 있다.
- 궁금한 점
    - antMatchers는 어떻게 권한 관리 대상을 메소드별로 지정할까?
    - 설정의 "진입점"은 무엇을 의미할까? 왜 진입점을 설정해 줘야 할까?

```java
// CustomOAuth2UserService

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
        String userNameAttributeName = userRequest.getClientRegistration().getProviderDetails()
                                                  .getUserInfoEndpoint().getUserNameAttributeName();

        OAuthAttributes attributes = OAuthAttributes.of(registrationId, userNameAttributeName,
                                                        oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);

        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(
                Collections.singleton(
                        new SimpleGrantedAuthority(user.getRoleKey())),
                attributes.getAttributes(),
                attributes.getNameAttributeKey());
    }

    private User saveOrUpdate(OAuthAttributes attributes) {
        User user = userRepository.findByEmail(attributes.getEmail())
                    .map(entity -> entity.update(attributes.
                            getName(), attributes.getPicture()))
                    .orElse(attributes.toEntity());

        return userRepository.save(user);
    }
}
```

- `registrationId`
    - 현재 로그인 진행 중인 서비스를 구분하는 코드이다.
- `userNameAttributeName`
    - OAuth 2 로그인 진행 시 키가 되는 필드값을 이야기한다. Primary Key와 같은 의미이다.
    - 구글의 경우 기본적으로 코드를 지원하지만, 네이버 카카오 등은 기본 지원하지 않는다. 구글의 기본 코드는 "sub"이다.
    - 이후 네이버 로그인와 구글 로그인을 동시 지원할 때 사용된다.
- `OAuthAttributes`
    - OAuth2UserService를 통해 가져온 OAuth2User의 attribute을 담을 클래스이다.
- `SessionUser`
    - 세션에 사용자 정보를 저장하기 위한 Dto 클래스이다.
    - 왜 **User 클래스를 쓰지 않고 새로 만들어서** 쓸까?
        - User 클래스를 그대로 사용하면 세션에 User 클래스를 저장하려고 할 때 **직렬화를 구현하지 않았다**는 의미의 에러가 발생한다. → User 클래스에 직렬화 코드를 넣으면 될까?
        - **User 클래스는 엔티티**이므로 언제 다른 엔티티와 관계가 형성될지 모른다. User 엔티티가 자식 엔티티를 갖고 있다면 직렬화 대상에 자식들까지 포함되므로 성능 이슈와 부수 효과가 발생할 수 있다.
        - 따라서 **직렬화 기능을 가진 세션 Dto**를 추가로 만드는 게 운영 및 유지 보수에 도움이 된다.

- OAuthAttributes는 Dto로 보고 config.auth.dto 패키지를 만들어 해당 패키지에 생성

```java
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

    private static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String, Object> attributes) {
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

- `of()`
    - OAuth2User에서 반환하는 사용자 정보는 Map이기 때문에 값 하나하나를 변환해야 한다.
- `toEntity()`
    - User 엔티티를 생성한다.
    - OAuthAttributes에서 엔티티를 생성하는 시점은 처음 가입할 때이다.
    - 가입할 때의 기본 권한을 GUEST로 주기 위해 role 빌더 값에는 Role.GUEST를 사용한다.
    - OAuthAttributes 클래스 생성이 끝나면 같은 패키지에 SessionUser 클래스를 생성한다.

```java
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

- SessionUser에는 **인증된 사용자 정보만 필요하다.**
- 그 외에 필요한 정보는 없으므로 name, email, picture만 필드로 선언한다.