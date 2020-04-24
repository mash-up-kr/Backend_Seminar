#3. 스프링부트에서 JPA로 DB다루기

####1.자바표준 ORM - JPA
#####1.1. JPA란?

개발자가 프로그래밍 한 객체지향 코드를 JPA가 데이터베이스에 맞게 SQL 대신 생성하여 실행.    
JPA는 인터페이스로 구현체가 필요. 구현체로 Hibernate, Eclipse Link등이 있으나 Spring에서는 Spring Data JPA 모듈(구현체를 추상화함) 사용 권장.    
`JPA←Hibernate←Spring Data JPA`   
Spring Data의 하위 프로젝트들은 기본적인 CRUD 인터페이스 같다.   
->구현체 교체의 용이성(Hibernate 외의 다른 구현체로 교체의 용이성), 저장소 교체의 용이성의 장점 가짐

#####1.2. Spring Data JPA적용

- 의존성 등록(build.gradle)
```java
dependencies {
    compile('org.springframework.boot:spring-boot-starter-data-jpa')   //jpa
    compile('com.h2database:h2')  //h2
}
```
__*h2*__   :인메모리 관계형 데이터베이스. 메모리에서 실행되기 되어 애플리케이션을 재시작할 때마다 초기화된다는 
점을 이용하여 테스트용도로 많이 사용.   
`In-memory Database : 데이터 스토리지의 메인메모리(휘발성)에 설치, 운영되는 DB관리 시스템.빠르고 IO성능이 좋다.`   

- src/main/java/com/jojoldu/book/springboot/domain/posts/Posts.java
```java
package com.jojoldu.book.springboot.domain.posts;


import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter
@NoArgsConstructor
@Entity
public class Posts {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

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
__*@Entity*__
:Entity클래스(실제 DB의 테이블과 매칭될 클래스). JPA사용하여 DB데이터에 작업할 경우 실제 쿼리를 날리기보다 Entity클래스의 수정을 통해 작업.   
`Entity클래스에 절대 Setter메소드 생성 금지->필드값 변경 필요할 땐 메서드 추가`   
`생성자/@Builder 통해 최종값 채운후 DB삽입&값 변경 필요시 해당 이벤트의 public 메서드 호출`

`cf) 스프링에서 Bean을 주입 받는 방법 3가지 = @Autowired / setter / 생성자(권장)`

__*@id*__
:해당 테이블의 PK필드

__*@GeneratedValue*__
:PK생성규칙표시. GenerationType.IDENTITY옵션 통해 auto_increment(권장)

__*@NoArgsConstructor*__
:기본 생성자 자동 추가

__*@Builder*__
:해당 클래스의 빌더 패턴 클래스 생성. 생성자 상단에 선언시 생성자에 포함된 필도만 빌더에 포함

- src/main/java/com/jojoldu/book/springboot/domain/posts/PostsRepository.java
   - Posts클래스로 Database접근하게 해줄 JpaRepository(인터페이스)(DB layer접근자)
   
```java
package com.jojoldu.book.springboot.domain.posts;

import org.springframework.data.jpa.repository.JpaRepository;

public interface PostsRepository extends JpaRepository<Posts, Long> {
}
```
인터페이스 생성 후 JpaRepository<Entity 클래스, PK 타입>상속하면 기본적인 CRUD 메소드 자동 생성.

`Entity클래스와 기본 Entity Repository함께 위치`

#####1.3. Spring Data JPA 테스트코드
- src/test/java/com/jojoldu/book/springboot/domain/posts/PostsRepositoryTest.java
```java
package com.jojoldu.book.springboot.domain.posts;


