# 3. 스프링 부트에서 JPA로 DB 다뤄보자

## 진도 기록

| 공부 날짜  | 공부한 페이지 수 |
| ---------- | ---------------- |
| 2020.04.08 | 78p ~ 100p       |
| 2020.04.09 | 100p ~ 124p      |

## 3.1. JPA 소개

웹 애플리케이션을 개발하다보면 SQL문을 작성하는 것에 많은 시간을 쏟게 된다.. 수십 수백개의 테이블에 대해 반복적인 SQL문을 작성하는 수고로움은 누구나 경험해봤을 것이다. 다양한 객체 모델링을 데이터 베이스로 구현하려고 노력하다보면 어느새 데이터베이스 모델링에 더 힘을 쏟게되는 자신을 발견하게 된다.

테이블 모델링에 드는 시간을 단축하고 애플리케이션보다 많아지는 문제를 방기하기위해 등장한 것이` JPA`이다. 

관계형 데이터 베이스와 객체지향 사이에서 발생하는 패러다임 불일치를 해결해주며 개발자는 JPA를 활용하여 객체지향적인 프로그래밍을 할 수 있으므로 **SQL에 종속적인 개발을 하지 않아도 된다!**

**JPA의 장점**

- 구현체 매핑을 지원해주기 때문에 쉽게 DB 교체가 가능하다.

> 관계형 데이터베이스 외에 다른 저장소로 쉽게 교체가 가능하다.

## 3.2. 프로젝트에 Spring Data Jpa 적용하기

### 의존성 알아보기

1. spring-boot-starter-data-jpa
   - 스프링 부트용 추상화 라이브러리이다. 
   - 버전에 맞춰 자동으로 JPA관련 라이브러리들의 버전을 관리해준다.
2. h2
   - 인메모리형 RDB이다.
   - 별도의 설치 필요없이 프로젝트 의존성만으로 관리가 가능하다!
   - 메모리에서 실행되므로  재시작시 초기화되어 테스트용으로 자주 사용된다.

### Domain

**도메인**이란 게시글, 댓글, 회원, 결제 등 소프트웨어에 대한 문제영역 또는 문제사항이다.

도메인에 어떤 코드들이 들어가고 관리되는지 확인해보자.

**Entity class**

- 모델의 정의가 이루어진다.
- Setter 클래스 사용을 지양하자! 어디서 변화했는지 명확히 구분이 어렵기 때문이다.
  - 명확하게 의도나 목적이 나타나는 메소드를 추가하도록하자.

```java
public class Order{
    public void cancelOrder(){
        this.status = false;
    }
}

public void 주문서비스_취소이벤트() {
    order.cancelOrder();
}
```

코드를 통해 Entity class가 어떻게 작성되는지 자세히 보도록하겠다.

```JAVA
@Getter
@NoArgsConstructor
@Entity 
// 필수 Annotation을 가까이 두는 센스,, 새언어 전환시 필요없는 부분을 수월하게 제거하기 위함이다.
public class Posts extends BaseTimeEntity {

    @Id // PK필드
    @GeneratedValue(strategy = GenerationType.IDENTITY) // AI옵션
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false) // 컬럼 옵션 추가
    private String content;
    
    private String author;

    @Builder // 생성자 대신 빌더 클래스를 통해 객체를 생성한다.
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

**EntityPostsRepository**

- CRUD 를 편하게 해준다!
- Entity Repository는 Entity 클래스와 함께 위치해야한다.

```java
public interface PostsRepository extends JpaRepository<Posts, Long> {
}
```

이 간단한 두줄만으로 CRUD가 가능해졌다!

## 3.3. Spring Data JPA 테스트 코드 

2장에서 테스트 코드의 중요성을 다루었다. 중요성을 아는 만큼 실제로 적용해보자.

>  remind: 테스트 코드는 개발 초기단계의 문제를 발견하게 해주고 불확실성을 감소시켜준다.

- 단위 테스트를 위한 프레임워크로 junit 을 사용한다.
- 테스트를 위한 DB로 H2를 사용한다.
  - 별다른 설정이 없는 경우 H2를 사용한다.

코드를 보며 이해해보자 (impot는 생략한다)

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class PostsRepositoryTest {

    @Autowired // 스프링이 관리하는 bean 주입하기
    PostsRepository postsRepository;

    @After // 단위 테스트가 끝나고 수행되는 메소드.
    public void cleanup(){
        postsRepository.deleteAll();
    }

    @Test
    public void loadPosts() {
        String title = "게시글";
        String content = "본문";

        postsRepository.save(Posts.builder()
        .title(title)
        .content(content)
        .author("jiss02@naver.com")
        .build());

        List<Posts> postsList = postsRepository.findAll();

        Posts posts = postsList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title); // chaining 가능!
        assertThat(posts.getContent()).isEqualTo(content);
    }

    @Test
    public void addBaseTimeEntity() {
        LocalDateTime now = LocalDateTime.of(2019,6,4,0,0,0);
        postsRepository.save(Posts.builder()
        .title("title")
        .content("content")
        .author("author")
        .build()); // insert/update 실행 (id 유무로 판단)

        List<Posts> postsList = postsRepository.findAll(); // posts의 모든 데이터 조회

        Posts posts = postsList.get(0);

        System.out.println(">>>>>>>>> create Date=" + posts.getCreatedDate() + ", modified Date=" + posts.getModifiedDate());

        assertThat(posts.getModifiedDate()).isAfter(now);
        assertThat(posts.getModifiedDate()).isAfter(now);
    }

}

```

