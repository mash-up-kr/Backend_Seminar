# 1.인텔리제이로 스프링 부트 시작하기  

## 진도 기록

|공부 날짜|공부한 페이지 수|
|---|---|
|2020.04.22|17p ~ 50p |

## 개요
- 스프링 부트로 웹 서비스를 만들기 위하여 개발환경을 구성해보자.
- 인텔리제이 설치, 스프링 부트 의존성 설정(Gradle 설정)

## 1.1 인텔리제이 소개
- 인텔리제이가 갖는 장점
  - 강력한 추천 기능
  - 훨씬 다양한 리팩토링과 디버깅 기능
  - 프로젝트 시작할 때 인덱싱을 하여 파일을 비롯한 자원들에 대한 빠른 검색 속도
  - HTML과 CSS, JS, XML에 대한 강력한 기능 지원
  - 자바, 스프링 부트 버전업에 맞춘 빠른 업데이트
- 인텔리제이의 유료버전과 무료버전
  - 무료버전으로 충분히 스프링 부트 개발이 가능하다.

## 1.2 인텔리제이 설치하기
- 설치중 메모리 할당 가능

## 1.3 인텔리제이 커뮤니티에서 프로젝트 생성하기
- 인텔리제이는 이클립스의 워크스페이스와 같은 개념은 없다.
- 모든 프로젝트를 한 번에 불러올 수 없다. 한 화면에서는 하나의 프로젝트만 가능
- ArtifactId
  - 프로젝트의 이름

## 1.4 그레이들 프로젝트를 스프링 부트 프로젝트로 변경하기
- 그레이들 프로젝트를 스프링 부트 프로젝트로 변경해보자
<pre><code>
buildscript {
    ext { //전역 변수
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories { //의존성을 가지고 오는 레파지토리
        mavenCentral()
        jcenter()
    }
    dependencies { // 의존성 추가
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

  group 'com.forest.grass'
  version '1.0'
  sourceCompatibility = 1.8
    
  apply plugin: 'java'
  apply plugin: 'org.springframework.boot'
  apply plugin: 'io.spring.dependency-management' // 의존성 버전 관리

  repositories {
      mavenCentral()
  }

  dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testCompile 'org.springframework.boot:spring-boot-starter-test'
  }
}
</code></pre>

## 1.5 인텔리제이에서 깃과 깃허브 사용하기
- 깃과 깃허브(인텔리제이 툴에서 깃허브 연동을 지원)
- .ignore(인텔리제이에서 플러그인으로 손쉽게 사용)

## 궁금한 점 스스로 찾아보기
- 없습니다.

## 회고
- 스프링부트를 개발하기위한 툴 설치 및 설정이였기 때문에 궁금한점이 없었던 것 같습니다.