# 1. 인텔리제이로 스프링 부트 시작하기

## 진도 기록

|공부 날짜|공부한 페이지 수|
|---|---|
|2020.04.07|4p ~ 50p|

## 1.1. 인텔리제이 소개

### # 이클립스에 비해 인텔리제이가 갖는 강점

- 강력한 추천 기능(Smart Completion)
  
- 훨씬 더 다양한 리팩토링과 디버깅 기능

- 이클립스의 깃(Git)에 비해 훨씬 높은 자유도

- 프로젝트 시작할 때 인덱싱을 하여 파일을 비롯한 자원들에 대한 빠른 검색 속도

- HTML과 CSS, JS, XML에 대한 강력한 기능 지원

- 자바, 스프링 부트 버전업에 맞춘 빠른 업데이트

→ 실제로 많은 IT 서비스 회사들이 사용하고 있습니다.

<br>

## 1.2. 인텔리제이 설치하기

### # [젯브레인 툴박스](https://www.jetbrains.com/toolbox/app/)를 이용한 설치

- 이미 설치되어 있으므로 과정 생략합니다.

💡 TIP) 학교 이메일이 있으면 Ultimate(유료)버전도 무료로 사용 가능합니다.

<br>

## 1.3. 인텔리제이 커뮤니티에서 프로젝트 생성하기

인텔리제이 : 프로젝트와 모듈, 한 화면에 하나의 프로젝트만 열립니다.

이클립스 : 워크스페이스, 모든 프로젝트를 한 번에 불러옵니다.

### # 프로젝트 생성

1. Create New Project

2. 프로젝트 유형 선택
   - Gradle → Java 선택

3. 프로젝트 그룹명과 아티펙트명 등록
   - GroupId는 프로젝트를 생성한 조직, 그룹명
   - ArtifactId는 프로젝트 이름

4. 프로젝트 생성 디렉토리 선택

<br>

## 1.3. 궁금한점

1. GroupId와 ArtifactId 네이밍

<br>

## 1.4. 그레이들 프로젝트를 스프링 부트 프로젝트로 변경하기

### # 플러그인 의존성 관리를 위한 설정

```java
/* build.gradle 파일 */

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

- ext라는 키워드는 build.gradle에서 사용하는 전역변수를 설정하겠다는 의미입니다.

- springBootVersion 전역변수를 생성, 그 값은 '2.1.7.RELEASE' 입니다.

→ spring-boot-gradle-plugin의 2.1.7.RELEASE를 의존성으로 받겠다는 의미입니다.

<br>

```java
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'
```

- 자바와 스프링 부트를 사용하기 위한 필수 플러그인들이니 항상 추가하면 됩니다.

- io.spring.dependency-management는 스프링 부트의 의존성들을 관리해 주는 플러그인이므로 반드시 추가해야만 합니다.

<br>

```java
repositories {
    mavenCentral()
    jcenter()
}

dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

- repositories는 각종 의존성(라이브러리)들을 어떤 원격 저장소에서 받을지 정합니다.

- 기본적으로 mavenCentral을 많이 사용하지만, 최근에는 jcenter도 많이 사용합니다.

  - 라이브러리 업로드 저장소로, 이 둘은 라이브러리 업로드 난이도 차이입니다.

  - mavenCentral은 직접 만든 라이브러리를 업로드하기 위해 많은 과정과 설정이 필요합니다.

  - jcenter는 이 문제점을 개선하여 간단하게 업로드가 가능하고, <br>
    mavenCentral에도 업로드될 수 있도록 자동화를 할 수 있습니다.

- dependencies는 프로젝트 개발에 필요한 의존성들을 선언하는 곳입니다.

  - 특정 버전을 명시하지 않아야만 처음 작성한 `${springBootVersion}`의 버전을 따라갑니다.

→ 이렇게 관리할 경우 각 라이브러리들의 버전 관리가 한 곳에 집중되고, 버전 충돌 문제도 해결됩니다.

