# 3. 스프링 부트에서 JPA로 데이터베이스 다뤄보자

## 진도 기록

|공부 날짜|공부한 페이지 수|
|---|---|
|2020년 4월|79p ~ 124p|

## 3.1 JPA 소개

현대의 웹 애플리케이션에서 관계형 데이터베이스(RDB, Relational Database)는 빠질 수 없는 요소이다. 그러다 보니 **객체를 관계형 데이터베이스에서 관리하는 것**이 무엇보다 중요하다.

* **관계형 데이터베이스**는 어떻게 데이터를 저장할지에 초점이 맞춰진 기술

* **객체지향 프로그래밍 언어**는 메시지를 기반으로 기능과 속성을 한 곳에서 관리하는 기술

이처럼 두개의 영역은 패러다임이 서로 다른데, 객체를 데이터베이스에 저장하려고 하니 문제가 발생한다. 이를 **`패러다임 불일치`** 라고 한다.

* 객체지향 프로그래밍에서 부모가 되는 객체를 가져오려면
```
User user = findUser();
Group group = user.getGroup();
```

User와 Group은 부모-자식 관계임을 명확하게 알 수 있다.

* 하지만 여기에 데이터베이스가 추가되면

```
User user = userDao.findUser();
Group group = groupDao.findGroup(user.getGroupId());
```

User 따로, Group 따로 조회하게 되므로 서로 어떤 관계인지 명확하게 파악이 어렵다. 

이처럼 다양한 객체 모델링을 데이터베이스로는 구현할 수 없으니, 웹 애플리케이션 개발은 데이터베이스 모델링에만 집중하게 된다. 

### `JPA` 란, 
* 서로 지향점이 다른 2개 영역(객체지향, 관계형 데이터베이스)을 **중간에서 패러다임 일치**를 시켜주기 위한 **자바 표준 ORM 기술**

* 개발자는 항상 객체지향적으로 프로그래밍을 표현할 수 있고, JPA가 이를 관계형 데이터베이스에 맞게 SQL을 대신 생성해서 실행한다.
  * SQL에 종속적인 개발을 하지 않아도 되는 것이다.
* 장점: 다양한 객체지향 프로그래밍 쉽게 가능, 객체 중심이므로 생산성 향상과 유지 보수의 편리, CRUD 쿼리 직접 작성할 필요 없음
* 단점: 높은 러닝 커브

### `Spring Data JPA`

* 인터페이스인 JPA를 사용하려면 구현체가 필요
  * Hibernate, Eclipse, Link 등
* Spring에서는 구현체들을 좀 더 쉽게 사용하고자 추상화시킨 `Spring Data JPA 모듈`을 이용해서 JPA 기술을 다룬다.
  * JPA <- Hibernate <- Spring Data JPA


* 등장 이유
  * 구현체 교체의 용이성 
  * 저장소 교체의 용이성

## 3.2 프로젝트에 Spring Data Jpa 적용하기

### **build.gradle에 의존성 등록**

#### spring-boot-starter-data-jpa
* 스프링 부트용 Spring Data Jpa 추상화 라이브러리
* 스프링 부트 버전에 맞춰 자동으로 JPA관련 라이브러리들의 버전 관리

#### h2
* 인메모리 관계형 데이터베이스로, 별도의 설치 필요 없이 프로젝트 의존성만으로 관리 가능
* 메모리에서 실행되어 애플리케이션을 재시작할 때마다 초기화된다는 점을 이용해 테스트 용도로 많이 사용

### **domain 패키지**

main의 web패키지와 같은 위치에 domain패키지를 생성한다.

**도메인**이란 

* 게시글, 댓글, 회원, 정산, 결제 등 소프트웨어에 대한 요구사항 혹은 문제영역을 의미
* xml에 쿼리를 담고, 클래스는 쿼리의 결과만 담던 일들이 모두 도메인 클래스에서 해결된다.

domain 패키지에 posts 패키지와 Posts 클래스를 생성한다.

**Posts 클래스**는 실제 DB의 테이블과 매칭될 클래스로, 보통 **Entity 클래스**라고 한다. JPA를 사용해서 DB를 작업할 경우, 실제 쿼리를 날리기보다는 Entity 클래스의 수정을 통해 작업한다.

**JPA 어노테이션**
* **`@Entity`**
  * 테이블과 링크될 클래스임을 나타낸다.
  * 기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍(_)으로 테이블 이름을 매칭한다

* **`@Id`**
  * 해당 테이블의 PK 필드를 나타낸다.
  
