# 1장 인텔리제이로 스프링 부트 시작하기

## Contents
- [인텔리제이 소개](#인텔리제이-소개)
- [인텔리제이 설치하기](#인텔리제이-설치하기)
- [인텔리제이 커뮤니티에서 프로젝트 생성하기](#인텔리제이-커뮤니티에서-프로젝트-생성하기)
- [그레이들 프로젝트를 스프링 부트 프로젝트로 변경하기](#그레이들-프로젝트를-스프링-부트-프로젝트로-변경하기)
- [인텔리제이에서 깃과 깃허브 사용하기](#인텔리제이에서-깃과-깃허브-사용하기)

---

### 인텔리제이 소개
- [IntelliJ IDEA](https://www.jetbrains.com/ko-kr/idea/)

### 인텔리제이 설치하기
- [Toolbox로 설치하기](https://www.jetbrains.com/toolbox-app/)

### 인텔리제이 커뮤니티에서 프로젝트 생성하기
- [Eclipse의 Workspace와 IntelliJ의 Project](https://jojoldu.tistory.com/334)
- 새로운 프로젝트 생성하기
  1. Create New Project 버튼 클릭
  2. Gradle 선택
  3. GroupId & ArtifactId(프로젝트의 이름) 등록
  4. 그레이들의 옵션은 기본 설정값 그대로
  5. 새로 만들 프로젝트의 디렉토리 위치 선택

### 그레이들 프로젝트를 스프링 부트 프로젝트로 변경하기
- build.gradle에 필요한 설정 추가하기
  ```gradle
  buildscript {
      ext {
          springBootVersion = '2.1.7.RELEASE'
      }
      repositories {
          mavenCentral()
          jcenter()
      }
      dependencies {
          classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
      }
  }
  ```
  - 플러그인 의존성 관리를 위한 설정
  - ext
    - build.gradle에서 사용하는 전역변수를 설정하겠다는 의미
    - 여기서는 springBootVersion 전역변수를 생성하고 그 값은 2.1.7RELEASE로 하겠다는 의미
      - spring-boot-gradle-plugin의 2.1.7.RELEASE를 의존성으로 받겠다.
  ```gradle
  apply plugin: 'eclipse'
  apply plugin: 'org.springframework.boot'
  apply plugin: 'io.spring.dependency-management'
  apply plugin: 'java'
  ```
  - 위의 네가지 플러그인은 자바와 스프링 부트를 사용하기 위해서는 필수인 플러그인들 항상 추가하여야 한다.
    - io.spring.dependency-management 플러그인은 스프링 부트의 의존성을 관리해 주는 플러그인이므로 꼭 추가해야만 한다.
  ```gradle
  repositories {
      mavenCentral()
      jcenter()
  }

  dependencies {
      compile('org.springframework.boot:spring-boot-starter-web')
      testCompile('org.springframework.boot:spring-boot-starter-test')
  }
  ```
  - repositories
    - 각종 의존성(라이브러리)들을 어떤 원격 저장소에서 받을지를 정한다.
  - dependencies
    - 프로젝트 개발에 필요한 의존성들을 선언하는 곳

### 인텔리제이에서 깃과 깃허브 사용하기
- 건너뛰기
