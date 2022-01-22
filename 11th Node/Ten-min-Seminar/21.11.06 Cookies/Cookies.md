# Cookies

발표 날짜: 2021년 11월 6일
발표 여부: Yes
발표자: 임지영

### 🍪 쿠키란

- 사용자의 브라우저에 저장되는 작은 데이터
- 이름, 값의 쌍으로 되어 있으며 구체적으로는 이름, 값, 만료 날짜/시간, 경로 정보 등의 정보가 담겨 있다.

![스크린샷 2021-11-03 오전 12.27.18.png](Cookies%203e7e3b2dae604261b3ce63e40cdcbfdb/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2021-11-03_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_12.27.18.png)

### 세션과의 차이

- 쿠키는 서버의 자원을 전혀 사용하지 않는 데 비해 세션은 쿠키를 기반하고 있지만 정보를 서버측에서 관리.
- 보안: 쿠키 < 세션 (쿠키는 로컬에 저장되기 때문에 request에서 스니핑 당할 우려 등이 있으나 세션은 세션 id만 저장하고 나머지는 서버에서 처리)
- 요청 속도: 쿠키 > 세션

### Nest에서 쿠키 사용하기 (with express)

**Step1. 의존성 설치**

```tsx
$ npm i cookie-parser
$ npm i -D @types/cookie-parser
```

**Step2. 글로벌 적용**

```tsx
import * as cookieParser from 'cookie-parser';

app.use(cookieParser());
// app.use(cookieParser(secret?, options?));
```

- `secret(optional)`: 쿠키 서명에 사용되는 문자열 또는 배열. 배열의 경우 순서대로 각 secret을 사용.
- `options(optional)`: cookie.parse에 전달되는 객체.
    - `decode`: 쿠키의 값을 디코딩할 때 사용할 메서드를 지정. 기본 메서드는 `decodeURIComponent`.

**쿠키 읽기**

```tsx
import { Req } from '@nestjs/common';
import { Request } from 'express';

@Get()
findAll(@Req() request: Request) {
  console.log(request.cookies);
	// or "request.cookies['cookieKey']"
  // or console.log(request.signedCookies);
}
```

**쿠키 보내기**

```tsx
@Get()
findAll(@Res({ passthrough: true }) response: Response) {
  response.cookie('key', 'value')
}
```

- `passthrough: true` → 쿠키만 설정하고 나머지는 프레임워크가 알아서 응답 처리하도록 하기 위해 사용.

### Nest에서 쿠키 사용하기 (with Fastify)

**Step1. 의존성 설치**

```tsx
$ npm i fastify-cookie
```

**Step2. 글로벌 적용**

```tsx
import fastifyCookie from 'fastify-cookie';

const app = await NestFactory.create<NestFastifyApplication>(
  AppModule,
  new FastifyAdapter(),
);
app.register(fastifyCookie, {
  secret: 'my-secret', // for cookies signature
});
```

**쿠키 읽기**

```tsx
@Get()
findAll(@Req() request: FastifyRequest) { // request 타입 빼곤 express와 동일
  console.log(request.cookies);
}
```

**쿠키 보내기**

```tsx
@Get()
findAll(@Res({ passthrough: true }) response: FastifyReply) {
  response.setCookie('key', 'value') // express는 .cookie()
}
```

### Custom decorator

쿠키에 쉽게 접근하기 위해 아래와 같이 데코레이터를 만들어 사용할 수 있다.

```tsx
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const Cookies = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return data ? request.cookies?.[data] : request.cookies;
  },
);
```

```tsx
@Get()
findAll(@Cookies('name') name: string) {}
```