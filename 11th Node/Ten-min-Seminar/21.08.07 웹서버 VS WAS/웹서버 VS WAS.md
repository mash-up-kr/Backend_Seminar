# 웹서버 VS WAS



### Static Pages VS Dynamic Pages

---

<img src="./images/2.PNG" alt="2" style="zoom:50%;" />

**정적 페이지(Static Pages)**

- 그대로 제공되는 것(served as-is)
- HTML, JavaScript, CSS 등의 언어로 작성된 것
- 서버가 웹 페이지에 대한 요청을 받으면 별도의 프로세스 없이 응답으로 보내줌
- 수동으로 변경하지 않는 이상 항상 동일하게 유지됨

**동적 페이지(Dynamic Pages)**

- 서버가 컨텐츠를 처리하는 것, 컨텐츠를 DB로부터 생성하는 것
- CGI, AJAX, ASP, ASP.NET 등과 같은 언어로 작성된 것
- 페이지의 내용이 그때그때 다름
- 정적 페이지에 비해 로드 시간이 많이 걸림



### Web Server

---

**개념**

- 웹 서버가 설치되어 있는 컴퓨터(하드웨어)

- 클라이언트로부터 요청을 받아 정적인 컨텐츠를 제공하는 컴퓨터 프로그램(소프트웨어)



**기능**

- WAS를 거치지 않고 바로 정적인 컨텐츠 제공
- 동적인 컨텐츠에 대한 요청을 WAS에 전달, WAS에서 처리한 결과물을 클라이언트에 응답



**예**

- Apache Server, Nginx, IIS 등



### WAS(Web Application Server)

---

**개념**

- 동적인 컨텐츠의 제공을 위한 어플리케이션 서버
- HTTP를 통해 컴퓨터나 장치에 있는 어플리케이션을 수행해주는 미들웨어
- 웹서버+웹 컨테이너
- 웹 컨테이너? 동적인 컨텐츠를 처리해 웹 서버에 정적인 파일로 만들어 보내주는 모듈



**기능**

- 프로그램 실행 환경과 DB 접속 기능 제공
- 여러개의 트랜잭션 관리
- 비즈니스 로직 수행



**예**

- Tomcat, JBoss, Jeus, Web Sphere 등



### 웹서버와 WAS 분리 이유

---

1. **서버 부하 방지**
   - WAS는 비즈니스 로직을 처리하여 동적인 컨텐츠를 제공하므로, 단순한 정적 컨텐츠는 웹서버가 빠르게 처리하는 것이 좋다
   - 만약 WAS가 정적 컨텐츠까지 처리한다면, 정적 데이터 처리로 인한 부하가 커지고 동적 컨텐츠의 처리 또한 지연되어 수행 속도가 느려지게 된다.
2. **물리적으로 분리하여 보안 강화**
   - SSL에 대한 암복호화 처리에 웹서버를 사용
3. **웹서버에 여러 대의 WAS 연결 가능**
   - 규모가 큰 서비스에서 요청을 분산하여 처리할 수 있도록 WAS를 여러 대 연결
   - 웹서버가 요청을 받고 WAS들에 적절하게 배분
   - 이와 같은 로드밸런싱으로 인해 하나의 WAS가 처리하는 요청 양이 줄어들어 안정적인 서비스 운영 가능
4. **여러 웹 어플리케이션 서비스 가능**
   - ex: Java 서버, PHP 서버를 함께 사용 가능



### Web Service Architecture

---

이렇게 웹서버와 WAS를 이용해 웹 서비스를 제작할 수 있으며, 다양한 아키텍처가 존재한다.

1. **Client -> Web Server -> DB**

<img src="./images/3.PNG" alt="3" style="zoom:50%;" />

2. **Client -> WAS -> DB**

<img src="./images/1.PNG" alt="1" style="zoom:50%;" />

3. **Client -> Web Server -> WAS -> DB**

<img src="./images/4.PNG" alt="4" style="zoom:50%;" />



# 참고

https://gmlwjd9405.github.io/2018/10/27/webserver-vs-was.html

https://developer.mozilla.org/ko/docs/Learn/Common_questions/What_is_a_web_server

https://medium.com/pocs/node-js-3-%EB%8F%99%EC%A0%81-%ED%8E%98%EC%9D%B4%EC%A7%80-%EC%83%9D%EC%84%B1-1695f802c40e

https://www.geeksforgeeks.org/difference-between-static-and-dynamic-web-pages/

https://tridentofposeidon2.tistory.com/m/43

https://www.educative.io/edpresso/web-server-vs-application-server

https://velog.io/@change/WEB%EC%84%9C%EB%B2%84-WAS-%EB%B6%84%EB%A6%AC-%EC%9D%B4%EC%9C%A0