import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

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
        //given
        String title = "테스트 게시글";
        String content = "테스트 본문";

        postsRepository.save(Posts.builder()
        .title(title)
        .content(content)
        .author("jojoldu@gmail.com")
        .build());

        //when
        List<Posts> postList = postsRepository.findAll();

        //then
        Posts posts = postList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title);
        assertThat(posts.getContent()).isEqualTo(content);
    }
}
```
__*@SpringBootTest*__
:h2 데이터베이스 자동 실행

__*@After*__
:단위 테스트 끝난 후 수행 메소드. 배포 전 테스트 수행 때 데이터 침범 막기 위해 사용.

__*@Test*__
:JUnit이 알아서 메소드 실행

__*Repository.save()*__
:해당 테이블에 insert(id값 없음)/update(id값 있음) 쿼리 실행

__*Repository.findAll()*__
:해당 테이블에 있는 모든 데이터 조회

***
###2. 등록/수정/조회 API 만들기

#####2.1. Spring 웹 계층
-Web Layer
 - 뷰 템플릿 영역(Controller, JSP 등)
 - 외부 요청, 응답에 대한 전반적인 영역(필터, 인터셉터, 컨트롤러 어드바이스 등)

-Service Layer
 - Controller와 Dao의 중간 영역에서 쓰임
 - @Service에 사용되는 서비스 영역, @Transaction이 쓰이는 영역

-Repository Layer

- Database와 같이 데이터 저장소에 접근하는 영역(=DAO영역)

-Dtos
 - 계층 간에 데이터 교환을 위한 객체(Dto)를 위한 공간

-Domain Model
 - 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해, 공유 할 수 있도록 단순화 시킨 것
 - 도메인 = @Entity사용된 영역, VO, 데이터 베이스의 테이블이 되는 영역
 - 비즈니스 로직 처리를 담당하는 영역


#####2.2. 등록, 수정, 조회 API 생성
`API만들기 위해 필요한 3개의 클래스 = Controller(API요청을 받는다)  
/ Service(트랜잭션, 도메인 기능 간의 순서 보장)
/ Dto(Request 데이터를 받는다)`

 - Controller     
   - src/main/java/com/jojoldu/book/springboot/web/PostsApiController.java

```java
package com.jojoldu.book.springboot.web;


import com.jojoldu.book.springboot.service.posts.PostsService;
import com.jojoldu.book.springboot.web.dto.PostsResponseDto;
import com.jojoldu.book.springboot.web.dto.PostsSaveRequestDto;
import com.jojoldu.book.springboot.web.dto.PostsUpdateRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RequiredArgsConstructor
@RestController
public class PostsApiController {

    private final PostsService postsService;

    @PostMapping("/api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto requestDto) {
        return postsService.save(requestDto);
    }

    @PostMapping("/api/v1/posts/{id}")
    public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto) {
        return postsService.update(id, requestDto);
    }

    @GetMapping("/api/v1/posts/{id}")
    public PostsResponseDto findById(@PathVariable Long id) {
        return postsService.findById(id);
    }
}
```
__*@RestController*__
:JSON반환 컨트롤러

__*@RequestBody*__
:HTTP 요청 몸체를 자바 객체로 변환하여 전달 

__*@PathVariable*__
:@RequestMapping의 값으로 {템플릿변수}를 지정하고 @PathVariable에서 동일한 템플릿 변수명을 갖는 parameter설정



- Service
  - src/main/java/com/jojoldu/book/springboot/service/posts/PostsService.java
```java
package com.jojoldu.book.springboot.service.posts;

import com.jojoldu.book.springboot.domain.posts.Posts;
import com.jojoldu.book.springboot.domain.posts.PostsRepository;
import com.jojoldu.book.springboot.web.dto.PostsResponseDto;
import com.jojoldu.book.springboot.web.dto.PostsSaveRequestDto;
import com.jojoldu.book.springboot.web.dto.PostsUpdateRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import javax.transaction.Transactional;

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
                .orElseThrow(() -> new IllegalArgumentException("해당 사용자가 없습니다. id=" + id));

        posts.update(requestDto.getTitle(), requestDto.getContent());

        return id;
    }

    public PostsResponseDto findById(Long id) {
        Posts entity = postsRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 사용자가 없습니다. id="+id));

        return new PostsResponseDto(entity);
    }
}
```
`Controller, Service에서 생성자로 Bean주입받는 방법을 사용 -> Lombok활용 `

JPA의 영속성 컨텍스트로 인해 update기능에서 DB에 쿼리를 날리는 부분이 불필요하다.(더티 체킹)

__*@Transactional*__
:메소드 내 작업처리를 하나로 묶기 위함

- Dto
  - src/main/java/com/jojoldu/book/springboot/web/dto/PostsSaveRequestDto.java


```java
package com.jojoldu.book.springboot.web.dto;


import com.jojoldu.book.springboot.domain.posts.Posts;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

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

`Entity와 별개로 Controller에서 쓸 Dto클래스를 생성해주어야 한다.     
Response와 Request용 Dto는 View를 위한 클래스로 자주 변경이 되기 때문에 Entity클래스를 Reqeust/Response클래스로 사용하면 안된다.   
View Layer와 DB Layer의 철저한 역할분리!
 `
 
 
  - src/main/java/com/jojoldu/book/springboot/web/dto/PostsResponseDto.java 

```java
package com.jojoldu.book.springboot.web.dto;


import com.jojoldu.book.springboot.domain.posts.Posts;
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
Entity 필드 중 일부만 사용하므로 생성자로 Entity 받아 필요한 값만 가져온다.

#####2.3. 등록/수정/조회 API 테스트 코드

- src/test/java/com/jojoldu/book/springboot/web/PostsApiControllerTest.java

```java 
package com.jojoldu.book.springboot.web;


