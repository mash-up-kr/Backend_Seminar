# 3장 스프링 부트에서 JPA로 데이터베이스 다뤄보자

## CONTENTS
- [JPA 소개](#JPA-소개)
- [프로젝트에 Spring Data Jpa 적용하기](#프로젝트에-Spring-Data-Jpa-적용하기)
- [Spring Data JPA 테스트 코드 작성하기](#Spring-Data-JPA-테스트-코드-작성하기)
- [등록/수정/조회 API 만들기](#등록/수정/조회-API-만들기)
- [JPA Auditing으로 생성시간/수정시간 자동화하기](#JPA-Auditing으로-생성시간/수정시간-자동화하기)

---

### JPA 소개
- 등장 배경
  - 단순 반복 작업의 문제
    - 반복적인 SQL 투성이
  - 패러다임 불일치
    - RDB와 OOP
      - 관계형 데이터베이스 => 어떻게 데이터를 저장할지
      - 객체지향 프로그래밍 => 메시지를 기반으로 기능과 속성을 한 곳에서 관리
    - 패러다임이 다른데 객체를 데이터베이스에 저장하려고 하니 여러 문제가 발생
      - SQL에 종속적인 개발
- ORM
  - RDB와 OOP 중간에서 패러다임 일치를 시켜주기 위한 기술
  - 개발자는 객체지향젹으로 프로그래밍을 하고, ORM이 이를 관계형 데이터베이스에 맞게 SQL을 대신 생성해서 실행한다.
      - SQL에 종속적이지 않아도 된다.
  - 생산성 향상은 물론 유지 보수 용이
- JPA
  - Java Persistence API
  - 기술 명세, 자바 표준명세서
  - 자바 어플리케이션에서 관계형 데이터베이스를 사용하는 방식을 정의한 인터페이스
- Hibernate, EclipseLink
  - JPA의 구현체
- Spring Data JPA
  - 스프링 진영에서 JPA의 구현체들을 좀 더 쉽게 사용하고자 추상화 시킨 모듈
  - Repository 인터페이스를 제공
  - 권장
    - 구현체 교체의 용의성
      - 구현체 매핑 지원
    - 저장소 교체의 용의성
      - 의존성 교체(기본적인 CRUD 인터페이스가 같다)

### 프로젝트에 Spring Data Jpa 적용하기
- build.gradle에 의존성 등록
  ```gradle
  compile('org.springframework.boot:spring-boot-starter-data-jpa')
  compile('com.h2database:h2')
  ```
  - spring-boot-start-data-jpa
    - 스프링 부트용 Srping Data Jpa 추상화 라이브러리
    - 스프링 부트 버전에 맞춰 자동으로 JPA 관련 라이브러리들의 버전을 관리해 줍니다.
  - h2
    - 인메모리 관계형 데이터베이스
    - 별도의 설치 필요 없이 프로젝트 의존성만으로 관리할 수 있다.
    - 메모리에서 실행되기 때문에 애플리케이션을 재시작할 대마다 초기화된다.
      - 테스트 용도로 많이 사용
- `Posts 클래스`
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
      public Posts(String title, String content, String author) {
          this.title = title;
          this.content = content;
          this.author = author;
      }
  }
  ```
  - 실제 DB 테이블과 매칭될 Entity 클래스
  - @Entity
    - 테이블과 링크될 클래스임을 나타낸다.
    - 기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍으로 테이블 이름을 매칭한다.
  - @Id
    - 해당 테이블의 PK 필드
  - @GeneratedValue
    - PK의 생성 규칙
    - 스프링부트 2.0에서는 GenerationType.IDENTITY 옵션을 추가행야만 AI가 된다.
      - [Spring Boot Data JPA 2.0 에서 id Auto_increment 문제 해결](https://jojoldu.tistory.com/295)
  - @Column
    - 테이블의 칼럼을 나타내며 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 칼럼이 된다.
    - 기본값 외에 추가로 변경이 필요한 옵션이 있으면 사용한다.
  - @NoArgsConstructor
    - 기본 생성자 자동 추가
  - @Builder
    - 해당 클래스의 빌더 패턴 클래스를 생성
    - 생성자 상단에 선언 시 생성장에 포함된 필드만 빌더에 포함
  - Entity의 PK
    - Long 타입의 auto_increment를 추천한다.
    - 주민등록번호와 같이 비즈니스상 유니크 키나, 복합키로 PK를 잡을 경우 난감한 상황 발샐
      - FK를 맺을 때 다른 테이블에서 복합키 전부를 갖고 있거나, 중간 테이블을 하나 더 둬야하는 상황 발생
      - 인덱스에 악영향
      - 유니크한 조건이 변경될 경우 PK 전체를 수정해야 하는 일이 발생
    - 주민등록번호, 복합키 등은 유니크 키로 별도로 추가하는 것을 추천
- `PostsRepository 인터페이스`
  ```java
  import org.springframework.data.jpa.repository.JpaRepository;

  public interface PostsRepository extends JpaRepository<Posts, Long> {
      
  }
  ```
  - Posts 클랴스로 DB를 접근하게 해줄 JpaRepository
    - MyBatis 등에서 Dao라고 불리는 DB Layer 접근자
  - JpaRepository를 상속하면 CRUD 메소드가 자동으로 생성된다.
    - PaginAndSortingRepository => CrudRepository
- Entity 클래스와 기본 Entity Repository는 함께 위치해야 한다.

### Spring Data JPA 테스트 코드 작성하기
- `PostsRepositoryTest 클래스`
  ```java
  @RunWith(SpringRunner.class)
  @SpringBootTest
  public class PostsRepositoryTest {
      @Autowired
      PostsRepository postsRepository;

      @After
      public void cleanup() {
          postsRepository.deleteAll();
      }

      @Test
      public void 게시글저장_불러오기() {
          // given
          String title = "테스트 게시글";
          String content = "테스트 본문";

          postsRepository.save(Posts.builder()
                  .title(title)
                  .content(content)
                  .author("jobata")
                  .build());

          // when
          List<Posts> postsList = postsRepository.findAll();

          // then
          Posts posts = postsList.get(0);
          assertThat(posts.getTitle()).isEqualTo(title);
          assertThat(posts.getContent()).isEqualTo(content);
      }
  }
  ```
  - save, findAll 기능을 테스트
  - @After
    - Junit에서 단위 테스트가 끝날 때마다 수행되는 메소드를 지정
    - 보통은 배포 전 전체 테스트를 수행할 때 테스트간 데이터 침법을 막기 위해 사용
      - 여러 테스트가 동시에 수행되면 테스트용 데이터베이스인 H2에 데이터가 그대로 남아 있어 다음 테스트 실행 시 테스트가 실패할 수 있다.
  - postsRepository.save
    - 테이블 posts에 insert/update 쿼리를 실행
      - id값이 있다면 update, 없다면 insert
  - postsRepository.findAll
    - 테이블 posts에 있는 모든 데이터를 조회
- 외부 설정 파일 생성
  - src/main/resources 디렉토리 아래에 `application.properties` 생성
    - yaml으로도 작성 가능.
  - 값을 정의하면 스프링 부트 어플리케이션의 환경설정 혹은 설정값을 정할 수 있다.

### 등록/수정/조회 API 만들기
- Spring 웹 계층
![image](https://www.petrikainulainen.net/wp-content/uploads/spring-web-app-architecture.png)
  - Web Layer
    - @Controller와 JSP/Freemarker 등의 뷰 템플릿 영역
    - 외부 요청과 응답에 대한 전반적인 영역
      - @Filter, 인터셉터, @ControllerAdvice 등
  - Service Layer
    - @Service에 사용되는 영역
    - Controller와 DAO의 중간 영역
    - @Transactional이 사용되어야 하는 영역
  - Repository Layer
    - DB와 같이 데이터 저장소에 접근하는 영역
      - DAO
  - Dtos
    - 계층 간에 데이터 교환을 위한 객체인 DTO 영역
  - Domain Moddel
    - 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화시킨 것
      - 예를 들면 택시 앱이라고 하면 배차, 탑승, 요금 등이 모두 도메인이 될 수 있다.
    - @Entity가 사용된 영역 역시 도메인 모델
    - 다만, 무조건 DB 테이블과 관계가 있엉야만 하는 것은 아니다.
      - VO와 같은 값 객체들도 역시 이 영역에 해당
- 오해
  - Service에서 비지니스 로직을 처리해야 한다? => 트랜잭션 스크립트
    - No, Service는 트랜잭션, 도메인 간 순서 보장의 역할
      - 그럼 비즈니스 로직은 누가?
        - Domain
  - 트랜잭션 스크립트
    ```java
    @Transactional
    public Order cancelOrder(int orderId) {
      OrdersDto order = ordersDao.selectOrders(orderId);
      BillingDto billing = billingDao.selectBilling(orderId);
      DeliveryDto delivery = deliveryDDao.selectBilling(orderId);

      String deliveryStatus = delivery.getStatus();

      if ("IN_PROGRESS".equals(deliveryStatus)) {
        delivery.setStatus("CANCEL");
        deleveryDao.update(delivery);
      }

      order.setStatus("CANCEL");
      ordersDao.update(order);

      billing.setStatus("CANCEL");
      deliveryDao.update(billing);

      return order;
    }
    ```
    - 모든 로직이 서비스 클래스 내부에서 처리된다.
    - 서비스 계층이 무의미하며, 객체란 단순히 데이터 덩어리 역할만 하게 된다.
  - 도메인 모델
    ```java
    @Transactional
    public Order cancelOrder(int OrderId) {
      Orders order = ordersRepository.findById(orderId);
      Billing billing = billingRepository.findByOrderId(orderId);
      Delivery delivery = deliveryRepository.findByOrderId(orderId);

      delivery.cancel();

      order.cancel();

      billing.cancel();

      return order;
    }
    ```
    - order, billing, delivery가 각자 본인의 취소 이벤트 처리를 한다.
    - 서비스 메소드는 트랜젹선과 도메인 간의 순서만 보장해준다.
- `PostsSaveRequestDto 클래스`
  ```java
  @Getter
  @NoArgsConstructor
  public class PostsSaveRequestDto {
      private String title;
      private String content;
      private String author;

      @Builder
      public PostsSaveRequestDto(String title, String content, String author) {
          this.title = title;
          this.content = content;
          this.author = author;
      }

      public Posts toEntity() {
          return Posts.builder()
                  .title(title)
                  .content(content)
                  .author(author)
                  .build();
      }
  }
  ```
  - Controller와 Service에서 사용할 Dto 클래스
  - 의문
    - 왜 Entity 클래스와 거의 유사한 형태임에도 Dto 클래스를 추가로 생성했을까?
      - Entity 클래스를 Request/Response 클래스로 사용해서는 안된다. why?
        - Entity 클래스는 DB와 맞닿은 핵심 클래스이다.
          - Entity 클래스를 기준으로 테이블이 생성되고 스키마가 변경된다.
        - View의 변경은 아주 사소한 기능 변경인데 이를 위해 테이블과 연결된 Entity 클래스를 변경하는 것은 너무 큰 변경이다.
          - 수많은 서비스 클래스나 비즈니스 로직들이 Entity 클래스를 기준으로 동작한다.
          - Entity 클래스가 변경되면 여러 클래스에 영향을 끼치지기 때문에 View를 위한 Dto 클래스를 따로 만들어 주는 것잉 좋다.
        - 결국 View Layer와 DB Layer의 역할 분리를 철저히 하기 위함!
- `PostsService 클래스`
  ```java
  @RequiredArgsConstructor
  @Service
  public class PostsService {
      private final PostsRepository postsRepository;

      @Transactional
      public Long save(PostsSaveRequestDto requestDto) {
          return postsRepository.save(requestDto.toEntity()).getId();
      }
  }
  ```
- `PostsApiController 클래스`
  ```java
  @RequiredArgsConstructor
  @RestController
  public class PostsApiController {
      private final PostsService postsService;

      @PostMapping("/api/v1/posts")
      public Long save(@RequestBody PostsSaveRequestDto requestDto) {
          return postsService.save(requestDto);
      }
  }
  ```
- Dependency Injection
  - Field / Setter / Constructor Based Injection
- [스프링 - 생성자 주입을 사용해야 하는 이유, 필드인젝션이 좋지 않은 이유](https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/)
- `PostApiControllerTest 클래스`
  ```java
  @RunWith(SpringRunner.class)
  @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
  public class PostsApiControllerTest {
      @LocalServerPort
      private int port;

      @Autowired
      private TestRestTemplate restTemplate;

      @Autowired
      private PostsRepository postsRepository;

      @After
      public void tearDown() throws Exception {
          postsRepository.deleteAll();
      }

      @Test
      public void Posts_등록된다() throws Exception {
          //given
          String title = "title";
          String content = "content";
          String author = "author";
          PostsSaveRequestDto requestDto = PostsSaveRequestDto.builder()
                  .title(title)
                  .content(content)
                  .author(author)
                  .build();

          String url = "http://localhost:" + port + "/api/v1/posts";

          //when
          ResponseEntity<Long> responseEntity = restTemplate.postForEntity(url, requestDto, Long.class);

          //then
          assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
          assertThat(responseEntity.getBody()).isGreaterThan(0L);

          List<Posts> all = postsRepository.findAll();
          assertThat(all.get(0).getTitle()).isEqualTo(title);
          assertThat(all.get(0).getContent()).isEqualTo(content);
          assertThat(all.get(0).getAuthor()).isEqualTo(author);
      }
  }
  ```
  - Api Controller 테스트
  - 이전의 HelloController 테스트의 @WebMvcTest는 JPA 기능이 작동하지 않는다.
    - Controller와 ControllerAdvice 등 외부 연동과 관련된 부분만 활성화 된다.
  - JPA 기능까지 한번에 테스트할 때는 @SpringBootTest와 @TestRestTemplate을 사용하면 된다.
- `PostUpdateRequestDto 클래스`
  ```java
  @Getter
  @NoArgsConstructor
  public class PostsUpdateRequestDto {
      private String title;
      private String content;
      
      @Builder
      public PostsUpdateRequestDto(String title, String content) {
          this.title = title;
          this.content = content;
      }
  }
  ```
- `PostsResponseDto 클래스`
  ```java
  @Getter
  public class PostsReponseDto {
      private Long id;
      private String title;
      private String content;
      private String author;
      
      public PostsReponseDto(Posts entity) {
          this.id = entity.getId();
          this.title = entity.getTitle();
          this.content = entity.getContent();
          this.author = entity.getAuthor();
      }
  }
  ```
- `Posts 클래스`
  ```java
  public void update(String title, String content) {
      this.title = title;
      this.content = content;
  }
  ```
- `PostsService 클래스`
  ```java
  @Transactional
  public Long update(Long id, PostsUpdateRequestDto requestDto) {
      Posts posts = postsRepository.findById(id).orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id=" + id));
      
      posts.update(requestDto.getTitle(), requestDto.getContent());

      return id;
  }

  public PostsResponseDto findById(Long id) {
      Posts entity = postsRepository.findById(id).orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id=" + id));

      return new PostsResponseDto(entity);
  }
  ```
  - update 부분에서 DB에 쿼리를 날리는 부분이 없다.
    - JPA의 영속성 컨텍스트 때문이다.
      - 엔티티를 영구 저장하는 환경
      - JPA의 핵심 내용은 엔티티가 영속성 컨텍스트에 포함되어 있냐 아니냐로 갈린다.
      - JPA의 EntityManager가 활성화된 상태로 트랜잭션 안에서 DB에서 데이터를 가져오면 이 데이터는 영속성 컨텍스트가 유지된 상태이다.
      - 이 상태에서 해당 데이터의 값을 변경하면 트랜잭션이 끝나는 시점에 해당 테이블에 변경분을 반영한다.
      - 즉, Entity 객체의 값만 변경하면 별도로 Update 쿼리를 날릴 필요가 없다.
      - [더티 체킹 (Dirty Checking)이란?](https://jojoldu.tistory.com/415)
- `PostsApiController 클래스`
  ```java
  @PutMapping("/api/v1/posts/{id}")
  public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto) {
      return postsService.update(id, requestDto);
  }
  
  @GetMapping("/api/v1/posts/{id}")
  public PostsResponseDto findById(@PathVariable Long id) {
      return postsService.findById(id);
  }
  ```
- `PostApiControllerTest 클래스`
  ```java
  @Test
  public void Posts_수정된다() throws Exception {
      //given
      Posts savedPosts = postsRepository.save(Posts.builder()
              .title("title")
              .content("content")
              .author("author")
              .build());

      Long updateId = savedPosts.getId();
      String expectedTitle = "title2";
      String expectedContent = "content2";

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
  ```
- H2 웹콘솔 활성화 하기
  - `application.properties`
    ```
    spring.h2.console.enabled=true
    ```

### JPA Auditing으로 생성시간/수정시간 자동화하기
- 보통 엔티티에는 해당 데이터의 생성시간과 수정시간을 포함한다.
  - 차후 유지보수에 굉장히 중요한 정보이다.
  - 매번 DB insert / update 작업에 날짜 데이터를 등록.수정하는 코드가 반복되면 코드가 지저분해진다.
  - JPA Auditing을 사용해보자.
- Java8 이전 Date와 Calendar 클래스의 문제점
  - 불변 객체가 아니다.
    - 멀티스레드 환경에서 문제가 발생한다.
  - Calendar는 Month 값 설계가 잘못되었다.
    - 1을 더해줘야함.
  - LocalDate를 사용해보자.
- `BaseTimeEntity 클래스`
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
  - 모든 Entity의 상위클래스가 되어 Entity들의 createdDate, modifiedDate를 자동으로 관리하는 역할
  - @MappedSuperclass
    - JPA Entity 클래스들이 BaseTimeEntity를 상속할 경우 필드들(createdDate, ModifiedDate)도 칼럼으로 인식하도록 한다.
  - @EntityListeners(AuditingEntityListener.class)
    - BaseTimeEntity 클래스에 Auditing 기능을 포함한다.
  - @CreatedDate
    - Entity가 생성되어 저장될 때 시간ㅇ이 자동 저장된다.
  - @LastModifiedDate
    - 조회한 Entity의 값을 변경할 때 시간이 자동 정장된다.
- `Posts 클래스`
  ```java
  public class Posts extends BaseTimeEntity {
  }
  ```
  - BaseTimeEntity를 상속받도록 한다.
- `Application 클래스`
  ```java
  @EnableJpaAuditing
  ```
  - JPA Auditing 어노테이션들을 모두 활성화할 수 있도록 Application 클래스에 활성화 어노테이션 하나를 추가
- `PostRepositoryTest 클래스`
  ```java
  @Test
  public void BaseTimeEntity_등록() {
      //given
      LocalDateTime now = LocalDateTime.of(2019, 6, 4, 0, 0, 0);

      postsRepository.save(Posts.builder()
              .title("title")
              .content("content")
              .author("author")
              .build());

      //when
      List<Posts> postsList = postsRepository.findAll();

      //then
      Posts posts = postsList.get(0);

      System.out.println(">>>>>>>>>> createDdate=" + posts.getCreatedDate() + ", modifiedDate = " + posts.getModifiedDate());

      assertThat(posts.getCreatedDate()).isAfter(now);
      assertThat(posts.getModifiedDate()).isAfter(now);
  }
  ```
  - 앞으로 추가될 엔티티들은 더이상 등록일/수정일로 고민할 필요가 없다.
    - BaseTimeEntity만 상속받으면 해결된다.
