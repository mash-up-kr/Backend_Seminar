
스프링부트에서 JPA로 데이터베이스 다뤄보자
관계형 데이터베이스를 이용하는 프로젝트에서 객체지향 프로그래밍 -> JPA 자바 표준 ORM로 해결
데이터베이스를 직접 다루지 않고 알아서 처리해준다.

JPA란? 
객체지향 프로그래밍 언어와 관계형 데이터베이스는 패러다임이 다르게 때문에 객체를 데이터베이스에 저장하려면 문제가 생긴다.
그것을 해결해주기위해 중간에서 패러다임을 일치시켜주는 기술로 JPA가 있다.

Spring Data JPA
JPA는 인터페이스로 사용하기 위해서 구현체가 필요하다. 
구현체를 좀 더 쉽게 사용하고자 추상화시킨 Spirng Data JPA라는 모듈을 이용한다.

JPA <- Hibernate <- Spring Data JPA

Spring Data JPA 필요한 이유
- 구현체 교체의 용이성
- 저장소 교체의 용이성

중간에 있는 Hibernate가 다른 것으로 바뀌게 되면 이것을 교체하는데 편리하게 할 수 있다.
의존서만 교체하면 되기 때문에 편하다.

3.2 프로젝트에 Spring Data Jpa 적용하기

dependencies {
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
    compile('com.h2database:h2')
}

spring-boot-starter-data-jpa
    - 스프링 부트용 Spring Data Jpa 추상화 라이버르러ㅣ
    - JPA 관련 라이브러리 버전관리

h2
    - 인메모리형 관계형 데이터베이스
    - 별도 설치 없이 프로젝트의 의존성 만으로 관리가능
    - 테스트 용도로 많이 사용

src
    main
        java
            org.example.springboot
                domain
                    posts
                        posts

도메인 부분에 게시글, 댓글, 회원 등을 만들기 위해 필요한 부분을 작성한다.

@Getter
@NoArgsConstructor
@Entity
public class Posts {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT" , nullable = false)
    private String contennt;

    private String author;

    @Builder
    pulbic Posts(String title, String content, String author) {
        this.title  = title;
        this.content = content;
        this.author = author;
       }
}

롬복의 어노테이션은 @Getter, @NoArgsConstructor 는 필수 어노테이션은 아니다

@Entity
    - 테이블과 링크될 클래스
    - 기본값으로 클래스의 카멜케이스 이름을 언더스토어 네이밍(_)으로 테이블 이름을 매칭
    ex) SalesManager.java -> sales_manager table

왜하는지?

@id
    - 해당 테이블의 PK 필드를 나타낸다

@GeneratedValue
    - PK의 생성 규칙을 나타냄

@Column
    - 테이블의 칼럼 나타내며 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 칼럼이 된다
    - 사용하는 이유는 기본값 외에 추가로 변경될 옵션이 있을 때 사용

Entity 클래스에서는 절대 Setter 메소드를 만들지 않는다.
대신 목적과 의도를 나타낼 수 있는 메소드를 추가해서 알려준다.

Setter 없이 어떻게 값을 채워 DB에 삽입할까?
기본적인 구조는 생성자를 통해 최종값을 채운 후 DB에 삽입하는 것
값 변경이 필요한 경우 해당 이벤트에 맞는 public 메소드 호출하여 변경
But,
여기에서는 생성자 대신에 @Builder를 통해 제공되는 빌더 클래스 사용하여 채워준다.
빌더를 통해 어떤 값을 채워야되는지 알려주기 때문에 용이하게 쓰인다.

다음으로 posts 패키지 안에 Posts 클래스로 Database를 접근하게 해줄 JpaRepository 생성
이는 인터페이스를 생성하는 것이다

public interface PostsRepository extends JpaRepository<Posts, Long> {

}

@Repository를 추가할 필요 없으나 Entity 클래스와 기본 Entity Repository는 함께 위치해야 한다.
Entity 클래스는 기본 Repository 없이는 제대로 역할을 할 수 없다.

3.3 Spring Data JPA 테스트 코드 작성하기
test
    -org.example.com.springboot
        domain.posts
            -PostsRepositoryTest

