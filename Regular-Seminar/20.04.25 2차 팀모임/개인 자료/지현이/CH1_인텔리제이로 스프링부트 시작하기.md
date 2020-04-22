# 1. 인텔리제이로 스프링부트 시작하기

스프링 부트로 웹 서비스를 위한 첫 번째 단계로 개발환경을 구성해보자!

> IntelliJ IDEA

## 진도 기록

| 공부 날짜  | 공부한 페이지 수 |
| ---------- | ---------------- |
| 2020.04.01 | 18p ~ 49p        |

## 1.1. 인텔리제이 소개

### 인텔리제이의 장점

- 강력한 추천기능
- 다양한 리팩토링과 디버깅 가능
- 높은 Git 자유도
- 자원에 대한 빠른 검색 속도
- 자바, 스프링 부트 버전에 대한 빠른 업데이트
- HTML, CSS, JS에 대한 강력한 기능 지원

> IntelliJ Community(무료) 로도 충분히 개발이 가능하다.
>
> 다만 HTML, CSS, JS에 대한 지원은 없으므로 다른 개발 도구를 사용하도록하자.

## 1.2. 인텔리제이 설치

툴박스를 [이곳](! https://www.jetbrains.com/toolbox-app/ )에서 설치하자.

툴박스는 인텔리제이의 버전관리를 용이하게 해주므로 이를 통해 설치하도록 한다.

## 1.3. 인텔리제이 커뮤니티에서 프로젝트 생성하기

### 설치

1. 테마 선택
2. 단축키 설정 < 다음
3. 시작 설정 스크립트 생성화면 < 다음
4. 기본 플러그인 설정 화면 < 다음
   - 스칼라같은 쓰고 싶은 플러그인을 옵션으로 선택할 수 있다.

5. 추가 플러그인 선택화면 < 다음

### 프로젝트 생성

1. 프로젝트 유형: 그레이들 프로젝트 생성 (JAVA check)
2. GroupId와 ArtifactId 등록
   - ArtifactId 는 프로젝트 Id가 된다.
3. 특별한 것 없이 다음으로 넘어가자.

## 1.4. 그레이들 프로젝트를 스프링 부트 프로젝트로 변경하기

`bulid.gradle` 파일을 열게되면 java 개발을 위한 기초적인 설정만 되어있다.

이곳에 스프링 부트에 필요한 설정을을 하나씩 추가해주자.

**왜 스프링 이니셜라이저를 안쓰나요**

bulid.gradle의 코드가 무슨 역할을 하고 의존성을 추가할 때 어떻게 해야할지 모르는 경우를 방지하기 위해 하나씩 보도록한다.

### bulid.gradle

**플러그인의 의존성 관리**

```
buildscript {
    ext { // 전역변수 설정
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

**플러그인 의존성 결정하기**

```
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management' // 필수! 스프링 부트의 의존성을 관리해준다.
```

**의존성(lib)를 어떤 원격 저장소에서 받을지 정하기**

```
repositories {
    mavenCentral()
    jcenter()
    google()
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    compile("org.springframework.boot:spring-boot-starter-test")
}
```

> 추가된 의존성들은 자동으로 변경된 내용들을 반영된다. (by gradle)

## 회고

설치할때는 늘 설랜다.