* **`@GeneratedValue`**
  * PK의 생성 규칙을 나타낸다.
  * 스프링 부트 2.0에서는 GenerationType.IDENTITY 옵션을 추가해야만 auto_increment가 된다.

* **`@Column`**
  * 테이블의 칼럼을 나타내며 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 칼럼이 된다.
  * 기본값 외에 추가로 변경이 필요한 옵션이 있으면 사용한다.

**롬복 어노테이션**

* **`@NoArgsConstructor`**
  * 기본 생성자 자동 추가
  * public Posts() {}와 같은 효과

* **`@Getter`**
  * 클래스 내 모든 필드의 Getter 메소드 자동생성
 
* **`@Builder`**
  * 해당 클래스의 빌더 패턴 클래스를 생성 
  * 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함

**Setter 메소드가 없다?**
* **Entity 클래스에서는 절대 Setter 메소드를 만들지 않는다.**
* 자바빈 규약을 생각하며 getter/setter를 무작정 생성하는 경우가 있다.이렇게 되면 해당 클래스의 인스턴스 값들이 언제 어디서 변해야 하는지 코드상으로 명확하게 구분할 수가 없어, 차후 기능 변경 시 복잡해진다.
* 필드의 값 변경이 필요하면, 명확히 목적과 의도를 나타낼 수 있는 메소드를 추가해야만 한다.

**Setter가 없는데 어떻게 값을 채워 DB에 삽입하는가?**
* 기본적인 구조는 **생성자를 통해** 최종값을 채운 후 DB에 삽입 하는 것
* 값 변경이 필요한 경우 해당 이벤트에 맞는 **public 메소드를 호출**하여 변경 하는 것을 전제로 한다.
* 책에서는 @Builder를 통해 제공되는 **빌더 클래스**를 사용
* 생성자나 빌더나 생성 시점에 값을 채워주는 역할은 똑같지만, 생성자의 경우 지금 채워야 할 필드가 무엇인지 명확히 지정할 수가 없다.