이곳에서는 save, findAll 기능을 테스트 한다.
           
<<코드>>

@After
    - Junit에서 단위테스트가 끝날 떄마다 수행되는 메소드를 지정
    - 전체 테스트 수행할 때 데스트간 데이터 침범 막기 위해 사용

@postsRepository.save
    - 테이블 posts에 insert/update 쿼리를 실행
    - id 값이 있다면 update가 없다면 insert 쿼리 실행

@postsRepository.findAll
    - 테이블 posts에 있는 모든 데이터를 조회해오는 메소드


3.4 등록/수정/조회 API 만들기
API 만들기 위해 총 3개의 클래스가 필요

    -Request 데이터를 받을 Dto
    -API 요청을 받을 Controller
    -트랜잭션, 도메인 기능 간의 순서를 보장하는 Service

Spring 웹 계층
    Web Layer           <D
                        T
    Service Layer       OS>   
                        <Domain
    Repository Layer    Model>


Web Layer
    - 컨트롤러 ,JSP 등의 뷰 템플릿 영역
    - 필터, 인터셉터 등 외부 요청과 응답에 대한 전반적인 영역

Service Layer
    - @Service에 사용되는 서비스 영역
    - Controller 와 Dao의 중간 영역에서 사용
    - @Transaction이 사용되어야 하는 영역

Repository Layer
    - Database와 같이 데이터 저장소에 접근하는 영역

Dtos
    - 계층 간에 데이터 교환을 위한 객체
Domain Model
    - 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화 시킨 곳

등록, 수정, 삭제 기능 만들기

PostsApiController
PostsService

Controller와 Service에서 @Autowired 사용 안하고 생성자 주입 받는 방식이 있다.
생성자로 Bean 객체 = @Autowired

생성자는 바로 @RequiredArgsConstructor에서 해결
final이 선언된 모든 필자를 인자값으로 생성자를 롬복이 대신 생성해준다.

Controller와 Service에서 사용할 Dto 클래스 생성
PostsSavaRequestDto

@Getter
@Setter
@NoArgsConstructor
public class PostsSaveRequestDto {

    private String title;
    private String content;
    private String author;

    public Posts toEntity(){
        return Posts.builder()
                .title(title)
                .content(content)
                .author(author)
                .build();
    }
}

Entity 클래스를 Request/Response 클래스로 사용해서는 안된다.
Entity 클래스는 데이터베이스와 맞닿는 핵심 클래스이므로 Entity 클래스를 변경하게 되는 경우가 많으면 여러 클래스에
영향이 끼치므로 분리하는 것이 좋다.

테스트 코드로 검증하려면 
web
    PostsApiControllerTest 생성


수정/조회기능 구현
PostsApiController

PostsResponseDto

PostsService

update 기능에 데이터베이스에 쿼리르 날리는 부분이 없다 이는 JPA의 영속성 컨텍스트 떄문이다
영속성 컨테스트란? 엔티티를 영구 저장하는 환경
트랜잭션 안에서 데이터베이스에서 데이터를 가져오면 이 데이터는 영속성 컨테스트가 유지되고 이 상태에서
해당 값을 변경하면 트랜잭션이 끝나는 시점에 해당 테이블에 변경분을 반영한다.
Entity 객체의 값만 변경하면 별도로 쿼리 날릴 필요 없는데 이 개념을 더티 체킹이라고 한다.
* 더티체킹이란

PostsApiController 통해 수정이 잘되는지 확인한다.


JPA Auditing으로 생성시간/수정시간 자동화하기
생성하고 수정하는 시간이 적혀 있어야 차후 유지보수에 있어 중요하다.
매번 DB에 저장하기 전에 날짜 데이터 등록/수정 코드를 하는 것은 지저분하므로 JPA Auditing을
사용하여 저장할 때 날짜와 시간이 적혀있도록 만든다.


<궁금한점>
@Entity
    - 테이블과 링크될 클래스
    - 기본값으로 클래스의 카멜케이스 이름을 언더스토어 네이밍(_)으로 테이블 이름을 매칭
    ex) SalesManager.java -> sales_manager table

왜하는지?

웹계층 몬가 알듯모를듯..

