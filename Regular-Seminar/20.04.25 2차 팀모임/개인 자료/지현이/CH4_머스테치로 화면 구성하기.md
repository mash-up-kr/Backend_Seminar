# 4. 머스테치로 화면 구성하기

## 진도 기록

| 공부 날짜  | 공부한 페이지 수 |
| ---------- | ---------------- |
| 2020.04.21 | 125p ~ 161p        |

## 4.1. 서버 템플릿 엔진과 머스테치 소개

**템플릿 엔진이란?**

지정된 템플릿 양식과 데이터가 합쳐서 HTML문서를 출력하는 소프트웨어이다.

**머스테치**

하나의 템플릿 엔진으로, 다양한 언어를 지원한다!

> 루비, 자바스크립트, 파이썬, PHP, 자바 등 

다양한 템플릿 엔진이 존재하는데 머스테치를 사용하는 이유는 무엇일까?

- 문법이 다른 템플릿 엔진보다 간단하다.
- 로직 코드를 사용할 수 없어 View와 서버의 역할을 확실히 한다.
- java와 js 모두 가능하기 때문에 클라이언트/서버 템플릿을 모두 사용가능하다.

> 책의 저자는 **템플릿 엔진의 경우 화면 역할에 충실**해야한다고 강조한다. 로직을 여기저기서 나누어 가지게 되면 유지보수가 까다로워지기 때문인 것 같다.

**설치**

plugins 탭에서 `mustache`를 검색해 설치해주면된다!

## 4.2. 기본페이지 만들기

1. **의존성 등록하기**

`compile('org.springframework.boot:spring-boot-starter-mustache')` 

의존성 하나만 추가하면 추가 설정없이 사용이 가능하다! 스프링 부트에서 공식 지원하는 템플릿 엔진이다.

2. **index.mustache 생성하기**

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>
        스프링 부트 웹서비스
    </title>
    <meta http-equiv="Content-Type" content="text/html" charset="UTF-8"/>
</head>
<body>
	<h1>스트링 부트로 시작하는 웹 서비스</h1>
</body>
</html>
```

3. **URL 매핑하기**

```java
@Controller
public class IndexController {
    @GetMapping("/")
    public String index() {
        return "index"; // 경로와 확장자는 자동으로 지정된다!
    }
}
```

이로써 "스트링 부트로 시작하는 웹 서비스"를 출력하는 페이지가 완성되었다!

게시글의 CRUD 화면을 하나하나 구현해보도록하자.

## 4.3. 게시글 등록 화면 만들기

화면을 구성하기 전에, 부트스트랩에 대해 이야기해보도록 하겠다.

기본 html만으로 화면을 구성한다고 생각해보자.. 너무 못생겼다.

이런 슬픔을 해결해주는 존재가 `부트스트랩`이다.

### 부트스트랩

오픈소스로 어느 정도 구색을 갖춘 웹 페이지를 제작할 수 있도록 도와준다.

설치 방법은 두가지가 있다.

- 외부 CDN 사용하기 << 코드에 몇줄만 추가하면 되므로 편리하다!
- 직접 다운 받아 사용하기

이번 장에서는 간단하게 외부 CDN을 사용해보겠다!

### 레이아웃 나누기

html을 작성하다보면 중복되는 부분이 발생한다. 

html의 기본적인 구조를 위한 태그(html, head)들이나 부트스트랩을 끌어오는 코드들이 이에 해당한다.

이런 중복되는 부분을 따로 나누어 만들어놓고 다른 html 파일에서 불러와 사용하면 더욱 편리할 것이다. 

코드의 중복을 막기위해 레이아웃을 먼저 생성하고 본격적으로 화면을 구성해보도록하겠다.

1. resouces/templates/header.mustache 생성

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>
        스프링 부트 웹서비스
    </title>
    <meta http-equiv="Content-Type" content="text/html" charset="UTF-8"/>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css"/>
</head>
<body>

```

2. resouces/templates/footer.mustache 생성

```html
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
</body>
</html>
```

페이지의 로딩 속도를 고려했을 때, **css는 헤더**에 **js는 footer**에 두는 것이 좋다.

HTML의 경우 위에서 부터 코드가 실행되는데, CSS의 경우 화면을 그리는 역할을 하므로 먼저 불러오는 것이 좋기 때문이다.

js의 경우 header에 존재하는데 용량이 크면 실행이 늦어지기 때문에 사용자는 빈 화면을 보아야 하기때문이다. 그러므로 js는 footer에 두어 화면이 다 그려진 뒤 호출 되도록 하자.

> 레이아웃은 아래처럼 적용된다.

```html
{{>layout/header}}

<h1>스트링 부트로 시작하는 웹 서비스</h1>

{{>layout/footer}}
```

### 글 등록 화면 만들기

1. **글 등록 버튼 만들기**

```HTML
{{>layout/header}}

<h1>스트링 부트로 시작하는 웹 서비스</h1>
<div class="col-md-12">
    <div class="row">
        <div class="col-md-6">
            <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
        </div>
    </div>
</div>
{{>layout/footer}}
```

