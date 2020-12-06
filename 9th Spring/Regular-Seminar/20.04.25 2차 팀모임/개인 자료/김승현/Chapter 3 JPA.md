# Chapter 3: 스프링 부트에서 JPA로 데이터베이스 다뤄 보자

Created: Mar 24, 2020 10:21 PM
Tags: Builder, Entity, JPA, Repository

어떻게 하면 관계형 데이터베이스를 이용하는 프로젝트에서 객체 지향 프로그래밍을 할 수 있을까?

- **JPA**라는 자바 표준 ORM(Object Relational Mapping) 기술
- MyBatis, iBatis는 SQL Mapper!
- ORM은 객체를 매핑하고, SQL Mapper는 쿼리를 매핑한다.

## 3.1 JPA 소개

- 현대의 웹 애플리케이션에서 관계형 데이터베이스는 빠질 수 없는 요소! → **객체를 관계형 데이터베이스에서 관리**하는 것이 중요하다.
- 관계형 데이터베이스를 사용해야만 하는 상황에서 SQL은 피할 수 없다. 관계형 데이터베이스가 SQL만을 인식할 수 있기 때문이다.
    - 관계형 데이터베이스는 어떻게 데이터를 저장할지에 초점이 맞춰진 기술인데 반해, 프로그래밍 언어는 기능과 속성을 한 곳에서 관리하는 기술이다. → 둘의 패러다임이 일치하지 않는 문제가 발생
    - JPA는 이를 해결하기 위해 등정하게 된다. → 둘의 패러다임을 중간에서 일치시키기
- JPA를 사용하면 개발자는 **객체 지향적으로 프로그래밍을 하고** JPA가 이를 관계형 데이터베이스에 맞게 SQL을 대신 생성해서 실행한다. → SQL에 종속적인 개발에서 벗어나게 된다.

### Spring Data JPA

- JPA는 인터페이스로서 자바 표준 명세서이다. 인터페이스인 JPA를 사용하기 위해서는 구현체가 필요하다.
    - 대표적으로 Hibernate, Eclipse Link 등이 있다.
    - 이들을 쉽게 사용하고자 추상화시킨 **Spring Data JPA**라는 모듈을 이용하여 JPA 기술을 다룬다.
- JPA ← Hibernate ← Spring Data JPA
- Spring Data JPA가 등장한 이유에는 크게 두 가지가 있다.
    - 구현체 교체의 용이성
        - Hibernate 외에 다른 구현체로 쉽게 교체하기 위함 ← Spring Data JPA 내부에서 구현체 매핑을 지원해 주기 때문에 쉽게 교체할 수 있다.
    - 저장소 교체의 용이성
        - 관계형 데이터베이스 외에 다른 저장소로 쉽게 교체하기 위함 ← 가령, MongoDB로 교체가 필요하다면 Spring Data JPA에서 Spring Data MongoDB로 의존성만 교체하면 된다.
        - 이는 Spring Data의 하위 프로젝트들은 기본적인 **CRUD 인터페이스 같기** 때문이다. 그렇기 때문에 저장소가 교체되어도 기본적인 기능은 변경할 것이 없다.

## 3.2 프로젝트에 Spring Data Jpa 적용하기

- `build.gradle` 에 의존성 추가하기

