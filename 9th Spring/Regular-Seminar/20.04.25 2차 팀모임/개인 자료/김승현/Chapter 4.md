# Chapter 4: 머스테치로 화면 구성하기

Created: Apr 03, 2020 9:57 PM
Tags: mustache, template

## 4.1 서버 템플릿 엔진과 머스테치 소개

- 템플릿 엔진이란?
    - 웹 개발에서 템플릿 엔진이란 **지정된 템플릿 양식과 데이터**가 합쳐져 HTML 문서를 출력하는 소프트웨어를 말한다. → 웹 사이트의 화면을 어떤 형태로 만들지 도와주는 양식
    - ex) JSP, Freemaker, React와 Vue의 View 파일 등

- 서버 템플릿 엔진과 클라이언트 템플릿 엔진

```jsx
<script type="text/javascript>

$(document).ready(function(){
	if(a=="1"){
	<%
			System.out.println("test");
	%>
	}
});
```

- 위의 코드는 if문과 관계없이 무조건 test을 콘솔에 출력한다.
    - 프론트엔드의 자바스크립트가 작동하는 영역과 JSP가 작동하는 영역이 다르기 때문인데, JSP를 비롯한 서버 템플릿 엔진은 **서버에서 구동**된다.
- 서버 템플릿 엔진을 이용한 화면 생성은 **서버에서 Java 코드로 문자열**을 만든 뒤 문자열을 HTML로 변환하여 **브라우저로 전달**한다. 앞선 코드는 HTML을 만드는 과정에서 `System.out.println("test");`를 실행할 뿐, 자바스크립트 코드는 이때 **단순한 문자열**이다.
- 반면 자바스크립트는 **브라우저 위에서 작동**한다.
    - 작성된 자바스크립트 코드가 실행되는 장소는 **브라우저**이다.
- Vue.js나 React.js를 이용한 Single Page Application은 **브라우저에서 화면을 생성**한다. 서버에서 이미 코드가 벗어난 경우이다.
    - 서버에서는 Json 혹은 Xml 형식의 데이터만 전달하고 클라이언트에서 조립한다.

### 머스테치란

- 여러 언어를 지원하는 가장 간단한 템플릿 엔진
    - 문법이 간단하다.
    - 로직 코드를 사용할 수 없어 View의 역할과 서버의 역할이 명확하게 분리된다.
    - Mustache.js와 [Mustache.java](http://mustache.java) 두 가지가 전부 있어 하나의 문법으로 클라이언트와 서버 템플릿을 모두 사용 가능하다.

## 4.2 기본 페이지 만들기

- 의존성 추가

```java
compile('org.springframework.boot:spring-boot-starter-mustache')
```

- 머스테치 파일 위치는 기본적으로 `src/main/resources/templates`
    - 이 위치에 머스테치 파일을 두면 스프링 부트에서 자동으로 로딩한다.
- 첫 페이지를 담당할 index.mustache를 해당 위치에 생성
- URL을 매핑 → URL 매핑은 Controller에서 진행한다.
- web 패키지 안에 IndexController 생성

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@RequiredArgsConstructor
@Controller
public class IndexController {

    @GetMapping("/")
    public String index() {
        return "index";
    }
}
```

- 머스테치 스타터 덕분에 컨트롤러에서 문자열을 반환할 때 **앞의 경로와 뒤의 파일 확장자는 자동으로 지정**
    - "index"를 반환하므로 src/main/resources/templates/index.mustache로 전환되어 View Resolver가 처리

- 테스트 코드

```java
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
        assertThat(body).contains("스프링 부트로 시작하는 웹 서비스");
    }
}
```

- 실제로 URL 호출 시 페이지의 내용이 제대로 호출되는지에 대한 테스트
    - `TestRestTemplate`를 통해 "/"로 호출했을 때 index.mustache에 포함된 코드들이 있는지 확인한다.

## 4.3 게시글 등록 화면 만들기

- 부트스트랩 사용
- 프론트엔드 라이브러리(부트스트랩, 제이쿼리 등)를 사용하는 방법에는 크게 2가지가 있다.
    - **외부 CDN(Content Delivery Network)**
    - 직접 라이브러리를 받아서 사용
- 레이아웃 방식으로 부트스트랩과 제이쿼리 추가
    - **공통 영역을 별도의 파일로 분리하여 필요한 곳에 가져다 쓰는 방식**
    - 라이브러리들이 **머스테치 화면 어디에서나 필요**하기 때문

- `src/main/resources/templates` 디렉토리에 layout 디렉토리 추가로 생성 후 `footer.mustache`와 `header.mustache` 추가
    - **페이지 로딩 속도를 높이기** 위해 css는 header에, js는 footer에 둔다. → HTML을 위에서부터 코드가 실행되므로 **head가 다 실행되고 나서야 body가 실행**
    - 부트스트랩의 경우 제이쿼리가 있어야만 하기 때문에 제이쿼리가 먼저 호출된다.
    - 라이브러리를 비롯한 기타 HTML 태그들이 모두 레이아웃에 추가되어 index.mustache에는 필요한 코드만 남게 된다.

- 글 등록 버튼 추가
- 이동할 페이지의 주소에 해당하는 컨트롤러 추가
    - 페이지에 관련된 컨트롤러는 모두 IndexController 사용

```java
@RequiredArgsConstructor
@Controller
public class IndexController {

