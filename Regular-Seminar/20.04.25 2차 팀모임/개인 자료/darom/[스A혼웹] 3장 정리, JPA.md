## [스A혼웹] 3장 정리, JPA

>  스프링 부트와 AWS로 혼자 구현하는 웹 서비스 3장 정리



### 03 스프링 부트에서 JPA로 데이터베이스 다뤄보자

어떻게 하면 관계형 데이터베이스를 이용하는 프로젝트에서 객체지향 프로그래밍을 할 수 있을까?

- JPA, 자바 표준 ORM(Object Relational Mapping)이 그 해결책이다.

MyBatis, iBatis는 ORM이 아닌 SQL Mapper입니다. ORM은 객체를 매핑하는 것이고, SQL Mapper는 쿼리를 매핑합니다.

---

#### 3.1 JPA 소개

> - Spring Data JPA
> - 실무에서 JPA
> - 요구사항 분석

JPA는 아래 문제를 해결하기 위해 등장

- 단순 반복 작업
- 패러다임 불일치
- 데이터베이스 모델링에만 집중



**Spring Data JPA**

JPA는 인터페이스로서 자바 표준명세서입니다. 인터페이스인 JPA를 사용하기 위해서는 구현체(Hibernate, Eclipse Link 등)가 필요합니다. 하지만 Spring에서 JPA를 사용할 때는 구현체들을 직접 다루지 않고 구현체들을 좀 더 쉽게 사용하고자 추상화시킨 Spring Data JPA라는 모듈을 이용하여 JPA 기술을 다룹니다.

- JPA ← Hibernate ← Spring Data JPA



이렇게 한 단계 더 감사놓은 Spring Data JPA가 등장한 이유는 두 가지가 있습니다.

- 구현체 교체의 용이성
  Hiberante 외에 다른 구현체로 쉽게 교체하기 위함
- 저장소 교체의 용이성
  관계형 데이터베이스 외에 다른 저장소로 쉽게 교체하기 위함 → 의존성만 교체하면 됨



**실무에서 JPA**

실무에서 JPA를 사용하지 못하는 가장 큰 이유는 **높은 러닝 커브**입니다. JPA를 잘 쓰러면 객체지향 프로그래밍과 관계형 데이터베이스를 둘 다 이해해야 합니다.

하지만 그만큼 JPA를 사용해서 얻는 보상은 큽니다.

- CRUD쿼리를 직접 작성할 필요 없습니다.
- 부모-자식 관계 표현, 1:N 관계 표현, 상태와 행위를 한 곳에서 관리하는 등 객체지향 프로그래밍을 쉽게 할 수 있습니다.



**요구사항 분석**

3~6장 까지 하나의 게시판을 만들어 보고 7장부터 10장 까지는 이 서비스를 AWS에 무중단 배포 하는것까지 진행합니다.

이 게시판의 요구사항은 다음과 같습니다.

- 게시판 기능
  - 게시글 조회
  - 게시글 등록
  - 게시글 수정
  - 게시글 삭제
- 회원 기능
- 구글/네이버 로그인
- 로그인한 사용자 글 작성 권한
- 본인 작성 글에 대한 권한 관리



#### 3.2 프로젝트에서 Spring Data Jpa 적용하기

1. build.gradle에 jpa 와 h2에 대한 의존성을 등록합니다.

   ```groovy
   compile('org.springframework.boot:spring-boot-starter-data-jpa')
   compile('com.h2database:h2')
   ```

   - spring-boot-stater-data-jpa
     - 스프링 부트용 Spring Data Jpa 추상화 라이브러리입니다.
     - 스프링 부트 버전에 맞춰 자동으로 JPA 관련 라이브러리들의 버전을 관리해 줍니다.
   - h2
     - 인메모리 관계형 데이터베이스입니다.
     - 별도의 설치가 필요 없이 프로젝트 의존성만으로 관리할 수 있습니다.
     - 메모리에서 실행되기 때문에 애플리케이션을 재시작할 때마다 초기화된다는 점을 이용하여 테스트 용도로 많이 사용됩니다.
     - 이 책에서는 JPA의 테스트, 로컬 환경에서의 구동에서 사용할 예정입니다.

