### 가드

가드는 `@Injectable()` 데코레이터가 붙는 클래스의 일종이며 `CanActivate` 인터페이스를 구현해야 합니다.

<figure><img src="/assets/Guards_1.png" /></figure>

가드는 **하나의 책임**만 집니다. 런타임 환경에서 주어지는 특정 조건(권한, 역할, ACLs 등)을 고려하여 주어진 요청을 핸들러가 처리할지 말지를 결정하며, 대게는 **인가**의 용도로 사용됩니다. Express 애플리케이션에서 인가(보통 그 사촌인 '인증'과 협력하는)는 전형적으로 [미들웨어](/middleware)를 통해 이루어졌습니다. 미들웨어를 사용하여 인증하는 것은 좋은 선택이긴 합니다. 토큰을 검사하고 `요청` 객체에 프로퍼티들을 붙이는 작업들을 특정 라우트 컨텍스트와 그 메타데이터에 국한시킬 수 있기 때문입니다.

하지만 원래 미들웨어는 조금 답답한 구석이 있습니다. 미들웨어는 입장에서는 어느 핸들러가 `next()`함수에 의해 호출되는지 모릅니다. 반면에, **가드**는 `ExecutionContext` 인스턴스에 접근할 수 있기 때문에 다음으로 무엇이 실행될지 정확하게 압니다. 가드는 예외 필터와 파이프, 인터셉터와 같이 요청부터 응답까지의 사이클 중 정확히 원하는 지점에 처리 로직을 아주 선언적으로 끼워 넣을 수 있도록 설계 되었습니다. 이 덕분에 더욱 DRY하고 선언적인 코드를 유지할 수 있습니다.

> info **힌트** 가드는 미들웨어보다 **이후**에, 파이프와 인터셉터보다는 **이전**에 실행됩니다.

#### 인가 가드

위에서 언급했듯, **인가**는 가드를 활용하는 좋은 사례입니다. 요청하는 사용자(보통은 이미 인증된 상태)가 충분한 권한을 가지고 있을 때만 라우트를 이용할 수 있기 때문입니다. 이제부터 작성해볼 `AuthGuard`는 사용자가 이미 인증 되었다고 가정하기 때문에 요청 헤더에 토큰이 붙어있을 것이라 생각합니다. 이 가드는 토큰을 추출 및 검증하여 확보한 정보를 보고 요청을 처리할지 말지 결정합니다.

```typescript
@@filename(auth.guard)
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
@@switch
import { Injectable } from '@nestjs/common';

@Injectable()
export class AuthGuard {
  async canActivate(context) {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
```

> info **힌트** 만약 애플리케이션에서 인증 메커니즘을 어떻게 구현하는지에 대한 실제 사례는 [이 챕터](/security/authentication)에서 확인할 수 있습니다.

`validateRequest()` 함수 내부의 로직은 필요에 따라 간단해질 수도 정교해질 수도 있습니다. 이번 예제의 핵심은 요청부터 응답까지의 사이클 속에서 알맞은 곳에 가드를 끼워넣는 것입니다.

모든 가드는 `canActivate()` 함수를 구현해야 하며 이 함수는 boolean을 반환해야 합니다. 반환되는 값은 요청이 허용될지 말지를 나타냅니다. 결과는 `Promise`나 `Observable`을 이용하여 비동기적으로도 반환할 수 있습니다. Nest는 이 반환값을 통해 다음 작업을 제어합니다.

- `true`를 반환하면 요청이 처리됩니다.
- `false`를 반환하면 Nest가 요청을 거절합니다.

<app-banner-enterprise></app-banner-enterprise>

#### 실행 컨텍스트