```
@Getter
@NoArgsConstructor
@Entity
public class Posts extends BaseTimeEntity {

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

### **PostsRepository.java**

Posts 클래스로 Database를 접근하게 해줄 `JpaRepository` 생성

DB Layer 접근자로, JPA에서는 Repository라고 부른다. 

단순히 인터페이스 생성 후, JpaRepository<Entity 클래스, PK 타입>를 상속하면 기본적인 CRUD 메소드가 자동으로 생성된다.

**Entity 클래스와 기본 Entity Repository는 함께 위치해야 한다 !**

```
import org.springframework.data.jpa.repository.JpaRepository;
public interface PostsRepository extends JpaRepository<Posts, Long> {}
```

## 3.3 Spring Data JPA 테스트 코드 작성하기

test 디렉토리에 domain.posts 패키지를 생성 후 PostsRepositoryTest 클래스를 생성 한다. 

* 별다른 설정 없는 경우 -> H2 데이터베이스 자동으로 실행

**`@After`**
* Junit에서 단위 테스트가 끝날 때마다 수행되는 메소드를 지정
* 배포 전 전체 테스트 수행시 테스트간 데이터 침범을 막기 위해 사용
* 여러 테스트가 동시에 수행되면 테스트용 데이터베이스인 H2에 데이터가 그대로 남아 다음 테스트 실행 시 테스트가 실패할 수 있다.

```
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
        List<Posts> postsList = postsRepository.findAll();

        //then
        Posts posts = postsList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title);
        assertThat(posts.getContent()).isEqualTo(content);
    }
}
```

## 3.4 등록/수정/조회 API 만들기

API를 만들기 위해서는 총 3개의 클래스가 필요하다.
* Request 데이터를 받을 Dto
* API 요청을 받을 Controller
* 트랜잭션, 도메인 기능 간의 순서를 보장하는 Service
  * Service는 위의 역할만 수행
  * 비즈니스 로직을 처리하는 곳은 Domain !

### Spring 웹 계층

* **Web Layer**
  * 컨트롤러(@Controller)와 JSP/Freemarker 등의 뷰 템플릿 영역
  * 외부 요청과 응답에 대한 전반적인 영역
    * @Filter, 인터셉터, @ControllerAdvice등

* **Service Layer**
  * @Service에 사용되는 서비스 영역
  * Controller와 Dao의 중간 영역에서 사용
  * @Transactional이 사용되어야 하는 영역이기도 하다.

* **Repository Layer**
  * Database와 같은 데이터 저장소에 접근하는 영역
  * Dao(Data Access Object)영역으로 생각 가능

* **Dtos**
  * Dto(Data Transfer Object)는 계층 간에 데이터 교환을 위한 객체 를 이야기하며 Dtos 는 이들의 영역을 의미
  * 뷰 템플릿 엔진에서 사용될 객체나 Repository Layer에서 결과로 넘겨준 객체 등을 의미

* **Domain Model**
  * 도메인이라 불리는 개발 대상을 단순화시킨 것
  * 택시앱이라고 하면 배차, 탑승, 요금 등이 모두 도메인이 될 수 있다.
  * @Entity가 사용된 영역 역시 도메인 모델
  * 무조건 데이터베이스의 테이블과 관계가 있어야만 하는 것은 아니다.

### Spring에서 Bean 주입받기

* @Autowired
* setter
* **생성자 (권장)**
  * 생성자로 Bean 객체를 받도록 하면 @Autowired와 동일할 효과 

롬복의 `@RequiredArgsConstructor` 로 final이 선언된 모든 필드를 인자값으로 하는 생성자를 생성할 수 있다.
* 해당 클래스의 의존성 관계가 변경될 때마다 생성자 코드를 계속해서 수정하는 번거로움을 해결하기 위해 롬복 어노테이션 사용

### Dto와 Entity 클래스 분리

* Entity 클래스는 데이터베이스와 맞닿은 핵심 클래스
* 수많은 서비스 클래스나 비즈니스 로직들이 Entity 클래스를 기준으로 동작하므로 **Entity 클래스가 변경되면 여러 클래스에 영향**을 끼친다.
* 따라서 자주 변경이 필요한 View를 위한 클래스로, **Request와 Response용 Dto**를 만들어 사용한다.

> Entity 클래스를 Request/Response 클래스로 사용해서는 안된다 !   
> View Layer와 DB Layer의 역할 분리를 철저하게 하는것이 좋다 !

### Api Controller  테스트
* @WebMvcTest의 경우 JPA 기능이 작동하지 않는다.
  * Controller와 ControllerAdvice 등 외부 연동과 관련된 부분만 활성화
* 지금처럼 JPA 기능까지 한번에 테스트할 때는 @SpringBootTest와 TestRestTemplate을 사용한다.

### 더티 체킹 (dirty checking)

* 수정/조회 기능을 추가하는 코드들의 update 기능에 데이터베이스에 쿼리를 날리는 부분이 없다.
  * JPA 의 영속성 컨텍스트 덕분에 가능한 것
* **영속성 컨텍스트** : 엔티티를 영구 저장하는 환경
  * JPA의 엔티티 메니저가 활성화된 상태로, 트랜잭션 안의 데이터베이스에서 데이터를 가져오면 이 데이터는 영속성 컨텍스트가 유지된 상태
  * 이 상태에서 해당 데이터의 값을 변경하면 트랜잭션이 끝나는 시점에 해당 테이블에 변경분을 반영
* Entity 객체의 값만 변경하면 별도로 Update 쿼리를 날릴 필요가 없다. => **더티 체킹**

## 3.5 JPA Auditing으로 생성시간/수정시간 자동화하기

주로 Entity에는 해당 데이터의 생성시간과 수정시간을 포함한다. 차후 유지보수에 있어 필요한 정보이기 때문이다. DB에 날짜 데이터를 등록/수정하는 코드가 여기저기 들어가면 코드가 보기 안좋아지므로 `JPA Auditing`을 사용한다.

### LocalDateTime

domain 패키지에 `BaseTimeEntity` 클래스를 생성

모든 Entity의 상위 클래스가 되어 Entity들의 createdDate, modifiedDate를 자동으로 관리하는 역할

```
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
* `@MappedSuperclass`
  * JPA Entity 클래스들이 BaseTimeEntity을 상속할 경우 필드들도 칼럼으로 인식하도록 한다.
* `@EntityListeners`(AuditingEntityListener.class)
  * BaseTimeEntity 클래스에 Auditing 기능을 포함시킨다.
* `@CreatedDate`
  * Entity가 생성되어 저장될 때 시간이 자동 저장
* `@LastModifiedDate`
  * 조회한 Entity의 값을 변경할 때 시간이 자동 저장

Posts 클래스가 BaseTimeEntity를 상속받도록 변경
```
public class Posts extends BaseTimeEntity{
    ...(중략)
}
```

Application 클래스에 JPA Auditing 어노테이션들을 모두 활성화하는 어노테이션 하나 추가

```
@EnableJpaAuditing
...
```

## 회고

어노테이션의 종류가 정말 다양하고 많다는 것을 느꼈다...