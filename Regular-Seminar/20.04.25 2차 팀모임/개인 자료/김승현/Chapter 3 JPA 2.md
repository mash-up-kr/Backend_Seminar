# Chapter 3: 스프링 부트에서 JPA로 데이터베이스 다뤄 보자

Created: Mar 25, 2020 2:26 PM
Tags: Web Layer, dto

## 3.3 Spring Data JPA에서 테스트 코드 작성하기

- test 디렉토리에 domain.posts 패키지를 생성하고, 테스트 클래스는 PostsRespositoryTest란 이름으로 생성한다.

```java
package example.org.practice.domain.posts;

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
public class PostRepositoryTest {

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
                                            .author("abc@def.com")
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

- `@After`
    - JUnit에서 단위 테스트가 끝날 때마다 수행되는 메소드를 저장
    - 보통은 배포 전 전체 테스트를 수행할 때 테스트간 데이터 침범을 막기 위해 사용한다.
    - 여러 테스트가 동시에 수행되면 테스트용 데이터베이스인 H2에 데이터가 그대로 남아 있어 다음 테스트 실행 시 테스트가 실패할 수 있다.
- `postsRepository.save`
    - 테이블 posts에 insert/update 쿼리를 실행한다.
    - id 값이 있다면 update가, 없다면 insert 쿼리가 실행된다.
- `postsRepository.findAll`
    - 테이블 posts에 있는 모든 데이터를 조회해 오는 메소드이다.
- 별다른 설정 없이 `@SpringBootTest`를 사용할 경우 **H2 데이터베이스**를 자동으로 실행해 준다.
- `src/main/resources` 디렉토리 아래에 `[application.properties](http://application.properties)` 파일 추가 후 `spring.jpa.show_sql=true`를 추가해 주면 콘솔에서 쿼리 로그를 확인할 수 있다.
    - 출력되는 쿼리 로그를 SQL 버전으로 변경하려면 같은 파일에 `spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect` 를 추가한다.

## 3.4 등록, 수정, 조회 API 만들기

- API를 만들기 위해서는 총 3개의 클래스가 필요하다.
    - Request 데이터를 받을 Dto
        - Date Transfer object, 데이터 전송에 사용하는 객체로 View 단에 사용하거나 다른 비즈니스 로직에 데이터를 전송할 때 사용한다.
    - API 요청을 받을 Controlloer
    - 트랜잭션, 도메인 기능 간의 순서를 보장하는 Service

- Spring 웹 계층은 다음과 같다.
    - Web Layer: 흔히 사용하는 컨트롤러(`@Controller`)와 JSP/Freemaker 등의 뷰 템플릿 영역이다. 이외에도 필터(`@Filter`), 인터셉터, 컨트롤러 어드바이스(`@ControllerAdvice`) 등 **외부 요청과 응답**에 대한 전반적인 영역을 이야기한다.
    - Service Layer: `@Service`에 사용되는 서비스 영역이다. 일반적으로 Controller와 Dao의 중간 영역에서 사용된다. `@Transactional`이 사용되어야 하는 영역이기도 하다.
        - Dao(Data Access Object): 데이터베이스의 데이터에 접근하기 위한 객체이다. 데이터베이스에 접근하기 위한 로직과 비즈니스 조직의 분리를 위해 사용하며, 데이터베이스를 사용해 데이터를 조회하거나 조작하는 기능을 전달하도록 만든 객체이다.
    - Repository Layer
        - **Database**와 같이 데이터 저장소에 접근하는 영역이다.
    - Dtos
        - Dto는 **계층 간에 데이터 교환을 위한 객체**를 이야기하며 Dtos는 이들의 영역을 이야기한다. 예를 들어 뷰 템플릿 엔진에서 사용될 객체나 Repository Layer에서 결과로 넘겨 준 객체 등이 이들을 이야기한다.
    - Domain model
        - 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화시킨 것을 도메인 모델이라고 한다. 이를테면 택시 앱이라고 하면 배차, 탑승, 요금 등이 모두 도메인이 될 수 있다. `@Entity`가 사용된 영역 역시 도메인 모델이라고 할 수 있다.
        - 다만, 무조건 데이터베이스의 테이블과 관계가 있어야만 하는 것은 아니다. VO처럼 값 객체들도 이 영역에 해당하기 때문이다.

- 위의 레이어들에서 비즈니스 처리를 담당해야 할 곳은 **Domain**이다.
    - 도메인에서 처리하는 경우 서비스 메소드는 **트랜잭션과 도메인 간의 순서만 보장**해 준다.

```java
@Transactional
public Order cancelOrder(int orderId){
	
	//1)
	Orders order = ordersRepository.findById(orderId);
	Billing billing = billingRepository.findByOrderId(orderId);
	Delivery delivery = deliveryRepository.findByOrderId(orderId);

	//2-3)
	delivery.cancel();
	
	//4)
	order.cancel();
	billing.cancel();

	return order;
}
```

- PostsApiController를 web 패키지에, PostsSaveRequestDto를 web.dto 패키지에, PostsService를 service 패키지에 생성한다.

```java
// PostsApiControlloer

package example.org.practice.web;

import example.org.practice.service.posts.PostsService;
import example.org.practice.web.dto.PostsSaveRequestDto;
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

}
```

```java
// PostsService

package example.org.practice.service.posts;

import example.org.practice.domain.posts.Posts;
import example.org.practice.web.dto.PostsSaveRequestDto;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

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

- Controller와 Service에 왜 `@Autowired`가 없을까?
- 스프링에서 Bean을 주입하는 방식들은 다음과 같다.
    - `@Autowired`
    - setter
    - 생성자
- 가장 권장되는 게 **생성자**로 주입받는 방식이다. 위의 코드에서 생성자는 `@RequiredArgsConstructor`가 해결해 준다. **final이 선언된 모든 필드**를 인자값으로 하는 생성자를 롬복의 `@RequiredArgsConstructor`가 대신 생성해 준 것이다.

```java
// PostsSaveRequestDto

package example.org.practice.web.dto;

import example.org.practice.domain.posts.Posts;
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
    public PostsSaveRequestDto(String title, String content,
                                            String author) {
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

- Entity 클래스와 거의 유사한 형태임에도 Dto 클래스를 추가로 생성했다. 그렇지만 절대 **Entity 클래스를 Request/Response 클래스로 사용해서는 안 된다.**
- Entity 클래스는 **데이터베이스와 맞닿은 핵심 클래스**로, 이를 기준으로 테이블이 생성되고 스키마가 변경된다. Entity 클래스가 변경되면 여러 클래스에 영향을 끼치지만 Request와 Response용 Dto는 View를 위한 클래스라 자주 변경이 필요하다.
    - View Layer와 DB Layer의 역할 분리는 철저한 게 좋다. 실제로 Controller에서 **결괏값으로 여러 테이블을 조인해서 줘야 할 경우**가 많으므로, Entity 클래스만으로 표현하기 어려운 경우가 많다.
    - 따라서 꼭 Entity 클래스와 Controller에서 쓸 Dto는 분리해서 사용해야 한다.

- 테스트의 web 패키지에 테스트 코드를 추가한다.

```java
// PostsApiControllerTest

package example.org.practice.web;
import example.org.practice.domain.posts.Posts;
import example.org.practice.domain.posts.PostsRepository;
import example.org.practice.web.dto.PostsSaveRequestDto;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.HttpStatus;
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
        assertThat(all.get(0).getContent()).
                                        isEqualTo(content);

    }

}
```

- `@WebMvcTest`의 경우 **JPA 기능이 작동하지 않기** 때문에, Controller와 ControlloerAdvice 등 **외부 연동과 관련된 부분**만 활성화되므로 JPA 기능까지 한번에 테스트할 때는 `@SpringBootTest`와 TestRestTemplate을 사용한다.