- @After
  - 배포 전 전체 테스트 수행시 테스트간의 데이터 침범을 방지하기위해 사용한다.
  - 여러 테스트가 동시에 실행되면 h2에 데이터가 그대로 남아 다음 테스트 실행시 실패할 수 있으니 조심하자.

> 감이 안잡힌다.. 몇번 반복하면서 따라치다보니 감이 오기는하는데... 좀 더 많이 코드를 작성해 봐야 할 것 같다.

## 3.4. 등록/수정/조회 API 만들기

### Spring 웹 계층

- Web layer
  - View template, controller, 예외 핸들러, filters
  - **외부 요청과 응답에 대한** 전반적인 영역
- Service Layer
  - **Contoller와 Dao의 중간**에서 사용된다.
  - @Transactional이 사용되어야 하는 영역이기도 하다.
  - 비지니스 로직이 사용되지 않는다!! **트랜젝션과 도메인 간의 순서 보장**만 해주도록하자.
  - 비지니스 로직은 `Domain`에서 처리한다.
- Repository Layer
  - DB와 같은 **데이터 저장소에 접근하는 영역**이다.
  - Dao 영역이라 생각하자.
- Dtos
  - 계층 간의 데이터 교환을 위한 객체이다. Entity 클래스와 유사하지만 다르다!
  - 템플릿 엔진에서 사용될 객체나 Repository Layer에서 결과로 넘겨준 객체들을 의미한다.
- Domain
  - 비지니스 로직을 담당한다. 

### Dto를 사용하는 이유

Entity 클래스는 DB와 맞닿는 핵심 클래스이기에 이가 변경되면 여러 클래스들에 영향을 주게된다.

그렇기에 자주 변경이 일어나는 View를 위한 클래스로 Request, Response용 Dto를 만들어 사용하는 것이다.

`View layer와 DB layer를 철저하게 나누어 Entity 클래스의 변경을 최소화하자`

### Bean 주입받기

1. @Autowired
2. setter
3. **생성자** < 권장!

**생성자로 Bean 객체를 받도록하면 ..?**

생성자로 객체를 받으면 @Autowired와 동일한 효과를 얻을 수 있다!

`@RequiredArsConstructor`를 사용하여 final이 선언된 모든 필드를 인자값으로 하는 생성자를 생성할 수 있다.

> 롬복의 어노테이션을 사용하면 의존성 관계가 변경되어도 생성자 코드를 수정하지 않아도 된다.

## 3.5. JPA Auditing으로 생성시간/수정시간 자동화하기

일반적으로 Entity에는 Eintity의 생성시간과 수정시간을 포함하는 경우가 많다.

그렇다 보니 매번 DB에 시간을 삽입하는 코드가 여기저기에 놓여지게 되는데, 이러한 반복적인 코드는 가독성을 떨어트리고 코드가 지저분해지므로 `JPA Auditing`을 통해 이 문제를 해결해보자.