`canActivate()` 함수는 `ExecutionContext` 인스턴스만 인자로 받습니다. `ExecutionContext`는 `ArgumentsHost`를 상속합니다. `AgumentsHost`는 이전에 예외 필터 챕터에 등장했었습니다. 위의 예제에서는 이전과 마찬가지로 `ArgumentsHost`에 정의되어 있는 헬퍼 메서드를 사용하여 `요청` 객체를 참조합니다. 이 주제에 대해 자세히 알고 싶다면 [예외 필터](https://docs.nestjs.com/exception-filters#arguments-host) 챕터의 **Arguments host** 섹션을 참조하세요.

`ExecutionContext`는 `ArgumentsHost`를 상속받기 때문에 새로운 헬퍼 메서드를 생성하여 현재 실행 프로세스에 대한 추가적인 세부사항들을 제공할 수 있습니다. 이 세부사항들은 전반적인 컨트롤러와 메서드, 실행 컨텍스트들을 아울러 동작하는 범용적인 가드를 작성할 때 유용할 것입니다. `ExecutionContext`에 대한 자세한 내용은 [여기](fundamentals/execution-context)를 참조해 주세요.

#### 역할 기반 인증

더욱 기능적인 가드를 작성하여 특정한 역할을 가진 사용자에게만 접근을 허가하도록 만들어 봅시다. 기본 가드 템플릿에서 시작하여 아래 섹션들을 거치면서 구성하겠습니다. 따라서 우선 지금은 모든 요청을 수행하도록 허가하고 있습니다:

```typescript
@@filename(roles.guard)
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    return true;
  }
}
@@switch
import { Injectable } from '@nestjs/common';

@Injectable()
export class RolesGuard {
  canActivate(context) {
    return true;
  }
}
```

#### 가드 적용하기

파이프나 예외 필터처럼 가드도 **컨트롤러 스코프**나 메서드 스코프, 전역 스코프를 가질 수 있습니다. 아래 예제에서는 `@UseGuards()` 데코레이터를 사용하여 컨트롤러 스코프를 가지는 가드를 적용했습니다. 이 데코레이터에는 단 하나의 값만 인자로 넘길 수도 있고, 여러 인자를 담은 배열을 인자로 넘길 수도 있습니다. 이 덕분에 하나의 데코레이터만으로도 적절한 가드들을 쉽게 적용할 수 있습니다.

```typescript
@@filename()
@Controller('cats')
@UseGuards(RolesGuard)
export class CatsController {}
```

> info **힌트** `@UseGuards()` 데코레이터는 `@nestjs/common` 패키지에서 import합니다.

위 예제처럼, `RolesGuard` 의 인스턴스가 아닌 타입을 전달함으로써 인스턴스화에 대한 책임을 프레임워크에 맡기고 의존성 주입을 가능하게 합니다. 파이프나 예외 필터를 적용할 때처럼 인스턴스로 넘기는 것도 가능합니다:

```typescript
@@filename()
@Controller('cats')
@UseGuards(new RolesGuard())
export class CatsController {}
```

위처럼 컨트롤러 레벨에 가드를 생성하면 해당 컨트롤러의 모든 핸들러에게 가드가 붙습니다. 가드를 특정 메서드에만 적용하려면 `@UseGuards()` 데코레이터를 **메서드 레벨**에 적용해야 합니다.

전역 가드를 적용하려면 Nest 애플리케이션 인스턴스의 `useGlobalGuards()`를 사용합니다.

```typescript
@@filename()
const app = await NestFactory.create(AppModule);
app.useGlobalGuards(new RolesGuard());
```

> warning **주의** 하이브리드 앱의 경우 `useGlobalGuards()` 메서드는 기본적으로 게이트웨이 및 마이크로서비스에 대해 가드를 적용시키지 않습니다(이 동작을 변경하는 방법은 [하이브리드 애플리케이션](/faq/hybrid-application)을 참조하세요). 하이브리드가 아닌 "표준" 마이크로서비스 앱에서는 `useGlobalGuards()`가 가드를 전역적으로 마운트시킵니다.

전역 가드는 모든 컨트롤러와 모든 라우트 핸들러, 즉 전체 애플리케이션의 어디에서나 사용됩니다. 의존성 주입 측면에서 볼 때, 위 예제에서 `useGlobalGuards()`할 때처럼 모든 모듈의 바깥에서 등록된 전역 가드는 의존성을 주입시킬 수 없습니다. 가드가 등록되는 일이 모든 모듈의 컨텍스트 바깥에서 일어나기 때문입니다. 이 이슈를 해결하려면 아래와 같이 모듈에서 가드를 직접 적용시켜야 합니다:

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: RolesGuard,
    },
  ],
})
export class AppModule {}
```

> info **힌트** 가드에 대한 의존성 주입을 위해 이러한 접근을 사용할 때 유념해야 할 점은 어느 모듈에서 적용하든 간에 그 가드는 사실 전역적이라는 것입니다. 그렇다면 위와 같은 작업은 어디서 해야 할까요? 우선 가드(위 에제에서는 `RolesGuard`)가 정의될 모듈을 선택합니다. 또한 `useClass`는 사용자 정의 프로바이더를 등록하는 유일한 방법이 아닙니다. 사용자 정의 프로바이더에 관한 자세한 정보는 [여기](/fundamentals/custom-providers)를 참조하세요.

#### 핸들러마다 역할 지정하기

우리의 `RolesGuard`는 동작은 하지만 아직은 그다지 똑똑하지 않습니다. 아직 가드의 중요한 기능인 [실행 컨텍스트](/fundamentals/execution-context)를 활용하지 않았기 때문이죠. 우리의 가드는 아직 각 핸들러마다 허용할 사용자의 역할에 대해 알지 못합니다. 예를 들어 `CatsController`의 각 라우터마다 다른 권한 규격을 가지게 하려고 합니다. 몇몇 핸들러는 관리자만 사용할 수 있고, 나머지는 모든 사용자가 이용할 수 있습니다. 어떻게 하면 유연하고 재사용 가능하게 관리자나 일반사용자 같은 역할을 라우트에 매칭시킬 수 있을까요?

바로 여기에서 **사용자 정의 메타데이터**를 사용합니다(사용자 정의 메타데이터에 대한 자세한 내용은 [여기](https://docs.nestjs.com/fundamentals/execution-context#reflection-and-metadata)를 참조하세요). Nest에서는 `@SetMetadata()` 데코레이터를 이용하여 라우트 핸들러에 사용자 정의 **메타데이터**를 붙일 수 있습니다. 이 메타데이터는 우리가 놓치고 있던 `role`데이터를 제공해 주며 우리의 가드는 비로소 똑똑하게 결정을 내릴 수 있게 됩니다. `@SetMetadata()`를 어떻게 사용하는 지 살펴봅시다:

```typescript
@@filename(cats.controller)
@Post()
@SetMetadata('roles', ['admin'])
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
@@switch
@Post()
@SetMetadata('roles', ['admin'])
@Bind(Body())
async create(createCatDto) {
  this.catsService.create(createCatDto);
}
```

> info **힌트** `@SetMetadata()` 데코레이터는 `@nestjs/common` 패키지에서 import합니다.

위와 같이 작성하면 `roles`라는 메타데이터(`roles`는 key이고, `['admin]`은 그 값입니다)가 `create()`메서드에 붙게 됩니다. 이러한 작업을 할 때, `@SetMetadata()`를 직접 라우트에 명시하는 것은 그다지 좋은 활용이 아닙니다. 대신에, 아래와 같이 자신만의 데코레이터를 작성합니다:

```typescript
@@filename(roles.decorator)
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
@@switch
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles) => SetMetadata('roles', roles);
```

이와 같이 접근하면 훨씬 깔끔하고 가독성이 좋으며 타입을 더욱 엄격하게 관리할 수 있습니다. 이제 `@Roles()`라는 사용자 정의 데코레이터가 생겼고, 이를 `create()`메서드에 붙일 수 있습니다.

```typescript
@@filename(cats.controller)
@Post()
@Roles('admin')
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
@@switch
@Post()
@Roles('admin')
@Bind(Body())
async create(createCatDto) {
  this.catsService.create(createCatDto);
}
```

#### 지금까지의 모든 것들을 적용하기

`RolesGuard`를 작성했던 곳으로 돌아가 봅시다. 아직까지는 단순하게 `true`만 반환하기 때문에 모든 요청이 처리됩니다. 이제는 **현재 사용자의 역할**과 처리될 예정인 라우트가 요구하는 역할을 비교하여 상황에 따라 다른 값을 반환하게 만들려고 합니다. 라우트의 역할(사용자 정의 메타데이터)에 접근하기 위해서는 `Reflector`라는 헬퍼 클래스를 사용해야 합니다. 이 클래스는 별도의 작업 없이 그저 `@nestjs/core`에서 가져다 사용하기만 하면 됩니다.

```typescript
@@filename(roles.guard)
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    if (!roles) {
      return true;
    }
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return matchRoles(roles, user.roles);
  }
}
@@switch
import { Injectable, Dependencies } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
@Dependencies(Reflector)
export class RolesGuard {
  constructor(reflector) {
    this.reflector = reflector;
  }

  canActivate(context) {
    const roles = this.reflector.get('roles', context.getHandler());
    if (!roles) {
      return true;
    }
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return matchRoles(roles, user.roles);
  }
}
```

> info **힌트** node.js 세계에서는 `request`객체에 프로퍼티들을 붙이는 경험을 흔하게 합니다. 따라서, 위의 에제에서는 `request.user`에 사용자의 인스턴스와 역할이 포함되어 있다고 가정합니다. 사용자 정의 **인증 가드**(또는 미들웨어)에서 관련 기능을 구현하게 될 것입니다. 해당 주제에 대한 자세한 정보는 [이 챕터](/security/authentication)에서 확인해 주세요.

> warning **주의** `matchRoels()` 함수 내부 로직은 필요에 따라 단순할 수도 복잡할 수도 있습니다. 중요한 건 이 예제는 그저 가드를 요청부터 응답까지의 사이클 속에 끼워 맞추는지 보여주기 위함입니다.

`Reflector`를 컨텍스트에 따라 다르게 활용하는 방법과 같이 보다 세부적인 내용은 **실행 컨텍스트** 챕터의 <a href="https://docs.nestjs.com/fundamentals/execution-context#reflection-and-metadata">Reflection과 메타데이터</a> 섹션에서 확인하실 수 있습니다.

권한이 불충분한 사용자가 요청을 한다면 Nest가 알아서 다음의 응답을 보냅니다:

```typescript
{
  "statusCode": 403,
  "message": "Forbidden resource",
  "error": "Forbidden"
}
```

가드가 `false`를 반환하면 프레임워크 내부적으로 `ForbiddenException`예외를 발생시킨다는 것을 기억하시길 바랍니다. 다른 에러를 응답하고 싶다면 아래 예시처럼 원하는 예외를 발생시키면 됩니다.

```typescript
throw new UnauthorizedException();
```

가드에서 발생시킨 예외는 [예외 계층](/exception-filters)(전역 예외 필터와 현재 컨텍스트에 적용된 예외 필터들)에서 처리됩니다.

> info **힌트** 실무에서 인가를 어떻게 구현하는지 알고 싶다면 [이 챕터](/security/authorization)를 확인해 보세요.
