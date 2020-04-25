# 2. 테스트코드
## 2.2 HelloController 테스트코드 작성
- @SpringBootApplication
    - 이 어노테이션으로 인해 스프링 부트의 자동 설정, 스프링 Bean 읽기와 생성이 모두 자동으로 설정됨
    - 이 어노테이션이 있는 위치부터 설정을 읽어가기 때문에 이 클래스는 항상 프로젝트 최상단에 위치해야 함
- main 메소드에서 실행하는 SpringApplication.run
    - 내장 WAS 실행
    - 서버에 톰캣 설치 불필요
    - 스프링 부트로 만들어진 Jar 파일로 실행하면 됨

```java
@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class)
class HelloControllerTest {
    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리천된다() throws Exception{
        String hello = "hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));

    }
}
```

1. @RunWith(SpringRunner.class)
- 테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킴
- 스프링 부트 테스트와 JUnit 사이에 연결자 역할

2. @WebMvcTest
- 여러 스프링 테스트 어노테이션 중 Web에 집중할 수 있는 어노테이션
- 선언할 경우 @Controller, @ControllerAdvice 등 사용 가능
- @Service, @Component, @Repository 등은 사용 불가

3. @Autowired
- 스프링이 관리하는 빈을 주입받음

4. private MockMvc mvc
- 웹 API를 테스트할 때 사용
- 스프링 MVC 테스트의 시작
- 이 클래스를 통해 HTTP GET, POST 등에 대한 API 테스트 가능

5. mvc.perform(get("/hello"))
- MockMvc를 통해 /hello 주소로 HTTP GET 요청을 함
- 체이닝이 지원되어 여러 검증 기능을 이어서 선언 가능

## 2.4 Hello Controller 코드를 롬복으로 전환하기
```java
class HelloResponseDtoTest {
    @Test
    public void 롬복_기능_테스트() {
        // given
        String name = "test";
        int amount = 1000;
        
        // when
        HelloResponseDto dto = new HelloResponseDto(name, amount);
        
        //then
        assertThat(dto.getName()).isEqualTo(name);
        assertThat(dto.getAmount()).isEqualTo(amount);
    }
}
```
1. assertThat
- assertj라는 테스트 검증 라이브러리의 검증 메소드
- 검증하고 싶은 대상을 메소드 인자로 받음
- 메소드 체이닝이 지원됨

2. isEqualTo
- assertj의 동등 비교 메소드
- assertThat에 있는 값과 isEqualTo의 값을 비교해서 같을 때만 성공

1. jsonPath
- JSON 응답값을 필드별로 검증할 수 있는 메소드
- $를 기준으로 필드명을 명시


# 3. 스프링 부트에서 JPA로 데이터베이스 다뤄보자
## 3.1 JPA 소개
- 관계형 데이터베이스와 객체지향 프로그래밍 언어 사이의 패러다임 불일치를 해결하기 위한 기술
- 스프링에서는 Spring Data JPA 모듈 이용
    - 구현체 교체의 용이성 : Hibernate 외에 다른 구현체로 쉽게 교체 가능
    - 저장소 교체의 용이성 : RDB 외에 다른 저장소로 쉽게 교체 가능

## 3.2 프로젝트에 Spring Data Jpa 적용하기
- Posts 클래스는 실제 DB 테이블과 매칭될 클래스이며 보통 Entity 클래스라고도 함

 ```java
@Getter
@NoArgsConstructor
@Entity
public class Posts {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;
    
    private String author;

    @Builder
    public Posts(String title, String content, String author){
        this.title = title;
        this.content = content;
        this.author = author;
    }
}
```
- setter 메소드가 없는 이유
    - setter가 있는 경우 해당 클래스의 인스턴스 값들이 언제 어디서 변해야 하는지 코드상으로 구분 불가
- 생성자를 통해 최종값을 채운 후 DB에 삽입해야 함
- @Builder를 통해 제공되는 빌더 클래스를 사용해 생성 시점에 값을 채워 줌

1. @Entitiy
- 테이블과 링크될 클래스임을 나타냄
- 기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍으로 테이블 이름을 매칭

2. @Id
- 해당 테이블의 PK 필드

3. @GeneratedValue
- PK의 생성 규칙을 나타냄
- 스프링 부트 2.0에서는 Genration Type.IDENTITY 옵션을 추가해야 auto_increment
```
- Entitiy의 PK는 Long 타입의 Auto_increment가 좋음
    - FK를 맺을 때 다른 테이블에서 복합키 전부를 갖고 있거나, 중간 테이블을 하나 더 둬야 하는 상황 발생
    - 인덱스에 좋지 않은 영향
    - 유니크한 조건이 변경될 경우 PK 전체를 수정해야 하는 일이 발생
- 주민등록번호, 복합키 등은 유니크 키로 별도로 추가하는 것이 나음
```

