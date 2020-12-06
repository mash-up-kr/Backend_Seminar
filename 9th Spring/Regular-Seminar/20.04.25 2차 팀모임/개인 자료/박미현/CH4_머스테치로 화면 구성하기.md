#4. 머스테치로 화면 구성하기

###1. 서버 템플릿 엔진과 머스테치
#####1.1. 템플릿엔진이란?

-템플릿엔진 : 지정된 템플릿 양식과 데이터가 합쳐져 HTML문서를 출력하는 SW

- 서버 템플릿 엔진

    - 서버에서 구동됨. (서버에서 Java코드로 문자열을 만들고 이 문자열을 HTML로 변환하여 브라우저로 전달)
    - eg) JSP, Freemarker

- 클라이언트 템플릿 엔진

    - 브라우저 위에서 작동->브라우저에서 작동될 때는 서버에서 제어 불가
    - 서버에서는 Json, Xml형식의 데이터만 전달, 클라이언트에서 조립
    - 리액트, 뷰(Vue)

#####1.2. 머스테치
많은 언어를 지원하는 심플한 템플릿 엔진으로 자바에서 사용될 때는 서버 템플릿 엔진으로, 
js에서 사용될 때는 클라이언트 템플릿 엔진으로 사용 가능.     
로직코드를 사용할 수 없어 View의 역할과 서버의 역할을 명확하게 분리.

***
###2. 기본 페이지 만들기
#####2.1. 의존성 등록

- build.gradle
```java
dependencies {
    compile('org.springframework.boot:spring-boot-starter-mustache')  //mustache
}
```    

#####2.2. 기본 페이지

`머스테치 파일의 위치는 src/main/resources/templates`
- src/main/resources/templates/index.mustache 
```html
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링부트 웹서비스</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

</head>
<body>
    <h1>스프링부트로 시작하는 웹 서비스</h1>
</body>
</html>
```

-  src/main/java/com/jojoldu/book/springboot/web/IndexController.java 
    - 머스테치에 URL매핑
```java
package com.jojoldu.book.springboot.web;


import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class IndexController {

    @GetMapping("/")
    public String index() {
        return "index";
    }
}
```
머스테치 스타터로 인해 문자열 반환 시 앞의 경로(src/main/resources/templates)와 
뒤의 파일 확장자(.mustache)자동 지정.     
위의 코드를 예로 src/main/resources/templates/index.mustache로 전환하여 ViewResolver가 처리

-  src/test/java/com/jojoldu/book/springboot/web/IndexControllerTest.java
     - 테스트 코드
```java
package com.jojoldu.book.springboot.web;


import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.boot.test.context.SpringBootTest.WebEnvironment.RANDOM_PORT;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = RANDOM_PORT)
public class IndexControllerTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void 메인페이지_로딩() {
        //when
        String body = this.restTemplate.getForObject("/", String.class);

        //then
        assertThat(body).contains("스프링부트로 시작하는 웹 서비스");
    }
}
```
__*restTemplate.getForObject()*__
:TestRestTemplate통해 지정한 URL주소로, HTTP GET메서드로, 객체로, 결과를 반환받는다 


###3. 게시글 등록 화면
#####3.1. 레이아웃 방식으로 외부 CDN사용하기
- 레이아웃 방식 : 공통 영역을 별도의 파일로 분리하여 필요한 곳에서 가져다 쓰는 방식
- 페이지 로딩속도를 높이기 위해 css는 header에, js는 footer에.(js용량이 클수록 body의 실행이 늦어져 화면이 다 그려진 후에 js호출하도록)

 - src/main/resources/templates/layout/header.mustache
```html
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링부트 웹서비스</title>
    <meta http-equiv="Content-Type" content="text/html; charset=URF-8"/>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
</head>
<body>
```
   - src/main/resources/templates/layout/footer.mustache
```html
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>

<!--index.js 추가-->
<script src="/js/app/index.js"></script>
</body>
</html>
```   

`스프링 부트는 src/main/resources/static에 위치한 js, css, 이미지 등의 정적
파일은 URL에서 /로 설정된다.`

- src/main/resources/templates/index.mustache
```html
{{>layout/header}}

<h1>스프링부트로 시작하는 웹 서비스</h1>
<div class="col-md-12">   //12칸짜리 1개
    <div class="row">
        <div class="col-md-6">  //6칸짜리 2개
            <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
        </div>
    </div>
</div>

{{>layout/footer}}
```
글 등록 버튼 생성
#####3.2. 등록 화면

