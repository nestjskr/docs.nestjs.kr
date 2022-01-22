### 캐싱

캐싱은 앱의 성능을 향상시킬 수 있는 훌륭하면서도 간단한 **테크닉** 입니다. 캐싱은 고성능 데이터 액세스를 제공하는 임시 데이터 저장소 역할을 합니다.  

#### 설치

먼저 캐싱을 사용하는 데 필요한 패키지를 설치합니다.

```bash
$ npm install cache-manager
$ npm install -D @types/cache-manager
```

#### 인메모리 캐시

네스트는 다양한 캐시 스토리지 프로바이더를 사용할 수 있도록 통합 API를 제공하며, 네스트 내에 인메모리 데이터 저장소가 내장되어 있습니다. 레디스 같은 포괄적인 솔루션도 쉽게 사용할 수 있습니다. 

캐싱을 사용하려면, `CacheModule` 를 임포트 하고 해당 모듈의 `register()` 매소드를 호출합니다.

```typescript
import { CacheModule, Module } from '@nestjs/common';
import { AppController } from './app.controller';

@Module({
  imports: [CacheModule.register()],
  controllers: [AppController],
})
export class AppModule {}
```

#### 캐시 저장소와 상호작용

캐시 매니저 인스턴스를 사용하려면, 아래와 같이 `CACHE_MANAGER` 토큰을 사용해 클래스에 주입합니다.

```typescript
constructor(@Inject(CACHE_MANAGER) private cacheManager: Cache) {}
```

> 정보 **힌트** `Cache` 클래스는 `cache-manager`에서 임포트 합니다. `CACHE_MANAGER` 토큰은 `@nestjs/common` 패키지에서 임포트 합니다.

`Cache` 인스턴스(`cache-manager` 패키지) 의 `get` 메소드는 캐시에서 아이템을 갖고오는데 사용합니다. 만약 캐시에 항목이 없으면 `null` 을 반환합니다.

```typescript
const value = await this.cacheManager.get('key');
```

캐시에 아이템을 추가하려면, `set` 메소드를 사용합니다:

```typescript
await this.cacheManager.set('key', 'value');
```

캐시의 기본 만료 시간은 5초 입니다.

다음과 같이 이 특정 키에 대한 TTL(초단위 만료시간)를 수동으로 지정할 수 있습니다. 

```typescript
await this.cacheManager.set('key', 'value', { ttl: 1000 });
```

캐시 만료를 비활성화하려면, `ttl` 구성 속성을 `0` 으로 설정합니다:

```typescript
await this.cacheManager.set('key', 'value', { ttl: 0 });
```

캐시에서 아이템을 삭제하려면, `del` 메소드를 사용합니다:

```typescript
await this.cacheManager.del('key');
```

캐시 전체를 비우려면, `reset` 메소드를 사용합니다:

```typescript
await this.cacheManager.reset();
```

#### 오토-캐싱 리스폰스(Auto-caching responses)

> **경고** [GraphQL](/graphql/quick-start) 어플리케이션에서 인터셉터는 각 필드 리졸버 마다 별도로 실행합니다. 따라서 `CacheModule` (인터셉터를 사용해서 응답을 캐시) 이 제대로 작동하지 않을 수 도 있습니다.

오토-캐싱 리스폰스를 활성화하려면, 캐시하려는 데이터에 `CacheInterceptor` 를 연결하면 됩니다.

```typescript
@Controller()
@UseInterceptors(CacheInterceptor)
export class AppController {
  @Get()
  findAll(): string[] {
    return [];
  }
}
```

> 경고**경고** 오직 `GET` 엔드포인트만 캐시합니다. 또한 네이티브 리스폰스 오브젝트(`@Res()`)를 주입한 HTTP 서버 라우트는 캐시 인터셉터를 사용할 수 없습니다.
> 자세한 내용은 <a href="https://docs.nestjs.com/interceptors#response-mapping">리스폰스 매핑</a> 을 참조하세요.

필요한 보일러플레이트의 양을 줄이기 위해, `CacheInterceptor` 를 전역으로 모든 엔드포인트에 바인딩 할 수 있습니다.

```typescript
import { CacheModule, Module, CacheInterceptor } from '@nestjs/common';
import { AppController } from './app.controller';
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  imports: [CacheModule.register()],
  controllers: [AppController],
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: CacheInterceptor,
    },
  ],
})
export class AppModule {}
```

