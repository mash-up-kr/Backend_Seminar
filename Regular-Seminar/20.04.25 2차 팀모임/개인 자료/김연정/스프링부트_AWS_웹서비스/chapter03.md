# 3. 스프링 부트에서 JPA로 데이터베이스 다뤄보자

## 진도 기록

|공부 날짜|공부한 페이지 수|
|---|---|
|2020.04.16|79p ~ 86p|
|2020.04.17|3.1 작성|
|2020.04.20|86p ~ 95p|
|2020.04.24|3.2 작성|

## 3.1. JPA 소개

<details>
  <summary> 소개 배경 (Click👆🏻)</summary>

  <br>
  
  MyBatis와 같은 SQL Mapper를 이용해서 데이터베이스의 쿼리를 작성하다보니, <br>
  실제로 개발하는 시간보다 SQL을 다루는 시간이 더 많았습니다.
  
  객체 모델링보다는 테이블 모델링에만 집중하고, 객체를 단순히 테이블에 맞추어 <br>
  데이터를 전달 역할만 하는 개발은 분명 기형적인 형태였습니다.

  이런 문제의 해결책으로 JPA라는 자바 표준 ORM(Object Relational Mapping) 기술을 만나게 됩니다. <br>

  > MyBatis, iBatis는 ORM이 아닌 SQL Mapper입니다. <br>
  > ORM은 객체를 매핑하는 것이고, SQL Mapper는 쿼리를 매핑합니다.

</details>

### # JPA의 필요성

현대의 웹 애플리케이션에서 관계형 데이터베이스(RDB)는 빠질 수 없는 요소이며, <br>
**객체를 관계형 데이터베이스에서 관리**하는 것이 무엇보다 중요합니다. <br>
관계형 데이터베이스가 계속해서 웹 서비스의 중심이 되면서 <br>
**애플리케이션 코드보다 SQL**로 가득하게 된 것입니다.

또한, 다음과 같은 문제점이 있습니다.

1. 각 테이블마다 기본적인 CRUD SQL을 매번 생성해야 합니다. <br>
   : 관계형 데이터베이스가 SQL만 인식할 수 있기 때문에 반복적인 SQL을 계속해서 만들어야 합니다. <br>
   자바 클래스를 잘 설계해도, SQL을 통해야만 데이터베이스에 저장하고 조회할 수 있어서 SQL은 피할 수 없습니다.

2. 패러다임 불일치 문제가 발생합니다. <br>
   : 관계형 데이터베이스는 **어떻게 데이터를 저장**할지에 초점이 맞춰진 기술입니다. <br>
   객체지향 프로그래밍 언어는 메시지를 기반으로 **기능과 속성을 한 곳에서 관리**하는 기술입니다. <br>
   이 둘은 패러다임이 서로 다른데, 객체를 데이터베이스에 저장하려고 하니 여러 문제가 발생합니다. <br>
   이것이 패러다임 불일치입니다.

JPA는 이러한 문제점을 해결하기 위해 등장하게 됩니다. <br>
서로 지향하는 바가 다른 2개 영역을 **중간에서 패러다임 일치**를 시켜주기 위한 기술입니다.
개발자는 SQL에 종속적인 개발을 하지 않고, 객체지향적으로 프로그래밍을 합니다. <br>
그러면 JPA가 이를 관계형 데이터베이스에 맞게 SQL을 대신 생성해서 실행합니다.

<br>

### # Spring Data JPA

JPA는 인터페이스로서 자바 표준명세서입니다. <br>
인터페이스인 JPA를 사용하기 위해서는 Hibernate, Eclipse Link 등과 같은 **구현체가 필요**합니다. <br>
하지만, Spring에서는 이 구현체들을 직접 다루지 않고 Spring Data JPA라는 모듈을 이용합니다. <br>
> JPA ← Hibernate ← Spring Data JPA

구현체들을 좀 더 쉽게 사용하고자 추상화시켜 한 단계 더 감싸놓은 Spring Data JPA가 등장한 이유는 다음과 같습니다.

- 구현체 교체의 용이성 <br>
  : Hibernate 외에 다른 구현체로 쉽게 교체하기 위함입니다. <br>
  내부에서 구현체 매핑을 지원해주기 때문에 새로운 JPA 구현체로 아주 쉽게 교체할 수 있습니다.