### BaseTimeEntity

BaseTimeEntity 클래스를 생성하자. 

이는 모든 Entity의 상위 클래스가 될 것이며 createDate와 modifiedDate를 자동으로 관리하는 역할을 한다!

코드는 다음과 같다.

1. BaseTimeEntity 

```java
@Getter
@MappedSuperclass // 상속받는 클래스들이 생성시간, 수정시간을 칼럼으로 인식하도록 한다.
@EntityListeners(AuditingEntityListener.class) // Auditing 기능을 포함시킨다.
public class BaseTimeEntity {

    @CreatedDate // 생성시 시간 자동저장
    private LocalDateTime createdDate;

    @LastModifiedDate // 수정시 시간 자동저장
    private LocalDateTime modifiedDate;
}
```

2. Posts 클래스가 BaseTimeEntity 상속받도록 변경

```java
	...
public class Posts extends BaseTimeEntity {
    ...
}
```

3. Application에 JPA Auditing 활성화 시키는 어노테이션 추가.

```java
@EnableJpaAuditing
...
```

## 궁금한 점 스스로 찾아보기

+) 다람쥐님이 코멘트 달아준 빈 .. 추가..

### 0. Bean..?

일반적인 객체인데, Ioc 컨테이너에 의해 관리되는 객체이다.

> 스프링 컨테이너에 의해 조립되거나 관리되는 자바 객체를 의미한다.

빈을 등록하는 방법은 두가지가 있는데,

1. 스프링부트의 경우 @Component, @Service, @Controller, @Repository, @Bean, @Configuration(빈 설정 파일 사용시 사용되는 어노테이션)과 같은 어노테이션으로 빈들을 등록하고 필요한 곳에서  @Autowired를 통해 주입받아 사용한다. 

   ```java
   @Configuration
   class SampleConfig {
       @Bean
       pulic SampleController sampleController() {
           return new SampleController(); // 리턴하는 객체가 빈으로 등록된다.
       }
   }
   ```

2.  Repository는 Jpa가 제공해주는 기능에 의해 Bean으로 등록이 된다. 특정한 어노테이션 없이 특정 인터페이스를 상속받으면 그 인터페이스를 상속받은 클래스들을 찾아 자동 등록이 되는 친절한 녀석.. 

### 1. Dao와 Dto

**Dao란**

Data Access Object의 약자로, **DB의 data에 접근하기 위한** 객체이다.

- DB에 접근하기 위한 로직과 비지니스 로직을 분리하기 위해 사용한다.

> Persistence Layer(DB에 data를 CRUD하는 계층)이다.

**Dto란**

Data Transfer Object로 **계층간 데이터 교환을 위한 자바 빈즈**를 의미한다.

- `로직을 가지지 않는` 순수한 데이터 객체이다.

- DB에서 데이터를 얻어 Service나 Controller 등으로 보낼 떄 사용하는 객체이다.
- DB에서 꺼낸 값을 임의로 변경할 필요가 없기 때문에 setter 없이 생성자에서 값을 할당한다.
- Request와 Response용 DTO는 View를 위한 클래스 이다.
  - 자주 변경이 일어난다.
  - toEntity 메소드를 이용해 DTO에서 필요한 부분을 이용해 Entity로 만든다
  - Controller Layer에서 ResponseDto 형태로 클라이언트에 전달한다.

### 2.디렉토리 구조

![img](/images/dir.jpg)

- Controller (web)
  - url에 따라 적절한 data처리와 View 매핑
  - ReponseDto를 body에 담아 클라이언트에 반환
- Service
  - @Autowired Repository를 통해 repository의 method를 이용한다.
  - DAO로 DB에 접근하고 DTO로 데이터를 전달받은 다음, 비지니스 로직을 처리해 적절한 데이터를 반환한다.

## 회고

어노테이션의 향연에 정신이 혼미했다. 

스프링 부트를 제대로 이해하려면 디렉토리 구조를  파악하는게 좋을 것 같아서 최대한 열심히 노트에 그려보았는데, 확실히 디렉토리 구조를 파악하니 공부하는데 조금 더 수월해졌다.

아직까지는 익숙치 않아서 감이 잡히지 않지만 계속 반복하다보면 익숙해질 것 같다.