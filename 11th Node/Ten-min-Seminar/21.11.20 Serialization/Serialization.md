# Serialization

발표 날짜: 2021년 11월 20일
발표 여부: Yes
발표자: 이소형

# 직렬화

- 직렬화는 개체가 네트워크 응답으로 변환되기 전에 발생하는 프로세스입니다.
- 클라이언트에 반환할 데이터를 전송에 적합한 다른 데이터 형식으로 변환하는 과정입니다.

예를 들어 비밀번호와 같은 민감한 데이터는 항상 응답에서 제외되어야 합니다.

또는 특정 속성에는 엔터티 속성의 하위 집합만 보내는 것과 같은 추가 변환이 필요할 수 있습니다.

이러한 변환을 수동으로 수행하는 것은 지루하고 오류가 발생하기 쉬우며 모든 사례가 다루어 졌는지 불확실할 수 있습니다.

<aside>
✅ 간단한 방식으로 수행할 수 있도록 지원하는 기본 제공 기능

</aside>

# `ClassSerializerInterceptor` 인터셉터

- class-transformer 패키지를 사용하여 객체를 변환합니다.
- 메소드 핸들러가 반환한 값을 가져와 class-transformer에서 `classToPlain()` 함수를 적용하여 `class-transformer` 데코레이터로 표현된 규칙을 적용할 수 있습니다.

## Exclude properties

속성을 자동으로 제외하려고 한다

```tsx
import { Exclude } from 'class-transformer';

export class UserEntity {
  id: number;
  firstName: string;
  lastName: string;

  **@Exclude()**
  password: string;

  constructor(partial: Partial<UserEntity>) {
    Object.assign(this, partial);
  }
}
```

`UserEntity` 클래스의 인스턴스를 반환하는 메서드 처리기가 있는 컨트롤러

```tsx
import { ClassSerializerInterceptor, UseInterceptors } from '@nestjs/common';

**@UseInterceptors(ClassSerializerInterceptor)**
@Get()
findOne(): UserEntity {
  return new UserEntity({
    id: 1,
    firstName: 'Kamil',
    lastName: 'Mysliwiec',
    password: 'password',
  });
}
```

이 엔드포인트가 요청되면 클라이언트는 다음과 같이 `password` 속성을 제거한 응답을 받습니다.

```json
{
  "id": 1,
  "firstName": "Kamil",
  "lastName": "Mysliwiec"
}
```

<aside>
✅ 애플리케이션 전체에 적용

```tsx
const app = await NestFactory.create(AppModule);
app.useGlobalInterceptors(new LoggingInterceptor());
```

```tsx
import { ClassSerializerInterceptor, Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
	providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: ClassSerializerInterceptor,
    },
  ],
})
export class AppModule {}
```

⇒ `UserEntity`를 반환하는 **모든** 메소드가 `password` 속성을 제거하도록 합니다.

</aside>

<aside>
👉🏻 `class-transformer` 데코레이터

## Expose properties

속성의 별칭 이름을 제공하거나 속성 값을 계산하는 함수를 실행할 수 있습니다 (getter)

```tsx
@Expose()
get fullName(): string {
	return `@{this.firstName} ${this.lastName}`;
}
```

## Transform

추가 데이터 변환을 수행할 수 있습니다.

```tsx
@Transform(({ value }) => value.name)
role: RoleEntity;
```

[GitHub - typestack/class-transformer: Decorator-based transformation, serialization, and deserialization between objects and classes.](https://github.com/typestack/class-transformer)

</aside>

## Pass options

변환 함수의 기본 설정을 재정의할 수 있습니다.

```tsx
@SerializeOptions({
	excludePrefixes: ['_'], // _ 접두사로 시작하는 모든 속성을 자동으로 제외합니다.
})
@Get()
findOne(): UserEntity {
	return new UserEntity();
}
```