2. domain 패키지 생성
   도메인을 담을 패키지. 도메인이란 게시글, 댓글, 회원, 정산, 결제 등 소프트웨어에 대한 요구사항 혹은 문제영역이라고 생각하면 됩니다.

3. domain 패키지에 posts 패키지와 Posts 클래스를 생성합니다.
   **Posts**

   ```java
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

   - @Entity
     - 테이블과 링크될 클래스임을 나타냅니다.
     - 기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍(_)으로 테이블 이름을 매칭합니다.
   - @Id
     - 해당 테이블의 PK필드를 나타냅니다.
   - @GeneratedValue
     - PK의 생성 규칙을 나타냅니다.
     - 스프링 부트 2.0에서는 GenerationType.IDENTITY 옵션을 추가해야만 auto_increment 가 됩니다.
     - 스프링 부트 2.0버전과 1.5 버전의 차이는 https://jojoldu.tistory.com/295 에서 더 확인할 수 있습니다.
   - @Column
     - 테이블의 칼럼을 나타내며 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 칼럼이 됩니다.
     - 사용하는 이유는, 기본값 외에 추가로 변경이 필요한 옵션이 있으면 사용합니다.
     - 문자열의 경우 VARCHAR(255)가 기본값인데, 사이즈를 500으로 늘리고 싶거나(ex: title) 타입을 TEXT로 변경하고 싶거나(ex: content) 등의 경우에 사용됩니다.
   - @NoArgsConstructor
     - 기본 생성자 자동 추가
     - public Posts(){} 와 같은 효과
   - @Getter
     - 클래스 내 모든 필드의 Getter 메소드 자동생성
   - @Builder
     - 해당 클래스의 빌더 패턴 클래스를 생성
     - 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함

   @Entity는 JPA  어노테이션이며, @Getter와 @NoArgsConstructor는 롬복의 어노테이션입니다.
   롬복은 코드를 단순화시켜 주지만 필수 어노테이션은 아닙니다. 그러다보니 주요 어노테이션인 @Entity를 클래스에 가깝게 두고, 롬복 어노테이션을 그 위로 두었습니다. 이렇게 하면 코틀린 등의 새 언어 전환으로 롬복이 더이상 필요 없을 경우 쉽게 삭제할 수 있습니다.

   Posts 클래스는 실제 DB의 테이블과 매칭될 클래스이며 보통 Entity 클래스라고도 합니다.

   JPA를 사용하면, DB 데이터를 작업할 경우 실제 쿼리를 날리기보다는, 이 Entity 클래스의 수정을 통해 작업합니다.

   롬복의 어노테이션들은 코드 변경량을 최소화시켜주기 때문에 적극적으로 사용합니다.

   

   이 Posts 클래스에는 한 가지 특이점이 있습니다. 

   * Setter 메소드가 없다는 점입니다.

     자바빈 규약을 생각하면서 getter/setter를 무작정 생성하는 경우가 있습니다.

     이렇게 되면 해당 클래스의 인스턴스 값들이 언제 어디서 변해야 하는지 코드상으로 명확하게 구분할 수가 없어, **차후 기능 변경 시 정말 복잡**해집니다.

     그래서 **Entity 클래스에서는 절대 Setter 메소드를 만들지 않습니다**.

     

   그렇다면 Setter가 없는 상황에서 어떻게 값을 채워 DB에 삽입 할까요?
   기본적인 구조는 생성자를 통해 최종값을 채운 후 DB에 삽입하는 것이며, 값 변경이 필요한 경우 해당 이벤트에 맞는 public 메소드를 호출하여 변경하는 것을 전제로 합니다.

   이 책에서는 생성자 대신에 @Builder를 통해 제공되는 빌더 클래스를 사용합니다. 생성자나 빌더나 생성 시점에 값을 채워주는 역할을 똑같습니다. 다만, 생성자의 경우 지금 채워야 할 필드가 무엇인지 명확하게 지정할 수가 없습니다. 하지만 빌더를 사용하면 어느 필드에 어떤 값을 채워야할지 명확하게 인지할 수 있습니다.

4. Posts 클래스로 Database를 접근하게 해줄 JpaRepository를 생성합니다.
   **PostsRepository**

   ```java
   import org.springframework.data.jpa.repository.JpaRepository;
   
   public interface PostsRepository extends JpaRepository<Posts, Long> {
   }
   ```

   보통 ibatis나 MyBatis 등에서 Dao라고 불리는 DB Layer 접근자입니다. JPA에서는 Repository라고 부르며 인터페이스로 생성합니다. 

   - 단순히 인터페이스를 생성 후 
   - JpaRepository<Entity 클래스, PK 타입>를 상속하면 기본적인 CRUD 메소드가 자동으로 생성됩니다.
   - @Repository를 추가할 필요도 없습니다.
   - 주의할 점은 Entity 클래스와 기본 Entity Repository는 함께 위치해야 하는 점입니다.
     둘은 아주 밀접한 관계이고, Entity 클래스는 기본 Repository 없이는 제대로 역할을 할 수가 없습니다.
     나중에 프로젝트 규모가 커져 도메인별로 프로젝트를 분리해야 한다면 이때 Entity 클래스와 Repository는 함께 움직여야 하므로 도메인 패키지에서 함께 관리합니다.



다 작성되었다면 간단하게 테스트 코드로 검증해보겠습니다.





#### 3.3 Spring Data JPA 테스트 코드 작성하기

1. test 디렉토리에 domain.posts 패키지를 생성하고, 테스트 클래스는 Posts RepositoryTest란 이름으로 생성합니다.
   PostsRespositoryTest

   ```java
   import com.mashup.springboot.domain.posts.Posts;
   import com.mashup.springboot.domain.posts.PostsRepository;
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
       public void cleanup(){
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
                                   .author("darom")
                                   .build());
   
           //when
           List<Posts> postsList = postsRepository.findAll();
   
           //then
           Posts posts = postsList.get(0);
           assertThat(posts.getTitle()).isEqualTo(title);
           assertThat(posts.getContent()).isEqualTo(content);
       }
   }
   ```

   - @After
     - Junit에서 단위 테스트가 끝날 때마다 수행되는 메소드를 지정
     - 보통은 배포 전 전체 테스트를 수행할 때 테스트간 데이터 침범을 막기 위해 사용합니다.
     - 여러 테스트가 동시에 수행되면 테스트용 데이터베이스인 H2에 데이터가 그대로 남아 있어 다음 테스트 실행 시 테스트가 실패할 수 있습니다.
   - postsRepository.save
     - 테이블 posts에 insert/update 쿼리를 실행합니다.
     - id값이 잆다면 update가, 없다면 insert 쿼리가 실행됩니다.
   - postsRepository.findAll
     - 테이블 posts에 있는 모든 데이터를 조회해오는 메소드입니다.'

2. 테스트를 실행해보면 성공!

3. 실제로 실행된 쿼리는 어떤 형태일까?
   src/main/resources 디렉토리 아래에 application.properties 파일을 생성합니다.


   application.properties

   ```
   spring.jpa.show-sql=true
   ```

   이렇게 한 줄만 추가하고 다시 테스트를 수행하면 콘솔에서 쿼리 로그를 확인할 수 있습니다.

   그런데, 쿼리 로그에서 아래 처럼 generated by default as identity 가 뜨는데 이는 H2의 쿼리 문법이 적용되었기 때문입니다.

   ```
   create table posts (id bigint generated by default as identity, ...
   ```

   

   MySQL의 쿼리를 수행해도 정상적으로 작동하기 때문에 이후 디버깅을 위해서 출력되는 쿼리 로그를 MySQL 버전으로 변경해 보겠습니다.

   ```
   spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
   ```

   위의 내용을 추가하고나면 로그가 아래처럼 바뀌어 출력됩니다.

   ```
   create table posts (id bigint not null auto_increment, ...
   ```

   

이제 본격적으로 API를 만들어 보겠습니다.



#### 3.4 등록/수정/조회 API 만들기

API를 만들기 위해 총 3개의 클래스가 필요합니다.

- Request 데이터를 받을 Dto
- API 요청을 받을 Controller
- 트랜잭션, 도메인 기능 간의 순서를 보장하는 Service

Service는 트랜잭션, 도메인 간 순서 보장의 역할만 합니다. 그럼 비즈니스 처리는?? 



**Spring 웹 계층**

![Spring 웹 계층](https://user-images.githubusercontent.com/44438366/80106239-947c4100-85b4-11ea-883e-fd45f008c665.png)

**1. Web Layer**

- 흔히 사용하는 컨트롤러(Controller)와 JSP/Freemarker 등의 뷰 템플릿 영역
- 이외에도 필터(@Filter), 인터셉터, 컨트롤러 어드바이스(@ControllerAdvice) 등 **외부 요청과 응답**에 대한 전반적인 영역을 의미한다.

**2. Service Layer**

- @Service에 사용되는 서비스 영역이다.
- 일반적으로 Controller와 Dao의 중간 영역에서 사용된다.
- @Transactional이 사용되어야 하는 영역이기도 하다.

**3. Repository Layer**

- **Database**와 같이 데이터 저장소에 접근하는 영역이다.
- Dao(Data Access Object) 영역이라고 생각하면 된다.

**4. Dtos**

- Dto(Data Transfer Object)는 **계층 간에 데이터 교환을 위한 객체**를 의미하며, Dtos는 이들의 영역을 의미한다.
- 예를 들어 뷰 템플릿 엔진에서 사용될 객체나 Repository Layer에서 결과로 넘겨준 객체 등이 이들을 이야기한다.

**5. Domain Model**

- 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화시킨 것을 도메인 모델이라고 한다.
- 비즈니스 로직을 처리하는 영역이다.
- 이를테면 택시 앱이라고 하면 배차, 탑승, 요금 등이 모두 도메인이 될 수 있다.
- @Entity가 사용된 영역 역시 도메인 모델이라고 이해하면 된다.
- 다만, 무조건 데이터베이스의 테이블과 관계가 있어야 하는 것은 아니다. 
- VO처럼 값 객체들도 이 영역에 해당하기 때문이다.



이 다섯가지 레이어에서 비즈니스 처리를 담당해야 할 곳은. Domain입니다. 

기존에 서비스로 처리하던 방식을 트랜잭션 스크립트라고 합니다. 



모든 로직이 서비스 클래스 내부에서 처리되면 서비스 계층이 무의미하며, 객체란 단순히 데이터 덩어리 역할만 하게 됩니다. 반면 도메인 모델에서 처리할 경우 서비스 메소드는 트랜잭션과 도메인 간의 순서만 보장해줍니다.

이 책에서는 계속 도메인 모델을 다루고 코드를 작성합니다.



이제 등록, 수정, 삭제 기능을 만들어 보겠습니다.

web/**PostsApiController**

```java
import com.mashup.springboot.service.posts.PostsService;
import com.mashup.springboot.web.dto.PostsSaveRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RequiredArgsConstructor
@RestController
public class PostsApiController {
    
    private final PostsService postsService;
    
    @PostMapping("/api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto requestDto){
        return postsService.save(requestDto);
    }
}
```



service/**PostsService**

```java
import com.mashup.springboot.domain.posts.PostsRepository;
import com.mashup.springboot.web.dto.PostsSaveRequestDto;
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
}
```



스프링에서 Bean을 주입받는 방식

- @Autowired
- setter
- 생성자

이 중 가장 권장하는 방식이 생성자로 주입받는 방식입니다. 즉, 생성자로 Bean 객체를 받도록 하면 @Autowired와 동일한 효과를 볼 수 있다는 것입니다. 그러면 앞에서 생성자는 어디있을까요?

바로 @RequiredArgsConstructor에서 해결해 줍니다. final이 선언된 모든 필드를 인자값으로 하는 생성자를 롬복의 @RequiredArgsConstructor가 대신 생성해 준 것입니다.

생성자를 직접 안 쓰고 롬복 어노테이션을 사용한 이유는 간단합니다. 해당 클래스의 의존성 관계가 변경될 때마다 생성자 코드를 계속해서 수정하는 번거로움을 피하기 위함입니다.

> 롬복 어노테이션이 있으면 해당 컨트롤러에 새로운 서비스를 추가하거나 기존 컴포넌트를 제거하는 등의 상황이 발생해도 생성자 코드는 전혀 손대지 않아도 됩니다. 편리하죠?



이제 Controller와 Service에서 사용할 Dto 클래스를 생성하겠습니다.

web/dto/**PostsSaveRequestDto**

```java
import com.mashup.springboot.domain.posts.Posts;
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

    public Posts toEntity(){
        return Posts.builder()
                .title(title)
                .content(content)
                .author(author)
                .build();
    }
}
```



여기서 Entity 클래스와 거의 유사한 형태임에도 Dto 클래스를 추가로 생성했습니다. 하지만, 절대로 **Entity 클래스를 Request/Response 클래스로 사용해서는 안 됩니다**.

Entity 클래스는 데이터베이스와 맞닿은 핵심 클래스입니다. 이 클래스를 기준으로 테이블이 생성되고, 스키마가 변경됩니다. 화면 변경은 아주 사소한 기능 변경인데, 이를 위해 테이블과 연결된 Entity 클래스를 변경하는 것은 너무 큰 변경입니다.

Request와 Response용 Dto는 View를 위한 클래스라 정말 자주 변경이 필요합니다. 

**View Layer와 DB Layer의 역할 분리를 철저하게** 하는 게 좋습니다. 





**이제 테스트코드로 검증.**

PostsApiControllerTest

```java
import com.mashup.springboot.domain.posts.Posts;
import com.mashup.springboot.domain.posts.PostsRepository;
import com.mashup.springboot.web.dto.PostsSaveRequestDto;
import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
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
        ResponseEntity<Long> responseEntity = restTemplate.
                postForEntity(url, requestDto, Long.class);

        //then
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);
    }
}
```

Api Controller를 테스트하는데 @WebMvcTest를 사용하지 않았습니다. @WebMvcTest 의 경우 JPA기능이 작동하지 않기 때문인데, Controller와 ControllerAdvice등 외부 연동과 관련된 부분만 활성화되니 지금 같이 JPA 기능까지 한번에 테스트 할 때는 @SpringBootTest와 TestRestTemplate을 사용하면 됩니다.



실행시키면 테스트 성공!



수정/조회도 이어서 만들기...

PostsApiController

```java
...
@PutMapping("/api/v1/posts/{id}")
    public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto){
        return postsService.update(id, requestDto);
    }

