# Chapter 1

Created: Mar 16, 2020 11:54 PM
Tags: gradle

### 1.4 그레이들 프로젝트를 스프링 부트 프로젝트로 변경하기

```java
// build.gradle

/* 프로젝트의 플러그인 의존성 관리를 위한 설정 */
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

/* 위에서 선언한 플러그인 의존성들을 적용할 것인지를 결정 */
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management' // 스프링 부트의 의존성들을 관리해 주는 플러그인이므로 반드시 추가해야

group 'org.example'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
    jcenter()
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.projectlombok:lombok')
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
    compile('org.springframework.boot:spring-boot-starter-mustache')
    compile('org.springframework.boot:spring-boot-starter-oauth2-client')
    compile('org.springframework.session:spring-session-jdbc')
    compile('com.h2database:h2')
    testCompile('org.springframework.boot:spring-boot-starter-test')
    testCompile('org.springframework.security:spring-security-test')
}
```

- `ext`: build.gradle에서 사용하는 전역 변수를 설정하겠다는 의미이다.
- `repositories`: 각종 의존성(라이브러리)들을 어떤 원격 저장소에서 받을지 결정한다.
    - 기본적으로 mavenCentral을 많이 사용하지만, 최근에는 **라이브러리 업로드 난이도** 때문에  jcenter도 많이 사용한다. → **라이브러리 업로드가 간단**
- `dependencies`: 프로젝트 개발에 필요한 의존성들을 선언한다.
- 깃허브 커밋 시 **.idea 디렉토리는 커밋하지 않는다**!
    - 프로젝트 실행 시 자동으로 생성되는 파일들이기 때문에 불필요하다.