4. @Column
- 추가로 변경이 필요한 옵션이 있는 경우 사용

5. @NoArgsConstructor, @Getter
- 롬복의 어노테이션들은 코드 변경량을 최소화시켜 줌

6. @Builder
- 해당 클래스의 빌더 패턴 클래스 생성
- 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함

```java
public interface PostsRepository extends JpaRepository<Posts, Long> {}
```
- Entity 클래스와 기본 Entity Repository는 함께 위치해야 함
- Entity 클래스는 기본 Repository 없이는 제대로 역할하지 못함

## 3.3 Spring Data JPA 테스트 코드 작성하기
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class PostRepositoryTest {
    @Autowired
    PostsRepository postsRepository;

    @After
    public void cleanup(){
        postsRepository.deleteAll();
    }

    @Test
    public void 게시글저장_불러오기(){
        // given
        String title = "test text";
        String content = "test body";

        postsRepository.save(Posts.builder()
            .title(title)
            .content(content)
            .author("aaa@gmail.com").build());

        // when
        List<Posts> postsList = postsRepository.findAll();

        // then
        Posts posts = postsList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title);
        assertThat(posts.getContent()).isEqualTo(content);
    }

}
```
1. @After
- Junit에서 단위 테스트가 끝날 때마다 수행되는 메소드를 지정
- 보통 배포 전 전체 테스트를 수행할 때 테스트 간 데이터 침범을 막기 위해 사용
- 여러 테스트가 동시에 수행되면 테스트용 DB인 H2에 데이터가 그대로 남아 있어 다음 테스트 실행 시 테스트가 실패할 수도 있음

2. postsReposity.save
- 테이블에 insert/update 쿼리 실행
- id값이 있으면 update, 없으면 insert 쿼리 실행

3. postsRepository.findAll
- 테이블에 있는 모든 데이터 조회

### cf) 
- 쿼리가 보고 싶은 경우 `application.properties`에 옵션 추가
```yml
spring.jpa.show_sql=ture
```
- 출력되는 쿼리 로그를 MySQL 버전으로 변경
```yml
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
```

## 3.4 등록/수정/조회 API 만들기
- API를 위해 필요한 클래스
    - Controller
    - Service
        - 비즈니스 로직을 처리하는 역할이 아님
        - 트랜잭션, 도메인 간 순서 보장의 역할
    - Dto

- Spring Web 계층
    - Web Layer
        - @Controller, 뷰 템플릿 영역
        - 필터, 인터셉터, 컨트롤러 어드바이스 등 외부 요청과 응답에 대한 전반적인 영역

    - Service Layer
        - 일반적으로 Controller와 Dao의 중간 영역에서 사용됨
        - @Transactional이 사용되어야 하는 영역
    
    - Repository Layer
        - 데이터 저장소에 접근하는 영역
    
    - Dtos
        - 계층 간 데이터 교환을 위한 객체

    - Domain Model
        - 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화시킨 것을 도메인 모델이라 함
        - Entity가 사용된 영역도 도메인 모델
        - VO 값 객체들도 도메인 영역에 해당
        - 무조건 데이터베이스 테이블과 관계가 있어야 하는 것은 아님
        - **비즈니스 처리를 담당해야 하는 곳**

- 트랜잭션 스크립트 : 기존에 서비스로 처리하던 방식
    - 모든 로직이 서비스 클래스 내부에서 처리됨
    - 따라서 서비스 계층이 무의미하며, 객체란 단순히 데이터 덩어리 역할만 하게 됨

- 도메인 모델에서 비스니스 처리
    - 객체가 각자의 이벤트 처리
    - 서비스 메소드는 트랜잭션과 도메인 간의 순서만 보장해 줌

- Spring Bean 주입 빋는 방식
    - 생성자 주입을 가장 권장
    - 생성자로 Bean 객체를 받도록 하면 @Autowired와 동일한 효과를 볼 수 있다.
    - 생성자는 @RequiredArgsConstructor에서 해결해줌
    - final이 선언된 모든 필드를 인자값으로 하는 생성자를 롬복의 어노테이션이 대신 생성해줌

- Entity 클래스를 Request/Response 클래스로 사용해서는 안됨
    - Dto 클래스는 Entity 클래스와 거의 유사한 형태
    - Entity 클래스는 데이터베이스와 맞닿은 핵심 클래스
    - Entity 클래스를 기준으로 테이블이 생성되고 스키마가 변경됨
    - Entity 클래스가 변경되면 여러 클래스에 영향을 끼치기 때문에 View를 위한 Dto 클래스는 변경이 잦아 역할 분리 필요
    
- Api Controller 테스트
    - @WebMvcTest의 경우 JPA 기능이 작동하지 않음
    - Controller와 ControllerAdvice 등 외부 연동과관련된 부분만 활성화 됨
    - JPA 기능까지 한번에 테스트할 때는 @SpringBootTest와 TestRestTemplate 사용하면 됨

- update 기능
    - 데이터베이스에 쿼리를 날리는 부분이 없음
    - JPA의 영속성 컨텍스트 때문에 가능
    - 영속성 컨텍스트 : 엔티티를 영구 저장하는 환경
    - 트랜잭션 안에서 DB에서 데이터를 가져오면 이 데이터는 영속성 컨텍스트가 유지된 상태
    - 이 상태에서 해당 데이터의 값을 변경하면 트랜잭션이 끝나는 시점에 해당 테이블에 변경분을 반영함
    - Entity 객체의 값만 변경하면 별도로 Update 쿼리를 날릴 필요가 없음 - **더티체킹**

    
## 3.5 JPA Auditing으로 생성시간/수정시간 자동화하기
### 3.5.1 LocalDate 사용
```java
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseTimeEntity {
    @CreatedDate
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime modifiedDate;
}
```
- @MappedSuperclass : JPA Entity 클래스들이 BaseTimeEntity를 상속할 경우, 필드들(createdDate, modifiedDate)도 칼럼으로 인식하도록 함
- @EntityListeners(AuditingEntityListener.class) : BaseTimeEntity 클래스에 Auditing 기능 포함
- @CreatedDate : Entity가 생성되어 저장될 때 시간이 자동 저장됨
- @LastModifiedDate : 조회한 Entity의 값을 변경할 때 시간이 자동 저장됨  

- Posts 클래스가 BaseTimeEntity를 상속받도록 변경
- JPA Auditing 어노테이션들을 모두 활성화할 수 있도록 Application 클래스에 활성화 어노테이션 추가
    (`@EnableJpaAuditing`)

# 4. 머스테치로 화면 구성하기
## 4.2 기본 페이지 만들기
- URL 매핑은 Controller에서

## 4.3 게시글 등록 화면 만들기
- 머스테치 스타터 덕분에 경로와 파일 확장자는 자동으로 지정됨
- `{{> }}` : 현재 머스테치 파일을 기준으로 다른 파일을 가져옴
- 등록 버튼 기능
    - index.js
        - 브라우저의 스코프는 공용 공간으로 쓰이기 때문에 나중에 로딩된 js의 init, save가 먼저 로딩된 js의 function을 덮어쓰게 됨
        - 중복된 함수명을 피하기 위해 index.js만의 유효범위를 만들어 사용
        - `index`라는 객체를 만들어서 해당 객체에서 필요한 모든 function을 선언함.
    ``` javascript
    var index = {
        init : function() {
            var _this = this;
            $('#btn-save').on('click', function () {
                _this.save();
            });
        },
        save : function() {
            var data = {
                title: $('#title').val(),
                author: $('#author').val(),
                content: $('#content').val()
            };

            $.ajax({
                type: 'POST',
                url: '/api/v1/posts',
                dataType: 'json',
                contentType: 'application/json; charset=utf-8',
                data: JSON.stringify(data)
            }).done(function () {
                alert('글이 등록되었습니다.');
                window.location.href = '/';
            }).fail(function (error){
                alert(JSON.stringify(error));
            });
        }
    };

    index.init();
    ```
## 4.4 전체 조회 화면 만들기
1. 머스테치
```html
    <tbody id="tbody">
    {{#posts}}
        <tr>
            <td>{{id}}</td>
            <td>{{title}}</td>
            <td>{{author}}</td>
            <td>{{modifiedDate}}</td>
        </tr
    {{/posts}}
    </tbody>
```
- {{#posts}} : posts라는 List를 순회. (for 문과 동일)
- {{변수명}} : List에서 뽑아낸 객체의 필드 사용

2. Repository
```java
    public interface PostsRepository extends JpaRepository<Posts, Long> {
        @Query("SELECT p FROM Posts p ORDER BY p.id DESC")
        List<Posts> findAllDesc();
    }
```
- `@Query` : SpringDataJpa에서 제공하지 않는 메소드는 쿼리로 작성 가능

3. Service
```java
    @Transactional(readOnly = true)
    public List<PostsListResponseDto> findAllDesc(){
        return postsRepository.findAllDesc().stream()
                .map(PostsListResponseDto::new)
                .collect(Collectors.toList());
    }
```
- 트랜잭션 어노테이션 옵션 추가 (readOnly) : 트랜잭션 범위는 유지하되, 조회 기능만 남겨두어 조회 속도 개선
- .map(PostsListResponseDto::new) = .map(posts -> new PostsListResponseDto(posts))
    - postsRepository 결과로 넘어온 Posts의 Stream을 map을 통해 PostsListResponseDto로 변환 -> List로 반환하는 메소드

3. DTO 생성

4. Controller 수정
```java
    @GetMapping("/")
    public String index(Model model){
        model.addAttribute("posts", postsService.findAllDesc());
        return "index";
    }
```
- Model : 서버템플릿 엔진에서 사용할 수 있는 객체를 저장할 수 있음
- postsService.findAllDesc()로 가져온 결과를 posts로 index.mustache에 전달

## 4.5 게시글 수정, 삭제 화면 만들기
### 4.5.1 게시글 수정
1. mustache
- 머스테치는 객체의 필드 접근 시 점으로 구분 (ex: post.id)
- readonly : Input 태그에 읽기 가능만 허용하는 속성

2. js
```javascript
var index = {
    init : function() {
        var _this = this;
        ...
        $('#btn-update').on('click', function() {
            _this.update();
        })
    },
    ...
    update : function() {
        var data = {
            title: $('#title').val(),
            content: $('#content').val()
        };

        var id = $('#id').val();

        $.ajax({
            type: 'PUT',
            url: '/api/v1/posts/'+id,
            dataType: 'json',
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function () {
            alert('글이 수정되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    }
```
- $('#btn-update').on('click') : btn-update라는 id를 가진 html 엘리먼트에 click 이벤트가 발생할 때 update function을 실행하도록 이벤트 등록

3. 전체목록에서 수정페이지로 이동하도는 기능 추가
- index.mustache에서 타이틀에 a 태그 추가
- <td><a href="/posts/update/{{id}}">{{title}}</a></td>

4. Controller
```java
    @GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model){
        PostsResponseDto dto = postsService.findById(id);
        model.addAttribute("post", dto);
        return "posts-update";
    }
```


# 5. 스프링 시큐리티와 OAuth2.0으로 로그인 기능 구현하기
- 스프링 시큐리티 : 막강한 인증과 인가 기능을 가진 프레임워크

## 5.1 스프링 시큐리티와 스프링 시큐리티 Oauth2 클라이언트
- 1.5 방식 : accessTokenUrl, userAuthorazationUrl 등 url 주소를 모두 명시해야 함
- 2.0 방식에서는 client 인증 정보(clientId, clientSecret)만 입력하면 됨
- 1.5 버전에서 직접 입력했던 값들은 모두 enum으로 대체됨
- CommonOAuth2Provider라는 enum이 추가되어 구글, 깃허브, 페이스북, 옥타의 기본 설정값을 모두 제공
- 이외의 소셜 로그인은 직접 추가해야 함

## 5.2 구글 서비스 등록
1. 구글 서비스에 신규 서비스를 생성해야 생성 시 발급된 인증 정보를 통해 로그인, 소셜 서비스 기능 사용 가능
2. application-oauth 등록
    - src/main/resources에 application-oauth.properties 파일 생성
    - scope를 별도로 등록한 이유
        - 기본값이 openid, profile, email인데, openid가 있으면 Open Id Provider로 인식
        - 그렇게 되면 OpenId Provider 이외의 서비스(네이버, 카카오 등)를 나눠서 만들어야 함
    - application properties에 `sprig.profiles.include=oauth` 추가
```
    spring.security.oauth2.client.registration.google.client-id=클라이언트 ID
    spring.security.oauth2.client.registration.google.client-secret=클라이언트 보안 비밀
    spring.security.oauth2.client.registration.google.scope=profile,email
```
- cf) properties
    - application-xxx.properties로 properties를 만들면, xxx라는 profile이 생성되어 관리 가능
    - profile=xxx라는 식으로 호출하면 해당 properties의 설정을 가져 올 수 있음
3. .gitignore등록
    - application-oauth.properties

## 5.3 구글 로그인 연동하기
1. User 클래스 생성
2. 각 사용자의 권한을 관리할 Enum 클래스 Role 생성
- 스프링 시큐리티에서는 권한 코드에 항상 ROLE_이 앞에 있어야 함
```java
@Getter
@RequiredArgsConstructor
public enum Role {
    GUEST("ROLE_GUEST", "게스트"),
    USER("ROLE_USER", "일반 사용자");
    
    private final String key;
    private final String title;
}
```
3. UserRepository 생성

### 5.3.1 스프링 시큐리티 설정
1. build.gradle에 스프링 시큐리티 관련 의존성 추가
    - compile('org.springframework.boot:spring-boot-starter-oauth2-client')
2. OAuth 라이브러리를 이용한 소셜 로그인 설정 코드 작성
    - config.auth 패키지 생성 : 시큐리티 관련 클래스 담당
    - SecurityConfig 클래스 생성
    - CustomOAuth2UserService 클래스 생성
        - 구글 로그인 이후 가져온 사용자의 정보들을 기반으로 가입 및 정보수정, 세션 저장 등의 기능 지원
    - OAuthAttributes클래스 생성
    - config.auth.dto 패키지에 SessionUser 클래스 추가
        - SessionUser에는 인증된 사용자 정보만 필요
        - User 대신 SessionUser 클래스를 사용하는 이유
            - User 클래스가 엔티티이기 때문에 다른 엔티티와 관계가 형성될 수 있다
            - User 클래스에 직렬화 코드를 구현할 경우, @OneToMany, @ManyToMay 등 자식 엔티티를 가지고 있다면, 직렬화 대상에 자식들까지 포함되어 성능 이슈 및 부수 효과가 발생할 수 있음
            - 직렬화 기능을 가진 세션 Dto를 추가로 만드는 것이 운영 및 유지보수에 효과적

### 5.3.2 로그인 테스트
1. 화면에 로그인 버튼 추가
2. index.mustache에서 userName을 사용할 수 있도록 IndexController에서 userName을 model에 저장하는 코드 추가


## 5.4 어노테이션 기반으로 개선하기
- 세션값을 가져오는 부분
    - index 메소드 외에 다른 컨트롤러와 메소드에서 세션값이 필요하면 그때마다 직접 세션에서 값을 가져와야 함
    - 같은 코드가 계속해서 반복됨
    - 메소드 인자로 세션값을 바로 받을 수 있도록 변경

- @LoginUser 어노테이션 생성
    - @Target : 이 어노테이션이 생성될 수 있는 위치 지정
        - PARAMETER이므로 메소드의 파라미터로 선언된 객체에만 사용할 수 있음
    - @interface : 이 파일을 어노테이션 클래스로 지정
        - LoginUser라는 어노테이션이 생성됨
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {
}
```

- LoginUserArgumentResolver 생성
    - HandlerMethodArgumentResolver 인터페이스를 구현한 클래스
    - HandlerMethodArgumentResolver : 조건에 맞는 경우 메소드가 있다면 HandlerMethodArgumentResolver의 구현체가 지정한 값으로 해당  메소드의 파라미터로 넘길 수 있다.
    - supportsParameter()
        - 컨트롤러 메소드의 특정 파라미터를 지원하는지 판단
        - 파라미터에 @LoginUser 어노테이션이 붙어있고, 파라미터 클래스 타입이 SessionUser.class인 경우 true 반환
    - resolveArgument()
        - 파라미터에 전달할 객체 생성(세션에서 객체를 가져옴)

```java
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

- LoginUserArgumentResolver를 스프링에서 인식될 수 있도록 WebMvcConfigurer에 추가
    - config 패키지에 WebConfig 클래스 생성
    - HandlerMethodArgumentResolver는 항상 WebMvcConfigure의 addArgumentResolvers()를 통해 추가해야 함

- IndexController 코드 @LoginUser로 개선하도록 개선


## 5.5 세션 저장소로 데이터베이스 사용하기
- 현재 서비스는 세션이 내장 톰캣의 메모리에 저장되기 때문에 어플리케이션을 재실행하면 로그인이 풀림
- 배포할 때마다 톰캣이 재시작됨
- 2대 이상의 서버에서 서비스하고 있다면 톰캣마다 세션 동기화 설정을 해야 함

- 세션 저장소에 대한 선택
    1. 톰캣 세션 사용
        - 2대 이상의 WAS가 구동되는 환경에서는 톰캣들 간의 세션 공유를 위한 추가 설정이 필요함
    2. 데이터베이스를 세션 저장소로 사용
        - 여러 WAS 간 공용 세션을 사용할 수 있는 가장 쉬운 방법
        - 로그인 요청시마다 DB IO가 발생함
        - 보통 로그인 요청이 많이 없는 배오피스, 사내 시스템 용도에서 사용
    3. Redis, Memcached와 같은 메모리 DB를 세션 저장소로 사용
    
### 5.5.1 spring-session-jdbc 등록
1. build.gradle에 의존성 등록
```    
compile('org.springframework.session:spring-session-jdbc')
```
2. application.properties에 세션 저장소를 jdbc로 선택하도록 코드 추가
```
spring.session.store-type=jdbc
```

## 5.6 네이버 로그인
### 5.6.1 네이버 API 등록
- 네이버에서는 스프링 시큐리티를 공식 지원하지 않기 때문에 Common-OAuth2Provider에서 해주던 값들도 전부 수동으로 입력해야 함
- 스프링 시큐리티에선 하위 필드를 명시할 수 없다.
- 최상위 필드들만 user_name으로 지정 가능. 네이버 응답값 최상위 필드는 resultCode, message, response.

### 5.6.2 스프링 시큐리티 설정 등록
1. OAuthAttriibutes에 네이버인지 판단하는 코드와 네이버 생성자 추가
2. index.mustache에 네이버 로그인 버튼 추가


## 5.7 기존 테스트에 시큐리티 적용하기
- 시큐리티 옵션이 활성화되면 인증된 사용자만 API를 호출할 수 있다.
- 기존의 API 테스트 코드들이 모두 인증에 대한 권한을 받지 못하였으므로, 테스트 코드마다 인증한 사용자가 호출한 것처럼 작동하도록 수정

### 문제1. CustomOAuth2UserService을 찾을 수 없음
- 소셜 로그인 관련 설정값들이 없기 때문에 발생
- src/main과 src/test 환경 차이 때문에 application-oauth.properties 설정을 읽지 못함
- test에 application.properties가 없으면 main의 설정을 그대로 가져옴
- 가짜 설정값 등록

### 문제2. 302 Status Code
- 스프링 시큐리티 설정 때문에 인증되지 않은 사용자의 요청은 이동시킨다.
- API 요청은 임의로 인증된 사용자를 추가하여 API만 테스트 할 수 있도록 한다.
- spring-security-test를 build.gradle에 추가
- PostsApiControllerTest의 2개 테스트 메소드에 임의 사용자 인증 추가
    - @WithMockUser(roles="USER")
        - 인증된 모의 사용자를 만들어서 사용
        - roles에 권한을 추가할 수 있음
        - 이 어노테이션은 MockMvc에서만 작동하기 때문에 @SpringBootTest로만 되어 있는 PostApiContrllerTest에서는 작동하지 않음

- @SpringBootTest에서 MockMvc를 사용하는 방법
    1. @Before : 매번 테스트가 시작되기 전 MockMvc 인스턴스 생성
    2. mvc.platform : 생성된 MockMvx를 통해 API 테스트. 본문 영역은 Object Mapper를 통해 문자열을 JSON으로 변환

### 문제3. @WebMvcTest에서 CustomOAuth2UserService를 찾을 수 없음
- HelloControllerTest는 @WebMvcTest를 사용함
- @WebMvcTest는 CustomOAuth2UserService를 스캔하지 않음
- @WebMvcTest는 WebSecurityConfigurerAdapter, WebMvcConfigurer를 비롯한 @ControllerAdvice, @Controller를 읽는다
- 따라서 SecurityConfig를 생성하기 위해 필요한 CustomOAuth2UserService는 읽을 수 없어서 에러 발생
- 스캔대상에서 SecurityConfig를 제거해야 함
    ``` java
    @WebMvcTest(controllers = HelloController.class, excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = SecurityConfig.class)
    })

    ```
- @WithMockUser를 사용해서 가짜로 인증된 사용자를 생성한다.
- @EnableJpaAuditing을 사용하기 위해서는 최소 하나의 @Entity 클래스가 필요함
    - @WebMvcTest는 @Entity 클래스가 없음
- @EnableJpaAuditing이 @SpringBootApplication과 함께 있어서 @WebMvcTest에서도 스캔하게 되므로 둘을 분리
    - Application.java에서 @EnableJpaAuditing 제거
    - config 패키지에 JpaConfig를 생성하여 @EnableJpaAuditing 추가
    