@GetMapping("/api/v1/posts/{id}")
    public PostsResponseDto findById (@PathVariable Long id){
        return postsService.findById(id);
    }
```



PostsResponseDto

```java
import com.mashup.springboot.domain.posts.Posts;
import lombok.Getter;

@Getter
public class PostsResponseDto {

    private Long id;
    private String title;
    private String content;
    private String author;

    public PostsResponseDto(Posts entity){
        this.id = entity.getId();
        this.title= entity.getTitle();
        this.content = entity.getContent();
        this.author = entity.getAuthor();
    }
}
```

PostsResponseDto 는 Entity의 필드 중 일부만 사용하므로 생성자로 Entity를 받아 필드에 값을 넣습니다. 굳이 모든 필드를 가진 생성자가 필요하진 않으므로 Dto는 Entity를 받아 처리합니다.



PostsUpdateRequestDto

```java
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

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



Posts

```java
public void update(String title, String content) {
    this.title = title;
    this.content = content;
}
```



PostsService

```java
@Transactional
    public Long update(Long id, PostsUpdateRequestDto requestDto) {
        Posts posts = postsRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 사용자가 없습니다. id=" + id));

        posts.update(requestDto.getTitle(), requestDto.getContent());

        return id;
    }

    public PostsResponseDto findById(Long id) {
        Posts entity = postsRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 사용자가 없습니다. id=" + id));

        return new PostsResponseDto(entity);
    }
```