- 저장소 교체의 용이성 <br>
  : 관계형 데이터베이스 외에 다른 저장소로 쉽게 교체하기 위함입니다. <br>
  점점 트래픽이 많아져 관계형 데이터베이스로 도저히 감당이 안 될 때, 만약 MongoDB로 교체가 <br> 필요하다면 Spring Data JPA에서 Spring Data MongoDB로 의존성만 교체하면 됩니다. <br>
  → Spring Data의 하위 프로젝트들은 기본적인 CRUD의 인터페이스가 같기 때문입니다.

<br>

### # 실무에서 JPA

실무에서 JPA를 사용하지 못하는 가장 큰 이유는 높은 러닝 커브입니다. <br>
JPA를 잘 쓰려면 객체지향 프로그래밍과 관계형 데이터베이스를 둘 다 이해해야하기 때문입니다.

JPA에서는 여러 성능 이슈 해결책들을 이미 준비해놓은 상태이기 때문에 <br>
이를 잘 활용하면 네이티브 쿼리만큼의 퍼포먼스를 낼 수 있습니다.

<br>

### # 요구사항 분석

앞으로 만들 하나의 게시판의 요구사항은 다음과 같습니다.

1. 게시판 기능
   
   - 게시글 조회
   - 게시글 등록
   - 게시글 수정
   - 게시글 삭제

2. 회원 기능
   
   - 구글/네이버 로그인
   - 로그인한 사용자 글 작성 권한
   - 본인 작성 글에 대한 권한 관리

메인 화면과 글 등록, 글 수정 역시 별도의 화면과 기능을 제공합니다.

<br>

## 3.2. 프로젝트에 Spring Data Jpa 적용하기

### # 의존성 등록

1. `build.gradle`에 다음과 같이 의존성들을 등록합니다.
   
   ```java
   dependencies {
     compile('org.springframework.boot:spring-boot-starter-web')
     compileOnly('org.projectlombok:lombok')
     annotationProcessor('org.projectlombok:lombok')
     /* 추가된 코드 */
     compile('org.springframework.boot:spring-boot-starter-data-jpa')
     compile('com.h2database:h2')
     /*************/
     testCompile('org.springframework.boot:spring-boot-starter-test')
   }
   ```

- spring-boot-starter-data-jpa
  
  스프링 부트용 Spring Data Jpa 추상화 라이브러리입니다. <br>
  스프링 부트 버전에 맞춰 자동으로 JPA관련 라이브러리들의 버전을 관리해줍니다.

- h2
  
  인메모리 관계형 데이터베이스입니다. <br>
  별도의 설치가 필요 없이 프로젝트의 의존성만으로 관리할 수 있습니다. <br>
  메모리에서 실행되기 때문에 애플리케이션을 재시작할 때마다 초기화된다는 점을 <br>
  이용하여 테스트 용도로 많이 사용됩니다. <br>
  JPA의 테스트, 로컬 환경에서의 구동에서 사용할 예정입니다.

<br>

### # Posts 클래스 작성

1. `src/main/java` 경로에 생성한 패키지 하위에 `domain` 패키지를 생성합니다.
   
   > 도메인을 담을 패키지입니다. <br>
   > 여기서 도메인이란 게시글, 댓글, 회원, 정산, 결제 등 <br>
   > 소프트웨어에 대한 요구사항 혹은 문제 영역이라고 생각하면 됩니다.

2. 생성한 `domain` 패키지 아래에 `posts` 패키지와 `Posts` 클래스를 생성합니다.
   
