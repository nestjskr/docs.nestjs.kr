### Injection scopes

서로 다른 개발언어 경험을 가진 사람들에게는, 들어오는 모든 요청 간에 거의 모든 것이 공유된다는 사실을 예상하지 못할 수 있습니다. 우리에게는 데이터베이스와의 연결 풀, 전역 상태를 가지는 싱글톤 서비스 등이 있습니다. Node.js는 모든 요청이 각각 분리된 스레드에서 처리되는 Multi-Threaded Stateless 모델을 따르지 않는다는 것을 기억하고 있어야 합니다. 이 때문에, 싱글톤 인스턴스를 사용하는 것이 우리의 애플리케이션에게 전적으로 **안전**합니다.

하지만 GraphQL 애플리케이션에서의 요청별 캐싱, 요청 트래킹, 멀티테넌시와 같이 인스턴스가 요청에 기반한 수명을 가지길 바라는 경우도 있습니다.

#### Provider scope

A provider can have any of the following scopes:

<table>
  <tr>
    <td><code>DEFAULT</code></td>
    <td>A single instance of the provider is shared across the entire application. The instance lifetime is tied directly to the application lifecycle. Once the application has bootstrapped, all singleton providers have been instantiated. Singleton scope is used by default.</td>
  </tr>
  <tr>
    <td><code>REQUEST</code></td>
    <td>A new instance of the provider is created exclusively for each incoming <strong>request</strong>.  The instance is garbage-collected after the request has completed processing.</td>
  </tr>
  <tr>
    <td><code>TRANSIENT</code></td>
    <td>Transient providers are not shared across consumers. Each consumer that injects a transient provider will receive a new, dedicated instance.</td>
  </tr>
</table>

> info **Hint** Using singleton scope is **recommended** for most use cases. Sharing providers across consumers and across requests means that an instance can be cached and its initialization occurs only once, during application startup.

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