여기서 신기한 점은 update기능에서 데이터베이스에 쿼리를 날리는 부분이 없습니다. 이게 가능한 이유는 **JPA의 영속성 컨텍스트** 때문입니다.

영속성 컨텍스트란, 엔티티를 영구 저장하는 환경입니다. 일종의 논리적 개념이라고 보시면 되며, JPA 핵심 내용은 엔티티가 영속성 컨텍스트에 포함되어 있냐 아니냐로 갈립니다.

JPA의 엔티티 매니저가 활성화된 상태로 **트랜잭션 안에서 데이터베이스에서 데이터를 가져오면** 이 데이터는 영속성 컨텍스트가 유지된 상태입니다.

이 상태에서 해당 데이터의 값을 변경하면 **트랜잭션이 끝나는 시점에 해당 테이블에 변경분을 반영**합니다. 즉, Entity 객체의 값만 변경하면 별도로 **Update 쿼리를 날릴 필요가 없다**는 것이죠. 이 개념을 **더티 체킹**이라고 합니다.



Update 쿼리를 정상적으로 수행하는지 테스트코드로 확인



PostsApiControllerTest

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.mashup.springboot.domain.posts.Posts;
import com.mashup.springboot.domain.posts.PostsRepository;
import com.mashup.springboot.web.dto.PostsSaveRequestDto;
import com.mashup.springboot.web.dto.PostsUpdateRequestDto;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.put;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PostsRepository postsRepository;

    @Autowired
    private WebApplicationContext context;

    private MockMvc mvc;

    @Before
    public void setup() {
        mvc = MockMvcBuilders
                .webAppContextSetup(context)
//                .apply(springSecurity())
                .build();
    }

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
        ResponseEntity<Long> responseEntity = restTemplate.
                postForEntity(url, requestDto, Long.class);

        //then
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

        //when
