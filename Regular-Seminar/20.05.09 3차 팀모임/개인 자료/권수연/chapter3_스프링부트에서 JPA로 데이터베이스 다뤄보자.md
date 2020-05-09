- JPA : 모델에 객체를 매핑, 자바표준ORM
- iBatis, MyBatis : SQL 매퍼, 쿼리를 매핑

### 3.1 JPA 소개

- 객체를 관계형 데이터베이스에서 관리하는 것이 중요하다.
- RDB는 **어떻게 데이터를 저장**할지에 초점
- OOP는 **기능과 속성을 한 곳에서 관리**하고자함
- RDB와 OOP의 패러다임 불일치 -> 중간에서 패러다임을 일치시켜주는 JPA 등장
    - 개발자는 **SQL에 종속적이지 않고, 객체지향적으로** 프로그래밍ok
    - JPA가 코드를 RDB에 맞게 **SQL을 대신 생성해서 실행**

- **Spring Data JPA**
    - JPA interface <- Hibernate 구현 <- 한단계 더 감싼 **Spring Data JPA 사용 권장**
    - 구현체 교체의 용이성 & 저장소 교체의 용이성으로 Hibernate 대신 Spring Data JPA 사용
    - 구현체 교체의 용이성
        - Hibernate 외에 다른 구현체로 쉽게 교체 가능
    - 저장소 교체의 용이성
        - RDB이외에 다른 저장소로 쉽게 교체 가능

- 높은 러닝커브
- OOP와 RDB 둘다 이해해야함
- CRUD 쿼리를 직접 짤 필요없음
- 성능이슈? 
    - 성능이슈해결 관련 자료를 활용하면 네이티브 쿼리만큼의 퍼포를 낼 수 있음

### 3.2 프로젝트에 Spring Data Jpa 적용하기

- `build.gradle`
```gradle
dependencies { //사용할 의존성
    compile("org.springframework.boot:spring-boot-starter-web")
    compile('org.projectlombok:lombok') //롬복
    compile('org.springframework.boot:spring-boot-starter-data-jpa') //Spring Data Jpa
    compile('com.h2database:h2') //h2 : 인메모리 RDB, 애플리케이션 재시작시마다 초기화->테스트용
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```
- 요구사항 : 게시판 만들기

- Post 도메인을 담을 `java/{package}/domain/posts/Posts.java` 생성

