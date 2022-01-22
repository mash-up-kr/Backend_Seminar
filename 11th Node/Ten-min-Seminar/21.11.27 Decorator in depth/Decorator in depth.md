# Decorator in depth

발표 날짜: 2021년 11월 27일
발표 여부: Yes
발표자: 정세훈

```tsx
type ClassDecorator = <TFunction extends Function>(target: TFunction) => TFunction | void;

type PropertyDecorator = (target: Object, propertyKey: string | symbol) => void;

type MethodDecorator = <T>(
  target: Object,
  propertyKey: string | symbol,
  descriptor: TypedPropertyDescriptor<T>
) => TypedPropertyDescriptor<T> | void;

type ParameterDecorator = (target: Object, propertyKey: string | symbol, parameterIndex: number) => void;

interface TypedPropertyDescriptor<T> {
    enumerable?: boolean;
    configurable?: boolean;
    writable?: boolean;
    value?: T;
    get?: () => T;
    set?: (value: T) => void;
}

```

### Decorator가 컴파일되는 모습

```tsx
type Config = {};
type User = {};
class Database {}

declare function Inject(key: string | symbol): ParameterDecorator;
declare function Query(key: string): ParameterDecorator;
declare function Controller(path?: string): ClassDecorator;
declare function Encrypt(): PropertyDecorator;
declare function Cached(key?: string): MethodDecorator;
declare function ApiResponse(option?: any): MethodDecorator;

@Controller()
class UserController {
  constructor(
    @Inject('CONFIG') private readonly config: Config,
    private readonly database: Database,
  ) {}

  @Encrypt()
  secret: string = '1234';

  @ApiResponse()
  @Cached('get-user')
  async getUser(@Query(':id') id: string) {
    return {};
  }
}

```

위 코드는 다음처럼 컴파일된다

```tsx
"use strict";
var __decorate = (this && this.__decorate) || function (decorators, target, key, desc) {
    var c = arguments.length, r = c < 3 ? target : desc === null ? desc = Object.getOwnPropertyDescriptor(target, key) : desc, d;
    if (typeof Reflect === "object" && typeof Reflect.decorate === "function") r = Reflect.decorate(decorators, target, key, desc);
    else for (var i = decorators.length - 1; i >= 0; i--) if (d = decorators[i]) r = (c < 3 ? d(r) : c > 3 ? d(target, key, r) : d(target, key)) || r;
    return c > 3 && r && Object.defineProperty(target, key, r), r;
};
var __metadata = (this && this.__metadata) || function (k, v) {
    if (typeof Reflect === "object" && typeof Reflect.metadata === "function") return Reflect.metadata(k, v);
};
var __param = (this && this.__param) || function (paramIndex, decorator) {
    return function (target, key) { decorator(target, key, paramIndex); }
};
class Database {
}
let UserService = class UserService {
    constructor(config, database) {
        this.config = config;
        this.database = database;
        this.secret = '1234';
    }
    async getUser(id) {
        return {};
    }
};
__decorate([
    Encrypt(),
    __metadata("design:type", String)
], UserService.prototype, "secret", void 0);
__decorate([
    ApiResponse(),
    Cached('get-user'),
    __param(0, Query(':id')),
    __metadata("design:type", Function),
    __metadata("design:paramtypes", [String]),
    __metadata("design:returntype", Promise)
], UserService.prototype, "getUser", null);
UserService = __decorate([
    Controller(),
    __param(0, Inject('CONFIG')),
    __metadata("design:paramtypes", [Object, Database])
], UserService);

```

만약 emitDecoratorMetadata 옵션을 키지 않을 경우

