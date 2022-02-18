### Module reference

Nest는 `ModuleRef` 클래스를 제공하여 프로파이더의 내부 목록을 탐색하고 인젝션 토큰을 조회 키로 사용하는 모든 프로파이더에 대한 참조를 가져옵니다. `ModuleRef` 클래스는 정적 및 범위 프로파이더를 모두 동적으로 인스턴스화하는 방법도 제공합니다. `ModuleRef`는 일반적인 방법으로 클래스에 인젝션될 수 있습니다:

```typescript
@@filename(cats.service)
@Injectable()
export class CatsService {
  constructor(private moduleRef: ModuleRef) {}
}
@@switch
@Injectable()
@Dependencies(ModuleRef)
export class CatsService {
  constructor(moduleRef) {
    this.moduleRef = moduleRef;
  }
}
```

> info **Hint** `ModuleRef` 클래스는 `@nestjs/core` 패키지에서 import됩니다.

#### Retrieving instances

`ModuleRef` 인스턴스 (이후 **모듈 참조**라고 함.)는 `get()` 메서드를 가집니다. 이 메서드는 인젝션 토큰/클래스 이름을 사용하여 **현재** 모듈에 존재하는 (인스턴스화된)프로바이더, 컨트롤러 또는 인젝터블(e.g., guard, interceptor, etc.)을 조회합니다.

```typescript
@@filename(cats.service)
@Injectable()
export class CatsService implements OnModuleInit {
  private service: Service;
  constructor(private moduleRef: ModuleRef) {}

  onModuleInit() {
    this.service = this.moduleRef.get(Service);
  }
}
@@switch
@Injectable()
@Dependencies(ModuleRef)
export class CatsService {
  constructor(moduleRef) {
    this.moduleRef = moduleRef;
  }

  onModuleInit() {
    this.service = this.moduleRef.get(Service);
  }
}
```

> warn **경고** `get()` 메서드로 스코프 프로바이더(transient or request-scoped)를 조회하면 안 됩니다. 대신에 <a href="https://docs.nestjs.kr/fundamentals/module-ref#resolving-scoped-providers">밑에</a> 서술된 기법을 사용하십시오. 스코프를 컨트롤 하는 방법 배우기 [here](/fundamentals/injection-scopes).

전역 컨텍스트에서(예를 들어, 프로바이더가 다른 모듈에서 인젝션되었다면) 프로바이더를 조회하기 위해서는 `{ strict: false }` 옵션을 `get()`에 두 번째 인자로 넘겨주십시오.

```typescript
this.moduleRef.get(Service, { strict: false });
```

#### Resolving scoped providers

스코프 프로바이더 (transient 또는 request-scoped)를 동적으로 리졸브하기 위해서는 인젝션 토큰을 인자로 넘겨주는`resolve()` 메서드를 사용하십시오.,

```typescript
@@filename(cats.service)
@Injectable()
export class CatsService implements OnModuleInit {
  private transientService: TransientService;
  constructor(private moduleRef: ModuleRef) {}

  async onModuleInit() {
    this.transientService = await this.moduleRef.resolve(TransientService);
  }
}
@@switch
@Injectable()
@Dependencies(ModuleRef)
export class CatsService {
  constructor(moduleRef) {
    this.moduleRef = moduleRef;
  }

  async onModuleInit() {
    this.transientService = await this.moduleRef.resolve(TransientService);
  }
}
```

`resolve()` 메서드는 프로바이더의 **DI container sub-tree**로부터 고유한 프로바이더 인스턴스를 반환합니다. 각각의 서브트리는 유니크한 **컨텍스트 식별자**를 가집니다. 그래서 만약 이 메서드를 한 번 이상 호출하고 인스턴스 참조를 비교한다면 이 둘이 같지 않음을 확인할 것입니다.

```typescript
@@filename(cats.service)
@Injectable()
export class CatsService implements OnModuleInit {
  constructor(private moduleRef: ModuleRef) {}

  async onModuleInit() {
    const transientServices = await Promise.all([
      this.moduleRef.resolve(TransientService),
      this.moduleRef.resolve(TransientService),
    ]);
    console.log(transientServices[0] === transientServices[1]); // false
  }
}
@@switch
@Injectable()
@Dependencies(ModuleRef)
export class CatsService {
  constructor(moduleRef) {
    this.moduleRef = moduleRef;
  }

  async onModuleInit() {
    const transientServices = await Promise.all([
      this.moduleRef.resolve(TransientService),
      this.moduleRef.resolve(TransientService),
    ]);
    console.log(transientServices[0] === transientServices[1]); // false
  }
}
```

