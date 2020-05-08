# 스프링 파일 업로드

스프링 부트로 멀티파트(Multi-part) 파일을 업로드하는 서버 애플리케이션을 소개하는 자료입니다.  
예제 프로젝트 파일은 [이 곳](https://github.com/spring-guides/gs-uploading-files/archive/master.zip)에서 다운로드 가능합니다.

## 구성 환경
- JDK 1.8 or later
- Gradle 4+ or Maven 3.2+

## 의존성 설정 파일

그레이들 설정 파일의 내용은 다음과 같다.

```gradle
plugins {
	id 'org.springframework.boot' version '2.2.0.RELEASE'
	id 'io.spring.dependency-management' version '1.0.8.RELEASE'
	id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
}

test {
	useJUnitPlatform()
}
```

주목할 점은 `implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'` 부분이다.  
**Thymeleaf**는 Mustache와 같은 템플릿 엔진이다. HTML, XML, JavaScript, CSS 및 일반 텍스트를 처리할 수 있는 웹 환경과 독립 환경에서 사용할 수 있는 Java 템플릿 엔진이다. HTML파일을 가져와 파싱한 다음 정해진 위치에 데이터를 치환해 웹 페이지를 만든다.

JSP와 다른 점은 JSP는 서블릿으로 변환되어 실행된다. 따라서 JSP 내에서 자바 코드도 사용이 가능하다. 반면 Thymeleaf는 직접적으로 자바 코드를 사용할 순 없다.

## 애플리케이션 클래스

스프링 부트 MVC 애플리케이션을 만드려면, 스타터가 필요하다. `spring-boot-starter-thymeleaf`와 `spring-boot-starter-web` 의존성이 이미 추가되어 있다. 서블릿 컨테이너와 함께 파일을 업로드하려면, web.xml 파일의 `<multipart-config>` 설정으로 `MultipartConfigElement` 클래스를 등록해야 한다. 스프링 부트는 자동으로 그 설정들을 해준다.

---

web.xml은 각종 서블릿의 설정과 매핑, 필터, 인코딩 등의 정보를 저장하는 설정 파일이다. 참고로 서블릿 3.0 부터는 web.xml 없이 개발 또는 배포가 가능하다. `@MultipartConfig` 어노테이션으로 지정해줄 수 있다.

```xml
<servlet>
  <servlet-name>MyServlet</servlet-name>
  <servlet-class>com.spring.mvc.MyServlet</servlet-class>
  <init-param>
    <param-name>myProperties</param-name>
    <param-value>C:\Workspace\spring\WebContent\myProperties.properties</param-value>
  </init-param>
  <!-- multipart 설정 -->
  <multipart-config>
      <location>C:\Workspace\spring\files\</location>
      <max-file-size>-1</max-file-size>
      <max-request-size>-1</max-request-size>
      <file-size-threshold>1024</file-size-threshold>
  </multipart-config>
</servlet>
```

```java
@MultipartConfig(
		fileSizeThreshold 	= <size in bytes>,
		maxFileSize 		= <size in bytes>,
		maxRequestSize		= <size in bytes>,
		location 		    = <save location>
)
```

---

## 업로드 컨트롤러 생성

```java
@Controller
public class FileUploadController {

	private final StorageService storageService;

	@Autowired
	public FileUploadController(StorageService storageService) {
		this.storageService = storageService;
	}

	@GetMapping("/")
	public String listUploadedFiles(Model model) throws IOException {

		model.addAttribute("files", storageService.loadAll().map(
				path -> MvcUriComponentsBuilder.fromMethodName(FileUploadController.class,
						"serveFile", path.getFileName().toString()).build().toUri().toString())
				.collect(Collectors.toList()));

		return "uploadForm";
	}

	@GetMapping("/files/{filename:.+}")
	@ResponseBody
	public ResponseEntity<Resource> serveFile(@PathVariable String filename) {

		Resource file = storageService.loadAsResource(filename);
		return ResponseEntity.ok().header(HttpHeaders.CONTENT_DISPOSITION,
				"attachment; filename=\"" + file.getFilename() + "\"").body(file);
	}

	@PostMapping("/")
	public String handleFileUpload(@RequestParam("file") MultipartFile file,
			RedirectAttributes redirectAttributes) {

		storageService.store(file);
		redirectAttributes.addFlashAttribute("message",
				"You successfully uploaded " + file.getOriginalFilename() + "!");

		return "redirect:/";
	}

	@ExceptionHandler(StorageFileNotFoundException.class)
	public ResponseEntity<?> handleStorageFileNotFound(StorageFileNotFoundException exc) {
		return ResponseEntity.notFound().build();
	}

}
```

- `GET /` : `StorageService`에서 현재 업로드된 파일 목록을 조회한다. Thymeleaf 템플릿으로 변환된다. `MvcUriComponentsBuilder`을 사용해 자료의 링크를 만든다.
- `GET /files/{filename}` : 파일이 존재한다면 파일을 불러온다. 그리고 다운로드 가능하도록 `Content-Disposition` 응답 헤더를 덧붙인다.
- `POST /` : 파일 업로드 메시지를 저장하고 `StorageService`에 매개변수로 넘어온 멀티파트 파일을 저장한다.

## StorageService 인터페이스

`StorageService`는 인터페이스로 컨트롤러와 파일 시스템과 같은 저장 레이어를 연결시켜준다.

```java
public interface StorageService {

	void init();

	void store(MultipartFile file);

	Stream<Path> loadAll();

	Path load(String filename);

	Resource loadAsResource(String filename);

	void deleteAll();

}
```

## HTML 템플릿 파일 만들기

`src/main/resources/templates/uploadForm.html` 파일에 Thymeleaf 템플릿 파일을 만든다.

```html
<html xmlns:th="https://www.thymeleaf.org">
<body>

    <!-- 1 -->
	<div th:if="${message}">
		<h2 th:text="${message}"/>
	</div>

    <!-- 2 -->
	<div>
		<form method="POST" enctype="multipart/form-data" action="/">
			<table>
				<tr><td>File to upload:</td><td><input type="file" name="file" /></td></tr>
				<tr><td></td><td><input type="submit" value="Upload" /></td></tr>
			</table>
		</form>
	</div>

    <!-- 3 -->
	<div>
		<ul>
			<li th:each="file : ${files}">
				<a th:href="${file}" th:text="${file}" />
			</li>
		</ul>
	</div>

</body>
</html>
```

- 1번 코드 : 메시지가 있다면 메시지를 띄운다. 파일 업로드에 성공하면 홈화면으로 리다이렉션되고 메시지 파라미터를 보낸다.
- 2번 코드 : 파일 업로드를 하는 Form 태그를 작성한다.
- 3번 코드 : 파일 목록 정보를 받는다면, `each` 속성으로 리스트를 출력한다.

## 파일 업로드 제한 설정

`src/main/resources/application.properties` 파일에 업로드 설정을 넣을 수 있다.

```
spring.servlet.multipart.max-file-size=128KB
spring.servlet.multipart.max-request-size=128KB
```

첫 번째 `max-file-size`는 파일 자체의 크기가 128KB를 넘지 못하도록 제한한다.  
두 번째 `max-request-size`는 `multipart/form-data` 요청 크기를 제한한다.

제한 없음은 `-1` 으로 설정한다.

## 애플리케이션 실행

```java
@SpringBootApplication
@EnableConfigurationProperties(StorageProperties.class)
public class UploadingFilesApplication {

	public static void main(String[] args) {
		SpringApplication.run(UploadingFilesApplication.class, args);
	}

	@Bean
	CommandLineRunner init(StorageService storageService) {
		return (args) -> {
			storageService.deleteAll();
			storageService.init();
		};
	}
}
```

- `@SpringBootApplication` 어노테이션은 다음 어노테이션을 추가해준다.
  - `@Configuration` : 애플리케이션 실행을 위한 Bean을 정의하는 소스 코드가 있는 클래스라고 표시를 해준다.
  - `@EnableAutoConfiguration` : 스프링 부트에게 클래스 경로의 설정과 다른 Bean들, 다양한 속성 설정을 기반으로 둔 빈들을 추가해달라고 한다.
  - `@ComponentScan` : 스프링이 같은 패키지로부터 다른 컴포넌트와 설정, 서비스, 컨트롤러를 찾도록 명령한다.

## 실행 Jar 파일로 만들기

Gradle을 사용한다면 `./gradlew bootRun` 으로 바로 실행이 가능하다. `./gradlew build`로 실행 가능한 JAR 파일로 만들 수 있다. `java -jar build/libs/gs-uploading-files-0.1.0.jar` 명령어로 JAR 파일을 실행한다.

Maven을 사용한다면 `./mvnw spring-boot:run` 으로 바로 실행할 수 있다. JAR 파일은 `./mvnw clean package` 으로 생성할 수 있다.

## 파일 업로드 테스트

파일 업로드 단위 테스트는 다음과 같다.  

```java
@AutoConfigureMockMvc
@SpringBootTest
public class FileUploadTests {

	@Autowired
	private MockMvc mvc;

	@MockBean
	private StorageService storageService;

	@Test
	public void shouldListAllFiles() throws Exception {
		given(this.storageService.loadAll())
				.willReturn(Stream.of(Paths.get("first.txt"), Paths.get("second.txt")));

		this.mvc.perform(get("/")).andExpect(status().isOk())
				.andExpect(model().attribute("files",
						Matchers.contains("http://localhost/files/first.txt",
								"http://localhost/files/second.txt")));
	}

	@Test
	public void shouldSaveUploadedFile() throws Exception {
		MockMultipartFile multipartFile = new MockMultipartFile("file", "test.txt",
				"text/plain", "Spring Framework".getBytes());
		this.mvc.perform(multipart("/").file(multipartFile))
				.andExpect(status().isFound())
				.andExpect(header().string("Location", "/"));

		then(this.storageService).should().store(multipartFile);
	}

	@SuppressWarnings("unchecked")
	@Test
	public void should404WhenMissingFile() throws Exception {
		given(this.storageService.loadAsResource("test.txt"))
				.willThrow(StorageFileNotFoundException.class);

		this.mvc.perform(get("/files/test.txt")).andExpect(status().isNotFound());
	}

}
```

`MockMultipartFile` 객체로 껍데기만 있는 멀티파트 파일을 만들 수 있다. `StorageService` 객체에 `@MockBean` 어노테이션을 붙여 사용한다. 기존에 사용하는 Bean의 인터페이스만 가져오고 내부 구현은 사용자에게 위임한 형태다. 해당 Bean의 어떤 메소드가 어떤 값이 입력 되면 어떤 값이 리턴 되어야 한다는 내용 모두 개발자 필요에 의해서 조작이 가능합니다.

`given` 메소드는 해당 Mock Bean이 어떤 행동을 취하면 어떤 결과를 반환해야 한다, 를 선언한다. 해당 테스트 코드를 실행 시 Mock Bean이 주입되어 실행된다. 메소드 호출 시에 `given`에서 설정한 대로 값이 반환되는 것을 확인할 수 있다.

통합 테스트는 다음과 같다.

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class FileUploadIntegrationTests {

	@Autowired
	private TestRestTemplate restTemplate;

	@MockBean
	private StorageService storageService;

	@LocalServerPort
	private int port;

	@Test
	public void shouldUploadFile() throws Exception {
		ClassPathResource resource = new ClassPathResource("testupload.txt", getClass());

		MultiValueMap<String, Object> map = new LinkedMultiValueMap<String, Object>();
		map.add("file", resource);
		ResponseEntity<String> response = this.restTemplate.postForEntity("/", map,
				String.class);

		assertThat(response.getStatusCode()).isEqualByComparingTo(HttpStatus.FOUND);
		assertThat(response.getHeaders().getLocation().toString())
				.startsWith("http://localhost:" + this.port + "/");
		then(storageService).should().store(any(MultipartFile.class));
	}

	@Test
	public void shouldDownloadFile() throws Exception {
		ClassPathResource resource = new ClassPathResource("testupload.txt", getClass());
		given(this.storageService.loadAsResource("testupload.txt")).willReturn(resource);

		ResponseEntity<String> response = this.restTemplate
				.getForEntity("/files/{filename}", String.class, "testupload.txt");

		assertThat(response.getStatusCodeValue()).isEqualTo(200);
		assertThat(response.getHeaders().getFirst(HttpHeaders.CONTENT_DISPOSITION))
				.isEqualTo("attachment; filename=\"testupload.txt\"");
		assertThat(response.getBody()).isEqualTo("Spring Framework");
	}

}
```

감사합니다.

## 참고 링크

- [Spring File Upload](https://spring.io/guides/gs/uploading-files/)
- [스프링 프레임워크 뷰로 사용되는 Thymeleaf 와 jsp 의 비교](https://offbyone.tistory.com/410)
- [SpringBoot @MockBean, @SpyBean 소개](https://jojoldu.tistory.com/226)