import com.jojoldu.book.springboot.domain.posts.Posts;
import com.jojoldu.book.springboot.domain.posts.PostsRepository;
import com.jojoldu.book.springboot.web.dto.PostsSaveRequestDto;
import com.jojoldu.book.springboot.web.dto.PostsUpdateRequestDto;
import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.test.context.junit4.SpringRunner;


import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

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
        PostsSaveRequestDto requestDto = PostsSaveRequestDto.builder()
                .title(title)
                .content(content)
                .author("author")
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

    }

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
        ResponseEntity<Long> responseEntity = restTemplate.exchange(url, HttpMethod.POST, requestEntity, Long.class);

        //then
        assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);

        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent()).isEqualTo(expectedContent);
    }
}
```

__*@SpringBootTest*__
:통합테스트에 쓰임.

__*TestRestTemplate*__
:컨테이너를 직접 실행한다

#####2.4. JPA Auditing으로 생성시간/수정시간 자동화

생성시간과 수정시간은 유지보수에 있어 중요한 정보로 Entity에 포함되는 경우가 대부분이다.

-  src/main/java/com/jojoldu/book/springboot/domain/BaseTimeEntity.java
```java
package com.jojoldu.book.springboot.domain;

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
__*@MappedSuperclass*__
:객체의 입장에서 공통 매핑 정보가 필요할 때 속성만 상속받아서 사용할 수 있게 해준다.
상속할 경우 필드들도 칼럼으로 인식되게 해준다.     
-@MappedSuperclass가 선언되어 있는 클래스는 entity가 아니므로 테이블에 매핑이 되지 않는다.
공통으로 사용하는 매핑 정보를 모으는 데 목적이 있으며 조회, 검색 불가하다.
추상클래스로 선언 권장한다(직접 생성하지 않으므로).

`위의 추상클래스만 상속받으면 Entity의 등록시간/수정시간 자동 해결`
__*@EntityListeners(AuditingEntityListener.class)*__
:해당 클래스에 Auditing기능 포함

__*@CreatedDate*__
:Entity 생성 시간 자동저장

__*@LastModifiedDate*__
:Entity조회시간 자동 저장

- JPA Auditing 활성화 위해 Application클래스에 추가
```java
@EnableJpaAuditing    //JPA Auditing 활성화
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
```



***
###3. 더 알아보기
#####3.1. ORM vs SQL Mapper
-ORM   
:(Object Relational Mapping) 객체-RDBMS테이블 객체지향적 사용위함. 
객체와 RBDMS테이블이 매핑되어 관계형 데이터를 객체처럼 사용. SQL문 없이 매핑만으로 DB내 데이터를 객체로 전달. DB설계가 잘 되어있음을 전제함.   
예)JPA

-SQL Mapper   
:객체-SQL문을 매핑하여 데이터를 객체화. SQL문의 질의와 객체 매핑   
예)MyBatis

#####3.2. Builder Pattern
새로운 객체 만들어 반환. 생성자에 들어갈 매개변수를 차례차례 받아들이고 모두 받은 후 통합하여 한번에 생성

#####3.3. DAO(Data Access Object)
- 데이터 베이스를 활용해 데이터를 조회하거나 조작하는(data 접근) 기능 전다의 객체
- 웹 서버가 DB와 연결하기 위해 생성한 하느이 커넥션을 가져오고,
그 커넥션을 가져온 객체가 모든 DB와의 연결을 한 것
(커넥션 풀, 커넥션에 대한 오버헤드 효율적 관리가 가능해짐)




#####3.4. JPA 영속성 컨텍스트

- 영속성 컨텍스트(Persistence Context)
: 'Entity를 영구저장하는 환경'이라는 논리적인 개념.

- 엔티티 생명주기(Entity LifeCycle)
   - 비영속(new/transient) : 객체를 생성만 한 상태
   - 영속(managed) : 영속성 컨텍스트에 저장 된 상태. DB에 저장되지 않음(쿼리 날라가지 않음). 트잭션의 커밋 시점에 영속성 컨텍스트에 있는 정보들이 DB에 저장됨.
   - 준영속(detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태
   - 삭제(removed) : 삭제된 상태. DB에서도 삭제.


#####3.5. TestRestTemplate

- RestTemplate = Rest API 호출 이후 응답 받을 때까지 기다림(동기 방식)
- Rest = HTTP기반으로 필요한 자원에 접근하는 방식을 정해놓은 아키텍쳐
- Rest API = Rest 통해 구현한 서비스 API

