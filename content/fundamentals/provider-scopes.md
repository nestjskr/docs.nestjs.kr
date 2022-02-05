### 스코프 주입하기

서로 다른 개발언어 경험을 가진 사람들에게는, 들어오는 모든 요청 간에 거의 모든 것이 공유된다는 사실을 예상하지 못할 수 있습니다. 우리에게는 데이터베이스와의 연결 풀, 전역 상태를 가지는 싱글톤 서비스 등이 있습니다. Node.js는 모든 요청이 각각 분리된 스레드에서 처리되는 Multi-Threaded Stateless 모델을 따르지 않는다는 것을 기억하고 있어야 합니다. 이 때문에, 싱글톤 인스턴스를 사용하는 것이 우리의 애플리케이션에게 전적으로 **안전**합니다.

하지만 GraphQL 애플리케이션에서의 요청별 캐싱, 요청 트래킹, 멀티테넌시와 같이 인스턴스가 요청에 기반한 수명을 가지길 바라는 경우도 있습니다.

#### 프로바이더의 스코프

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

#### 사용법

주입되는 프로바이더의 스코프를 정하려면 `@Injectable()`데코레이터의 options객체에 `scope`프로퍼티를 넘겨야 합니다:

```typescript
import { Injectable, Scope } from "@nestjs/common";

@Injectable({ scope: Scope.REQUEST })
export class CatsService {}
```

비슷하게, [custom providers](/fundamentals/custom-providers)에서도 프로바이더를 등록하는 과정에서 `scope`프로퍼티를 지정했습니다.

```typescript
{
  provide: 'CACHE_MANAGER',
  useClass: CacheManager,
  scope: Scope.TRANSIENT,
}
```

> info **힌트** `Scope` enum은 `@nestjs/common`에서 import합니다

> warning **주의** Gateway는 실제 소켓을 캡슐화하기 때문에 여러 번 인스턴스화 될 수 없습니다. 즉, Gateway는 무조건 싱글톤으로 동작해야 하기에 요청기반 스코프를 가지는 프로바이더를 사용할 수 없습니다.

싱글톤 스코프는 기본으로 적용되기에 따로 명시할 필요가 없습니다. 만약 프로바이더가 싱글톤 스코프를 가지도록 명시하고 싶다면, `scope`프로퍼티의 값으로 `Scope.DEFAULT`를 넣으면 됩니다.

#### 컨트롤러의 스코프

컨트롤러 또한 스코프를 가질 수 있으며, 해당 컨트롤러에 속한 모든 요청핸들러 메서드들에게 적용됩니다. 프로바이더의 스코프처럼, 컨트롤러의 스코프도 그 수명을 결정합니다. 요청기반 스코프를 가지는 컨트롤러는 들어오는 각 요청에 대해 새로운 인스턴스를 생성하고 요청이 완전히 처리되면 가비지 컬렉션에 의해 수거됩니다.

컨트롤러의 스코프를 정의하려면 `ControllerOptions`객체의 `scope`프로퍼티를 사용합니다:

```typescript
@Controller({
  path: "cats",
  scope: Scope.REQUEST,
})
export class CatsController {}
```

#### 스코프 계층구조

스코프의 주입은 연쇄적으로 버블링 됩니다. 어떤 컨트롤러가 요청기반 스코프의 프로바이더에 의존한다면, 그 컨트롤러도 요청기반 스코프를 가지게 됩니다.

다음과 같은 의존성 구조를 상상해 봅시다: `CatsController <- CatsService <- CatsRepository`. 만약 `CatsService`만 요청기반 스코프를 가지고 나머지는 기본 스코프인 싱글톤 스코프를 가진다면, `CatsController`는 주입받는 서비스에 의존하므로 요청기반 스코프를 가지게 됩니다. `CatsRepository`는 서비스에 의존하지 않으므로 싱글톤 스코프 그대로 남습니다.

<app-banner-courses></app-banner-courses>

#### 요청 프로바이더

HTTP 서버 기반의 애플리케이션(이를테면 `@nestjs/platform-express` 혹은 `@nestjs/platform-fastify`)에서, 요청기반 스코프의 프로바이더를 사용하면서 원래의 요청객체를 참조하고 싶을 수 있습니다. 이는 `REQUEST`객체를 주입함으로써 가능합니다.

```typescript
import { Injectable, Scope, Inject } from "@nestjs/common";
import { REQUEST } from "@nestjs/core";
import { Request } from "express";

@Injectable({ scope: Scope.REQUEST })
export class CatsService {
  constructor(@Inject(REQUEST) private request: Request) {}
}
```

기본적인 플랫폼이나 프로토콜의 차이로 인해, 마이크로서비스나 GraphQL을 사용하는 분들은 들어오는 요청에 접근하는 방법이 아주 살짝 다릅니다. [GraphQL](/graphql/quick-start) 애플리케이션에서는, `REQUEST`대신에 `CONTEXT`를 주입합니다.

```typescript
import { Injectable, Scope, Inject } from "@nestjs/common";
import { CONTEXT } from "@nestjs/graphql";

@Injectable({ scope: Scope.REQUEST })
export class CatsService {
  constructor(@Inject(CONTEXT) private context) {}
}
```

그 다음, `request`프로퍼티를 포함시키도록 `GraphQLModule`에서 `context`의 값을 구성해야 합니다.

#### Performance

Using request-scoped providers will have an impact on application performance. While Nest tries to cache as much metadata as possible, it will still have to create an instance of your class on each request. Hence, it will slow down your average response time and overall benchmarking result. Unless a provider must be request-scoped, it is strongly recommended that you use the default singleton scope.
