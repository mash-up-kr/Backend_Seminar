스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현하기

5.1 스프링 시큐리티와 스프링 시큐리티 Oauth2 클라이언트
구현하는데에는 많은 시간이 걸리기 때문에 소셜 로그인 서비스를 대신 이용한다.

5.2 구글 서비스 등록
구글 서비스에서 발급된 인증 정보를 통해서 로그인 기능, 소셜 서비스 기능 사용 가능하다.

1. 구글 클라우드 플랫폼 주소에서 새 프로젝트를 생성
2. API 및 서비스 카테고리에서 사용자 인증 정보 만들기
3. OAuth 클라이언트 ID 만들기
* 승인된 리디렉션 URI
서비스에서 파라미터로 인증 정보 주었을 때 인증이 성공하면 구글에서 리다이렉트할 URL
4. 생성된 애플리케이션에서 인증정보 보기 가능(Client ID, Client 보안 비밀 코드)
5. src/main/resources/ 디렉토리에 application-oauth.properties 파일 생성하여 4번의 Id, 보안 비밀코드 등록

scope = profile, email
- 강제로 등록한 이유는 ..? 이해안댐..

.gitignore에 위와 같은 ID, 보안비밀코드 올라가지 않도록 입력
application-oauth.properties 추가

5.3 구글 로그인 연동하기

사용자 정보를 담당할 도메인인 User 클래스 생성
domain
    user 패키지


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

    @Column(nullable = false)
    private String principalId;

    @Column
    private String picture;

    @Builder
    public User (String name, String email, String principalId, String picture) {
        this.name = name;
        this.email = email;
        this.principalId = principalId;
        this.picture = picture;

    public User update(String nmae, String picture) {
        this.name = name;
        this.picture = picture;

        return this;
    }

    public String getRoleKey() {
        return this.role.getKey();
    }
}

@Enumerated(EnumType.STRING)
    - JPA로 데이터베이스로 저장할때 Enum 값을 어떤 형태로 저장할지 결정
    - 기본적으로 int로된 숫자가 저장
    - 숫자 저장 시 확인할 때 그 값이 무슨 코드인지 의미를 알 수 없으므로 문자열로 저장할 수 있도록 선언

각 사용자 권한을 관리할 Enum 클래스 Role를 생성
Getter
@RequiredArgsConstructor
public enum Role {

    GUEST("ROLE_GUEST", "손님"),
    USER("ROLE_USER", "일반 사용자");

    private final String key;
    private final String title;
}
* 권한코드 앞에는 항상 ROLE_ 이 있어야한다.

CRUD 책임질 UserRepository 생성

findByEmail
    - 소셜로그인으로 반환된 값 중 email 통해 이미 생성된 것인지 처음 가입자 인지 판단

스프링 시큐리티 설정
build.gradle에 스프링 시큐리티 관련 의존성 추가
compile('org.springframework.boot:spring-boot-starter-oauth2-cline')

SecurityConfig 클래스 생성( 시큐리티 관련 클래스 )
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
                    .antMatchers("/", "/css/**", "/images/**", "/js/**", "/h2-console/**", "/profile").permitAll()
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

@EnableWebSecurity
    -Spring Security 설정들 활성화

csrf().disable().headers().frameOptions().disable()
    -h2-console 화면을 사용하기 위해 해당 옵션들을 disable

antMatchers
    -권한 관리 대상 지정하는 옵션
    - "/" 등 지정된 URL들은 permitAll()옵션을 통해 전체 열람 권한 가능
    - POST 메소드이면서 "/api/v1/**" 주소를 가진 API는 USER 권한을 가진 사람만 가능

anyRequest
    -인증된 사용자 즉 로그인한 사용자만 허용하게 한다.

logout().logoutSuccessUrl("/")
    - 로그아웃 기능에 대한 여러 설정의 진입점
    - 로그아웃 성공시 / 주소로 이동

oauth2Login
    - OAuth 2 로그인 기능에 대한 여러 설정의 진입점