//        ResponseEntity<Long> responseEntity = restTemplate.exchange(url, HttpMethod.PUT, requestDto, Long.class);
        mvc.perform(put(url)
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(new ObjectMapper().writeValueAsString(requestDto)))
                .andExpect(status().isOk());


        //then
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent()).isEqualTo(expectedContent);
    }
}
```

테스트 결과를 보면 update 쿼리가 수행되는 것을 확인할 수 있습니다.





조회 기능은 실제로 톰캣을 실행해서 확인해 보겠습니다. 

로컬환경에서는 데이터베이스로 H2를 사용합니다.  메모리에서 실행하기 때문에 직접 접근하려면 웹 콘솔을 사용해야만 합니다.

application.properties에 다음과 같이 옵션을 추가합니다.

```
spring.h2.console.enabled=true
```



Application 클래스의 main 메소드를 실행한 후 웹 브라우저 `http://localhost:8080/h2-console` 로 접속하면 웹 콘솔 화면이 등장합니다. 이때 JDBC URL이 아래와 똑같지 않다면 똑같이 작성해주셔야 합니다. 작성한 후 connet 버튼을 클릭하면 H2관리 페이지로 이동합니다.

```
jdbc:h2:mem:testdb
```



POSTS 테이블이 정상적으로 노출되어야만 합니다.

