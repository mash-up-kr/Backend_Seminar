# WebSocket

발표 날짜: 2021년 9월 25일
발표 여부: Yes
발표자: 김선재

## 1. Gateways : 게이트웨이란?

Nest 에서 게이트웨이는 `@WebSocketGateway()` 데코레이터로 주석이 달린 클래스다.
플랫폼에 의존적이지 않고 웹소켓 어댑터에 맞춰 호환된다.

빌트인 웹소켓 플랫폼은 다음 두 가지가 있다.

- Socket.io
- WS

![출처 - [https://docs.nestjs.kr/websockets/gateways](https://docs.nestjs.kr/websockets/gateways)](WebSocket%200d6db0f8dca045ecbd60f34ef19824c1/Untitled.png)

출처 - [https://docs.nestjs.kr/websockets/gateways](https://docs.nestjs.kr/websockets/gateways)

참고로 게이트웨이는 프로바이더로 취급되며, 다른 클래스 생성자에 종속성을 주입할 수 있다.

브라우저와 호환되는 WS 보다는 [Socket.io](http://socket.io) 라이브러리의 기능이 더 많아 예제로 Socket.io 플랫폼을 사용했다.
Nest v8 에서 사용하는 Socket.io 라이브러리의 버전은 v4 이다.
( 참고로 Nest v7 에서 사용하는 Socket.io 라이브러리의 버전은 v2 이다. )

## 2. 설치하기

아래 명령어로 웹소켓 서버에 필요한 라이브러리를 설치받는다.

```tsx
$ npm i --save @nestjs/websockets @nestjs/platform-socket.io
```

## 3. 가볍게 훑어보기

```tsx
import { WebSocketGateway } from '@nestjs/websockets';

@WebSocketGateway(80)
export class OneToOneChattingGateway {}

@WebSocketGateway(80, { namespace: 'events' })
export class OneToOneChattingGateway {}
```

`namespace` 옵션으로 네임스페이스를 지정할 수 있다.
`ws://localhost:80/events` 식으로 Path 를 지정해줄 수 있다.

클라이언트로부터 메시지를 받기 위해선, `@SubscribeMessage('')` 데코레이터로 메시지를 구독해야 합니다.
아래는 똑같은 데이터로 클라이언트에게 응답하는 메서드입니다.

```tsx
import {
  MessageBody,
	SubscribeMessage,
} from '@nestjs/websockets';

@SubscribeMessage('echo')
sendEcho(@MessageBody() chat: string): string {
  return chat;
}
```

`@MessageBody()` 에 인자로 속성키를 전달하여 수신 메시지 본문에서 가져올 수도 있다.

```tsx
import {
  MessageBody,
	SubscribeMessage,
} from '@nestjs/websockets';

@SubscribeMessage('echoId')
sendEchoId(@MessageBody('id') id: number): number {
	// id === messageBody.id
  return id;
}
```

`@MessageBody()` 데코레이터를 사용하지 않을 시 아래 코드와 동일합니다.

```tsx
import {
	SubscribeMessage,
} from '@nestjs/websockets';

import { Socket } from 'socket.io';

@SubscribeMessage('echo')
sendEcho(client: Socket, data: string): string {
  return chat;
}
```

첫 번째 매개변수에는 플랫폼별 소켓 인스턴스를 가진다. 현재 예에선 [Socket.io](http://socket.io) 의 `Socket` 
두 번째는 클라이언트에서 수신한 데이터다.
이 방법은 테스트 시에 `client: Socket` 인스턴스를 모의화해야하므로 권장되지는 않는다.

서버에서 연결된 소켓 인스턴스에 접근하려면, `@ConnectedSocket()` 데코레이터로 가져올 수 있다.

```tsx

import {
	ConnectedSocket,
  MessageBody,
	SubscribeMessage,
} from '@nestjs/websockets';

@SubscribeMessage('echo')
sendEcho(
	@MessageBody() chat: string,
	@ConnectedSocket() client: Socket,
): string {
  return chat;
}
```

직접 `client.emit()` 을 해주는 것과 메서드에서 `return` 해주는 건 어떤 차이점이 있을까?

첫 번째의 경우 클라이언트에서 따로 메시지를 수신하도록 설정해야 한다.

```tsx
// 서버 코드

import {
	ConnectedSocket,
  MessageBody,
	SubscribeMessage,
} from '@nestjs/websockets';

@SubscribeMessage('echo')
sendEcho(
	@MessageBody() chat: string,
	@ConnectedSocket() client: Socket,
): string {
	client.emit('sendEcho', chat);
}

// 클라이언트 코드

io.on('sendEcho', (chat) => {
	console.log(chat);
});
```

두 번째 `return` 의 경우 클라이언트에서 `emit()` 메서드에 ack (acknowledgement) 콜백 매개변수를 둬야한다.

```tsx
// 서버 코드

import {
	ConnectedSocket,
  MessageBody,
	SubscribeMessage,
} from '@nestjs/websockets';

@SubscribeMessage('echo')
sendEcho(
	@MessageBody() chat: string,
	@ConnectedSocket() client: Socket,
): string {
	return chat;
}

// 클라이언트 코드

// socket.emit(eventName[, ...args][, ack])

io.emit('echo', chat, (responseChat) => {
	console.log(responseChat);
});
```

***클라이언트와 원만하게 협의할 것!***

클라이언트에게 응답하고 싶지 않을 땐 `return` 을 건너뛰거나 명시적으로 falsy 한 값(예, `undefined`)을 반환한다. 

## 4. Return 으로 다른 이벤트명으로 전달하기

기본적으로 return 할 시 `acknowledgement` 콜백과 연결된다.
`return` 문으로 다른 이벤트로 보내기 위해선 두 가지 속성으로 구성된 객체를 반환해야 한다.
생성된 이벤트의 이름인 `event` 와 클라이언트로 전달하는 `data` 두 개로 구성된 객체를 반환한다.

```tsx
import {
  MessageBody,
	SubscribeMessage,
	WsResponse,
} from '@nestjs/websockets';

@SubscribeMessage('events')
handleEvent(@MessageBody() data: unknown): WsResponse<unknown> {
  const event = 'events';
  return { event, data };
}
```

```tsx
socket.on('events', (data) => console.log(data));
```

개인적으로는 위 `return { event, data }` 가 다른 이벤트로 Emit 한다는 걸 이해하기 힘들 것 같아 직접 `ConnectedSocket` 에 `emit` 를 해주는 편

- 이 때는 테스트를 할 때 `client: Sokcet` 을 모의화해야 하는 번거로움이 있다.

## 5. 비동기 응답

메시지 핸들러는 비동기로도 응답할 수 있다. `Observable` 을 반환해줄 수 있다. 스트림이 완료될 때 까지 결과값이 보내지게 된다. 즉 아래 예에선 3번 응답하게 된다.

```tsx
import {
  MessageBody,
	SubscribeMessage,
	WsResponse,
} from '@nestjs/websockets';

import { Observable } from 'rxjs';

@SubscribeMessage('events')
onEvent(@MessageBody() data: unknown): Observable<WsResponse<number>> {
  const event = 'events';
  const response = [1, 2, 3];

  return from(response).pipe(
    map(data => ({ event, data })),
  );
}
```

## 6. 라이프사이클

[라이프사이클](https://www.notion.so/b97dc2bcf1514f98bf09165e4282b40e)

서버 인스턴스에 접근하려면, 위 라이프사이클의 `OnGatewayInit` 구현의 `afterInit(server: Server)` 으로 가져오거나, `@WebSocketServer()` 데코레이터로 가져올 수 있다.

```tsx
import { WebSocketServer } from '@nestjs/websockets';

@WebSocketServer()
server: Server;
```

## 7. Exception Filter

기존 HTTP 서버의 Exception Filter 와 차이는 `HttpException` 을 던지는 대신 `WsException` 을 던져야 한다.

```tsx
import { WsException } from '@nestjs/websockets';

throw new WsException('Invalid credentials.');
```

```tsx
{
  status: 'error',
  message: 'Invalid credentials.'
}
```

위 형태로 클라이언트에게 보내줄 수 있다.

게이트웨이에서 `@UseFilters()` 데코레이터를 붙여 사용한다.

```tsx
@UseFilters(new WsExceptionFilter())
@SubscribeMessage('events')
onEvent(client, data: any): WsResponse<any> {
  const event = 'events';
  return { event, data };
}
```

예외 필터를 커스텀하는 방법은 다음과 같다. 기본 예외 처리는 `BaseWsExceptionFilter` 에 위임하고 이를 확장하여 `catch()` 메서드를 오버라이딩할 수 있다.

```tsx
import { Catch, ArgumentsHost } from '@nestjs/common';
import { BaseWsExceptionFilter } from '@nestjs/websockets';

@Catch()
export class AllExceptionsFilter extends BaseWsExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
		// Buisiness logics...

    super.catch(exception, host);
  }
}
```

## 8. Pipes

기본 HTTP 서버의 파이프와 다른점은 Exception Filter 와 마찬가지로 `HttpException` 대신 `WsException` 을 사용해야 한다. 모든 파이프는 아래 코드를 예를 들어 `data` 매개변수에만 적용된다. ( `client` 인스턴스의 유효성을 검사하거나 변환하는 작업을 할 필요가 없다고 Nest 에서 생각하는 듯 하다. )

```tsx
@UsePipes(new ValidationPipe())
@SubscribeMessage('events')
handleEvent(client: Client, data: unknown): WsResponse<unknown> {
  const event = 'events';
  return { event, data };
}
```

## 9. Guards

마찬가지로 기본 HTTP 가드와 다르지 않고, 예외를 `WsException` 으로 사용하면 된다.

```tsx
@UseGuards(AuthGuard)
@SubscribeMessage('events')
handleEvent(client: Client, data: unknown): WsResponse<unknown> {
  const event = 'events';
  return { event, data };
}
```

## 10. Interceptors

마찬가지로 기본 인터셉터와 다르지 않고, 예외를 `WsException` 으로 사용하면 된다.

```tsx
@UseInterceptors(new TransformInterceptor())
@SubscribeMessage('events')
handleEvent(client: Client, data: unknown): WsResponse<unknown> {
  const event = 'events';
  return { event, data };
}
```

## 11. 어댑터

`WebSocketAdapter` 인터페이스를 사용하여 아래 메서드를 구현하면 커스텀 어댑터를 만들 수 있다.

1. `create` : 전달된 인수를 기반으로 소켓 인스턴스를 만듭니다.
2. `bindClientConnect` : 클라이언트 연결 이벤트를 바인드합니다.
3. `bindClientDisconnect` : 클라이언트 연결 해제 이벤트를 바인딩합니다(선택사항*).
4. `bindMessageHandlers` : 수신 메시지를 해당 메시지 핸들러에 바인드합니다.
5. `close` : 서버 인스턴스를 종료합니다.

예시로 [socket.io](http://socket.io)-redis 패키지를 사용하여 socket.io 패키지의 `IoAdapter` 를 확장하여 socket.io 서버를 인스턴스화하는 단일 메소드를 재정의한다.

```tsx
$ npm i --save socket.io-redis
```

```tsx
import { IoAdapter } from '@nestjs/platform-socket.io';
import { RedisClient } from 'redis';
import { ServerOptions } from 'socket.io';
import { createAdapter } from 'socket.io-redis';
const pubClient = new RedisClient({ host: 'localhost', port: 6379 });
const subClient = pubClient.duplicate();
const redisAdapter = createAdapter({ pubClient, subClient });

export class RedisIoAdapter extends IoAdapter {
    createIOServer(port: number, options?: ServerOptions): any {
    const server = super.createIOServer(port, options);
    server.adapter(redisAdapter);
    return server;
  }
}
```

```tsx
const app = await NestFactory.create(AppModule);
app.useWebSocketAdapter(new RedisIoAdapter(app));
```

Ws 어댑터를 사용하려면 다음과 같다. [socket.io](http://socket.io) 보다는 제공되는 기능이 훨씬 적지만, 브라우저와 완벽하게 호환되고 속도가 빠르다는 장점이 있다. ( 예를들어, namespace 를 지원하지 않음. 모방하기 위해 `path` 로 서버를 다른 경로에 지정할 수 있다. )

```tsx
$ npm i --save @nestjs/platform-ws
```

```tsx
const app = await NestFactory.create(AppModule);
app.useWebSocketAdapter(new WsAdapter(app));
```

```tsx
import * as WebSocket from 'ws';
import { WebSocketAdapter, INestApplicationContext } from '@nestjs/common';
import { MessageMappingProperties } from '@nestjs/websockets';
import { Observable, fromEvent, EMPTY } from 'rxjs';
import { mergeMap, filter } from 'rxjs/operators';

export class WsAdapter implements WebSocketAdapter {
  constructor(private app: INestApplicationContext) {}

  create(port: number, options: any = {}): any {
    return new WebSocket.Server({ port, ...options });
  }

  bindClientConnect(server, callback: Function) {
    server.on('connection', callback);
  }

  bindMessageHandlers(
    client: WebSocket,
    handlers: MessageMappingProperties[],
    process: (data: any) => Observable<any>,
  ) {
    fromEvent(client, 'message')
      .pipe(
        mergeMap(data => this.bindMessageHandler(data, handlers, process)),
        filter(result => result),
      )
      .subscribe(response => client.send(JSON.stringify(response)));
  }

  bindMessageHandler(
    buffer,
    handlers: MessageMappingProperties[],
    process: (data: any) => Observable<any>,
  ): Observable<any> {
    const message = JSON.parse(buffer.data);
    const messageHandler = handlers.find(
      handler => handler.message === message.event,
    );
    if (!messageHandler) {
      return EMPTY;
    }
    return process(messageHandler.callback(message.data));
  }

  close(server) {
    server.close();
  }
}
```

위 예시는 WSAdapter 가 어떻게 구현되었는지 데모 수준으로 구현한 예시다.

## 12. 클라이언트 Socket 인스턴스에 정보 저장하기

실무에서 사용했을 때는 각 클라이언트 `Socket` 인스턴스에 정보를 저장하여 사용했다.
각 클라이언트마다 어떤 상태인지, 어떤 데이터를 가지고 있는지를 Socket 인스턴스 안에 저장하니 굉장히 사용하기 편리했음

```tsx
import { Socket as SocketIO } from 'socket.io';

export interface Socket extends SocketIO {
  user: User;
  storage: Storage;
}

export interface User {
	email: string;
	nickname: string;
	age: number;
}

export interface Storage {
	token: string;
	status: Status;
	room: string;
}

export interface Status {
	state: StateType;
	isAuth: boolean;
	authAt: Date | null;
}

export type StateType = "BEFORE_AUTHENTICAITON" | "AUTHENTICATION" | "IN_CHAT";
```

```tsx
handleConnection(client: Socket) {
	client.user = {} as User;
	client.storage = {
		...client.storage,
		status: {
			state: "BEFORE_AUTHENTICATION"
			isAuth: false,
			authAt: null,
		}
}
```