CustomOAuth2UserService 클래스
< 구글 로그인 이후 가져온 사용자의 정보 기반으로 가입/정보수정/세션저장 등의 기능 지원>

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

        OAuthAttributes attributes = OAuthAttributes.of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);
        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                attributes.getAttributes(),
                attributes.getNameAttributeKey());
    }


    private User saveOrUpdate(OAuthAttributes attributes) {
        User user = userRepository.findByEmail(attributes.getEmail())
                .map(entity -> entity.update(attributes.getName(), attributes.getPicture()))
                .orElse(attributes.toEntity());

        return userRepository.save(user);
    }
}

userNameAttributeName
    -OAuth2 로그인 진행 시 키가 되는 필드값. PK 같은 의미

OAuthAttributes
    -OAuth2UserService를 통해 가져온 OAuth2user의 attribute를 담을 클래스

SessionUser
    - 세션에 사용자 정보를 저장하기 위한 Dto 클래스

CustomOAuth2UserService 클래스 생성 후 OAuthAttributes 클래스 생성한다.
OAuthAttributes는 Dto로 보기 때문에 config.auth.dto 패키지를 만들어 해당 패키지에 생성

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

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes) {
        if("naver".equals(registrationId)) {
            return ofNaver("id", attributes);
        }

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

    public User toEntity() {
        return User.builder()
                .name(name)
                .email(email)
                .picture(picture)
                .role(Role.GUEST)
                .build();
    }
}

of()
    -OAuth2User에서 반환되는 사용자 정보는 Map이므로 값 하나하나 변환해야함

toEntity()
    -User 엔터티 생성
    -OAuthAttributes에서 엔티티를 생성하는 시점은 처음 가입할 때

<config.auth.dto 패키지에 SessionUser 패키지 추가>
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

SessionUser에는 인증된 사용자 정보만 필요하다.

User 클래스 사용 안하는 이유?
User 클래스에서 직렬화를 구현하지 않는 이유는 User 클래스는 엔티티이기 때문에 언제 다른 엔티티와 관계를 형성할 지 모른다.
엔티티 클래스는 자식 엔티티에도 직렬화 대상에 포함되므로 성능 이슈 등이 날 수 있다.
그러므로 직렬화 기능을 가진 Dto를 하나 추가로 만드는 것이 유지보수에 도움이 된다.

로그인 테스트
화면에 로그인 버튼과 로그인 성공 시 사용자 이름 보여주는 코드 구현
index.mustache

<code>

index.mustache에서 userName 사용할 수 있게 IndexController에서 userName을 model에 저장하는 코드 추가

@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;
    private final HttpSession httpSession;

    @GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user) {
        model.addAttribute("posts", postsService.findAllDesc());

        SessionUser user = (User) httpSession.getAttribute("user");

        if (user != null) {
            model.addAttribute("userName", user.getName());
        }
        return "index";
    }
}

(User) httpSession.getAttribute("user")
    - 로그인 성공시 CustomOAuth2UserService 세션에 SessionUser를 저장하도록 구현
    - 로그인 성공시 httpSession.getAttribute("user") 에서 값을 가져올 수 있다.

if(user != null)
    - 세션에 저장된 값이 있을 때에만 model에 userName 등록

5.4 어노테이션 기반으로 개선하기
같은 코드 반복하는 것을 개선한다.
IndexController에서 세션값을 가져오는 부분을 개선할 필요가 있다.

메소드 인자로 세션값을 바로 받을 수 있도록 변경한다.
config.auth 패키지에 다음과 같이 @LoginUser 어노테이션 생성

@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {
}

@Target(ElementType.PARAMETER)
    - 이 어노테이션이 생성될 수 있는 위치를 지정
    - PARAMETER로 지정했으니 메소드의 파라미터로 선언된 객체에서만 사용할 수 있다

@interface
    - 이 파일을 어노테이션 클래스로 지정


같은 위치에 LoginUserArgumentResolver 생성
UserArgumentResolver라는 HandlerMethodArgumentResolver 인터페이스를 구현한 클래스다

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

resolveArgument()
    - 파라미터에 전달할 객체를 생성
    - 여기에서는 세션에서 객체를 가져온다

UserArgumentResolver가 스프링에서 인식할 수 잇도록 WebMvcConfigurer에 추가

