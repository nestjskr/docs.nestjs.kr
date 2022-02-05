### Injection scopes

서로 다른 개발언어 경험을 가진 사람들에게는, 들어오는 모든 요청 간에 거의 모든 것이 공유된다는 사실을 예상하지 못할 수 있습니다. 우리에게는 데이터베이스와의 연결 풀, 전역 상태를 가지는 싱글톤 서비스 등이 있습니다. Node.js는 모든 요청이 각각 분리된 스레드에서 처리되는 Multi-Threaded Stateless 모델을 따르지 않는다는 것을 기억하고 있어야 합니다. 이 때문에, 싱글톤 인스턴스를 사용하는 것이 우리의 애플리케이션에게 전적으로 **안전**합니다.

하지만 GraphQL 애플리케이션에서의 요청별 캐싱, 요청 트래킹, 멀티테넌시와 같이 인스턴스가 요청에 기반한 수명을 가지길 바라는 경우도 있습니다.

#### Provider scope

A provider can have any of the following scopes:
프로바이더는 다음 스코프들 중 하나를 가질 수 있습니다:

<table>
  <tr>
    <td><code>DEFAULT</code></td>
    <td>하나의 인스턴스를 가지는 프로바이더로서 전체 애플리케이션 간에 공유됩니다. 인스턴스의 수명은 애플리케이션 생명주기와 직결됩니다. 애플리케이션이 구동되고 나면, 모든 싱글톤 프로바이더들이 인스턴스화 되며 기본적으로 싱글톤 스코프를 가집니다.</td>
  </tr>
  <tr>
    <td><code>REQUEST</code></td>
    <td>들어오는 각각의 <strong>요청</strong>에 대해 새로운 프로바이더 인스턴스가 생성됩니다. 생성된 인스턴스는 요청이 완전히 처리된 이후에 가비지 콜렉션에 의해 수거됩니다.</td>
  </tr>
  <tr>
    <td><code>TRANSIENT</code></td>
    <td>Transient 프로바이더는 소비자(프로바이더를 주입받아 사용하는 컴포넌트)들 사이에서 공유되지 않습니다. 하나의 transient 프로바이더를 주입하는 여러 소비자들은 각자 새로운 인스턴스를 할당 받습니다.</td>
  </tr>
</table>

> info **힌트** 대부분의 경우에는 싱글톤 스코프를 사용하는 것을 **권장**합니다. 프로바이더를 모든 소비자들과 모든 요청들이 공유한다는 것은 인스턴스의 초기화 작업이 애플리케이션이 구동되는 시점에 딱 한 번 이루어진 후에 그 인스턴스가 캐싱된다는 것을 뜻합니다.

#### Usage

Specify injection scope by passing the `scope` property to the `@Injectable()` decorator options object:

```typescript
import { Injectable, Scope } from "@nestjs/common";

@Injectable({ scope: Scope.REQUEST })
export class CatsService {}
```

Similarly, for [custom providers](/fundamentals/custom-providers), set the `scope` property in the long-hand form for a provider registration:

```typescript
{
  provide: 'CACHE_MANAGER',
  useClass: CacheManager,
  scope: Scope.TRANSIENT,
}
```

> info **Hint** Import the `Scope` enum from `@nestjs/common`

> warning **Notice** Gateways should not use request-scoped providers because they must act as singletons. Each gateway encapsulates a real socket and cannot be instantiated multiple times.

Singleton scope is used by default, and need not be declared. If you do want to declare a provider as singleton scoped, use the `Scope.DEFAULT` value for the `scope` property.

#### Controller scope

Controllers can also have scope, which applies to all request method handlers declared in that controller. Like provider scope, the scope of a controller declares its lifetime. For a request-scoped controller, a new instance is created for each inbound request, and garbage-collected when the request has completed processing.

Declare controller scope with the `scope` property of the `ControllerOptions` object:

```typescript
@Controller({
  path: "cats",
  scope: Scope.REQUEST,
})
export class CatsController {}
```

#### Scope hierarchy

Scope bubbles up the injection chain. A controller that depends on a request-scoped provider will, itself, be request-scoped.

Imagine the following dependency graph: `CatsController <- CatsService <- CatsRepository`. If `CatsService` is request-scoped (and the others are default singletons), the `CatsController` will become request-scoped as it is dependent on the injected service. The `CatsRepository`, which is not dependent, would remain singleton-scoped.

<app-banner-courses></app-banner-courses>

#### Request provider

In an HTTP server-based application (e.g., using `@nestjs/platform-express` or `@nestjs/platform-fastify`), you may want to access a reference to the original request object when using request-scoped providers. You can do this by injecting the `REQUEST` object.

```typescript
import { Injectable, Scope, Inject } from "@nestjs/common";
import { REQUEST } from "@nestjs/core";
import { Request } from "express";

@Injectable({ scope: Scope.REQUEST })
export class CatsService {
  constructor(@Inject(REQUEST) private request: Request) {}
}
```

Because of underlying platform/protocol differences, you access the inbound request slightly differently for Microservice or GraphQL applications. In [GraphQL](/graphql/quick-start) applications, you inject `CONTEXT` instead of `REQUEST`:

```typescript
import { Injectable, Scope, Inject } from "@nestjs/common";
import { CONTEXT } from "@nestjs/graphql";

@Injectable({ scope: Scope.REQUEST })
export class CatsService {
  constructor(@Inject(CONTEXT) private context) {}
}
```

You then configure your `context` value (in the `GraphQLModule`) to contain `request` as its property.

#### Performance

Using request-scoped providers will have an impact on application performance. While Nest tries to cache as much metadata as possible, it will still have to create an instance of your class on each request. Hence, it will slow down your average response time and overall benchmarking result. Unless a provider must be request-scoped, it is strongly recommended that you use the default singleton scope.
