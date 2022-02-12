### 비율 제한

**비율 제한**은 브루트-포스 공격으로부터 애플리케이션을 보호하는 일반적인 방법입니다. 이를 위해 `@nestjs/throttler` 패키지를 설치해야 합니다.

```bash
$ npm i --save @nestjs/throttler
```

설치가 완료되면, `ThrottlerModule`을 다른 Nest 패키지들처럼 `forRoot`나 `forRootAsync` 메서드를 이용하여 설정할 수 있습니다.

```typescript
@Module({
  imports: [
    ThrottlerModule.forRoot({
      ttl: 60,
      limit: 10,
    }),
  ],
})
export class AppModule {}
```

위의 예제에서는 보호받을 라우트들에게 적용될 전역 옵션을 설정합니다. 단위 시간인 `ttl`과 ttl동안의 최대 요청 횟수인 `limit`을 설정했습니다.

모듈이 import되면, `ThrottlerGuard`를 어떻게 바인딩할지 선택할 수 있습니다. [가드](/guards) 섹션에서 언급한 어느 유형의 바인딩이든 사용할 수 있습니다. 가드를 전역적으로 바인딩하고 싶다면, 어느 모듈에서든 아래 예와 같이 해당 프로바이더를 추가하면 됩니다:

```typescript
{
  provide: APP_GUARD,
  useClass: ThrottlerGuard
}
```

#### 사용자 정의

가드를 전역이나 컨트롤러에 바인딩하고 싶지만, 하나 혹은 여러 엔드포인트에 대해서는 비율 제한을 비활성화하고 싶은 경우도 있을 수 있습니다. 그런 경우에는 `@SkipThrottler()` 데코레이터를 사용하여 전체 클래스 또는 하나의 라우트에 쓰로틀러를 배제시킬 수 있습니다. 전체 라우트는 아니지만 특정 컨트롤러의 *대부분*은 제외하고 싶다면 `@SkipThrottler()` 데코레이터에 boolean을 취할 수도 있습니다.

보안 옵션을 더 엄격하거나 느슨하게 조절할 수 있도록, 전역 모듈에 설정된 `limit`과 `ttl`을 덮어씌울 수 있는 `@Throttele()` 데코레이터도 있습니다. 이 데코레이터 또한 클래스나 함수 위에 사용할 수 있습니다. 이 데코레이터에는 인자를 `limit, ttl`의 순서로 넘기는 것에 유의해야 합니다.

#### 프록시

