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

#### Binding guards

Like pipes and exception filters, guards can be **controller-scoped**, method-scoped, or global-scoped. Below, we set up a controller-scoped guard using the `@UseGuards()` decorator. This decorator may take a single argument, or a comma-separated list of arguments. This lets you easily apply the appropriate set of guards with one declaration.

```typescript
@@filename()
@Controller('cats')
@UseGuards(RolesGuard)
export class CatsController {}
```

> info **Hint** The `@UseGuards()` decorator is imported from the `@nestjs/common` package.

Above, we passed the `RolesGuard` type (instead of an instance), leaving responsibility for instantiation to the framework and enabling dependency injection. As with pipes and exception filters, we can also pass an in-place instance:

```typescript
@@filename()
@Controller('cats')
@UseGuards(new RolesGuard())
export class CatsController {}
```

The construction above attaches the guard to every handler declared by this controller. If we wish the guard to apply only to a single method, we apply the `@UseGuards()` decorator at the **method level**.

In order to set up a global guard, use the `useGlobalGuards()` method of the Nest application instance:

```typescript
@@filename()
const app = await NestFactory.create(AppModule);
app.useGlobalGuards(new RolesGuard());
```

> warning **Notice** In the case of hybrid apps the `useGlobalGuards()` method doesn't set up guards for gateways and micro services by default (see [Hybrid application](/faq/hybrid-application) for information on how to change this behavior). For "standard" (non-hybrid) microservice apps, `useGlobalGuards()` does mount the guards globally.

Global guards are used across the whole application, for every controller and every route handler. In terms of dependency injection, global guards registered from outside of any module (with `useGlobalGuards()` as in the example above) cannot inject dependencies since this is done outside the context of any module. In order to solve this issue, you can set up a guard directly from any module using the following construction:

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

> info **Hint** When using this approach to perform dependency injection for the guard, note that regardless of the
> module where this construction is employed, the guard is, in fact, global. Where should this be done? Choose the module
> where the guard (`RolesGuard` in the example above) is defined. Also, `useClass` is not the only way of dealing with
> custom provider registration. Learn more [here](/fundamentals/custom-providers).

#### Setting roles per handler

Our `RolesGuard` is working, but it's not very smart yet. We're not yet taking advantage of the most important guard feature - the [execution context](/fundamentals/execution-context). It doesn't yet know about roles, or which roles are allowed for each handler. The `CatsController`, for example, could have different permission schemes for different routes. Some might be available only for an admin user, and others could be open for everyone. How can we match roles to routes in a flexible and reusable way?

This is where **custom metadata** comes into play (learn more [here](https://docs.nestjs.com/fundamentals/execution-context#reflection-and-metadata)). Nest provides the ability to attach custom **metadata** to route handlers through the `@SetMetadata()` decorator. This metadata supplies our missing `role` data, which a smart guard needs to make decisions. Let's take a look at using `@SetMetadata()`:

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

> info **Hint** The `@SetMetadata()` decorator is imported from the `@nestjs/common` package.

With the construction above, we attached the `roles` metadata (`roles` is a key, while `['admin']` is a particular value) to the `create()` method. While this works, it's not good practice to use `@SetMetadata()` directly in your routes. Instead, create your own decorators, as shown below:

```typescript
@@filename(roles.decorator)
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
@@switch
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles) => SetMetadata('roles', roles);
```

This approach is much cleaner and more readable, and is strongly typed. Now that we have a custom `@Roles()` decorator, we can use it to decorate the `create()` method.

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

#### Putting it all together

Let's now go back and tie this together with our `RolesGuard`. Currently, it simply returns `true` in all cases, allowing every request to proceed. We want to make the return value conditional based on the comparing the **roles assigned to the current user** to the actual roles required by the current route being processed. In order to access the route's role(s) (custom metadata), we'll use the `Reflector` helper class, which is provided out of the box by the framework and exposed from the `@nestjs/core` package.

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

> info **Hint** In the node.js world, it's common practice to attach the authorized user to the `request` object. Thus, in our sample code above, we are assuming that `request.user` contains the user instance and allowed roles. In your app, you will probably make that association in your custom **authentication guard** (or middleware). Check [this chapter](/security/authentication) for more information on this topic.

> warning **Warning** The logic inside the `matchRoles()` function can be as simple or sophisticated as needed. The main point of this example is to show how guards fit into the request/response cycle.

Refer to the <a href="https://docs.nestjs.com/fundamentals/execution-context#reflection-and-metadata">Reflection and metadata</a> section of the **Execution context** chapter for more details on utilizing `Reflector` in a context-sensitive way.

When a user with insufficient privileges requests an endpoint, Nest automatically returns the following response:

```typescript
{
  "statusCode": 403,
  "message": "Forbidden resource",
  "error": "Forbidden"
}
```

Note that behind the scenes, when a guard returns `false`, the framework throws a `ForbiddenException`. If you want to return a different error response, you should throw your own specific exception. For example:

```typescript
throw new UnauthorizedException();
```

Any exception thrown by a guard will be handled by the [exceptions layer](/exception-filters) (global exceptions filter and any exceptions filters that are applied to the current context).

> info **Hint** If you are looking for a real-world example on how to implement authorization, check [this chapter](/security/authorization).
