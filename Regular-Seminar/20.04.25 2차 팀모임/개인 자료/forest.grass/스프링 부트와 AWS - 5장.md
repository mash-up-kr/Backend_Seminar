# 5.스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현하기

## 진도 기록

|공부 날짜|공부한 페이지 수|
|---|---|
|2020.04.24|163p ~ 223p |

## 개요
- 스프링 시큐리티는 막강한 인증과 인가 기능을 가진 프레임워크입니다.
- 스프링 기반의 애플리케이션에서는 보안을 위한 표준이라고 보면 됩니다.
- 스프링의 대부분 프로젝트는 확장성을 고려한 프레임워크다.
- OAuth 2.0을 구현한 구글 로그인을 연동하여 로그인 기능을 만들어 보자.

## 5.1 스프링 시큐리티와 스프링 시큐리티 Oauth2 클라이언트
- 인증과 인가 기능을 직접 구현할 경우 배보다 배꼽이 커지는 경우가 많다.
- 직접 구현 한다면
  - 로그인시 보안
  - 비밀번호 찾기
  - 회원가입시 이메일 혹은 전화번호 인증
  - 비밀번호 변경
  - 회원정보 변경
- 스프링 부트 1.5 vs 스프링 부트 2.0
  - 설정 방법에 크게 차이가 없는 경우를 볼 수 있다.
  - spring-security-oauth2-autoconfigure 라이브러리 덕분
  - 기존에 안전하게 작동하던 코드를 사용하는 것이 아무래도 더 확실하므로 많은 개발자가 선호한다.
  - 1.5에 비해 2.0은 설정이 간소화(enum으로 Oauth 도메인을 관리하고 있다.)
- 네이버, 카카오 도메인은 직접 추가해야한다.

## 5.2 구글 서비스 등록
- 구글 서비스에 신규 서비스(나의 서버)를 생성
- 발급된 인증정보를 통해서 로그인 기능과 소셜 서비스 기능을 사용할 수 있으니 무조건 발급받고 시작해야 합니다.
- 애플리케이션 이름
- 지원 이메일
- Google API의 범위
- application-xxx.properties
  - xxx라는 이름의 profile이 생성되어 이를 통해 관리할 수 있습니다.
  - 즉, profile=xxx라는 식으로 호출하면 해당 properties의 설정들을 가져올 수 있습니다.
- .gitignore
  - 중요한 정보가 들어 있는 application.properties 파일을 ignore하자.

## 5.3 구글 연동하기
- 스프링 시큐리티 설정
  - spring boot starter oauth2 client 의존성 추가
- @EnableWebSecurity
- @csrf().disable().headers().frameOptions().disable
- userService
- CustomoAuth2UserService
  - 로그인 이후 가져온 사용자의 정보들을 기반으로 가입 및 정보수정, 세션 저장 등의 기능을 지원합니다.
- 직렬화를 이용하여 따로 UserSession 클래스를 만드는 이유
  - entity를 그대로 사용하면 추후 관계 맵핑에서 연관되어 있는 class까지 연결되어 사이드이펙트가 발생할 수 있다.
  - 세션을 위한 직렬화 클래스를 따로 만들자.
- 머스테치 문법
  - {{#****}} - 값이 있으면
  - {{^****}} - 값이 없으면

## 5.4 어노테이션 기반으로 개선하기
- 같은 코드가 반복되는 부분은 생산성을 낮춘다.
- 어노테이션을 이용하여 중복되는 코드를 줄이자
- 세션을 일일이 불러오는 코드 -> 메서드 인자로 세션값을 바로 받을 있도록 변경(어노테이션 사용)
- @Traget(ElementType.PARAMETER)
- @interface
- HandlerMethodArgumentResolver
  - 조건에 맞는 경우 메소드가 있다면 HandlerMethodArgumentResolver의 구현체가 지정한 값으로 해당 메소드의 파라미터로 넘길 수 있습니다.
  - HandlerMethodArgumentResolver는 항상 WebMvcConfigurer의 addArgumentResolvers()를 통해 추가해야 합니다. 다른 Hadler-MethodArgumentResolver가 필요하다면 같은 방식으로 추가해 주면 됩니다.

## 5.5 세션 저장소로 데이터베이스 사용하기
- 세션을 서버 메모리에 저장한다면 서버를 늘린다면 동기화가 필요하고 재부팅을 한다면 메모리가 날아가 복구가 불가능하다.
- 메모리에 저장하지말고 데이터베이스를 이용하자.
- 세션을 저장하는 세가지 방법
  - 톰캣 세션
  - 데이터베이스
  - 메모리 디비

## 5.6 네이버 로그인

## 5.7 기존 테스트에 시큐리티 적용하기
- 테스트코드에서 시큐리티를 적용을 해야할까?
- 불필요한곳에는 시큐리티는 적용시키지 않는게 좋다.(컨트롤러가 아닌 단위 테스트)
- 테스트에도 시큐리티를 적용하고 싶으면 테스트를 위한 스프링 시큐리티 테스트 의존성이 있다. 사용하자.
- test의 application.properties가 존재 하지 않는다면 main의 application.properties를 사용한다.
- 하지만 main의 application-xxx.properties는 가져오지 않는다. 따로 테스트를 위하여 만들어주자.
- @WithMockUser(roles="User")
- @WebMvcTest(excludeFilters = {@ComponentScan.Filter(....)})
- @EnableJpaAuditing 위치 변경

## 궁금한 점 스스로 찾아보기
### 1.스프링 시큐리티는 로그인후 프론트와 무엇을 사용하여 인증 처리를 할까??
- [spring security 파헤치기 (구조, 인증과정, 설정, 핸들러 및 암호화 예제, @Secured, @AuthenticationPrincipal, taglib)](https://sjh836.tistory.com/165)

### 2.H2 console을 사용하기 위해 CSRF, X-Frame-Options을 disable하는데 그 이유는 무엇일까??
- [Spring Security, H2 연동했을때 문제해결](https://www.slipp.net/questions/546)


## 회고
- 스프링 시큐리티를 사용해본적이 거의 없어 이번기회에 좋은 공부를 했다고 생각합니다.
- 전체적인 돌아가는 플로우(Oauth2 포함)를 공부하는 기회가 되었습니다.
- 설정이 어렵다고 생각했는데 하나하나 설명해주고 있어서 매우 좋았습니다.