만약 애플리케이션이 프록시 서버 뒤에서 실행된다면, `프록시 신뢰` 옵션에 대한 HTTP 어댑터 옴션([express](http://expressjs.com/en/guide/behind-proxies.html), [fastify](https://www.fastify.io/docs/latest/Server/#trustproxy))을 찾아서 활성화 하면 됩니다. 그렇게 하면 `X-Forward-For` 헤더에 원래 IP 주소가 포함되며, `getTracker()` 메서드를 오버라이딩하여 `req.ip`가 아니라 헤더에서 그 값을 받아올 수 있습니다. 아래 예제는 express와 fastify 모두 동일합니다:

```ts
// throttler-behind-proxy.guard.ts
import { ThrottlerGuard } from '@nestjs/throttler';
import { Injectable } from '@nestjs/common';

@Injectable()
export class ThrottlerBehindProxyGuard extends ThrottlerGuard {
  protected getTracker(req: Record<string, any>): string {
    return req.ips.length ? req.ips[0] : req.ip; // individualize IP extraction to meet your own needs
  }
}

// app.controller.ts
import { ThrottlerBehindProxyGuard } from './throttler-behind-proxy.guard';

@UseGuards(ThrottlerBehindProxyGuard)
```

> info **힌트** 요청 객체 `req`에 대한 API는 express는 [여기](https://expressjs.com/en/api.html#req.ips)에서, fastify는 [여기](https://www.fastify.io/docs/latest/Request/)에서 확인할 수 있습니다.

#### 웹소켓

이 모듈은 웹소켓에서도 동작하지만 몇 가지 확장 클래스가 필요합니다. 다음과 같이 `ThrottlerGuard`를 상속받고 `handleRequest` 메서드를 오버라이딩합니다:

```typescript
@Injectable()
export class WsThrottlerGuard extends ThrottlerGuard {
  async handleRequest(
    context: ExecutionContext,
    limit: number,
    ttl: number
  ): Promise<boolean> {
    const client = context.switchToWs().getClient();
    const ip = client.conn.remoteAddress;
    const key = this.generateKey(context, ip);
    const ttls = await this.storageService.getRecord(key);

    if (ttls.length >= limit) {
      throw new ThrottlerException();
    }

    await this.storageService.addRecord(key, ttl);
    return true;
  }
}
```

> info **힌트** 만약 `@nestjs/platform-ws` 패키지를 사용한다면 `client._socket.remoteAddress`를 사용해야 합니다.

#### GraphQL

`ThrottlerGuard`는 GraphQL 요청에 대해서도 동작합니다. 역시 가드를 확장해야 하는데, 이번에는 `getRequestResponse` 메서드를 오버라이딩합니다.

```typescript
@Injectable()
export class GqlThrottlerGuard extends ThrottlerGuard {
  getRequestResponse(context: ExecutionContext) {
    const gqlCtx = GqlExecutionContext.create(context);
    const ctx = gqlCtx.getContext();
    return { req: ctx.req, res: ctx.res };
  }
}
```

#### 설정

아래는 `ThrottlerModule`이 가질 수 있는 옵션들입니다:

<table>
  <tr>
    <td><code>ttl</code></td>
    <td>각 요청이 저장소에 존재하는 시간</td>
    <td></td>
  </tr>
  <tr>
    <td><code>limit</code></td>
    <td>TTL 제한 내 최대 요청 횟수</td>
  </tr>
  <tr>
    <td><code>ignoreUserAgents</code></td>
    <td>쓰로틀링 요청이 들어왔을 때 무시할 user-agents에서 무시할 정규표현식들을 담은 배열</td>
  </tr>
  <tr>
    <td><code>storage</code></td>
    <td>요청을 추적하는 방법에 대한 저장소 설정</td>
  </tr>
</table>

#### 비동기 설정

비율 제한에 대한 설정을 동기적이 아닌 비동기적으로 하고 싶다면, `forRootAsync()` 메서드를 사용하여 `async` 메서드와 의존성 주입을 허용할 수 있습니다.

팩토리 함수를 사용하는 것도 하나의 방법입니다:

```typescript
@Module({
  imports: [
    ThrottlerModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        ttl: config.get("THROTTLE_TTL"),
        limit: config.get("THROTTLE_LIMIT"),
      }),
    }),
  ],
})
export class AppModule {}
```

`useClass` 구문을 사용할 수도 있습니다:

```typescript
@Module({
  imports: [
    ThrottlerModule.forRootAsync({
      imports: [ConfigModule],
      useClass: ThrottlerConfigService,
    }),
  ],
})
export class AppModule {}
```

이는 `ThrottlerConfigService`가 구현하는 `ThrottlerOptionsFactory` 인터페이스를 따르는 한 얼마든지 자유롭게 작성할 수 있습니다.

#### 저장소

내장 저장소는 메모리에 캐시되어, 전역 옵션으로 설정한 TTL값을 넘길 때까지 요청을 추적합니다. `ThrottlerModule`의 `storage` 옵션에 `ThrottlerStorage` 인터페이스를 따르는 한에서 자유롭게 자신의 저장소 옵션을 추가할 수 있습니다.

분산 서버 환경의 경우, [Redis](https://github.com/kkoomen/nestjs-throttler-storage-redis)용 커뮤니티 저장소 프로바이더를 사용함으로써 신뢰할 수 있는 단일 소스를 가질 수 있습니다.

> info **참고** `ThrottlerStorage`는 `@nestjs/throttler`에서 import합니다.