```mysql
insert into posts (author, content, title) values ('author', 'content', 'title');
```

작성한 후 run을 누른 다음, 브라우저로 http://localhost:8080/api/v1/posts/1 API 조회 기능을 테스트해봅니다.

> Chrome에 JSON Viewer 라는 플러그인을 설치하면 정렬된 JSON 형태를 볼 수 있습니다.



이렇게 기본적인 등록/수정/조회 기능을 모두 만들고 테스트해 보았습니다. 특히 등록/수정은 테스트 코드로 보호해 주고 있으니 이후 변경 사항이 있어도 안전하게 변경할 수 있습니다.



 

#### 3.5 JPA Auditing 으로 생성시간/수정시간 자동화하기

> - LocalDate 사용
> - JPA Auditing 테스트 코드 작성하기

보통 엔티티에는 해당 데이터의 생성시간과 수정시간을 포함합니다. 언제 만들어졌는지, 언제 수정되었는지 등은 차후 유지보수에 있어 굉장히 중요한 정보이기 때문입니다. 그렇다 보니 매번 DB에 삽입하기 전, 갱신하기 전에 날짜 데이터를 등록/수정하는 코드가 여기저기 들어가게 됩니다.

이런 단순하고 반복접인 코드가 모든 테이블과 서비스 메소드에 포함되어야 한다고 생각하면 어마어마하게 귀찮고 코드가 지저분해집니다. 그래서 이 문제를 해결하고자 JPA Auditing를 사용하겠습니다.



