# 5. 스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현하기

## 진도 기록

| 공부 날짜  | 공부한 페이지 수 |
| ---------- | ---------------- |
| 2020.04.23 | 162p ~ 223p        |

## 5.1. 스프링 시큐리티와 스프링 시큐리티 Oauth2 클라이언트

스프링 시큐리티는 인증과 권한 부여기능을 가진 프레임 워크이다.

인터셉터, 필터기반의 보안 기능을 구현하는 것보다 스프링 시큐리티를 통해 구현하는 것이 적극 권장 되므로 열심히 배워보자. 구글, 네이버를 통한 소셜로그인을 구현해보도록 하겠다!

### 소셜로그인을 사용하는 이유?

id/password 방식을 사용시 구현해야할 사항이 너무나도 많다. 

- 로그인 시 보안
- 이메일 / 전화번호 인증
- 회원정보 변경
- 비밀번호 찾기 및 변경

앞선 모든 것들을 고려하면서 개발하다보면 로그인 기능에 시간을 쏟게되는 경우가 많기때문에, 

이를 구글, 페이스북, 네이버에 맡기고 서비스 개발에 집중해보도록하자.

**스프링 2.0부터 바뀐점**

CommonOAuth2Provider 라는 enum이 새롭게 추가되어 기본 설정값은 모두 여기서 제공하게 된다.

기존의 설정에서는 `application.properties`에 url까지 명시되어야했으나, 2.0에서는 간단하게 client인증 정보만 입력해주면 된다!

## 5.3. 구글 로그인 연동을 통해 구조 파악해보기

![디렉토리](./images/dir2.jpg)

일단 아이패드로 다시 정리..

스프링 시큐리티는  FilterChainProxy라는 이름 하에 여러 필터들이 동작하고 있다.

기본적으로 동작하는 여러 필터들 덕분에 별도의 로직없이 로그인/로그아웃 처리가 가능하다고 한다.

### SecurityConfig

 WebSecurityConfigurerAdapter 라는 클래스를 상속받아 메소드를 오버라이딩하여 설정을 조정할수있는 자바 파일이다.

- @EnableWebSecurity

- - @Configuration 클래스에 @EnableWebSecurity 어노테이션을 추가하여 Spring Security 설정할 클래스라고 정의한다.
  - 설정은 WebSebSecurityConfigurerAdapter 클래스를 상속받아 메서드를 구현하는 것이 일반적인 방법이다.

> config/auth/SecurityConfig.java

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

    private final CustomOAuth2UserService CustomOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        
        http
            	// h2 사용을 위한 disable
                .csrf().disable()
                .headers().frameOptions().disable()
                .and()
                    .authorizeRequests()
            		// 페이지 권한 설정 및 관리 대상 지정
                    .antMatchers("/", "/css/**", "/images/**",
                            "/js/**", "/h2-console/**").permitAll()
                    .antMatchers("/api/vi/**").hasRole(Role.USER.name())
                    .anyRequest().authenticated()
                .and()
            		// 로그아웃 설정
                    .logout()
                        .logoutSuccessUrl("/")
                .and()
                    .oauth2Login()
                        .userInfoEndpoint()
                            .userService(CustomOAuth2UserService);

    }

}
```

- **configure(HttpSecurity http)**
  - HTTP 요청에 대한 웹 기반 보안을 구성할 수 있다.
- **authorizeRequests()**
  - URL별 권한 관리를 하겠다는 시작점. 이가 선언되어야 antMatchers()를 사용할 수 있다.

### CustomOAuth2UserService

로그인 이후 가져온 사용자의 정보를 기반으로 회원가입, 정보수정, 세션 저장등의 기능을 지원한다.

OAuth2 로그인 성공 후 후속조치를 담당하는 역할! 

> SecurityConfig에 bean으로 주입된다.

```java
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
                .getClientRegistration().getRegistrationId();
        String userNameAttributeName = userRequest
                .getClientRegistration().getProviderDetails()
                .getUserInfoEndpoint().getUserNameAttributeName();
        OAuthAttributes attributes = OAuthAttributes
                .of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);
        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                attributes.getAttributes(),
                attributes.getNameAttributeKey()
        );

    }
    private User saveOrUpdate(OAuthAttributes attributes) {
        User user = userRepository.findByEmail(attributes.getEmail())
                .map(entity -> entity.update(attributes.getName(), attributes.getPicture()))
                .orElse(attributes.toEntity());

        return userRepository.save(user);
    }
}
```

### OAuthAttributes

OAuth2UserService를 통해 가져온 OAuth2User의 attribute를 담을 클래스이다.

dto로 간주하고 코딩하자!

```java
@Getter
public class OAuthAttributes {