- src/main/java/com/jojoldu/book/springboot/web/IndexController.java 
```java

    @GetMapping("/posts/save")
    public String postsSave() {
        return "posts-save";
    }
}
```
-  src/main/resources/templates/posts-save.mustache
```html
{{>layout/header}}

<h1>게시글 등록</h1>

<div class="col-md-12">
    <div class="col-md-4">
        <form>
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control" id="title" placeholder="제목을 입력하세요">
            </div>
            <div class="form-group">
                <label for="author">작성자</label>
                <input type="text" class="form-control" id="author" placeholder="작성자를 입력하세요">
            </div>
            <div class="form-group">
                <label for="content">내용</label>
                <textarea class="form-control" id="content" placeholder="내용을 입력하세요"></textarea>
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secondary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-save">등록</button>
    </div>
</div>

{{>layout/footer}} 
```
__*\<label\>*__
:\<input\>디자인이 힘들 때 \<label\>태그로 연결하여 디자인 도움주고 클릭 편의성 높임.
for 속성을 이용하여 \<input\>태그의 id속성에 연계하여 사용
(label태그의 for값과 input태그의 id일치)

- src/main/resources/static/js/app/index.js
```jshint
var index = {
    init : function () {
        var _this = this;
        $('#btn-save').on('click', function () {
        _this.save();
        });
    },
        save : function () {
            var data = {
            title: $('#title').val(),
            author: $('#author').val(),
            content: $('#content').val()
        };

        $.ajax({
            type: 'POST',
            url: '/api/v1/posts',
            dataType: 'json',
            contentType: 'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 등록되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    }
}

index.init();
```
` js파일 각각만의 유효범위를 만들어 사용하기 위해 객체를 만들어 해당 객체 안에서 필요한 모든 function을 선언 → 중복된 함수명으로 인한 문제 해결`

###4. 전체 조회 화면
- src/main/resources/templates/index.mustache 수정
```html
{{>layout/header}}
<h1>스프링부트로 시작하는 웹 서비스</h1>
<div class="col-md-12">
    <div class="row">
        <div class="col-md-6">
            <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
        </div>
    </div>
</div>
<br>
<!-- 목록 출력 영역-->
<table class="table table-horizontal table-bordered">
    <thread class="thread-strong">
        <tr>
            <th>게시글번호</th>
            <th>제목</th>
            <th>작성자</th>
            <th>최종수정일</th>
        </tr>
    </thread>
    <tbody id="tbody">
    {{#posts}}
        <tr>
            <td>{{id}}</td>
            <td>{{title}}</td>
            <td>{{author}}</td>
            <td>{{modifiedDate}}</td>
        </tr>
    {{/posts}}
    </tbody>
</table>

{{>layout/footer}}
```
__*html 표 만들 때 사용하는 태그*__
- \<table\> : 표 만들기
- \<tr\> : 행
- \<th\> : 열의 제목(테이블의 헤더)
- \<td\> : 내용
- \<thead\> : 테이블 제목
- \<tbody\> : 테이블 내용
<br>
<br>

-Controller, Service, Repository코드 작성
- Repository
    - src/main/java/com/jojoldu/book/springboot/domain/posts/PostsRepository.java 인터페이스 쿼리 추가
```java
public interface PostsRepository extends JpaRepository<Posts, Long> {
    @Query("SELECT p FROM Posts p ORDER BY p.id DESC")
    List<Posts> findAllDesc();
}
```
SpringDataJpa에서 제공하지 않는 메소드는 쿼리로 작성 가능하다.     <br>

`규모가 큰 프로젝트에서 데이터 조회는 조회용 프레임워크(querydsl 등)를 추가로 사용한다.
조회는 조회용 프레임워크에서, 등록/수정/삭제는 SpringDataJpa로 진행한다.`

- Service
  -  src/main/java/com/jojoldu/book/springboot/service/posts/PostsService.java 코드 추가
```java

    @Transactional(readOnly = true)
    public List<PostsListResponseDto> findAllDesc() {
        return postsRepository.findAllDesc().stream()
                .map(PostsListResponseDto::new)
                .collect(Collectors.toList());
    }
```
__*@Transactional(readOnly=true)*__
:옵션으로 (readOnly=true)를 주면 트랜잭션 범위는 유지하되, 
조회기능만 남겨두어 속도 개선 효과. 등록/수정/삭제 기능이 없는 메소드에서 추천.