```java
@Getter
@NoArgsConstructor //기본생성자 자동추가
@Entity //JPA, DB 테이블과 매칭될 클래스=Entity class, 절대 Setter를 만들지 않는다.
public class Posts {
    @Id //PK
    @GeneratedValue(strategy = GenerationType.IDENTITY) //PK 생성규칙 GenerationType.IDENTITY만 auto-increment됨
    private Long id;

    @Column(length = 500, nullable = false) //칼럼, 어노테이션 안붙여도 모든 필드는 칼럼이 됨
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author; //VARCHAR 는 255가 기본값

    @Builder //빌더패턴 클래스 생성, 생성자에 포함된 필드만 빌더에 포함
    private Posts(String title, String content, String author){
        this.title = title;
        this.content = content;
        this.author = author;
    }
}
```
- [빌더 패턴이란?](https://jdm.kr/blog/217)
    -  생성자에 들어갈 매개 변수가 많든 적든 차례차례 매개 변수를 받아들이고 모든 매개 변수를 받은 뒤에 이 변수들을 통합해서 한번에 사용

- @Entity 클래스에는 **절대 Setter를 만들지 않는다**
    - 그럼?
    - 생성자를 통해 최종값을 채운 후 insert
    - 값 변경이 필요한 경우, 해당 이벤트에 맞는 public 메소드를 호출

- Repository 생성
    - `java/{package}/domain/posts/PostsRepository.java` 생성

    ```java
    import org.springframework.data.jpa.repository.JpaRepository;

    //DB Layer 접근자
    //JpaRepository<Entitiy 클래스, PK타입> 상속->CRUD 자동생성
    //Entity 클래스와 Entity Repository는 함께 위치해야함
    public interface PostsRepository extends JpaRepository<Posts, Long> {
    
    }
    ```

### 3.3 Spring Data JPA 테스트 코드 작성하기

- 테스트(`/test/java/{패키지}/PostsRepositoryTest.java`)

```java
@RunWith(SpringRunner.class)
@SpringBootTest //h2 db 자동으로 실행해줌
public class PostsRepositoryTest {

    @Autowired
    PostsRepository postsRepository;

    @After // 단위 테스트가 끝나고 수행됨, 테스트간 데이터 침범을 막기위해
    public void cleanup(){
        postsRepository.deleteAll();
    }

    @Test
    public void loadPosts() {
        //given
        String title = "테스트 게시글";
        String content = "테스트 본문";

        //post 테이블에 insert,update 쿼리 실행
        postsRepository.save(Posts.builder()
                .title(title)
                .content(content)
                .author("kwonsye@naver.com")
                .build());
        //when
        List<Posts> postsList = postsRepository.findAll(); //모든 데이터 조회
        //then
        Posts posts = postsList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title);
        assertThat(posts.getContent()).isEqualTo(content);
    }
}
```

### 3.4 등록/수정/조회 API 만들기

- Service layer 에서는 
    - 비지니스 로직을 처리x
    - **트랜잭션과 도메인 간의 순서 보장**만
- **Domain 에서 비즈니스 로직**을 처리함

- Spring 웹 계층

    - Web layer
        - View template, controller 등
        - 외부 요청과 응답에 대한 영역
    - Service Layer
        - **Contoller와 Dao의 중간**에서 사용된다.
        - @Transactional이 사용되어야 하는 영역
    - Repository Layer
        - **DB와 같이 데이터 저장소에 접근하는 영역**이다.
    - DTOs
        - 계층 간의 데이터 교환을 위한 객체이다. 
        - Web layer에서 사용될 객체나 Repository Layer에서 결과로 넘겨준 객체들
        - View를 위한 객체 클래스
    - Domain Model
        - 도메인이라고 불리는 개발대상
        - @Entity 가 사용된 영역
        - VO 


- Controller 생성
```java
@RequiredArgsConstructor //Autowired 없어도 빈주입되는 이유
@RestController
public class PostsApiController {

    private final PostsService postsService;

    @PostMapping("/api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto requestDto){
        return postsService.save(requestDto);
    }

    @PutMapping("/api/v1/posts/{id}")
    public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto){
        return postsService.update(id, requestDto);
    }

    @GetMapping("/api/v1/posts/{id}")
    public PostsResponseDto findById(@PathVariable Long id){
        return postsService.findById(id);
    }
}
```

- Service 생성
```java
@RequiredArgsConstructor //Autowired가 없어도 주입되는 이유
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    @Transactional
    public Long save(PostsSaveRequestDto requestDto){
        return postsRepository.save(requestDto.toEntity()).getId();
    }

    @Transactional
    public Long update(Long id, PostsUpdateRequestDto requestDto){
        Posts post = postsRepository.findById(id)
                .orElseThrow(()->new IllegalArgumentException("해당 게시글이 없습니다. id="+id));

        //update 쿼리를 날리는 부분이 없음, JPA의 영속성 컨텍스트(엔티티를 영구 저장하는 환경) 때문임
        post.update(requestDto.getTitle(), requestDto.getContent());
        return id;
    }

    public PostsResponseDto findById(Long id){
        Posts entity = postsRepository.findById(id)
                .orElseThrow(()->new IllegalArgumentException("해당 게시글이 없습니다. id="+id));

        return new PostsResponseDto(entity);
    }
}
``` 
- **JPA 영속성 컨텍스트**
    - **엔티티를 영구 저장**하는 환경

    - JPA의 엔티티 메니저가 활성화된 상태(Spring Data Jpa의 기본옵션)로 **트랜잭션 안에서 DB에서 데이터를 가져오면** 이 데이터는 영속성 컨텍스트가 유지된 상태

    - 트랜잭션 안에서 DB 데이터를 가져와서 데이터 값을 변경하면 **트랜잭션이 끝나는 시점에 자동으로 해당 테이블에 변경분을 반영** = 더티체킹(dirty checking)

<br>

- 글 저장 Request시 받을 PostsSaveRequestDto 생성
```java
@Getter
@NoArgsConstructor
public class PostsSaveRequestDto {
    //Posts Entity와 거의 동일하지만 새로생성<-절대 Entity 클래스를 Request/Response 클래스로 사용하면 안됨
    private String title;
    private String content;
    private String author;

    @Builder
    public PostsSaveRequestDto(String title, String content, String author){
        this.title=title;
        this.content=content;
        this.author=author;
    }

    public Posts toEntity(){
        return Posts.builder()
                .title(title)
                .content(content)
                .author(author)
                .build();
    }
}
```

- 해당 id의 게시글을 수정할때 요청받을 PostsUpdateRequestDto 생성
```java
@Getter
@NoArgsConstructor
public class PostsUpdateRequestDto {
    private String title;
    private String content;

    @Builder
    public PostsUpdateRequestDto(String title, String content){
        this.title = title;
        this.content = content;
    }
}
```
<br>

- 해당 id의 게시글을 찾아 줄때 담을 PostsResponseDto 생성

```java

@Getter
public class PostsResponseDto {

    private Long id;
    private String title;
    private String content;
    private String author;

    public PostsResponseDto(Posts entity){
        this.id = entity.getId();
        this.title = entity.getTitle();
        this.content = entity.getContent();
        this.author = entity.getAuthor();
    }
}
```


- 글 등록/수정 테스트

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT) //JPA 기능도 test 가능
public class PostsApiControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;//JPA 기능도 test 가능

    @Autowired
    private PostsRepository postsRepository;

    @After
    public void tearDown() throws Exception{
        postsRepository.deleteAll();
    }

    @Test //글 등록 테스트
    public void registerPosts() throws Exception{
        //given
        String title = "title";
        String content = "content";
        PostsSaveRequestDto requestDto = PostsSaveRequestDto.builder()
                .title(title)
                .content(content)
                .author("author")
                .build();
        String url = "http://localhost:" + port + "/api/v1/posts";

        //when
        ResponseEntity<Long> responseEntity = restTemplate.postForEntity(url,requestDto,Long.class);

        //then
        assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);

        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);

    }
    
    @Test //글 수정 테스트
    public void updatePosts() throws Exception{
        //given
        Posts savedPost = postsRepository.save(Posts.builder()
            .title("test title")
            .content("test content")
            .author("test author")
            .build());
        
        Long updateId = savedPost.getId();
        String expectedTitle = "test title2";
        String expectedContent = "test content2";
        PostsUpdateRequestDto requestDto = PostsUpdateRequestDto.builder()
                .title(expectedTitle)
                .content(expectedContent)
                .build();
        
        String url = "http://localhost:" + port + "/api/v1/posts/" + updateId;

        HttpEntity<PostsUpdateRequestDto> requestEntity = new HttpEntity<>(requestDto);
        
        //when
        ResponseEntity<Long> responseEntity = restTemplate.exchange(url, HttpMethod.PUT, requestEntity, Long.class);
        
        //then
        assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);
        
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent()).isEqualTo(expectedContent);
        
    }
}
```

<br>

### 3.5 JPA Auditing 으로 생성시간/수정시간 자동화하기

- Java8 LocalDate 사용
- `/java/{package}/domain/BaseTimeEntity.java`

```java
@Getter
@MappedSuperclass //JPA Entity 클래스들이 이 클래스를 상속할 경우 필드들을 칼럼에 추가함, 장고의 abstract = True 랑 비슷
@EntityListeners(AuditingEntityListener.class) //Auditing 기능 포함
public class BaseTimeEntity {
    
    @CreatedDate //entity가 생성되어 저장될 때 시간이 자동저장되는 필드
    private LocalDateTime createdDate;
    
    @LastModifiedDate // entity 값이 변경될 때 시간이 자동 저장되는 필드
    private LocalDateTime modifiedDate; 
}
```

- [JPA Auditing 기능이란?]('https://webcoding-start.tistory.com/53')
- `Posts Entity`에서 `BaseTimeEntity` 상속
- `Application.java`에서 `@EnableJpaAuditing` 추가

- JPA Auditing 테스트

```java
@Test
    public void addBaseTimeEntity() {
        LocalDateTime now = LocalDateTime.of(2020,4,24,0,0,0);
        postsRepository.save(Posts.builder()
                .title("title")
                .content("content")
                .author("author")
                .build());

        List<Posts> postsList = postsRepository.findAll();
        Posts post = postsList.get(0);

        System.out.println(">>>>>>>>> create Date=" + post.getCreatedDate() +
                ", modified Date=" + post.getModifiedDate()); //BaseTimeEtitiy의 필드들이 자동으로 추가됨

        assertThat(post.getModifiedDate()).isAfter(now);
        assertThat(post.getModifiedDate()).isAfter(now);
    }
    ```