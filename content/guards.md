### Guards

가드는 `@Injectable()` 데코레이터로 주석이 달린 클래스입니다. 가드는 `CanActivate` 인터페이스를 구현해야합니다.

<figure><img src="/assets/Guards_1.png" /></figure>

가드는 **단독 책임**이 있습니다. 런타임에 존재하는 특정 조건 (예: 권한, 역할, ACL 등)에 따라 지정된 요청이 라우트 핸들러에 의해 처리될 지 여부를 결정합니다. 이를 종종 **권한**이라고합니다. 인증 (및 일반적으로 공동 작업 인 **인증**)은 일반적으로 기존 Express 응용 프로그램의 [미들웨어](/middleware)에 의해 처리되었습니다. 토큰 유효성 검사 및 속성을 `요청(Request)`개체에 연결하는 것과 같은 것은 특정 경로 컨텍스트 (및 해당 메타 데이터)와 강력하게 연결되어 있지 않기 때문에 미들웨어는 인증에 적합한 선택입니다.

그러나 미들웨어는 본질적으로 바보입니다. `next()`함수를 호출한 후 어떤 핸들러가 실행될지 알 수 없습니다. 반면에 **Guards**는 `ExecutionContext` 인스턴스에 액세스할 수 있으므로 다음에 무엇이 실행 될지 정확히 알 수 있습니다. 요청/응답 주기의 정확한 시점에 처리 로직을 삽입하고 선언적으로 처리할 수 있도록 예외 필터, 파이프 및 인터셉터와 같이 설계되었습니다. 이렇게 하면 코드를 건조하고 선언적으로 유지할 수 있습니다.

> info **힌트** 가드는 각 미들웨어 **후**에 실행되지만, 인터셉터 나 파이프는 **전**에 실행됩니다.

#### Authorization guard

언급 한 바와 같이 **권한**은 발신자 (일반적으로 특정 인증된 사용자)에게 충분한 권한이 있는 경우에만 특정 경로를 사용할 수 있어야 하기 때문에 Guards의 훌륭한 사용 사례입니다. 우리가 구축 할 `AuthGuard`는 이제 인증된 사용자를 가정합니다 (따라서 토큰이 요청 헤더에 첨부되어 있음). 토큰을 추출하고 유효성을 검사하고 추출된 정보를 사용하여 요청을 진행할 수 있는지 여부를 결정합니다.

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

> info **Hint** If you are looking for a real-world example on how to implement an authentication mechanism in your application, visit [this chapter](/security/authentication). Likewise, for more sophisticated authorization example, check [this page](/security/authorization).

`validateRequest()` 함수 내부의 논리는 필요한 만큼 간단하거나 정교할 수 있습니다. 이 예의 요점은 보호자가 요청/응답 주기에 어떻게 맞는지 보여줍니다.

모든 가드는 `canActivate()`함수를 구현해야합니다. 이 함수는 현재 요청이 허용되는지 여부를 나타내는 조건값을 반환해야 합니다. 응답을 동기식 또는 비동기식으로 반환 할 수 있습니다 (`Promise` 또는 `Observable을` 통해). Nest는 리턴 값을 사용하여 다음 조치를 제어합니다.

- `true`를 반환하면 요청이 처리됩니다.
- `false`를 반환하면 Nest는 요청을 거부합니다.

<app-banner-enterprise></app-banner-enterprise>

#### Execution context

