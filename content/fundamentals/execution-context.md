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

`ExecutionContext`는 `ArgumentsHost`를 상속 받으며, 현재 실행 프로세스에 대한 추가적인 세부사항을 제공합니다. `ArgumentsHost`와 같이, Nest는 우리가 원하는 곳에 `ExecutionContext` 인스턴스를 제공하며, [가드](https://docs.nestjs.com/guards#execution-context)의 `canActive()`메서드나 [인터셉터](https://docs.nestjs.com/interceptors#execution-context)의 `intercept()`메서드가 그 예시입니다. 인스턴스는 아래 메서드들을 제공합니다:

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

#### Reflection과 메타데이터

Nest는 **사용자 정의 메타데이터** 붙일 수 있는 기능을 제공하며 이는 라우트 핸들러에서 `@SetMetadata()`데코레이터를 통해 가능합니다. 이후 클래스 내에서 이 메타데이터에 접근하여 특정 결정을 내릴 수 있습니다.

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

위의 예제에서는 `roles`라는 메타데이터를 `create()`메서드에 붙였습니다(`roles`는 메타데이터 키이며 `['admin']`은 그 값입니다). 라우트에서 `@SetMetadata()`데코레이터를 직접 쓰는 것은 그다지 좋은 방법은 아닙니다. 대신에 아래와 같이 사용자 정의 데코레이터를 작성합니다:

```typescript
@@filename(roles.decorator)
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
@@switch
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles) => SetMetadata('roles', roles);
```

이러한 접근 방식이 훨씬 깔끔하고 가독성 있으며, 타입을 더욱 엄격하게 관리할 수 있습니다. 이제 `@Roles()`데코레이터를 `create()`메서드에 사용합니다.

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

라우트에서 role이라는 사용자 정의 메타데이터에 접근하기 위해서는 `Reflector` 헬퍼 클래스를 사용하면 됩니다. 이 클래스는 별도의 작업 없이 그저 `@nestjs/core`에서 가져다 사용하기만 하면 됩니다. `Reflector` 일반적인 방법으로 클래스에 주입시킬 수 있습니다:

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

> info **힌트** `Reflector` 클래스는 `@nestjs/core` 패키지에서 import합니다.
> Now, to read the handler metadata, use the `get()` method.

```typescript
const roles = this.reflector.get<string[]>("roles", context.getHandler());
```

메타데이터의 **키**와 메타데이터를 찾을 **컨텍스트** (데코레이터가 붙는 대상) 두 가지를 `Reflector#get` 메서드에 인자로 넘김으로써 쉽게 메타데이터에 접근할 수 있습니다. 이 예제에서는 위에서 작성한 `roles.decorator.ts`파일에서 `SetMetadata()`를 통해 만들었던 `'roles'`라는 **키**가 있습니다. 컨텍스트는 `context.getHandler()`를 호출하면 얻을 수 있고, 이 둘을 넘긴 결과로 현재 처리되는 라우트 핸들러의 메타데이터를 추출할 수 있습니다. `getHandler()`는 라우트 핸들러 함수의 **참조**를 반환한다는 점을 기억해야 합니다.

또한, 컨트롤러 레벨에 메타데이터를 적용하여 해당 컨트롤러의 모든 라우트들에 메타데이터를 적용시킬 수 있습니다.

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

이 경우에는, `context.getHandler()` 대신에 `context.getClass()`를 두 번째 인자로 넣어야 하며, 이는 메타데이터를 추출하기 위한 컨텍스트로 해당 컨트롤러 클래스를 제공하기 위함입니다.

```typescript
@@filename(roles.guard)
const roles = this.reflector.get<string[]>('roles', context.getClass());
@@switch
const roles = this.reflector.get('roles', context.getClass());
```

하나의 메타데이터를 다양한 레벨에서 동시에 지정한다면, 다수의 컨텍스트로부터 메타데이터를 추출하고 합쳐야 할 것입니다. `Reflector` 클래스는 이를 위해 두 가지의 유틸리티 메서드를 제공합니다. 이 메서드들은 컨트롤러와 메서드 **두 곳 모두**에서 메타데이터를 추출하고, 각기 다른 방법으로 조합합니다.

`'roles'`메타데이터를 두 가지 레벨에서 동시에 공급한다고 가정해 봅시다.

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

위 코드의 의도가 `'user'` role을 기본으로 취급하되 몇몇 메서드에서만 role을 덮어씌우고자 하는 것이라면, `getAllAndOverride()` 메서드를 사용하면 됩니다.

```typescript
const roles = this.reflector.getAllAndOverride<string[]>("roles", [
  context.getHandler(),
  context.getClass(),
]);
```

`created()` 메서드의 컨텍스트 속에서 동작하는 가드에서 위와 같이 코드를 작성하면 `roles`의 값은 `['admin']`가 됩니다.

두 메타데이터를 병합하려면 `getAllAndMerge()` 메서드를 사용합니다. 이 메서드는 두 배열을 하나로 병합합니다.

```typescript
const roles = this.reflector.getAllAndMerge<string[]>("roles", [
  context.getHandler(),
  context.getClass(),
]);
```

위와 같이 작성하면 `roles`의 값은 `['user', 'admin']`가 됩니다.

이 두 가지 메서드를 사용할 때에는 첫 번째 인자로 메타데이터 키를, 두 번재 인자로는 메타데이터 대상 컨텍스트(`getHandler()`나 `getClass()` 메서드를 호출한 결과)가 담긴 배열을 건네야 합니다.