        private Map<String, Object> attributes;
        private String nameAttributeKey;
        private String name;
        private String email;
        private String picture;

        @Builder
        public OAuthAttributes(Map<String, Object> attributes, String nameAttributeKey, String name, String email, String picture) {
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

- of() 연산자
  - OAuth2User에서 반환하는 사용자 정보는 Mpa이다! 값하나하나 변환해야만 한다.

### SessionUser

세션에 사용자 정보를 저장하기 위한 Dto 클래스이다. 인증된 사용자 정보만 필요로 하니 name, email, picture만 필드로 선언한다.

```java
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

- User 클래스는 엔티티이기때문에, 다른 엔티티들과 관계가 형성될 수 있다 (1:N, M;N 등). 이런경우 자식들까지 직렬화 대상에 포함되어 성능이슈가 발생할 확률이 높아질 수 있기 때문에 **직렬화 기능을 가진 Dto**를 추가로 만든것이다.

## 궁금했던 점

### 1. OAuth

좀 부끄럽지만 그냥 소셜로그인할 때 쓰는 녀석~ 이라고만 생각하고 개발할 때가 많았던지라.. 개념이라도 제대로 알아놓으려한다.

#### OAuth1

Open Authorization, Open Authentication! 

페이스북, 구글, 트위터 같은 유저의 비밀번호를 Third party앱에 제공 없이 인증, 인가를 할 수 있는 오픈 스탠다드 프로토콜이다. OAuth 인증을 통해 API에 대한 접근 권한을 얻을 수 있다.

기본인증인 id/password 방식은 보안상 신경쓸 것이 많고 취약하기 때문에 문제가 많았는데, 이를 보완하기 위해 나온 것이 OAuth이다. 

OAuth 인증은 서버(API를 제공하는)에서 진행하고, 유저가 인증되면 이에대한 Access Token을 발급한다.

#### OAuth2

OAuth1과 전반적인 목적이나 인증 절차는 비슷하나 웹, 앱, 데스크탑의 인증방식을 강화하고 개발을 간소화하는 것을 중심으로 개발되었다!

OAuth1.0은 디지털 서명기반이었지만 OAuth2.0의 암호화는 https에 맡겨 개발자 입장에더는 더 쉬워지는 것이다.

> 소셜로그인할때 쓰는 녀석~~

### 2. 싱글톤

해당 클래스에서 인스턴스가 하나만 생성이 되는 것이며, 어디든지 그 인스턴스에 접근이 가능하도록 하는 패턴이다.

- 어떤 클래스가 최초 한번만 메모리를 할당(Static)하고 그메모리에 인스턴스를 생성.
  - 이후 생성자를 시도하면 최초에 생성된 객체를 리턴하도록한다.

#### 구현법

1. 외부 클래스에서의 인스턴스 생성을 막기위해 생성자는 private로 선언한다.
2. 어느 영역에서든 접근이 가능하도록 참조는 static으로 정의한다.

**Thread safe lazy + Doubled checked locking**

```java
public class TreadSafe {
    private static ThreadSafe instance;
    
    private TreadSafe(){}
    
    public static TreadSafe getInstance() {
        if(instnace == null){ // 존재여부 확인하고
            // syncronized를 거는 것은 자신이 포함된 객체에 lock을 건다.
            synchronized(ThreadSafe.class) // 두번째 체크전에 동기화
                if(instance == null) { 
                    instance = new TreadSafe();
                }
        }
        return instance;
    }
}
```

더 일반적으로 사용하는 방법이 있는데 이는 더 공부해보고 추가해볼 생각이다!

> 추후에..추후에..

#### 장점

- 메모리 낭비 방지
  - 한번의 new로 인스턴스를 생성하기 때문
- 전역 인스턴스이기 때문에 다른 클래스 인스턴스에서 데이터를 공유하는게 용이하다.

#### 유의점

- 멀티 스레드 기반의 애플리케이션인경우!!
  - 객체가 생성되지 않은 시점에서 생성자 호출
  - 동시에 객체 유무 체크 => 객체는 하나만 생성되어야함!!!

클래스에 mutual exclusion(상호배제)를 사용하여 객체가 생성되고 있음을 알려야한다.

> 운영체제에서 배워서 그런지 괜히 반갑다. 어떻게 적용되는지 나중에 찾아봐야겠다.



## 회고

로그인 어려워,,!,,! 그래도 두번보고 세번보면 익숙해지지 않을까 한다.