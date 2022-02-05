### 실행 컨텍스트

Nest는 여러 애플리케이션 컨텍스트들(예를 들면 Nest HTTP 서버 기반과 마이크로서비스, 웹소켓 컨텍스트)을 아울러서 작동하는 애플리케이션을 쉽게 작성할 수 있도록 몇 가지 유틸리티 클래스들을 제공합니다. 이 유틸리티들은 전반적인 컨트롤러, 메서드, 실행 컨텍스트를 아울러 동작하는 범용적인 [가드](/guards), [필터](/exception-filters), [인터셉터](/interceptors)를 만들 수 있도록 현재 실행 컨텍스트에 대한 정보를 제공합니다.

이 챕터에서는 `ArgumentsHost`, `ExecutionContext` 두 클래스를 다룹니다.

#### ArgumentsHost 클래스

`ArgumentsHost` 클래스는 핸들러에 전달 된 인자들을 찾을 수 있는 메서드들을 제공하며, 인자들을 찾을 수 있는 적절한 컨텍스트(예를 들면 HTTP, RPC(마이크로서비스), 웹소켓)를 선택할 수 있게 해줍니다. 프레임워크는 `ArgumentsHost`의 인스턴스를 제공하며, 접근하려는 곳에서는 일반적으로 `host`라는 이름의 매개변수로 참조됩니다. 예를 들어, [예외 필터](https://docs.nestjs.com/exception-filters#arguments-host)의 `catch()`메서드는 `ArgumentsHost` 인스턴스와 함께 호출됩니다.

`ArgumentsHost`는 단순하게 핸들러의 전체적인 인자에 대한 추상화 역할을 수행합니다. HTTP 서버 애플리케이션(`@nestjs/platform-express`를 사용할 때)을 예를 들면 `host`객체가 Express의 `[request, response, next]`배열을 캡슐화 합니다. 여기서 `request`는 요청 객체를, `response`는 응답 객체를, `next`는 애플리케이션의 요청-응답 사이클을 제어하는 함수입니다. 반면에 [GraphQL](/graphql/quick-start) 애플리케이션에서의 `host`객체는 `[root, args, context, info]` 배열을 포함합니다.

#### 현재 애플리케이션 컨텍스트

여러 애플리케이션 컨텍스트를 아울러 동작하는 일반적인 [가드](/guards), [필터](/exception-filters), [인터셉터](/interceptors)를 만들기 위해서는 현재 메서드가 동작하고 있는 애플리케이션 타입을 결정지어야 하며, 이는 `ArgumentsHost`의 `getType()`메서드를 통해 가능합니다:

```typescript
if (host.getType() === "http") {
  // do something that is only important in the context of regular HTTP requests (REST)
} else if (host.getType() === "rpc") {
  // do something that is only important in the context of Microservice requests
} else if (host.getType<GqlContextType>() === "graphql") {
  // do something that is only important in the context of GraphQL requests
}
```

> info **힌트** `GqlContextType`은 `@nestjs/graphql`패키지에서 import합니다.

이처럼, 사용 가능한 애플리케이션 타입을 활용하여 보다 범용적인 컴포넌트를 작성할 수 있습니다.

#### 호스트 핸들러 매개변수

핸들러에 전달 된 인자를 찾기 위해 host 객체의 `getArgs()`메서드를 사용할 수 있습니다.

```typescript
const [req, res, next] = host.getArgs();
```

`getArgByIndex()`메서드를 사용하여 특정 인자만 찾을 수 있습니다:

```typescript
const request = host.getArgByIndex(0);
const response = host.getArgByIndex(1);
```

이 예제들에서 요청 객체 및 응답 객체를 인덱스를 통해 찾았는데, 일반적으로 이는 어플리케이션을 특정한 하나의 실행 컨텍스트에 국한시키는 것이므로 권장되지 않습니다. 대신에, `host`객체에는 적절한 어플리케이션 컨텍스트로 전환할 수 있는 유틸리티 메서드가 있으므로 이를 사용하여 더욱 안정되고 재사용가능한 코드를 작성할 수 있습니다. 아래는 컨텍스트를 전환하는 유틸리티 메서드들입니다.

```typescript
/**
 * Switch context to RPC.
 */
switchToRpc(): RpcArgumentsHost;
/**
 * Switch context to HTTP.
 */
switchToHttp(): HttpArgumentsHost;
/**
 * Switch context to WebSockets.
 */
switchToWs(): WsArgumentsHost;
```

직전의 예제를 `switchToHttp()`메서드를 사용하여 다시 작성해 봅시다. `host.switchToHttp()`는 HTTP 애플리케이션 컨텍스트에 걸맞는 `HttpArgugentsHost`객체를 반환합니다. `HttpArgugentsHost`객체는 우리가 원하는 객체를 받아올 수 있도록 두 가지 유용한 메서드를 제공합니다. 이 경우에는 Express 본연의 타입을 가진 객체를 반환받을 수 있도록 Express의 타입을 type assertion 해줍니다:

```typescript
const ctx = host.switchToHttp();
const request = ctx.getRequest<Request>();
const response = ctx.getResponse<Response>();
```

`WsArgumentsHost`와 `RpcArgumentsHost`에도 각각 웹소켓 컨텍스트와 마이크로서비스 컨텍스트에서 적당한 객체를 반환하는 메서드들이 있습니다. 아래는 `WsArgMentsHost`의 메서드들입니다.

```typescript
export interface WsArgumentsHost {
  /**
   * Returns the data object.
   */
  getData<T>(): T;
  /**
   * Returns the client object.
   */
  getClient<T>(): T;
}
```

이어서 `RpcArgumentsHost`의 메서드들입니다:

```typescript
export interface RpcArgumentsHost {
  /**
   * Returns the data object.
   */
  getData<T>(): T;

  /**
   * Returns the context object.
   */
  getContext<T>(): T;
}
```

#### ExecutionContext 클래스

`ExecutionContext`는 `ArgumentsHost`를 상속 받으며, 현재 실행 프로세스에 대한 추가적인 세부사항을 제공합니다. `ArgumentsHost`와 같이, Nest는 우리가 원하는 곳에 `ExecutionContext` 인스턴스를 제공하며, [가드]()의 `canActive()`메서드나 [인터셉터]()의 `intercept()`메서드가 그 예시입니다. 인스턴스는 아래 메서드들을 제공합니다:

```typescript
export interface ExecutionContext extends ArgumentsHost {
  /**
   * Returns the type of the controller class which the current handler belongs to.
   */
  getClass<T>(): Type<T>;
  /**
   * Returns a reference to the handler (method) that will be invoked next in the
   * request pipeline.
   */
  getHandler(): Function;
}
```

`getHandler()`메서드는 실행 될 핸들러의 참조를 반환합니다. `getClass()`메서드는 그 핸들러가 속한 `Controller`클래스의 타입을 반환합니다. HTTP 컨텍스트에서의 예시로, 현재 처리되는 요청이 `POST`요청이고 `CatsController`의 `create()`메서드가 실행된다면 `getHandler()`는 `create()`메서드의 참조를 반환하고 `getClass()`는 `CatsController`의 인스턴스가 아닌 **타입**을 반환합니다.

```typescript
const methodKey = ctx.getHandler().name; // "create"
const className = ctx.getClass().name; // "CatsController"
```

현재 클래스와 핸들러 메서드 둘의 참조 모두에 접근하는 기능은 굉장한 유연성을 제공합니다. 가장 중요한 건, 연관된 가드나 인터셉터에서 `@SetMetadata()`데코레이터를 통해 지정한 메타데이터에 접근할 수 있는 기회가 된다는 점입니다. 이 사례에 대해서는 아래에서 다루어 보겠습니다.

<app-banner-enterprise></app-banner-enterprise>

#### Reflection and metadata

Nest provides the ability to attach **custom metadata** to route handlers through the `@SetMetadata()` decorator. We can then access this metadata from within our class to make certain decisions.

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

With the construction above, we attached the `roles` metadata (`roles` is a metadata key and `['admin']` is the associated value) to the `create()` method. While this works, it's not good practice to use `@SetMetadata()` directly in your routes. Instead, create your own decorators, as shown below:

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

To access the route's role(s) (custom metadata), we'll use the `Reflector` helper class, which is provided out of the box by the framework and exposed from the `@nestjs/core` package. `Reflector` can be injected into a class in the normal way:

```typescript
@@filename(roles.guard)
@Injectable()
export class RolesGuard {
  constructor(private reflector: Reflector) {}
}
@@switch
@Injectable()
@Dependencies(Reflector)
export class CatsService {
  constructor(reflector) {
    this.reflector = reflector;
  }
}
```

> info **Hint** The `Reflector` class is imported from the `@nestjs/core` package.

Now, to read the handler metadata, use the `get()` method.

```typescript
const roles = this.reflector.get<string[]>("roles", context.getHandler());
```

The `Reflector#get` method allows us to easily access the metadata by passing in two arguments: a metadata **key** and a **context** (decorator target) to retrieve the metadata from. In this example, the specified **key** is `'roles'` (refer back to the `roles.decorator.ts` file above and the `SetMetadata()` call made there). The context is provided by the call to `context.getHandler()`, which results in extracting the metadata for the currently processed route handler. Remember, `getHandler()` gives us a **reference** to the route handler function.

Alternatively, we may organize our controller by applying metadata at the controller level, applying to all routes in the controller class.

```typescript
@@filename(cats.controller)
@Roles('admin')
@Controller('cats')
export class CatsController {}
@@switch
@Roles('admin')
@Controller('cats')
export class CatsController {}
```

In this case, to extract controller metadata, we pass `context.getClass()` as the second argument (to provide the controller class as the context for metadata extraction) instead of `context.getHandler()`:

```typescript
@@filename(roles.guard)
const roles = this.reflector.get<string[]>('roles', context.getClass());
@@switch
const roles = this.reflector.get('roles', context.getClass());
```

Given the ability to provide metadata at multiple levels, you may need to extract and merge metadata from several contexts. The `Reflector` class provides two utility methods used to help with this. These methods extract **both** controller and method metadata at once, and combine them in different ways.

Consider the following scenario, where you've supplied `'roles'` metadata at both levels.

```typescript
@@filename(cats.controller)
@Roles('user')
@Controller('cats')
export class CatsController {
  @Post()
  @Roles('admin')
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }
}
@@switch
@Roles('user')
@Controller('cats')
export class CatsController {}
  @Post()
  @Roles('admin')
  @Bind(Body())
  async create(createCatDto) {
    this.catsService.create(createCatDto);
  }
}
```

If your intent is to specify `'user'` as the default role, and override it selectively for certain methods, you would probably use the `getAllAndOverride()` method.

```typescript
const roles = this.reflector.getAllAndOverride<string[]>("roles", [
  context.getHandler(),
  context.getClass(),
]);
```

A guard with this code, running in the context of the `create()` method, with the above metadata, would result in `roles` containing `['admin']`.

To get metadata for both and merge it (this method merges both arrays and objects), use the `getAllAndMerge()` method:

```typescript
const roles = this.reflector.getAllAndMerge<string[]>("roles", [
  context.getHandler(),
  context.getClass(),
]);
```

This would result in `roles` containing `['user', 'admin']`.

For both of these merge methods, you pass the metadata key as the first argument, and an array of metadata target contexts (i.e., calls to the `getHandler()` and/or `getClass())` methods) as the second argument.