- Dto
  - src/main/java/com/jojoldu/book/springboot/web/dto/PostsListResponseDto.java 
```java
package com.jojoldu.book.springboot.web.dto;

import com.fasterxml.jackson.annotation.JsonFormat;
import com.jojoldu.book.springboot.domain.posts.Posts;
import lombok.Getter;

import java.time.LocalDate;
import java.time.LocalDateTime;

@Getter
public class PostsListResponseDto {
    private Long id;
    private String title;
    private String author;

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern="yyyy-MM-dd HH:mm:ss", timezone = "Asia/Seoul")
    private LocalDateTime modifiedDate;

    public PostsListResponseDto(Posts entity) {
        this.id = entity.getId();
        this.title = entity.getTitle();
        this.author = entity.getAuthor();
        this.modifiedDate = entity.getModifiedDate();
    }
}
```

- Controller
   - src/main/java/com/jojoldu/book/springboot/web/IndexController.java 수정
```java
@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;

    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("posts", postsService.findAllDesc());
        return "index";
    }
```
__*Model*__
:서버 템플릿 엔진에서 사용할 수 있는 객체 저장
(위의 코드에서는 Service에서 가져온 결과를 posts로 index.mustache에 전달)

###5. 수정, 삭제 화면

- src/main/resources/templates/layout/posts-update.mustache 
```html
{{>layout/header}}
<h1>게시글 수정</h1>
<div class="col-md-12">
    <div class="col-md-4">
        <form>
            <div class="form-group">
                <label for="id">글 번호</label>
                <input type="text" class="form-control"
                       id="id" value="{{post.id}}" readonly>
            </div>
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control"
                       id="title" value="{{post.title}}">
            </div>
            <div class="form-group">
                <label for="author">작성자</label>
                <input type="text" class="form-control"
                       id="author" value="{{post.author}}" readonly>
            </div>
            <div class="form-group">
                <label for="content">내용</label>
                <textarea class="form-control" id="content">{{post.content}}</textarea>
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secondary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-update">수정 완료</button>
        <button type="button" class="btn btn-danger" id="btn-delete">삭제</button>
    </div>
</div>
```

- src/main/resources/static/js/app/index.js 코드 추가 
```jshint
var main = {
    init : function () {
        $('#btn-update').on('click', function () {
        _this.update();
        });
        $('#btn-delete').on('click', function () {
        _this.delete();
        });
    },
    update : function () {
        var data = {
            title: $('#title').val(),
            content: $('#content').val()
        };
        var id = $('#id').val();
        $.ajax({
            type: 'PUT',
            url: '/api/v1/posts/'+id,
            dataType:'json',
            contentType: 'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 수정되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    },
    delete : function () {
        var id = $('#id').val();

        $.ajax({
            type: 'DELETE',
            url: '/api/v1/posts/'+id,
            dataType: 'json',
            contentType: 'application/json; charset=utf-8'
        }).done(function () {
            alert('글이 삭제되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
```
__*type:*__
Controller에 있는 API에서 선언한 REST 그대로 사용

- CRUD-HTTP Method 매핑
   - 생성(Create) - POST
   - 읽기(Read) - GET
   - 수정(Update) - PUT
   - 삭제(Delete) - DELETE

- src/main/resources/templates/index.mustache 수정 a태그 추가
```html
<td><a href="/posts/update/{{id}}">{{title}}</a></td>
```

-Controller
- src/main/java/com/jojoldu/book/springboot/web/IndexController.java
```java
    @GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model) {
        PostsResponseDto dto = postsService.findById(id);
        model.addAttribute("post", dto);

        return "posts-update";
    }
```

-Service
-  src/main/java/com/jojoldu/book/springboot/service/posts/PostsService.java 삭제기능 추가
```java
    @Transactional
    public void delete (Long id) {
        Posts posts = postsRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 사용자가 없습니다. id=" + id));

        postsRepository.delete(posts);
    }
```
__*Repository.delete()*__
:JpaRepository에서 delete메소드 지원. 엔티티를 파라미터로 삭제, deleteById 메소드 이용하면 id로 바로 삭제 가능

-Controller
```java
    @DeleteMapping("/api/v1/posts/{id}")
    public Long delete(@PathVariable Long id) {
        postsService.delete(id);
        return id;
    }
```

***
###. 더 알아보기
#####.1. View Resolver
URL 요청의 결과를 전달할 타입과 값을 지정하는 관리자 격
    

    