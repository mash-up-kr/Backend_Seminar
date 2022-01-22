# caching

발표 날짜: 2021년 11월 20일
발표 여부: Yes
발표자: 김진호

# Caching

- 고성능 데이터 액세스를 제공하는 임시 데이터 저장소 역할
- 캐싱은 앱의 성능을 개선하는 데 도움이 되는 훌륭하고 간단한 **기술**

```tsx
설치 명령어
$ npm install cache-manager
$ npm install -D @types/cache-manager
```

### **In-memory cache**

- Nest는 다양한 캐시 스토리지 제공업체를 위한 통합 API를 제공
- 내장된 것은 메모리내 데이터 저장소

```tsx
import { CacheModule, Module } from '@nestjs/common';
import { AppController } from './app.controller';

@Module({
  imports: [CacheModule.register()],
  controllers: [AppController],
})
export class AppModule {}
```

### 사용법

```tsx
# 조회
const value = await this.cacheManager.get('key');

# 추가
await this.cacheManager.set('key', 'value');

# ttl 활성화
await this.cacheManager.set('key', 'value', { ttl: 1000 }); 

# ttl 비활성화
await this.cacheManager.set('key', 'value', { ttl: 0 });

# 특정 키값 삭제
await this.cacheManager.del('key');

# 전체 캐싱 삭제
await this.cacheManager.reset();
```

### **Auto-caching responses**

- 자동 캐싱 응답을 사용하려면 데이터를 캐시하려는 위치에 `CacheInterceptor`를 연결
- `GET` 만 가능 , @Res 객체를 그대로 리턴하는 곳에서는 사용 불가

```tsx
@Controller()
@UseInterceptors(CacheInterceptor)
export class AppController {
  @Get()
  findAll(): string[] {
    return [];
  }
}

==========================================================
=====================전역으로도 사용 가능 =================
==========================================================
import { CacheModule, Module, CacheInterceptor } from '@nestjs/common';
import { AppController } from './app.controller';
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  imports: [CacheModule.register()],
  controllers: [AppController],
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: CacheInterceptor,
    },
  ],
})
export class AppModule {}
```

### **Global cache overrides**

- 전역 캐시가 활성화된 동안 캐시 항목은 라우트 경로에 따라 자동 생성되는 `CacheKey` 아래에 저장
- 특정 캐시 설정 (`@CacheKey()` 및 `@CacheTTL()`)을 메서드별로 재정의하여 개별 컨트롤러 메서드에 대한 사용자 지정 캐싱 전략을 허용 (다른 캐시 저장소 사용 가능)

```jsx
@Controller()
export class AppController {
  @CacheKey('custom_key')
  @CacheTTL(20)
  findAll(): string[] {
    return [];
  }
}
```

### **WebSockets and Microservices**

- WebSocket 구독자 및 Microservice의 패턴(사용중인 전송 방법에 관계없이)에 `CacheInterceptor`를 적용 할 수 있음

```jsx
@CacheTTL(10)
@UseInterceptors(CacheInterceptor)
@SubscribeMessage('events')
handleEvent(client: Client, data: string[]): Observable<string[]> {
  return [];
}
```

### **Different stores**

- 이 서비스는 내부적으로 `cache-manager`를 활용
- `cache-manager` 패키지는 **[Redis](https://github.com/dabroek/node-cache-manager-redis-store)** 저장소와 같은 다양한 유용한 저장소(store)를 지원

```tsx
import * as redisStore from 'cache-manager-redis-store';
import { CacheModule, Module } from '@nestjs/common';
import { AppController } from './app.controller';

# general

@Module({
  imports: [
    CacheModule.register({
      store: redisStore,
      host: 'localhost',
      port: 6379,
    }),
  ],
  controllers: [AppController],
})
export class AppModule {}

# async

CacheModule.registerAsync({
  useFactory: () => ({
    ttl: 5,
  }),
});

# 모듈 펙토리

CacheModule.registerAsync({
  imports: [ConfigModule],
  useFactory: async (configService: ConfigService) => ({
    ttl: configService.get('CACHE_TTL'),
  }),
  inject: [ConfigService],
});

# useClass 메소드 사용

CacheModule.registerAsync({
  useClass: CacheConfigService,
});

@Injectable()
class CacheConfigService implements CacheOptionsFactory {
  createCacheOptions(): CacheModuleOptions {
    return {
      ttl: 5,
    };
  }
}
```

# redis

- Redis는 Remote DIctionary Server의 약자로 메모리 기반의 key-value 구조 데이터 관리 시스템입니다.
- 디스크가 아닌 메모리에 데이터를 저장하므로 매우 빠른 속도를 보장합니다.

etc

- nestjs-simple-redis-lock

```tsx
import { RedisModule } from 'nestjs-redis';
import { RedisLockModule } from 'nestjs-simple-redis-lock';

@Module({
  imports: [
    ...
    RedisModule.forRootAsync({ // import RedisModule before RedisLockModule
      imports: [ConfigModule],
      useFactory: (config: ConfigService) => ({
        host: config.get('REDIS_HOST'),
        port: config.get('REDIS_PORT'),
        db: parseInt(config.get('REDIS_DB'), 10),
        password: config.get('REDIS_PASSWORD'),
        keyPrefix: config.get('REDIS_KEY_PREFIX'),
      }),
      inject: [ConfigService],
    }),
    RedisLockModule.register({}), // import RedisLockModule, use default configuration
  ]
})
export class AppModule {}
```

```tsx
import { RedisLockService } from 'nestjs-simple-redis-lock';

export class FooService {
  constructor(
    protected readonly lockService: RedisLockService, // inject RedisLockService 
  ) {}

  async test1() {
    try {
      /**
       * Get a lock by name
       * Automatically unlock after 1min
       * Try again after 100ms
       * The max times to retry is 36000, about 1h
       */
      await this.lockService.lock('test1');
      // Do somethings
    } finally { // use 'finally' to ensure unlocking
      this.lockService.unlock('test1'); // unlock
      // Or: await this.lockService.unlock('test1'); wait for the unlocking
    }
  }
```

- Bull (redis를 사용한 queue)
    - **[Redis](https://redis.io/)**를 사용하여 작업 데이터를 유지
    - 일부 대기열 producers와 consumers 및 listeners는 하나(또는 여러) 노드의 Nest에서 실행되고 다른 producers, consumers 및 listeners 다른 네트워크 노드의 다른 Node.js 플랫폼에서 실행되는 리스너