@RequiredArgsConstructor
@Configuration
public class WebConfig implements WebMvcConfigurer {
    private final LoginUserArgumentResolver loginUserArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(loginUserArgumentResolver);
    }
}

모든 설정이 끝나면 IndexController의 코드에서 반복되는 부분들을 모두 @LoginUser로 개선


5.6 네이버 로그인

네이버 서비스 등록하여
ClientID, CilentSecret 생성시킨다.
이 키 값들을 application-oauth.properties에 등록

스프링 시큐리티 설정 등록

구글 로그인 했으므로 거기에 네이버인지 판단하는 코드와 네이버 생성자만 추가하면 된다.

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

마지막으로 index.mustache에 네이버 로그인 버튼 추가


5.7 기존 테스트에 시큐리티 적용하기

문제1. CustomOAuth2UserService 찾을 수 없음
CustomOAuth2UserService 생성하는데 필요한 소셜 로그인 관련 설정값 없기 때문에 발생

src/main 환경과 src/test 환경이 달라서 application-oauth.properties에 설정값들을 추가했는데도 없다고 말하는 것이다.
이 문제 해결하기 위해서는 가짜 설정값을 등록한다.
# Test OAuth

spring.security.oauth2.client.registration.google.client-id=test
spring.security.oauth2.client.registration.google.client-secret=test
spring.security.oauth2.client.registration.google.scope=profile,email
를 설정해서 잇는것처럼 보이게 한다

문제2. 302 Status Code
200를 예상했는데 302가 뜨는 이유는 인증되지 않는 사용자의 요청이기 때문이다.
그러므로 임의로 인증된 사용자를 추가하여 API 테스트 하도록 한다.
build.gradle에 아래를 추가한다
testCompile("org.springframework.security:spring-security-test")

그리소 PostApiControllerTest의 2개 테스트 메소드에는 다음과 같이 임의 사용자 인증을 추가한다.

    @Test
    @WithMockUser(roles="USER")
    public void Posts_등록된다() throws Exception {

    @Test
    @WithMockUser(roles="USER")
    public void Posts_수정된다() throws Exception {

@WithMockUser(roles="USER")
    -인증된 가짜 사용자를 만들어서 사용한다.
    -roles에 권한을 추가

@WithMockUser가 MockMvc에서만 작동해서 실제로 작동안된다.

@SpringBootTest에서 MockMvc를 사용하는 방법은 코드를 수정해야한다.

 @Before
    public void setup() {
        mvc = MockMvcBuilders
                .webAppContextSetup(context)
                .apply(springSecurity())
                .build();
    }
...

 mvc.perform(post(url)
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(new ObjectMapper().writeValueAsString(requestDto)))
                .andExpect(status().isOk());

문제3. @WebMvcTest에서 CustomOAuth2UserService을 찾을 수 없음
1번쨰 문제와 같은 오류가 난다.
여기에서는 @WebMvcTest 사용해서 문제가 난다. @WebMvcTest기 CustomOAuth2userService를 스캔하지 않기 때문이다.

@WebMvcTest는 @ControllerAdvice, @Controller는 읽지만 @Repository, @Service, @Component는 스캔 대상이 아니다.
그리하여 SecurityConfig 생성하기 위해 필요한 CustomOAuth2UserService을 읽을 수 없어 에러가 난다.
해결하기 위해서는 스캔 대상에서 SecurityConfig를 제거한다.

이곳에서는 가짜 인증된 사용자로 @WithMockUser 사용한다.
다음 에러는 java.lang.IllegalArgumentException : At least one JPA metamodel must be present!
이 에러는 @EnableJpaAuditing으로 인해 발생
@EnableJpaAuditing 사용하기 위해서는 최소 하나의 @Entity 클래스 필요하다.

@EnableJpaAuditing가 @SpringBootApplication와 함께 있다보니 @WebMvcTest에서도 스캔하게 된다.
그래서 이 둘을 분리해야 한다.
Application.java에서 @EnableJpaAuditing 제거

그리고 config 패키지에 JpaConfig 생성하여 @EnableJpaAuditing 추가하면 모든 오류가 제거되고 전체 테스트 통과하게 된다.

  
