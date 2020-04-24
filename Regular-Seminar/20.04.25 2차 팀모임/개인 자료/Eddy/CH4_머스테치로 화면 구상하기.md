
머스테치로 화면 구상하기

머스테치는 템플릿엔진이다.
화면에 보여지는 View 부분을 위해 사용하는 것

머스테치 스타터 의존성을 먼저 등록해야한다.

compile('org.springframework.boot:spring-boot-starter-mustache')

머스테치의 파일 위치는 기본적으로 src/main/resources/templates이다.
이곳에 머스테치 파일을 두면 자동으로 로딩한다.
HTMl파일같은 것같다. 

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

머스테치에 URL 매핑 - web 패키지 안에 IndexController 생성
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

테스트 검증을 위해 test 패키지에 IndexControllerTest 클래스 생성
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

4.3 게시글 등록 화면 만들기

오픈소스 라이브러리 이용하여 더 이쁘게 만든다. 부트스트랩, 제이쿼리
src/main/resources/templates 디렉토리에 layout 디렉토리 추가로 생성
그 속에 footer.mustache, header.mustache 파일 생성

css, js 구분 -- 페이지 로딩속도를 높이기 위해 css - header, js는 footer

HTML은 위에서부터 코드 실행되므로 css가 먼저 화면을 그리는 것이기 때문에 header에 css가 먼저하는게 낫다.
글 등록 버튼을 구현

등록 버튼 해당하는 IndexController 사용

게시글 등록 mustache 파일 생성

But, 기능이 없으므로 기능을 구현해야 한다.
index.js
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
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 등록되었습니다.');
            location.reload();
        }).fail(function (error) {
            alert(error);
        });
    }

};

main.init(); 

window.location.href='/'
    - 글 등록이 성공하면 메인페이지(/)로 이동

인덱스라는 변수의 속성으로 function을 추가한 이유?
여러 사람이 참여하는 프로젝트에서 중복된 함수 이름으로 겹칠 수가 있다.
이를 방지하기 위해서 index 객체 안에서만 function이 유효하기 만들어서 겹칠위험을 사라지게 한다.

index.js를 머스테치 파일이 쓸 수 있게 footer.mustache에 추가
<<게시글 등록 하기 가능 구현>>

전체 조회 화면 만들기

index.mustache UI 변경
<h1>스프링부트로 시작하는 웹 서비스</h1>
    <div class="col-md-12">
        <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#savePostsModal">글 등록</button>

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
            {{#each posts}}
                <tr>
                    <td>{{id}}</td>
                    <td>{{title}}</td>
                    <td>{{author}}</td>
                    <td>{{modifiedDate}}</td>
                </tr>
            {{/each}}
            </tbody>
        </table>
    </div>

기존에 있는 PostsRepository 인터페이스에 쿼리 추가

import org.springframework.data.jpa.repository.Query;

import java.util.List;

public interface PostsRepository extends JpaRepository<Posts, Long> {

    @Query("SELECT p FROM Posts p ORDER BY p.id DESC")
    List<Posts> findAllDesc();

PostsService에 코드 추가
    @Transactional(readOnly = true)
    public List<PostsListResponseDto> findAllDesc() {
        return postsRepository.findAllDesc().stream()
                .map(PostsListResponseDto::new)
                .collect(Collectors.toList());
    }

findAllDesc 메소드의 트랜잭션 어노테이션에 옵션이 추가됨( readOnly = true) 주면
트랜잭션 범위 유지하되 조회 기능만 남겨 조회 속도 개선.. 등록/수정/삭제 기능 전혀 없는 서비스 메소드에서 추천

PostListResponseDto

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
	
Controller 변경
@RequiredArgsConstructor
@Controller	@Controller
public class IndexController {	


    private final PostsService postsService;

    @GetMapping("/")	 
    	    public String index(Model model) {
        model.addAttribute("posts", postsService.findAllDesc());
            return "index";
    	    }
	}


4.5 게시글 수정, 삭제 화면 만들기

게시글 수정
<div class="modal-body">
                    <form>
                        <div class="form-group">
                            <label for="title">제목</label>
                            <input type="text" class="form-control" id="title" placeholder="제목을 입력하세요">
                        </div>
                        <div class="form-group">
                            <label for="author"> 작성자 </label>
                            <input type="text" class="form-control" id="author" placeholder="작성자를 입력하세요">
                        </div>
                        <div class="form-group">
                            <label for="content"> 내용 </label>
                            <textarea class="form-control" id="content" placeholder="내용을 입력하세요"></textarea>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">취소</button>
                    <button type="button" class="btn btn-primary" id="btn-save">등록</button>
                </div>
            </div>
        </div>
    </div>

{{post.id}}
    - 머스테치는 객체의 필드 접근시 점(Dot)으로 구분
    - 즉 Post 클래스의 id에 대한 접근은 post.id로 사용

readonly
    - input 태그에 읽기 가능만 허용하는 속성
    - id와 author는 수정할 수 없도록 읽기만 허용하도록 추

그리고 update 기능 호출할 수 있도록  index.js 파일에 update function 추가

수정 페이지 이동할 수 있도록 페이지 이동기능 추가

수정화면 연결할 Controller 코드 작업


게시글 삭제

삭제 이벤트 진행할 JS 코드 추가

삭제 API 만들기
서비스 메소드

PostsService

public class PostsService {
        
    @Transactional
    public void delete (Long id) {
        Posts posts = postsRepository.findById(id)
            .orElseThrow(() -> new
IllegalArgumentException("해당 게시글이 없습니다. id =" + id));
        
        postsRepository.delete(posts);
    }
        ..

}

postsRepository.delete(posts)
    - 엔티리를 파라미터로 삭제할 수도 잇고 deleteByld 메소드 이용하면 id로 삭제할 수도 있다
    - 존재하는 Posts인지 확인을 위해 엔티티 조회 후 그대로 삭제


PostsAPiController 구현