<br>

## 1.4. 궁금한점

1. Gradle과 Maven이란?
2. Gradle과 Maven의 차이점

<br>

## 1.5. 인텔리제이에서 깃과 깃허브 사용하기

깃의 원격 저장소 역할을 하는 서비스 : 깃허브, 깃랩 등

### # 프로젝트와 깃허브 연동

1. 단축키 `[Ctrl+Shift+A]` → share project on github 을 검색합니다.

2. 깃허브 계정으로 로그인합니다.

3. 깃허브에 생성할 저장소 정보를 입력합니다.

4. 커밋할 파일 선택 및 커밋 메시지를 작성합니다.
   
   > `.idea` 디렉토리는 커밋하지 않습니다. - 프로젝트 실행시 자동으로 생성되는 파일들

5. `.gitignore` 파일 작성
   
   > 인텔리제이에서는 `.gitignore`에 대한 기본적인 지원이 없고, 플러그인에서 지원합니다.
   
   1 ) 단축키 `[Ctrl+Shift+A]` → plugins 을 검색합니다. <br>
   2 ) .ignore 검색 및 설치 후 재시작합니다. <br>
   3 ) 단축키 `[Alt+Insert]` → `.ignore file` → `gitignore file[Git]` <br>
   4 ) 인텔리제이에서 자동으로 생성되는 파일들 모두 이그노어 처리합니다.

6. 단축키 `[Ctrl+K]` → Commit

7. 단축키 `[Ctrl+Shift+K]` → Push

💡 TIP) [gitignore.io](https://www.gitignore.io/) 사이트를 이용하면 쉽게 `.gitignore`파일을 생성할 수 있습니다.

<br>

## 궁금한 점 스스로 찾아보기

### 1. GroupId와 ArtifactId 네이밍

#### # GroupId

group id는 프로젝트마다 구별할 수 있는 고유한 이름으로, 보통은 java의 패키지 네이밍을 따릅니다. <br>
ex) org.apache.maven, org.apache.commons
  
하지만 이 규칙이 강제적인 것은 아닙니다.

GroupId에 많은 하위 group을 만들수 있는데 좋은 방법은 프로젝트 구조로 만드는 것입니다. <br>
만약 프로젝트가 멀티 프로젝트가 된다면, 새로운 식별자만 부모의 groupid 뒤에 붙이면 됩니다. <br>
ex) org.apache.maven, org.apache.maven.plugins, org.apache.maven.reporting

#### # ArtifactId

artifact id는 jar파일에서 버전 정보를 뺀 이름입니다. <br>
소문자를 사용하고 이상한 특수문자는 사용하지 않습니다. <br>
ex) maven, commons-math

##### 출처 : [Hyunpro 발전하는개발자](https://writemylife.tistory.com/74)

<br>

### 2. Gradle과 Maven이란?

#### 빌드도구(Build Tool)

빠른 기간동안에 계속해서 늘어나는 라이브러리의 추가와 <br>
프로젝트를 진행하며 라이브러리의 버전을 동기화하기 어렵기 때문에 등장하였습니다. <br>
초기의 JAVA 빌드 도구로 Ant라는 도구를 많이 사용하였으나 <br>
늘어나는 라이브러리를 관리하기 위해 Maven, Gradle등의 기존 Ant를 보완한 빌드 도구들도 생기게 되었습니다.

#### # Maven
  
Maven은 사용할 라이브러리 뿐만 아니라 해당 라이브러리가 작동하는데 필요한 다른 라이브러리들까지 관리하여 네트워크를 통해 자동으로 다운 받아주며, 프로젝트의 전체적인 라이프사이클을 관리하는 도구입니다.

Maven은 프로젝트에 필요한 모든 'Dependency (종속성)'를 리스트의 형태로 Maven에게 알려 관리할 수 있도록 돕는 방식입니다.
  
- Dependency를 관리하고, 표준화된 프로젝트(Standardized project)를 제공합니다.