    ...

    @GetMapping("/posts/save")
    public String postsSave() {
        return "posts-save";
    }
}
```

- index.mustache와 같은 위치에 posts-save.mustache 생성

- 버튼을 기능하게 하기 위해 `src/main/resources` 에 `static/js/app` 디렉토리 생성
    - index.js 생성

```jsx
var main = {
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
        }).done(function () {
            alert('글이 등록되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        });
    }

};

main.init();
```

- `var main = { ... }` 이라는 코드를 선언한 이유?
    - 브라우저의 스코프는 **공용 공간**으로 쓰이지 때문에 나중에 로딩된 js의 동일한 이름의 함수가 먼저 로딩된 js의 함수를 **덮어쓰게 된다**.
    - 여러 사람이 참여하는 프로젝트에서는 중복된 함수 이름은 자주 발생할 수 있기 때문에, 이런 문제를 피하기 위해 index.js만의 유효 범위를 만들어 사용한다.
        - 방법은 `var main`이라는 객체를 만들어 해당 객체에서 필요한 모든 function을 선언하는 것이다. 이렇게 하면 이 객체 안에서만 function이 유효하다.

- 생성된 index.js를 머스테치 파일이 쓸 수 있게 footer.mustache에 추가

```jsx
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>

<!--index.js 추가-->
<script src="/js/app/index.js"></script>
</body>
</html>
```

- index.js 호출 코드는 **절대 경로**로 시작
- 스프링 부트는 기본적으로 src/main/resources/static에 위치한 자바스크립트, CSS, 이미지 등 정적 파일들은 URL에서 /로 설정된다.

## 4.4 전체 조회 화면 만들기

- index.mustache 변경

```jsx
{{>layout/header}}

    <h1>스프링 부트로 시작하는 웹 서비스</h1>
    <div class="col-md-12">
        <!-- 로그인 기능 영역 -->
        <div class="row">
            <div class="col-md-6">
                <a href="posts/save" role="button" class="btn btn-primary">글 등록</a>
            </div>
        </div>
        <br>
        <!-- 목록 출력 영역 -->
        <table class="table table-horizontal table-bordered">
            <thead class="thead-strong">
            <tr>
                <th>게시글번호</th>
                <th>제목</th>
                <th>작성자</th>
                <th>최종수정일</th>
            </tr>
            </thead>
            <tbody id="tbody">
            {{#posts}}
                <tr>
                    <td>{{id}}</td>
                    <td><a href="/posts/update/{{id}}">{{title}}</a></td>
                    <td>{{author}}</td>
                    <td>{{modifiedDate}}</td>
                </tr>
            {{/posts}}
            </tbody>
        </table>
    </div>

{{>layout/footer}}
```

- `{{#posts}}`
    - posts라는 List를 순회
- `{{id}}` 등의 `{{변수명}}`
    - List에서 뽑아낸 객체의 필드 사용

- PostsRepository 인터페이스에 쿼리 추가

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;

import java.util.List;

public interface PostsRepository extends JpaRepository<Posts, Long> {
    @Query("SELECT p FROM Posts p ORDER BY p.id DESC")
    List<Posts> findAllDesc();
}
```

- SpringDataJpa에서 제공하지 않는 메소드는 쿼리로 작성할 수 있다.
- Entity 클래스만으로 데이터 조회를 처리하기 어려운 경우 조회용 프레임워크를 추가로 사용한다.
    - Querydsl을 사용하면 타입 안정성을 보장할 수 있다. 또한 많은 레퍼런스가 있다.

```java
//  PostsService

import java.util.List;
import java.util.stream.Collectors;

@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    ...

    @Transactional(readOnly = true)
    public List<PostsListResponseDto> findAllDesc() {
        return postsRepository.findAllDesc().stream()
                    .map(PostsListResponseDto::new)
                    .collect(Collectors.toList());
    }
}
```

- 트랜잭션 어노테이션에 `readOnly = true`를 주면 **트랜잭션 범위**는 유지하되, 조회 기능만 남겨서 **조회 속도가 개선**된다.
- `.map(PostsListResponseDto::new)`
    - postsRepository 결과로 넘어 온 Posts의 Stream을 map을 통해 PostsListResponseDto로 변환 → List로 반환하는 메소드이다.

```java
// PostsListResponseDto

import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Getter;
import java.time.LocalDateTime;

@Getter
public class PostsListResponseDto {
    private Long id;
    private String title;
    private String author;
    private LocalDateTime modifiedDate;

    public PostsListResponseDto(Posts entity) {
        this.id = entity.getId();
        this.title = entity.getTitle();
        this.author = entity.getAuthor();
        this.modifiedDate = entity.getModifiedDate();
    }
}
```

- Controller 변경

```java
// IndexController

@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;

    @GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user) {
        model.addAttribute("posts", postsService.findAllDesc());
        return "index";
    }
}
```

- `Model`
    - 서버 템플릿 엔진에서 사용할 수 있는 객체를 저장할 수 있다.
    - `postsService.findAllDesc()`로 가져온 결과를 posts로 index.mustache에 전달한다.

## 4.5 게시글 수정, 삭제 화면 만들기

- posts-update.mustache 파일 생성

```jsx
{{>layout/header}}

<h1>게시글 수정</h1>

<div class="col-md-12">
    <div class="col-md-4">
        <form>
            <div class="form-group">
                <label for="title">글 번호</label>
                <input type="text" class="form-control" id="id" value="{{post.id}}" readonly>
            </div>
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control" id="title" value="{{post.title}}">
            </div>
            <div class="form-group">
                <label for="author"> 작성자 </label>
                <input type="text" class="form-control" id="author" value="{{post.author}}" readonly>
            </div>
            <div class="form-group">
                <label for="content"> 내용 </label>
                <textarea class="form-control" id="content">{{post.content}}</textarea>
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secondary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-update">수정 완료</button>
        <button type="button" class="btn btn-danger" id="btn-delete">삭제</button>
    </div>
</div>

{{>layout/footer}}
```

- `{{post.id}}`
    - 객체의 필드 접근 시 점으로 구분
- `readonly`
    - input 태그에 읽기 가능한 허용하는 속성

- index.js 파일에 update function 추가

```jsx
var main = {
    init : function () {
        var _this = this;
        $('#btn-save').on('click', function () {
            _this.save();
        });

        $('#btn-update').on('click', function () {
            _this.update();
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
        }).done(function () {
            alert('글이 등록되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
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
            dataType: 'json',
            contentType: 'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function () {
            alert('글이 수정되었습니다.');
            window.location.href = '/';
        }).fail(function (error) {
            alert(JSON.stringify(error));
        })
    }
};

main.init();
```

- `$'(#btn-update').on('click')`
    - btn-update란 id를 가진 HTML 엘리먼트에 click 이벤트가 발생할 때 update function을 실행하도록 한다.
- `type: 'PUT'`
    - 여러 HTTP method 중 PUT 메소드를 선택한다.
    - PostsApiController에 있는 API에서 이미 `@PutMapping`으로 선언했기 때문에 PUT을 사용해야 한다. 이는 REST 규약에 맞게 설정된 것이다.
    - REST에서 CRUD는 다음와 같이 HTTP method에 매핑된다.
        - 생성(create): POST
        - 읽기(read): GET
        - 수정(update): PUT
        - 삭제(delete): DELETE
- `url: '/api/v1/posts/'+id`
    - 어느 게시글을 수정할지 URL Path로 구분하기 위해 Path에 id를 추가한다.

- IndexController에 메소드 추가

```java
@RequiredArgsConstructor
@Controller
public class IndexController {

    ...

    @GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model) {
        PostsResponseDto dto = postsService.findById(id);
        model.addAttribute("post", dto);

        return "posts-update";
    }

}
```

- 게시글 삭제도 마찬가지로 삭제 이벤트를 진행할 js 코드를 추가해 준다.
- 삭제 API 추가

```java
// PostsService

@RequiredArgsConstructor
@Service
public class PostsService {
    
		...

    @Transactional
    public void delete (Long id) {
        Posts posts = postsRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id=" + id));

        postsRepository.delete(posts);
    }
}
```

- `postsRepository.delete(posts)`
    - JpaRepository에서 이미 delete 메소드를 지원하고 있으므로 활용한다.
    - 엔티티를 파라미터로 삭제할 수도 있고, `deleteById` 메소드를 이용하면 id로 삭제할 수 있다.
    - 존재하는 Posts 인지 확인하기 위해 엔티티 조회 후 삭제한다.

```java
// PostsApiController

@RequiredArgsConstructor
@RestController
public class PostsApiController {

    ...

    @DeleteMapping("/api/v1/posts/{id}")
    public Long delete(@PathVariable Long id) {
        postsService.delete(id);
        return id;
    }

}
```