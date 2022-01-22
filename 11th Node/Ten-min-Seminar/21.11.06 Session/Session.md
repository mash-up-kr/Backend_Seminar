# Session

발표 날짜: 2021년 11월 6일
발표 여부: Yes
발표자: 조현아

# HTTP

<aside>
💡 비연결성(Connectionless),비상태성(Stateless)으로 연결 상태가 유지되지 않고, 연결 해제 후에 상태 정보가 저장되지 않는다.

</aside>

이런  특성을 보완하기 위해 쿠키와 세션을 사용해 서버가 클라이언트를 기억할 수 있게 한다!

# 세션

![B2777374-4E16-45D3-8FD0-4BFBC9B8EB55.jpeg](Session%206acabef275c14792aeb288b07de25ccf/B2777374-4E16-45D3-8FD0-4BFBC9B8EB55.jpeg)

<aside>
💡 웹 서버쪽의 컨테이너에 상태를 유지하기 위한 정보를 저장한다.                                                웹 브라우저당 1개씩 생성되어 웹 컨테이너에 저장된다.

</aside>

![0D7CCDAF-726C-4EEA-985E-637E8EB975EC.jpeg](Session%206acabef275c14792aeb288b07de25ccf/0D7CCDAF-726C-4EEA-985E-637E8EB975EC.jpeg)

1. 웹브라우저가 서버에 요청한다.
2. 서버가 해당 웹브라우저(클라이언트)에 Session ID를 부여하고 저장한다. 
3. 서버가 HTTP 헤더(Set-Cookie)에  Session ID를 담아 응답한다.
4. 브라우저는 서버에 요청할 때 Session ID가 담겨있는 쿠키를 HTTP 헤더에 넣어 전송한다.
5. 서버는 세션 ID를 확인하고 유효하면 응답한다.

✔패키지 설치

```jsx
$ npm i express-session
$ npm i -D @types/express-session
```

✔ `express-session` 미들웨어를 글로벌 미들웨어로 적용한다.

```jsx
import * as session from 'express-session';
// somewhere in your initialization file
app.use(
  session({
    secret: 'my-secret',
    resave: false,
    saveUninitialized: false,
  }),
);
```

-`secret`  : 세션 ID 쿠키에 서명하는 데 사용된다.

하나의 문자열이거나 여러 secret의 배열일 수 있다. 비밀 배열이 제공되면 첫번째 요소만 세션 ID 쿠키에 서명하는데 사용된다.

-`resave` : 요청중에 세션이 수정되지 않은 경우에도 세션을 세션 저장소에 다시 저장할지 선택한다.

-`saveUninitialized` :  초기화되지 않은 세션을 저장소에 저장할지 선택한다. 

✔라우트 핸들러 내에서 세션값을 설정하고 읽을 수 있다.

```jsx
@Get()
findAll(@Req() request: Request) {
  request.session.visits = request.session.visits ? request.session.visits + 1 : 1;
}
```

✔`@Session()` 데코레이터를 사용하여 요청에서 세션 객체를 추출할 수 있다.

```jsx
@Get()
findAll(@Session() session: Record<string, any>) {
  session.visits = session.visits ? session.visits + 1 : 1;
}
```

Fastify 사용하면?

✔패키지 설치

```jsx
$ npm i fastify-secure-session
```

✔`fastify-secure-session` 플러그인을 등록

```jsx
import secureSession from 'fastify-secure-session';

// somewhere in your initialization file
const app = await NestFactory.create<NestFastifyApplication>(
  AppModule,
  new FastifyAdapter(),
);
app.register(secureSession, {
  secret: 'averylogphrasebiggerthanthirtytwochars',
  salt: 'mq9hDxBVDbspDR6n',
});
```

✔라우트 핸들러내에서 세션값을 설정하고 읽을 수 있다.

```jsx
@Get()
findAll(@Req() request: FastifyRequest) {
  const visits = request.session.get('visits');
  request.session.set('visits', visits ? visits + 1 : 1);
}
```

✔@Session() 데코레이터를 사용하여 요청에서 세션 객체를 추출할 수 있다.

```jsx
@Get()
findAll(@Session() session: secureSession.Session) {
  const visits = session.get('visits');
  session.set('visits', visits ? visits + 1 : 1);
}
```