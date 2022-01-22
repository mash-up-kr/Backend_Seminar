# gRPC

발표 날짜: 2021년 9월 25일
발표 여부: Yes
발표자: 이소연

### gRPC

> 다른 서비스 간 언어 제약 없이 사용할 수 있는 오픈소스 고성능 원격 프로시저 호출 프레임워크이다.
> 

![Untitled](gRPC%2093f1a9787e324d4290ee95f4759264d3/Untitled.png)

위의 그림을 살펴보면 client는 java 혹은 ruby로 구현되어 있다. 또한 서버의 경우 c++로 구현되어 있는데 이렇게 다른 환경으로 구성되어 있는  것을 통신 가능하게 해준다.

정확히 말하자면 데이터를 주고 받을 때 프로토콜 버퍼들 사이에서 gRPC 통신을 한다.

gRPC는 기본적으로 프로토콜 버퍼를 사용한다. 

프로토콜 버퍼는 csv, json, xml과 같은 데이터 구조이다. 프로토콘 버퍼와 다른 데이터 구조와의 차이는 프로토콜 버퍼의 경우 구글에서 제작하였고 직렬화가 아닌 바이너리 파일로 되어면서 속도가 더 빠르다는 장점이 있다.

특징:

- http/2 : 한 connection으로 동시에 여러 개 메시지를 주고 받고 header를 압축하여 중복 제거 후 전달하기에 http/1.1에 비해 훨씬 효율적.
- ProtoBuf: 바이너리 파일(사람 읽을 수 x)

### Nest에서 gRPC

1. 필요한 패키지 설치

```tsx
npm i --save grpc @grpc/proto-loader
```

1. main.ts 변경

```tsx
const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
  transport: Transport.GRPC,
  options: {
    package: 'hero',
    protoPath: join(__dirname, 'hero/hero.proto'),
  },
});
```

1. nest-cli 변경

```tsx
{
  "compilerOptions": {
    "assets": ["**/*.proto"],
    "watchAssets": true
  }
}
```

 

gRPC의 경우 .proto 파일에 응답 형식을 지정해 놓는다. 이를 dist에도 복사되도록 watchAsset 을 추가.

### Proto 파일 작성

```tsx
syntax = "proto3";

package hero;

service HeroesService {
  **rpc FindOne (HeroById) returns (Hero) {}**
}

message HeroById {
  **int32 id = 1;**
}

message Hero {
  int32 id = 1;
  string name = 2;
}
```

 proto 파일에서 service를 지정 하고 해당 리턴하는 부분을 모두 명시해준다.(Protocol Buffer의 기본 정보를 명세 하여 인코딩, 디코딩)

proto파일에서는 응답으로 heroById일 때 id가 요청, 응답은 hero로 되는데 인자와 해당 인자에 번호를 부여한다.

일반적으로 proto 파일은 protoidl 레포를 따로 두고 서브 모듈로 사용한다고 알고있다.

패키지 이름 명시, 

서비스 명시: RPC를 통해 서버가 클라이언트에게 제공할 함수의 형태를 정의.

![Untitled](gRPC%2093f1a9787e324d4290ee95f4759264d3/Untitled%201.png)

### controller 작성

```tsx
@Controller()
export class HeroesController {
  **@GrpcMethod('HeroesService', 'FindOne') // @GrpcMethod('UserService', 'GoogleLogin')**
  findOne(data: HeroById, metadata: Metadata, call: ServerUnaryCall<any>): Hero {
    const items = [
      { id: 1, name: 'John' },
      { id: 2, name: 'Doe' },
    ];
    return items.find(({ id }) => id === data.id);
  }
}
```

`GrpcMethod` : proto파일에서 지정한 서비스 명과 메소드 명을 주입 : 인수는 모두 선택사항

첫번째 인수를 선택사항으로 두려면 HeroesController ⇒ HeroesService로 이름을 변경하여 proto파일에서 연관시킬 수 있도록 해주어야한다.

### Client

proto 파일에 정의된 서비스를 사용할 수 있는 client 

서비스가 main - auth - resource 이렇게 3가지로 나뉘었다고 하면

main에 클라이언트로 auth, resoucre 등록해주면 된다.

- `ClientGprc` 객체 얻는 법
    - `ClientModule` 가져오기

```tsx
imports: [
  ClientsModule.register([
    {
      name: 'HERO_PACKAGE',
      **transport: Transport.GRPC**,
      options: {
        package: 'hero', // proto 파일에 명시
        protoPath: join(__dirname, 'hero/hero.proto'),
      },
    },
  ]),
];
```

```tsx
@Injectable()
export class AppService implements OnModuleInit {
  private heroesService: HeroesService;

  **constructor(@Inject('HERO_PACKAGE') private client: ClientGrpc) {}
// clietGrpc 객체 삽입
// 이를 통해서 getService로 서비스가져올 수 있다.**

  onModuleInit() {
    this.heroesService = this.client.getService<HeroesService>('HeroesService');
  }

  getHero(): Observable<string> {
    return this.heroesService.findOne({ id: 1 });
  }
}
```

// `ClientProxyFactory` 를 이용해서 동적으로 구성된 클라이언트를 삽입할 수 있다고한다.

→ 잘 모르겠음..

 → 아마 이름이 FindOne, findOne이면 잘 연결시켜준다는거같은데.. ㅜ

느낀점: 마이크로서비스 통신 어려운거같아요 ㅋ_ㅋ

## MSA 노우!

누구야!