#### 커스터마이즈 캐시 

캐시된 모든 데이터에는 고유한 만료 시간이 있습니다. ([TTL](https://en.wikipedia.org/wiki/Time_to_live)). 기본값을 사용자가 정의하려면, options 객체를 `register()` 메소드로 전달합니다.

```typescript
CacheModule.register({
  ttl: 5, // 초
  max: 10, // 캐시의 최대 아이템 수
});
```

#### 모듈 전역 사용

다른 모듈에서 `CacheModule` 을 사용하려면, 임포트를 해야 합니다. (모든 Nest 모듈이 기본적으로 임포트가 필요합니다). 또는 아래와 같이 옵션 객체의 `isGlobal` 속성을 `true` 로 설정해 [전역 모듈](https://docs.nestjs.com/modules#global-modules) 로 선언할 수 있습니다. 이런 경우에는, 루트 모듈 (예: `AppModule`) 이 로드 된 뒤 다른 모듈에 `CacheModule` 를 임포트할 필요가 없습니다.

```typescript
CacheModule.register({
  isGlobal: true,
});
```

#### 글로벌 캐시 오버라이드

전역 캐시가 활성화되어 있는 동안에는 경로를 기반으로 자동 생성되는 `CacheKey` 아래에 캐시 항목을 저장합니다. 메소드 별로 특정 캐시 설정 (`@CacheKey()` 과 `@CacheTTL()`)을 재정의 해 개별 컨트롤러 메소드마다 맞춤형 캐싱 전략을 사용할 수 있습니다. [다른 캐시 저장소.](https://docs.nestjs.com/techniques/caching#different-stores)를 사용할 때 필요할 수 있습니다.

```typescript
@Controller()
export class AppController {
  @CacheKey('custom_key')
  @CacheTTL(20)
  findAll(): string[] {
    return [];
  }
}
```

> 정보 **힌트** `@CacheKey()` 와 `@CacheTTL()` 데코레이터는 `@nestjs/common` 패키지에서 임포트 합니다.

`@CacheKey()` 데코레이터는 `@CacheTTL()` 데코레이터와 함께 또는 단독으로 사용할 수 있으며, 그 반대도 가능합니다. `@CacheKey()` 만, `@CacheTTL()` 만 오버라이드 하도록 선택할 수 있으며 데코레이터로 오버라이드하지 않은 설정은 전역으로 등록된 기본값을 사용합니다. ([캐시 사용자 지정](https://docs.nestjs.com/techniques/caching#customize-caching) 보기).

#### 웹소켓과 마이크로서비스

웹소켓 구독자와 마이크로서비스 패턴에 `CacheInterceptor` 적용할 수 도 있습니다. (사용중인 전송방법과 관계없이 적용 가능합니다).

```typescript
@@filename()
@CacheKey('events')
@UseInterceptors(CacheInterceptor)
@SubscribeMessage('events')
handleEvent(client: Client, data: string[]): Observable<string[]> {
  return [];
}
@@switch
@CacheKey('events')
@UseInterceptors(CacheInterceptor)
@SubscribeMessage('events')
handleEvent(client, data) {
  return [];
}
```

그러나 데이터를 캐시에 저장하거나 가져오는 데 사용하는 키를 지정하려면 `@CacheKey()` 데코레이터가 추가로 필요합니다. 또한 **모든 걸 캐시하면 안된다**는 걸 명심하세요. 단순 데이터 쿼리가 아닌 비즈니스 작업을 수행하는 작업은 캐시해서는 안됩니다. 
 
또한 `@CacheTTL()` 데코레이터를 사용해서 캐시 만료 시간(TTL)을 지정하면 전역 캐시 만료 시간(TTL)을 오버라이드합니다.

```typescript
@@filename()
@CacheTTL(10)
@UseInterceptors(CacheInterceptor)
@SubscribeMessage('events')
handleEvent(client: Client, data: string[]): Observable<string[]> {
  return [];
}
@@switch
@CacheTTL(10)
@UseInterceptors(CacheInterceptor)
@SubscribeMessage('events')
handleEvent(client, data) {
  return [];
}
```

> 정보 **힌트** `@CacheTTL()` 데코레이터는 `CacheModule` 데코레이터와 함께 또는 단독으로 사용할 수 있습니다. 

#### 트래킹 조정

기본적으로 Nest는 요청 URL(HTTP 앱에서) 또는 캐시 키(웹 소켓 및 마이크로서비스 앱에서 `@CacheKey()` 데코레이터를 통해 설정)를 사용하여 캐시 레코드를 엔드포인트와 연결합니다. 그럼에도 불구하고 HTTP 헤더(예: `profile` 엔드포인트를 올바르게 식별하기 위한 `Authorization`)를 사용과 같이 다양한 요소를 기반으로 추적을 설정하고 싶을 수도 있습니다.

이렇게 하려면 `CacheInterceptor` 의 서브클래스를 생성하고 `trackBy()` 메소드를 사용해 오버라이드 합니다.

```typescript
@Injectable()
class HttpCacheInterceptor extends CacheInterceptor {
  trackBy(context: ExecutionContext): string | undefined {
    return 'key';
  }
}
```

#### 다른 저장소

이 서비스는 [캐시 매니저](https://github.com/BryanDonovan/node-cache-manager)를 사용합니다. `cache-manager` 패키지는 [레디스](https://github.com/dabroek/node-cache-manager-redis-store)와 같은 다양한 유용한 저장소를 지원합니다. 지원되는 스토어의 전체 목록은 [여기](https://github.com/BryanDonovan/node-cache-manager#store-engines)에서 확인할 수 있습니다. 레디스를 설정하려면 해당 옵션과 함께 패키지를 `register()` 메소드에 전달하기만 하면 됩니다.

```typescript
import type { ClientOpts as RedisClientOpts } from 'redis'
import * as redisStore from 'cache-manager-redis-store';
import { CacheModule, Module } from '@nestjs/common';
import { AppController } from './app.controller';

@Module({
  imports: [
    CacheModule.register<RedisClientOpts>({
      store: redisStore,
      // Store-specific configuration:
      host: 'localhost',
      port: 6379,
    }),
  ],
  controllers: [AppController],
})
export class AppModule {}
```

#### 비동기 구성

컴파일 시간에 정적으로 전달하는 대신, 모듈 옵션을 비동기로 전달할 수 있습니다. 이 경우 비동기 구성을 처리하는 여러가지 방법을 제공하는 `registerAsync()`  메소드를 사용합니다.  

팩토리 함수를 사용합니다:

```typescript
CacheModule.registerAsync({
  useFactory: () => ({
    ttl: 5,
  }),
});
```

팩토리는 다른 비동기 모듈 팩토리처럼 작동합니다. (`async` 이거나 `inject`를 통해 종속성을 주입받을 수 있습니다).

```typescript
CacheModule.registerAsync({
  imports: [ConfigModule],
  useFactory: async (configService: ConfigService) => ({
    ttl: configService.get('CACHE_TTL'),
  }),
  inject: [ConfigService],
});
```

또는 `useClass` 메소드를 사용할 수 있습니다:

```typescript
CacheModule.registerAsync({
  useClass: CacheConfigService,
});
```

위의 구성은 `CacheModule` 내부의 `CacheConfigService` 를 인스턴스화 하고 이를 사용해 옵션 객체를 가져옵니다. `CacheConfigService` 는 `CacheOptionsFactory` 인터페이스를 구현해야 구성옵션을 제공할 수 있습니다:

```typescript
@Injectable()
class CacheConfigService implements CacheOptionsFactory {
  createCacheOptions(): CacheModuleOptions {
    return {
      ttl: 5,
    };
  }
}
```

다른 모듈에서 가져온 기존 구성 공급자를 사용하려면 `useExisting` 구문을 사용합니다:

```typescript
CacheModule.registerAsync({
  imports: [ConfigModule],
  useExisting: ConfigService,
});
```

이것은 `useClass` 와 동일하게 작동하지만 단 한가지 큰 차이점이 있습니다. `CacheModule` 은 인스턴스를 생성하지 않고, 이미 생성된 `ConfigService` 을 재사용하기 위해 임포트 하고 있는 모듈을 조회합니다.

> 정보 **힌트** `CacheModule#register` 와 `CacheModule#registerAsync` 와 `CacheOptionsFactory` 는 저장소별 구성 옵션 까지 좁혀진 선택적 제네릭 (타입 아규먼트) 을 사용해 타입을 안전하게 합니다. 

#### 예제

작업 예시는 [여기](https://github.com/nestjs/nest/tree/master/sample/20-cache)에서 볼 수 있습니다.