3. `Posts` 클래스의 코드를 다음과 같이 작성합니다.
   
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
       this. content = content;
       this.author = author;
     }
   }
   ```

<details>
  <summary> 필자의 TIP (Click👆🏻)</summary>

  필자는 어노테이션 순서를 주요 어노테이션을 클래스에 가깝게 둡니다. <br>
  `@Entity`는 JPA의 어노테이션이며, `@Getter`와 `NoArgsConstructor`는 롬복의 어노테이션입니다. <br>
  롬복은 코드를 단순화시켜 주지만 필수 어노테이션은 아니므로, <br>
  코틀린 등의 새 언어 전환으로 롬복이 더이상 필요 없을 경우 쉽게 삭제할 수 있습니다.

</details>

Posts 클래스는 실제 DB의 테이블과 매칭될 클래스이며 보통 Entity 클래스라고 합니다. <br>
JPA를 사용하면 Entity 클래스의 수정을 통해 DB 데이터 작업을 합니다.

다음은 JPA에서 제공하는 어노테이션입니다.

- `@Entity`
  
  테이블과 링크될 클래스임을 나타냅니다. <br>
  기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍(_)으로 테이블 이름을 매칭합니다. <br>
  ex) SalesManager.java → sales_manager table

- `@Id`
  
  해당 테이블의 PK 필드를 나타냅니다.

- `@GeneratedValue`
  
  PK의 생성 규칙을 나타냅니다. <br>
  스프링 부터 2.0에서는 GenerationType.IDENTITY 옵션을 추가해야만 auto_increment가 됩니다.

- `@Column`
  
  테이블의 칼럼을 나타내며 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 칼럼이 됩니다. <br>
  사용하는 이유는 기본값 외에 추가로 변경이 필요한 옵션이 있으면 사용합니다. <br>
  문자열의 경우 VARCHAR(255)가 기본값인데, 사이즈를 늘리고 싶거나, 타입을 TEXT로 변경하고 싶은 등의 경우에 사용됩니다.

> 웬만하면 Entity의 PK는 Long 타입의 Auto_increment를 추천합니다. <br>
> (이렇게 하면 MySQL 기준으로 bigint 타입이 됩니다) <br>
> 주민등록번호와 같이 비즈니스상 유니크 키나, 여러 키를 조합한 복합키로 PK를 잡을 경우 <br>
> 1. FK를 맺을 때 다른 테이블에서 복합키 전부를 갖고 있거나, 중간 테이블을 하나 더 둬야하는 상황이 발생합니다. <br>
> 2. 인덱스에 좋은 영향을 끼치지 못합니다. <br>
> 3. 유니크한 조건이 변경될 경우 PK 전체를 수정해야 하는 일이 발생합니다. <br>
> 
> 따라서 주민등록번호, 복합키는 유니크 키로 별도로 추가하시는 것을 추천합니다.

다음은 롬복에서 재공하는 어노테이션입니다.

- `@NoArgsConstructor`
  
  기본 생성자 자동 추가 <br>
  `public Posts() {}`와 같은 효과

- `@Getter`
  
  클래스 내 모든 필드의 Getter 메소드를 자동생성

- `@Builder`
  
  해당 클래스의 빌더 패턴 클래스를 생성 <br>
  생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 
  
<details>
  <summary> ✨ Setter 메소드가 없는 이유 (Click👆🏻)</summary>

  <br>

  자바빈 규약을 생각하면서 getter/setter를 무작정 생성하는 경우가 있습니다. <br>
  이렇게 되면 해당 클래스의 인스턴스 값들이 언제 어디서 변해야 하는지 <br>
  코드상으로 명확하게 구분할 수가 없어, 차후 기능 변경 시 정말 복잡해집니다. <br>
  그래서 **Entity 클래스에서는 절대 Setter 메소드를 만들지 않습니다.** <br>
  대신, 해당 필드의 값 변경이 필요하면 명확히 그 목적과 의도를 나타낼 수 있는 메소드를 추가해야만 합니다. <br>

  ```java
  // 잘못된 사용
  public class Order {
    public void setStatus(boolean status) {
      this.status = status;
    }
  }

  public void 주문서비스의_취소이벤트() {
    order.setStatus(false);
  }

  // 올바른 사용
  public class Order {
    public void cancelOrder() {
      this.status = false;
    }
  }

  public void 주문서비스의_취소이벤트() {
    order.cancelOrder();
  }
  ```

  Setter가 없는 상황에서 어떻게 값을 채워 DB에 삽입을 하냐면, <br>
  기본적인 구조는 생성자를 통해 최종값을 채운 후 DB에 삽입하는 것이며, <br>
  값 변경이 필요한 경우 해당 이벤트에 맞는 public 메소드를 호출하여 변경하는 것을 전제로 합니다.

  생성자 대신에 `@Builder`를 통해 제공되는 빌더 클래스를 사용하는데, <br>
  생성자나 빌더나 생성 시점에 값을 채워주는 역할은 같습니다. <br>
  다만, 생성자의 경우 지금 채워야 할 필드가 무엇인지 명확이 지정할 수가 없습니다. <br>
  하지만 빌더의 경우 어느 필드에 어떤 값을 채워야 할지 명확하게 인지할 수 있습니다.

  ```java
  Example.builder()
    .a(a)
    .b(b)
    .build()
  ```

</details>

<br>

### # PostsRepository 인터페이스 작성

작성한 코드가 제대로 작동하는지 테스트를 하는데 <br>
WAS를 실행하지 않고, **테스트 코드로 검증**해 봅니다.


1. `src/test/java` 경로에 생성한 `posts` 패키지에 `PostsRepository` 인터페이스를 생성합니다.
   
   > 보통 ibatis나 Mybatis 등에서 Dao라고 불리는 DB Layer 접근자입니다. <br>
   > JPA에선 Repository라고 부르며 인터페이스로 생성합니다. <br>

2. `PostsRepository` 인터페이스의 코드를 다음과 같이 작성합니다.
   
   ```java
   import org.springframework.data.jpa.repository.JpaRepository;

   public interface PostsRepository extends JpaRepository<Posts, Long> {
   }
   ```

단순히 인터페이스 생성 후, `JpaRepository<Entity 클래스, PK 타입>`를 상속하면 <br>
기본적인 CRUD 메소드가 자동으로 생성됩니다. `@Repository`를 추가할 필요도 없습니다.

<details>
  <summary> 주의점 ‼ (Click👆🏻)</summary>

  <br>

  Entity 클래스와 기본 Entity Repository는 함께 위치해야 합니다. <br>
  둘은 아주 밀접한 관계이고, Entity 클래스는 기본 Repository 없이는 제대로 역할을 할 수가 없습니다. <br>
  나중에 프로젝트 규모가 커져 도메인별로 프로젝트를 분리해야 한다면 <br>
  Entity 클래스와 기본 Repository는 함께 움직여야 하므로 도메인 패키지에서 함께 관리합니다.


</details>

<br>

## 3.3. Spring Data JPA 테스트 코드 작성하기

### # PostsRepositoryTest 클래스 작성

1. `src/test/java` 경로 아래에 `domain.posts` 패키지를 추가합니다.

2. 생성한 패키지 아래에 `PostsRepositoryTest` 클래스를 생성합니다.

3. `PostsRepositoryTest` 클래스의 코드를 다음과 같이 작성합니다.
   
   ```java
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
       // given
       String title = "테스트 게시글";
       String content = "테스트 본문";

       postsRepository.save(Posts.builder()
                .title(title)
                .content(content)
                .author("yjeongisme@gmail.com")
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

별다른 설정 없이 `@SpringBootTest`를 사용할 경우 **H2 데이터베이스**를 자동으로 실행합니다.

- `@After`
  
  Junit에서 단위 테스트가 끝날 때마다 수행되는 메소드를 지정합니다. <br>
  보통은 배포 전 전체 테스트를 수행할 때 테스트간 데이터 침범을 막기 위해 사용합니다. <br>
  여러 테스트가 동시에 수행되면 테스트용 데이터베이스인 H2에 데이터가 그대로 남아있어 <br>
  다음 테스트 실행 시 테스트가 실패할 수 있습니다.

- `postsRepository.save`
  
  테이블 posts에 insert/update 쿼리를 실행합니다. <br>
  id 값이 있다면 update 쿼리, 없다면 insert 쿼리가 실행됩니다.

- `postsRepository.findAll`
  
  테이블 posts에 있는 모든 데이터를 조회해오는 메소드입니다.

<br>

### # 실행된 쿼리 확인

실제로 실행된 쿼리의 형태를 확인하기 위해 쿼리를 로그로 확인할 수 있습니다. <br>
쿼리 로그를 ON/OFF 할 수 있는 설정이 있으며, Java 클래스로 구현할 수 있습니다. <br>
스프링 부트에서는 `application.properties`, `application.yml` 등의 파일로 <br>
한 줄의 코드를 설정할 수 있도록 지원하고 권장합니다.

1. `src/main/resources` 경로에 `application.properties` 파일을 생성합니다.
   
2. `application.properties` 파일의 코드를 다음과 같이 작성합니다.
   
   ```java
   spring.jpa.show_sql=true
   ```

create table 쿼리를 확인해보면 `id bigint generated by default as identity`라는 옵션으로 <br>
생성되는데, 이는 H2의 쿼리 문법이 적용되었기 때문입니다. <br>
H2는 MySQL의 쿼리를 수행해도 정상 작동하기 때문에 MySQL 버전으로 변경해 봅니다.

3. 마찬가지로 `application.properties` 파일의 코드를 다음과 같이 추가 작성합니다.

   ```java
   spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
   ```

`id bigint not null auto_increment`라는 옵션이 적용된 것을 확인할 수 있습니다.

<br>

## 3.4. 등록/수정/조회 API 만들기

<br>

### 3.4. 궁금한점

<br>

## 3.5. JPA Auditing으로 생성시간/수정시간 자동화하기

<br>

## 궁금한 점 스스로 찾아보기

<br>

## 회고