```java
dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.projectlombok:lombok')
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
    compile('com.h2database:h2')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

- spring-boot-starter-data-jpa
    - 스프링 부트용 Spring Data Jpa 추상화 라이브러리
    - 스프링 부트 버전에 맞추어 자동으로 JPA 관련 라이브러리들의 버전을 관리해 줍니다.
- h2
    - 인메모리 RDB
    - 설치 필요 없이 프로젝트 의존성만으로 관리할 수 있다.
    - 메모리에서 실행되기 때문에 애플리케이션을 재시작할 때마다 초기화된다는 점을 이용하여 테스트 용도로 많이 사용한다.

- domian 패키지 생성 → **도메인을 담을 패키지**
    - 도메인이란 게시 글, 댓글, 회원, 정산, 결제 등 소프트웨어 대한 요구사항 혹은 문제 영역이다.
- domain 패키지에 **posts 패키지와 Posts 클래스** 생성

```java
package example.org.practice.domain.posts;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

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

}
```

- **주요 어노테이션을 클래스에 가깝게**
- Posts 클래스는 실제 DB의 테이블과 매칭될 클래스이며 Entity 클래스라고도 한다.
- `@Entity`
    - 테이블과 링크될 클래스임을 나타낸다.
    - 기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍으로 테이블 이름을 매칭한다. (ex. [SalseManager.java](http://salsemanager.java) → salse_manager table)
- `@Id`
    - 해당 테이블의 PK 필드를 나타낸다.
    - Long 타입의 auto_increment가 좋다. 유니크 키가 여러 키를 조합한 복합키를 PK로 잡을 경우 문제가 발생할 수 있다.
        - FK를 맺을 때 다른 테이블에서 복합키 전부를 갖고 있거나 중간 테이블을 하나 더 둬야 하는 상황이 발생한다.
        - 인덱스에 좋은 영향을 끼치지 못한다.
        - 유니크한 조건이 변경될 경우 PK 전체를 수정해야 하는 일이 발생한다.
- `@GeneratedValue`
    - PK의 생성 규칙을 나타낸다.
    - 스프링 부트 2.0에서는 GenerationType.IDENTITY 옵션을 추가해야만 auto_increment가 된다.
- `@Column`
    - 테이블의 칼럼을 나타내며 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 칼럼이 된다.
    - 기본값 외에 추가로 변경이 필요한 옵션이 있으면 사용한다.

롬복 라이브러리의 어노테이션들

- `@NoArgsConstructor`
    - 기본 생성자 자동 추가
    - `public Posts() {}`  같은 효과
- `@Getter`
    - 클래스 내 모든 필드의 Getter 메소드를 자동 생성 → 선언된 모든 필드의 get 메소드 생성
- `@Builder`
    - 해당 클래스의 빌더 패던 클래스를 생성
    - 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함

- getter/setter를 무작정 생성하면 해당 클래스의 인스턴스 값들이 언제 어디서 변화하는지 코드상으로 명확하게 구분할 수 없어, 차후 기능 변성 시 복잡해진다. → **Entitiy 클래스에서는 절대 Setter 메소드를 만들지 않는다.**
    - 해당 필드의 값 변경이 필요하면 명확히 목적과 의도를 나타낼 수 있는 메소드를 추가해야 한다.
- Setter가 없는 상황에서 어떻게 값을 채워 DB에 삽입할까?
    - 기본적인 구조는 **생성자를 통해** 최종값을 채운 후 DB에 삽입 → 책에서는 생성자 대신 **@Builder를 통해 제공되는 빌더 클래스** 사용
        - **어느 필드에 어떤 값을 채워야 할지 명확하게 인지** 가능
    - 값 변경이 필요한 경우 해당 이벤트에 맞는 **public 메소드를 호출**

- Posts 클래스로 데이터베이스를 접근(DB layer 접근)하게 해 줄 JpaRepository 생성

```java
package example.org.practice.domain.posts;

import org.springframework.data.jpa.repository.JpaRepository;

public interface PostsRepository extends JpaRepository<Posts, Long> {
}
```

- 인터페이스 생성 후 `JpaRepository<Entity 클래스, PK 타입>`을 상속하면 기본적인 CRUD 메소드가 자동으로 생성된다. → `@Respository`를 추가할 필요 X
- **Entity 클래스와 기본 Entity Repository는 함께 위치**해야 → Entity 클래스는 기본 Repository 없이는 제대로 역할을 할 수 없다!
- 프로젝트 규모가 커져 도메인별로 프로젝트를 분리해야 한다면 이때 Entiy 클래스와 기본 Repository는 함께 움직여야 하므로 **도메인 패키지에서 함께 관리**