- XML, remote repository를 가져 올 수 있습니다. <br>
  : 개발에 필요한 종속되는 'jar', 'class path'를 다운로드 할 필요 없이 선언만으로 사용 가능합니다.

- 상속형 : 하위 XML이 필요 없는 속성도 모두 표기합니다.
  
즉, `POM.xml`이라는 Maven 파일에 필요한 'Jar', 'Class Path'를 선언해 주면 <br>
직접 다운로드 할 필요 없이 Maven은 Repository에서 필요한 모든 파일들을 해당 프로젝트로 불러와 줍니다. <br>
이러한 장점에도 불구하고, Maven은 몇가지 단점이 있는데 그것은 바로 아래와 같습니다.

- 라이브러리가 서로 종속할 경우 XML이 복잡해집니다.

- 계층적인 데이터를 표현하기에 좋지만, 플로우나 조건부 상황을 표현하기엔 어려습니다.

- 편리하나 맞춤화된 로직 실행이 어려습니다.

#### # Gradle
  
Gradle은 JVM 기반의 빌드 도구로 기존의 Ant와 Maven을 보완하였습니다. <br>
따라서 JAVA 혹은 Groovy를 이용해 logic을 개발자의 의도에 따라 설계할 수 있습니다.

- 오픈소스기반의 build 자동화 시스템으로 Groovy 기반 DSL(Domain-Specific Language)로 작성합니다.

- Build-by-convention을 바탕으로함: 스크립트 규모가 작고 읽기 쉽습니다.

  - Multi 프로젝트의 빌드를 지원하기 위해 설계되었습니다.

  - 설정 주입 방식 (Configuration Injection) 입니다.
  
따라서 초기 프로젝트 설정에 드는 시간을 절약할 수 있으며 기존의 Maven이나 Ivy등과 같은 빌드 도구들과도 호완이 가능합니다.

##### 출처 : [JJ's Once a week](https://jj-one-a-week.blogspot.com/2017/05/ant-maven-gradle.html)

<br>

### 3. Gradle과 Maven의 차이점

Maven의 경우 XML(`pom.xml`)로 라이브러리를 정의하고 활용하도록 되어 있으나, <br>
Gradle의 경우 별도의 빌드스크립트를 통하여 사용할 어플리케이션 버전, 라이브러리등의 항목을 설정 할 수 있습니다.

Maven에는 Gadle과 비교문서가 없지만, Gradle에는 비교문서가 있습니다.

Gradle이 시기적으로 늦게 나온만큼 사용성, 성능 등 비교적 뛰어난 스펙을 가지고 있습니다.

#### # Gradle이 Maven보다 좋은점

- Build라는 동적인 요소를 XML로 정의하기에는 어려운 부분이 많습니다.

  - 설정 내용이 길어지고 가독성 떨어집니다.
  
  - 의존관계가 복잡한 프로젝트 설정하기에는 부적절합니다.
  
  - 상속구조를 이용한 멀티 모듈 구현
  
  - 특정 설정을 소수의 모듈에서 공유하기 위해서는 부모 프로젝트를 생성하여 상속하게 해야합니다. (상속의 단점이 생김)

- Gradle은 그루비를 사용하기 때문에, 동적인 빌드는 Groovy 스크립트로 플러그인을 호출하거나 직접 코드를 짜면 됩니다.

  - Configuration Injection 방식을 사용해서 공통 모듈을 상속해서 사용하는 단점을 커버합니다.

  - 설정 주입시 프로젝트의 조건을 체크할 수 있어서 프로젝트별로 주입되는 설정을 다르게 할 수 있습니다.
  
→ Gradle은 Maven보다 최대 100배 빠릅니다.

##### 출처 : [어쩌다, 블로그](https://bkim.tistory.com/13)

<br>

## 회고

1장은 원래 워밍업!(부터 빡세게 정리한 듯하다ㅎ)