**LocalDate 사용**

Java8부터 LocalDate와 LocalDateTime이 등장했습니다. 그간 Java의 기본 날짜 타입인 Date의 문제점을 제대로 고친 타입이라 Java8일 경우 무조건 써야 한다고 생각하면 됩니다.



Java8이 나오기 전까지 사용되었던 Date와 Calendar 클래스의 문제점

- 불변객체가 아니다.
  - 멀티스레드 환경에서 언제든 문제가 발생할 수 있습니다.
- Calendar는 월(Month)값 설계가 잘못되었습니다.
  - 10월을 나타내는 Calendar.OCTOBER의 숫자 값은 '9'입니다.
  - 당연시 10으로 생각했던 개발자들에게 큰 혼란

JodaTime이라는 오픈소스를 사용해서 문제점들을 피했었고, Java8에선 LocalDate를 통해 해결했습니다.



LocalDate와 LocalDateTime이 데이터베이스에 제대로 매핑되지 않는 이슈가 Hibernate 5.2.10 버전에서 해결되었습니다.



1. domain 패키지에 BaseTimeEntity 클래스를 생성
   BaseTimeEntity

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

   BaseTimeEntity 클래스는 **모든 Entity의 상위 클래스**가 되어 **Entity들의 createdDate, modifiedDate를 자동으로 관리하는 역할**입니다.

   - @MappedSuperclass
     - JPA Entity 클래스들이 BaseTimeEntity을 상속할 경우 필드들(createdDate, modifiedDate)도 칼럼으로 인식하도록 합니다.
   - @EntityListeners(AuditingEntityListener.class)
     - BaseTimeEntity 클래스에 Auditing 기능을 포함시킵니다.
   - @CreatedDate
     - Entity가 생성되어 저장될 때 시간이 자동 저장됩니다.
   - @LastmodifiedDate
     - 조회한 Entity의 값을 변경할 때 시간이 자동 저장됩니다.
       

2. 그리고 Posts 클래스가 BaseTimeEntity를 상속받도록 변경

   ```java
   public class Posts extends BaseTimeEntity {
       ...
   }
   ```

3. 마지막으로 JPA Auditing 어노테이션들을 모두 활성화할 수 있도록 Application 클래스에 활성화 어노테이션 하나를 추가

   ```java
   @EnableJpaAuditing // JPA Auditing 활성화
   @SpringBootApplication
   public class Application {
       public static void main(String[] args) {
           SpringApplication.run(Application.class, args);
       }
   }
   ```

   

   잘 동작하는지 확인을 위해 테스트코드 작성.



**JPA Auditing 테스트 코드 작성하기**

PostsRepositoryTest 클래스에 테스트 메소드 하나 더 추가.

```java
@Test
    public void BaseTimeEntity_등록(){
        //given
        LocalDateTime now = LocalDateTime.of(2020, 4, 24, 0, 0, 0);
        postsRepository.save(Posts.builder()
                .title("title")
                .content("content")
                .author("author")
                .build());
        //when
        List<Posts> postsList = postsRepository.findAll();

        //then
        Posts posts = postsList.get(0);

        System.out.println(">>>>>>>>> createDate=" + posts.getCreatedDate() + ", modifiedDate=" + posts.getModifiedDate());

        assertThat(posts.getCreatedDate()).isAfter(now);
        assertThat(posts.getModifiedDate()).isAfter(now);
    }
```

테스트 코드를 수행해 보면 

다음과 같이 실제 시간이 잘 저장된 것을 확인할 수 있습니다.

```
>>>>>>>>> createDate=2020-04-24T01:08:18.133, modifiedDate=2020-04-24T01:08:18.133
```



앞으로 추가될 엔티티들은 더이상 등록일/수정일로 고민할 필요가 없습니다. BaseTimeEntity만 상속받으면 자동으로 해결되기 때문입니다. 다음 장에서는 템플릿 엔진을 이용하여 화면을 만들어 보겠습니다.