```tsx
"use strict";
var __decorate = (this && this.__decorate) || function (decorators, target, key, desc) {
    var c = arguments.length, r = c < 3 ? target : desc === null ? desc = Object.getOwnPropertyDescriptor(target, key) : desc, d;
    if (typeof Reflect === "object" && typeof Reflect.decorate === "function") r = Reflect.decorate(decorators, target, key, desc);
    else for (var i = decorators.length - 1; i >= 0; i--) if (d = decorators[i]) r = (c < 3 ? d(r) : c > 3 ? d(target, key, r) : d(target, key)) || r;
    return c > 3 && r && Object.defineProperty(target, key, r), r;
};
var __param = (this && this.__param) || function (paramIndex, decorator) {
    return function (target, key) { decorator(target, key, paramIndex); }
};
class Database {
}
let UserService = class UserService {
    constructor(config, database) {
        this.config = config;
        this.database = database;
        this.secret = '1234';
    }
    async getUser(id) {
        return {};
    }
};
__decorate([
    Encrypt()
], UserService.prototype, "secret", void 0);
__decorate([
    ApiResponse(),
    Cached('get-user'),
    __param(0, Query(':id'))
], UserService.prototype, "getUser", null);
UserService = __decorate([
    Controller(),
    __param(0, Inject('CONFIG'))
], UserService);

```

### `__decorate`를 찬찬히 살펴보자.

반복적으로 나오는 다음 구문은 스코프에 해당 함수가 있는지를 체크할뿐이다. 따라서 지워버려도 된다.

```tsx
var foo = (this && this.foo) || function () { ... }

```

코드를 조금 정리하면 다음같은 모습이 된다.

```tsx
function __decorate(decorators, target, key, desc) {
  let c = arguments.length;

  // arguments가 3미만일 경우 -> decorators와 target만 주어졌다 -> class decorator이다.
  // arguments가 3개 이상일 경우 -> method, property, parameter 데코레이터이다.
  let r = c < 3 ? target : desc === null ? (desc = Object.getOwnPropertyDescriptor(target, key)) : desc;

  if (typeof Reflect === 'object' && typeof Reflect.decorate === 'function') {
    r = Reflect.decorate(decorators, target, key, desc);
  } else {
    let d;
    // decorators 배열을 역순으로 순회하면서 적용한다.
    for (const decorator of decorators.reverse()) {
      if(c < 3) {
       // class decorator
        r = decorator(r) ?? r;
      } else if (c > 3) {
        // method, parameter, property decorator
        r = decorator(target, key, r) ?? r; // r은 property descriptor
      } else {
        // 케이스 없음... 왜있는거지?
        r = decorator(target, key) ?? r;
      }
    }
  }

  return c > 3 && r && Object.defineProperty(target, key, r), r;
}

```

### import 'reflect-metadata'는 무슨 역할을 하는걸까?

빌트인 객체(클래스가 아니다) Reflect에 메소드들을 더 추가한다. 그리고 Reflect 네임스페이스에 타입정의를 추가한다.

- [Reflect.decorate](https://github.com/rbuckton/reflect-metadata/blob/master/Reflect.ts#L730)
- [Reflect.metadata](https://github.com/rbuckton/reflect-metadata/blob/master/Reflect.ts#L730)

### Decorator는 어떤 순서로 적용될까??

```tsx
@foo
@bar
@baz
class User {
  constructor(name: string) { ... }
}

```

이 코드는 다음처럼 컴파일된다.

```tsx
let User = class User { ... }
User = __decorate([  // 사실 얘는 Reflect.decorate로 대체된다.
  foo,
  bar,
  baz,
  // 만약 emitDecoratorMetadata가 true일 경우
  __metadata('design:paramtypes', [String]) // 이 또한 Reflect.metadata(key, value)로 대체된다.
], User);

```

### Decorator 패키지들

- [https://github.com/andreypopp/autobind-decorator](https://github.com/andreypopp/autobind-decorator)
- [https://github.com/vcfvct/typescript-retry-decorator](https://github.com/vcfvct/typescript-retry-decorator)
- [https://github.com/vlio20/utils-decorators](https://github.com/vlio20/utils-decorators)

### References

- [https://netbasal.com/behind-the-scenes-how-typescript-decorators-operate-28f8dcacb224](https://netbasal.com/behind-the-scenes-how-typescript-decorators-operate-28f8dcacb224)
- [https://medium.com/google-developers/exploring-es7-decorators-76ecb65fb841](https://medium.com/google-developers/exploring-es7-decorators-76ecb65fb841)