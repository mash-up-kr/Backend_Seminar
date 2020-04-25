# Chapter 3: 스프링 부트에서 JPA로 데이터베이스 다뤄 보자

Created: Mar 29, 2020 9:07 PM
Tags: JPA Auditing

- 수정, 조회 기능 만들기

```java
// PostsApiController
package example.org.practice.web;

import example.org.practice.service.posts.PostsService;
import example.org.practice.web.dto.PostsResponseDto;
import example.org.practice.web.dto.PostsSaveRequestDto;
import example.org.practice.web.dto.PostsUpdateRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RequiredArgsConstructor
@RestController
public class PostsApiController {

    private final PostsService postsService;

    @PostMapping("/api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto
                                           requestDto) {

        return postsService.save(requestDto);
    }

    @PutMapping("/api/v1/posts/{id}")
    public Long update(@PathVariable Long id, @RequestBody
            PostsUpdateRequestDto requestDto) {
        return postsService.update(id, requestDto);
    }

    @GetMapping("/api/v1/posts/{id}")
    public PostsResponseDto findById (@PathVariable Long id) {
        return postsService.findById(id);
    }

}
```

```java
// PostsResponseDto

package example.org.practice.web.dto;

import example.org.practice.domain.posts.Posts;
import lombok.Getter;

@Getter
public class PostsResponseDto {

    private Long id;
    private String title;
    private String content;
    private String author;

    public PostsResponseDto(Posts entity) {
        this.id = entity.getId();
        this.title = entity.getTitle();
        this.content = entity.getContent();
        this.author = entity.getAuthor();
    }
}
```

- **Entity 의 필드 중 일부만 사용하므로** 생성자로 Entity를 받아 필드에 값을 넣는다.

```java
// PostsUpdateRequestDto

package example.org.practice.web.dto;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

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

```java
// Posts
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

    public void update(String title, String content) {
        this.title = title;
        this.content = content;
    }
}
```

```java
// PostsService