여러 `resolve()`호출을 걸쳐 단일 인스턴스를 생성하고, 이들이 동일한 생성된 DI container sub-tree를 공유하도록 보장하려면 컨텍스트 식별자를 `resolve()` 메서드에 전달하면 됩니다. 컨텍스트 식별자를 생성하기 위해서 `ContextIdFactory` 클래스를 사용하십시오. 이 클래스는 적절한 고유 식별자를 반환하는 `create()` 메서드를 제공합니다.

```typescript
@@filename(cats.service)
@Injectable()
export class CatsService implements OnModuleInit {
  constructor(private moduleRef: ModuleRef) {}

  async onModuleInit() {
    const contextId = ContextIdFactory.create();
    const transientServices = await Promise.all([
      this.moduleRef.resolve(TransientService, contextId),
      this.moduleRef.resolve(TransientService, contextId),
    ]);
    console.log(transientServices[0] === transientServices[1]); // true
  }
}
@@switch
@Injectable()
@Dependencies(ModuleRef)
export class CatsService {
  constructor(moduleRef) {
    this.moduleRef = moduleRef;
  }

  async onModuleInit() {
    const contextId = ContextIdFactory.create();
    const transientServices = await Promise.all([
      this.moduleRef.resolve(TransientService, contextId),
      this.moduleRef.resolve(TransientService, contextId),
    ]);
    console.log(transientServices[0] === transientServices[1]); // true
  }
}
```

> info **힌트** `ContextIdFactory` 클래스는 `@nestjs/core` 패키지에서 import 됩니다.

#### Registering `REQUEST` provider

(`ContextIdFactory.create()`로) 수동으로 생성된 컨텍스트 식별자는 Nest 의존성 주입 시스템에 의해 인스턴스화 되고 관리되지 않으므로 `REQUEST` 프로바이더가 `undefined` DI sub-trees를 나타낸다.

수동으로 생성된 DI sub-tree를 위해 커스텀 `REQUEST` 오브젝트를 등록하기 위해서, `ModuleRef#registerRequestByContextId()` 메서드를 다음과 같이 사용하십시오:

```typescript
const contextId = ContextIdFactory.create();
this.moduleRef.registerRequestByContextId(/* YOUR_REQUEST_OBJECT */, contextId);
```

#### Getting current sub-tree

경우에 따라 **request context** 안에 있는 request-scoped 프로바이더 인스턴스를 리졸브하고 싶을 수도 있습니다. `CatsService`가 request-scoped 이고 당신이 request-scoped 프로바이더로 표시된`CatsRepository` 인스턴스를 리졸브 하고 싶다고 해봅시다. In order to share the 같은 DI container sub-tree를 공유하기 위해서, (e.g., 위에서 보여준 것과 같이`ContextIdFactory.create()` 함수로 생성했던) 새로운 컨텍스트 식별자를 생성하는 대신 현재 컨텍스트 식별자를 얻어야만 합니다. 현재 컨텍스트 식별자를 얻기 위해서, `@Inject()` 데코레이더를 사용해서 요청 객체를 주입하는 것으로 시작하십시오.

```typescript
@@filename(cats.service)
@Injectable()
export class CatsService {
  constructor(
    @Inject(REQUEST) private request: Record<string, unknown>,
  ) {}
}
@@switch
@Injectable()
@Dependencies(REQUEST)
export class CatsService {
  constructor(request) {
    this.request = request;
  }
}
```

> info **힌트** 요청 프로바이더에 더 배우고 싶다면 [here](https://docs.nestjs.kr/fundamentals/injection-scopes#request-provider).

이제, 요청 객체에 기반을 둔 컨텍스트 id를 생성하기 위해 `ContextIdFactory` 클래스의 `getByRequest()` 메서드를 사용하고 컨텍스트 id를 `resolve()` 호출에 전달하십시오:

```typescript
const contextId = ContextIdFactory.getByRequest(this.request);
const catsRepository = await this.moduleRef.resolve(CatsRepository, contextId);
```

#### Instantiating custom classes dynamically

동적으로 **이전에 프로바이더로 등록되지 않은** 클래스를 인스턴스화 시키기 위해서, 모듈 참조의 `create()` 메서드를 사용하십시오.

```typescript
@@filename(cats.service)
@Injectable()
export class CatsService implements OnModuleInit {
  private catsFactory: CatsFactory;
  constructor(private moduleRef: ModuleRef) {}

  async onModuleInit() {
    this.catsFactory = await this.moduleRef.create(CatsFactory);
  }
}
@@switch
@Injectable()
@Dependencies(ModuleRef)
export class CatsService {
  constructor(moduleRef) {
    this.moduleRef = moduleRef;
  }

  async onModuleInit() {
    this.catsFactory = await this.moduleRef.create(CatsFactory);
  }
}
```

이 기법은 조건적으로 프레임워크 컨테이너 바깥의 다른 클래스를 인스턴스화 할 수 있게 해줍니다.

<app-banner-shop></app-banner-shop>