The `canActivate()` function takes a single argument, the `ExecutionContext` instance. The `ExecutionContext` inherits from `ArgumentsHost`. We saw `ArgumentsHost` previously in the exception filters chapter. In the sample above, we are just using the same helper methods defined on `ArgumentsHost` that we used earlier, to get a reference to the `Request` object. You can refer back to the **Arguments host** section of the [exception filters](https://docs.nestjs.com/exception-filters#arguments-host) chapter for more on this topic.

By extending `ArgumentsHost`, `ExecutionContext` also adds several new helper methods that provide additional details about the current execution process. These details can be helpful in building more generic guards that can work across a broad set of controllers, methods, and execution contexts. Learn more about `ExecutionContext` [here](/fundamentals/execution-context).

#### Role-based authentication

특정 역할을 가진 사용자만 액세스 할 수 있는 보다 기능적인 보호 기능을 구축해 보겠습니다. 기본 가드 템플릿으로 시작하여 다음 섹션에서 작성합니다. 현재로서는 모든 요청을 진행할 수 있습니다.

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

파이프 및 예외 필터와 마찬가지로 가드는 **컨트롤러 범위(controller-scoped)**, 메소드 범위(method-scoped) 또는 전역 범위(global-scoped) 일 수 있습니다. 아래는 `@UseGuards()`데코레이터를 사용하여 컨트롤러 범위의 보호대를 설정했습니다. 이 데코레이터는 단일 인수 또는 쉼표로 구분된 인수 목록을 사용할 수 있습니다. 이를 통해 한 번의 선언으로 적절한 가드 세트를 쉽게 적용할 수 있습니다.

```typescript
@@filename()
@Controller('cats')
@UseGuards(RolesGuard)
export class CatsController {}
```

> info **힌트** `@UseGuards()` 데코레이터는 `@nestjs/common` 패키지에서 가져옵니다.

위에서 우리는 인스턴스 대신에 `롤 가드 (RolesGuard)` 타입을 전달하여 프레임워크에 인스턴스화와 의존성 주입을 가능하게 했습니다. 파이프 및 예외 필터와 마찬가지로 내부 인스턴스를 전달할 수도 있습니다.

```typescript
@@filename()
@Controller('cats')
@UseGuards(new RolesGuard())
export class CatsController {}
```

위의 구성은 이 컨트롤러가 선언한 모든 핸들러에 가드를 연결합니다. 가드가 단일 메소드에만 적용되도록 하려면 **메소드 수준**에서 `@UseGuards()` 데코레이터를 적용합니다.

전역 가드를 설정하려면 Nest 애플리케이션 인스턴스의 `useGlobalGuards()`메소드를 사용하십시오.

```typescript
@@filename()
const app = await NestFactory.create(AppModule);
app.useGlobalGuards(new RolesGuard());
```

> warning **알림** 하이브리드 앱의 경우 `useGlobalGuards()` 메소드는 기본적으로 게이트웨이 및 마이크로 서비스에 대한 보호를 설정하지 않습니다. ([하이프리드 어플리케이션](faq/hybrid-application)에서 해당 설정 변경에 대한 정보를 볼 수 있습니다.) "표준"(하이브리드가 아닌) 마이크로 서비스 앱의 경우 `useGlobalGuards()` 는 가드를 전역으로 마운트합니다.

전역 가드는 모든 컨트롤러와 모든 경로 처리기에 대해 전체 응용 프로그램에서 사용됩니다. 의존성 주입의 관점에서, 모듈 외부에서 등록된 전역 가드 (위의 예에서와 같이 `useGlobalGuards()` 로)는 의존성이 주입될 수 없습니다. 이는 모듈의 컨텍스트 밖에서 수행되기 때문입니다. 이 문제를 해결하기 위해 다음 구성을 사용하여 모든 모듈에서 직접 가드를 설정할 수 있습니다.

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

> info **힌트** 가드에 대한 의존성 주입을 수행하기 위해 이 접근 방식을 사용할 때,
> 이 구성이 사용되는 모듈에 관계없이, 가드는 전역적입니다.
> 가드가 (위 예제에서 `RolesGuard`)가 정의된 모듈을 선택하십시오. 또한, 커스텀 프로 바이더 등록을 다루는 유일한 방법은 `useClass`가 아닙니다.
> 여기에 대해 자세히 알아보십시오 [here](/fundamentals/custom-providers).

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