@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    @Transactional
    public Long save(PostsSaveRequestDto requestDto) {
        return postsRepository.save(requestDto.toEntity()).getId();
    }

    @Transactional
    public Long update(Long id, PostsUpdateRequestDto requestDto) {
        Posts posts = postsRepository.findById(id)
                .orElseThrow(() -> new
            IllegalArgumentException("해당 게시글이 없습니다. id=" + id));

        posts.update(requestDto.getTitle(), requestDto.getContent());

        return id;
    }

    public PostsResponseDto findById (Long id) {
        Posts entity = postsRepository.findById(id)
                .orElseThrow(() -> new
                        IllegalArgumentException("해당 게시글이 없습니다. id=" + id));

        return new PostsResponseDto(entity);
    }
}
```

- update 기능에서 데이터베이스에 **쿼리를 날리지 않는 이유**는 무엇일까?
    - 이게 가능한 이유는 JPA의 **영속성 컨텍스트** 때문이다.
    - 영속성 컨텍스트란, **엔티티를 영구 저장하는 환경**을 말한다. 일종의 논리적 개념으로 JPA의 핵심 내용은 **엔티티가 영속성 컨텍스트 포함되어 있냐 아니냐**로 갈린다.
    - JPA의 엔티티 매니저가 활성화된 상태로(Spring Data Jpa를 쓴다면 기본 옵션) **트랜잭션 안에서 데이터베이스에서 데이터를 가져오면** 이 데이터는 영속성 컨텍스트가 유지된 상태이다.
    - 이 상태에서 해당 데이터의 값을 변경하면 **트랜잭션이 끝나는 시점에 해당 테이블에 변경분을 반영**한다. 즉, Entity 객체의 값만 변경하면 별도로 **Update 쿼리를 날릴 필요가 없다**.
    - 이 개념을 **더티 체킹(dirty checking)**이라고 한다.

```java
// PostsApiControllerTest

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {

    ...

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

        PostsUpdateRequestDto requestDto =
                              PostsUpdateRequestDto.builder()
                .title(expectedTitle)
                .content(expectedContent)
                .build();

        String url = "http://localhost:" + port + "api/v1/posts/" + updateId;

        HttpEntity<PostsUpdateRequestDto> requestEntity = new HttpEntity<>(requestDto);

        //when
        ResponseEntity<Long> responseEntity = restTemplate.
														 exchange(url, HttpMethod.PUT,
														 requestEntity, Long.class);

        //then
				assertThat(responseEntity.getStatusCode()).
																		isEqualTo(HttpStatus.OK);
				assertThat(responseEntity.getBody()).isGreaterThan(0L);

        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle())
                                .isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent())
                                .isEqualTo(expectedContent);
    }

}
```

- 좀 더 객체 지향적으로 코딩 가능!
- 조회 기능은 실제로 톰캣을 실행해서 확인
    - `application.properties`에 `spring.h2.console.enabled=true` 옵션을 추가한다.
    - [http://localhost:8000/h2-console](http://localhost:8000/h2-console) 접속 후  JDBC URL을 jdbc:h2:mem:testdb로 설정한 후 Connect 버튼을 클릭한다.

## 3.5 JPA Auditing으로 생성 시간/수정 시간 자동화하기

- 보통 엔티티에는 데이터의 생성 시간과 수정 시간을 포함한다.
    - 언제 만들어졌는지, 언제 수정되었는지 등은 차후 유지 보수에 있어 중요한 정보이기 때문이다.
    - 매번 DB에 삽입하기 전, 갱신하기 전에 날짜 데이터를 등록, 수정하는 코드가 여기저기에 들어가게 된다.
    - 이렇게 단순하고 반복적인 코드가 모든 테이블과 서비스 메소드에 포함된다고 하면 코드가 지저분해진다.
    - → 이 문제를 해결하고자 JPA Auditing를 사용!

### LocalDate 사용

- Java 8부터 `LocalDate`와 `LocalDateTime`이라는 Java의 기본 날짜 타입이 등장
- domain 패키지에 BaseTimeEntity 클래스 생성

```java
import lombok.Getter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.EntityListeners;
import javax.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class BaseTimeEntity {

    @CreatedDate
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime modifiedDate;
}
```

- 모든 Entity의 상위 클래스가 되어 **Entity들의 createdDate, modifiedDate를 자동으로 관리하는 역할**을 수행

- `@MappedSuperclass`
    - JPA Entity 클래스들이 BaseTimeEntity를 상속할 경우 필드들(createdDate, modifiedDate)도 칼럼으로 인식하도록 한다.
- `@EntityListeners(AuditingEntityListener.class)`
    - BaseTimeEntity 클래스에 Auditing 기능을 포함시킨다.
    - 생성일, 수정일 컬럼은 대단히 중요한 데이터이므로, JPA에서는 `Audit`이라는 기능을 제공한다. Audit은 감시하다라는 뜻으로 Spring Data JPA에서 시간에 대해 자동으로 값을 넣어 주는 기능이다. 도메인을 영속성 컨텍스트에 저장하거나 조회를 수행한 후에 update를 하는 경우 매번 시간 데이터를 입력해 주어야 하는데, audit을 이용하면 자동으로 시간을 매핑하여 데이터베이스의 테이블에 넣어 주게 된다.
- `@CreatedDate`
    - Entity가 생성되어 저장될 때 시간이 자동 저장됨
- `@LastModifiedDate`
    - 조회한 Entity의 값을 변경할 때 시간이 자동 저장됨.

- Posts 클래스가 BaseTimeEntity를 상속받도록 변경해 준다.
- JPA Auditing 어노테이션을 모두 활성화할 수 있도록 Application 클래스에 활성화 어노테이션을 추가한다.

```java
@EnableJpaAuditing
@SpringBootApplication
public class Application {
    public static void main(String[] args){
        SpringApplication.run(Application.class, args);
    }
}
```

### JPA Auditing 테스트 코드 작성하기

- PostsRepositoryTest 클래스에 테스트 메소드 추가

```java
		@Test
    public void BaseTimeEntity_등록() {
        //given
        LocalDateTime now = LocalDateTime.of(2020, 3, 6, 17, 52, 0);
        postsRepository.save(Posts.builder()
                .title("title")
                .content("content")
                .author("author")
                .build());

        //when
        List<Posts> postList = postsRepository.findAll();

        //then
        Posts posts = postList.get(0);

        System.out.println(">>>>>>>> createDate=" + posts.getCreatedDate() +
                            ", modifiedDate=" + posts.getModifiedDate());

        assertThat(posts.getCreatedDate()).isAfter(now);
        assertThat(posts.getModifiedDate()).isAfter(now);
    }
```