2. **IndexContoller**로 페이지 매핑하기 
   - url => /posts/save

```java
@GetMapping("/posts/save")
public String postsSave() {
    return "posts-save";
}
```

3. **post-save.mustache 생성하기**

```html
{{>layout/header}}

<h1>게시글 등록</h1>
<div class="col-md-12">
    <div class="col-md-4">
        <form>
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control" id="title" placeholder="제목을 입력하세요"/>
            </div>
            <div class="form-group">
                <label for="author">작성자</label>
                <input type="text" class="form-control" id="author" placeholder="작성자를 입력하세요"/>
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

예! 등록화면을 구성했다! 하지만 버튼을 눌러도 아무런 동작도 하지 않는다..

**API를 호출하는 JS코드가 없기 때문**이다!

4. **resources/static/js/app에 index.js 생성하기**
   - 등록 버튼을 눌렀을 때 API를 호출하고 데이터를 담아 POST 요청을 보낸다.

```javascript
var main = {
    init: function() {
        var _this = this;
        $('#btn-save').on('click', function() {
            _this.save();
        });
    },
    save: function() {
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
};

main.init();
```

등록 기능을 하는 JS 파일을 생성했다! 이제 footer에서 js 파일을 불러주자

```html
...
<script src="/js/app/index.js"></script>
</body>
...
```

Create 기능이 완성되었다! 이제 쓴 글들을 확인하는 코드를 작성해보자.

## 4.4. 전체 조회 화면 만들기 

1. **홈 화면에서 모든 글 목록을 볼 수 있도록 수정하자.**

```html
{{>layout/header}}

<h1>스트링 부트로 시작하는 웹 서비스</h1>
<div class="col-md-12">
    <div class="row">
        <div class="col-md-6">
            <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
        </div>
    </div>
</div>
<br/>
<table class="table table-horizontal table-bordered">
    <thead class="thead=strong">
        <tr>
            <th>게시글 번호</th>
            <th>제목</th>
            <th>작성자</th>
            <th>최종수정일</th>
        </tr>
    </thead>
    <tbody id="tbody">
        {{#posts}}
            <tr>
                <td>{{id}}</td>
                <td>{{title}}</a></td>
                <td>{{author}}</td>
                <td>{{modifiedDate}}</td>
            </tr>
        {{/posts}}
    </tbody>
</table>
{{>layout/footer}}
```

머스테치의 문법이 사용된다.

- {{#posts}}
  - posts에 담긴 객체들을 하나씩 뽑아온다.
- {{변수명}}
  - LIST에서 뽑아낸 객체의 필드응 사용한다.

화면이 잘 구성되었다. 이제 Controller와 Service, Repository 코드를 수정해보자.

2. **PostsRespository 코드 작성**

```java
public interface PostsRespository extends JpaRepository<Posts, Long> {
    //SpringDataJpa에서 제공하지 않는 메소드는 쿼리로 작성해도 된다!
    @Query("SELECT p FROM Posts ORDER BY p.id DESC")
    Lists<Posts> findAllDesc();
}
```

3. **PostsService 코드 작성**

```java
...
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;
	
	...

    // 조회기능만 남겨두어 조회 속도를 개선한다.
    @Transactional(readOnly = true)
    public List<PostsListResponseDto> findAllDesc() {
        return postsRepository.findAllDesc().stream()
                .map(PostsListResponseDto::new)
                .collect(Collectors.toList());
    }
}
```

4. PostsListResponseDto 작성

```java
package com.jojoldu.book.springboot.web.dto;
import com.jojoldu.book.springboot.domain.posts.Posts;
import lombok.Getter;

import java.time.LocalDateTime;

@Getter
public class PostsListResponseDto {
    private Long id;
    private String title;
    private String author;
    private LocalDateTime modifiedDate;

    public PostsListResponseDto(Posts entity){
        this.id = entity.getId();
        this.title = entity.getTitle();
        this.author = entity.getAuthor();
        this.modifiedDate = entity.getModifiedDate();
    }
}
```

5. **Controller 작성**

```java
@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;

    @GetMapping("/")
    public String index(Model model) {
        // 결과를 posts에 담아 index로 전달한다.
        model.addAttribute("posts", postsService.findAllDesc());
        return "index";
    }

    @GetMapping("/posts/save")
    public String postsSave() {
        return "posts-save";
    }

    @GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model) {
        PostsResponseDto dto = postsService.findById(id);
        model.addAttribute("post", dto);
        return "posts-update";
    }
}

```

## 4.5. 게시글 수정/삭제 화면 만들기

위에서 한 것과같이 기능을 하나하나 추가하면 된다.

코드만 작성하면 되므로 책을 참고하여 따라치도록하자 :)

## 회고

장고 템플릿이나 퍼그를 썼던 경험이 있어서 수월했던 것 같다!

수염 귀여워