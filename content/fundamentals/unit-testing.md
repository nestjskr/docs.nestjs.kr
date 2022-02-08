### Testing

자동화된 테스트는 진지한 소프트웨어 개발의 필수적인 부분 이라 간주됩니다. 테스트 자동화는 개발하는 도중 각각의 테스트, 혹은 테스트 묶음을 쉽게 혹은 빠르게 반복할 수 있게 해줍니다. 이는 Release가 품질과 성능 목표치를 충족하는지 보장하는데 도움을 줍니다. 테스트 자동화는 보증 범위를 늘려주며, 빠른 테스트 피드백 반복 주기를 개발자에게 제공합니다. 자동화는 개별 개발자의 생산성을 높이며 소스 코드 제어 check-in, 기능 통합 및 버전 릴리즈와 같은 중요한 개발 생명 주기 시점에서 테스트가 실행 될 수 있도록 보장합니다.

이러한 테스트는 종종 단위 다양한 여러 유형으로, 유닛테스트를 포함한, 종단간 테스트, 통합 테스트등 다양한 유형에 걸쳐있습니다. 장점은 의심할 여지가 없습니다, 테스트를 설정하는건 지루할 수 있습니다. NEST는 효과적인 테스트를 포함하여 모범적인 개발사례를 발전시키기 위해 노력하고 있습니다, 그래서 다음과 같이 개발자들과 그 팀들이 자동화된 테스트를 구현하기 위한 기능들을 포함하고 있습니다. NEST :

- automatically scaffolds default unit tests for components and e2e tests for applications
- provides default tooling (such as a test runner that builds an isolated module/application loader)
- provides integration with [Jest](https://github.com/facebook/jest) and [Supertest](https://github.com/visionmedia/supertest) out-of-the-box, while remaining agnostic to testing tools
- makes the Nest dependency injection system available in the testing environment for easily mocking components

언급 했듯, 당신이 사용하고 싶은 어떠한 **테스트 프레임워크**라도 사용할 수 있습니다, Nest는 어떠한 특별한 테스팅 툴을 강제하지 않습니다. 간단하게 필요한 요소 ( 예를들어 테스트 러너 ) 를 교체하면 됩니다, 그럼 기존에 사용하던 그대로 Nest의 ready-made 테스트 시설들을 이용할 수 있습니다.

#### 설치방법

시작에 앞서, 필요한 패키지를 설치하여야 합니다:

```bash
$ npm i --save-dev @nestjs/testing
```

#### Unit Testing

이하 예시에서는, 두가지 Case를 테스트해볼 예정입니다.: `CatsController` 그리고 `CatsService`. 언급했듯이, [Jest](https://github.com/facebook/jest) 는 기본으로 제공되는 테스팅 프레임워크입니다. 테스트 러너 역할을 하며 assert function 기능과 mocking 과 spying, 및 기타 기능을 제공하는 test-double uttilities 기능을 제공합니다 . 이하 기본 테스트를 통해, 다음 기본 테스트에서, 이러한 클래스를 수동으로 인스턴스화하고 컨트롤러와 서비스를 실행하여 API가 제대로 동작하는지 확인합니다.

```typescript
@@filename(cats.controller.spec)
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

describe('CatsController', () => {
  let catsController: CatsController;
  let catsService: CatsService;

  beforeEach(() => {
    catsService = new CatsService();
    catsController = new CatsController(catsService);
  });

  describe('findAll', () => {
    it('should return an array of cats', async () => {
      const result = ['test'];
      jest.spyOn(catsService, 'findAll').mockImplementation(() => result);

      expect(await catsController.findAll()).toBe(result);
    });
  });
});
@@switch
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

describe('CatsController', () => {
  let catsController;
  let catsService;

  beforeEach(() => {
    catsService = new CatsService();
    catsController = new CatsController(catsService);
  });

  describe('findAll', () => {
    it('should return an array of cats', async () => {
      const result = ['test'];
      jest.spyOn(catsService, 'findAll').mockImplementation(() => result);

      expect(await catsController.findAll()).toBe(result);
    });
  });
});
```

> info **Hint** Keep your test files located near the classes they test. Testing files should have a `.spec` or `.test` suffix.

위의 예시는 사소한 것이기 때문에, Nest-Specific 한것을 테스트했다고 볼수는 없습니다. 대신, "실제로 우리는 의존성 주입을 사용하지도 않았습니다. ( `CatsService`를  `catsController` 로 건내주기만 했다는걸 기억해주시기 바랍니다). 이러한 형태의 테스트 - 테스트 하려는 클래스를 수동으로 인스턴수화 하는 것 - 이를 프레임워크에서 독립되어있다고 하여 **독립 테스트** 라고 명명하곤 합니다. Nest의 기능을 보다 광범위하게 사용하는, 애플리케이션 테스트에 도움되는 몇가지 고급 기능을 소개하겠습니다.

#### 테스팅 도구

The `@nestjs/testing` 패키지는 보다 강력한 테스트 프로세스를 가능하게 하는 유틸리티를 제공합니다. 내장된 `Test` class 를 사용하여 이전 예제를 다시 작성해 보도록 하겠습니다.:

```typescript
@@filename(cats.controller.spec)
import { Test } from '@nestjs/testing';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

describe('CatsController', () => {
  let catsController: CatsController;
  let catsService: CatsService;

  beforeEach(async () => {
    const moduleRef = await Test.createTestingModule({
        controllers: [CatsController],
        providers: [CatsService],
      }).compile();

    catsService = moduleRef.get<CatsService>(CatsService);
    catsController = moduleRef.get<CatsController>(CatsController);
  });

  describe('findAll', () => {
    it('should return an array of cats', async () => {
      const result = ['test'];
      jest.spyOn(catsService, 'findAll').mockImplementation(() => result);

      expect(await catsController.findAll()).toBe(result);
    });
  });
});
@@switch
import { Test } from '@nestjs/testing';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

describe('CatsController', () => {
  let catsController;
  let catsService;

  beforeEach(async () => {
    const moduleRef = await Test.createTestingModule({
        controllers: [CatsController],
        providers: [CatsService],
      }).compile();

    catsService = moduleRef.get(CatsService);
    catsController = moduleRef.get(CatsController);
  });

  describe('findAll', () => {
    it('should return an array of cats', async () => {
      const result = ['test'];
      jest.spyOn(catsService, 'findAll').mockImplementation(() => result);

      expect(await catsController.findAll()).toBe(result);
    });
  });
});
```

The `Test` 클래스는 전체 Nest 런타임을 모의화하는 애플리케이션 실행 컨텍스트를 제공하는데 유용합니다, 이는  클래스 인스턴스 관리 와 mocking, overriding 을 쉽게 관리 할 수 있는 hook 을 제공합니다. The `Test` 클래스는 `createTestingModule()` 모듈 메타데이터 개체를 인수로 사용하는 메서드 ( `@Module()` decorator 로 전달되는 동일한 객체 ) 를 사용합니다.  `TestingModule` 메소드는  몇 가지 메서드를 차례로 제공하는 인스턴스를 반환합니다. 유닛 테스트를 위해서, 무엇보다 중요한건 `compile()` 메소드 입니다. 이 메소드는 종속성이 있는 모듈을 bootstrap 합니다 (응용 프로그램이 기존의 `main.ts` 파일의 `NestFactory.create()`), 그리고 테스트가 준비된 모듈을 반환합니다.

> 정보) **힌트** The `compile()` 메소드는 **비동기 함수**로, await 되어야 합니다. 모듈이 compile 되면, `get()` 메소드를 이용하여 정의된 **static** instance ( 컨트롤러 그리고 프로바이더들) 을 가져올 수 있습니다.

`TestingModule` 은  [module reference](/fundamentals/module-ref) 클래스로 부터 상속되었습니다, 따라서 scoped provider(transient or request-scoped)를 동적으로 가져올수 있습니다. `resolve()` 함수로 가능합니다. (the `get()` 메소드는 정적인 인스턴스를 가져올 수 있습니다)

```typescript
const moduleRef = await Test.createTestingModule({
  controllers: [CatsController],
  providers: [CatsService],
}).compile();

catsService = await moduleRef.resolve(CatsService);
```

> 주의 **주의** The `resolve()` 메소드는 공급자의 고유한 인스턴스를 **DI 컨테이너 하위 트리** 에서 반환합니다. 각 하위 트리에는 고유한 컨텍스트 식별자가 존재합니다. 따라서 이 메서드를 두 번 이상 호출하고 인스턴스 참조를 비교하면 동일하지 않음을 알 수 있습니다.

> 정보 **힌트** 모듈 참조 기능에 대해 자세히 알아보기 [here](/fundamentals/module-ref).

공급자의 프로덕션 버전을 사용하는 대신 테스트를 위해 다음으로 재정의 할 수 있습니다 [custom provider](/fundamentals/custom-providers). 예를 들어, live 데이터베이스에 접속하는 대신, 데이터베이스 서비스를 mock 합니다. Mocking은 다음 섹션에서 다루지만 단위 테스트에서도 사용할 수 있습니다.

<app-banner-courses></app-banner-courses>

#### Auto mocking

Nest는 또한 누락된 모든 종속성에 적용할 mock factory를 정의할 수 있습니다. 클래스에 많은 수의 종속성이 있고 모든 종속성을 mocking 하는 데 오랜 시간과 많은 설정이 필요한 경우 유용합니다. 이 기능을 사용하려면, the `createTestingModule()` 과 `useMocker()` 메소드의  연결이 필요합니다,  의존성 mock 을 위한 factory 데이터 전달. 이 factory는 선택적으로 토큰을 가질 수 있습니다, Nest 제공자에 대한 유효한 모든 토큰인 instance 토큰이며 mock 구현을 반환합니다. 다음은 mocker 인  [`jest-mock`](https://www.npmjs.com/package/jest-mock) 을 사용하여  `CatsService`의 `jest.fn()`를 사용한 generic mocker 구현 예시입니다.

```typescript
const moduleMocker = new ModuleMocker(global);

describe('CatsController', () => {
  let controller: CatsController;

  beforeEach(async () => {
    const moduleRef = await Test.createTestingModule({
      controllers: [CatsController],
    })
    .useMocker((token) => {
      if (token === CatsService) {
        return { findAll: jest.fn().mockResolveValue(results) };
      }
      if (typeof token === 'function') {
        const mockMetadata = moduleMocker.getMetadata(token) as MockFunctionMetadata<any, any>;
        const Mock = moduleMocker.generateFromMetadata(mockMetadata);
        return new Mock();
      }
    })
    .compile();
    
    controller = moduleRef.get(CatsController);
  });
})
```

> 정보 **힌트** 일반적인 mock factory, like `createMock` from [`@golevelup/ts-jest`](https://github.com/golevelup/nestjs/tree/master/packages/testing) can also be passed directly.

You can also retrieve these mocks out of the testing container as you normally would custom providers, `moduleRef.get(CatsService)`.

#### End-to-end testing

단위 테스트와는 달리, 개별 모듈 및 클래스에 중점을 둡니다, 엔드 투 엔드 ( e2e ) 테스트는 보다 종합적인 스준에서 클래스와 모듈의 상호 작용을 다룹니다 -- 최종 사용자가 생산 시스템과 갖게 될 상호 작용 유형에 보다 가깝습니다. 애플리케이션이 커질수록, API endpoint를 수동으로 테스트 한다는것은 불가능에 가깝습니다. 자동화된 종단 간 테스트는 시스템의 전반적인 동작이 정확하고 프로젝트 요구 사항을 충족하는지 확인하는 데 도움이 됩니다. e2e 테스트를 수행하기 위해 **단위 테스트**에서 다룬 것과 유사한 구성을 사용합니다. 추가적으로NEST 는 [Supertest](https://github.com/visionmedia/supertest) 라이브러리를 사용하여 HTTP requests 를 시뮬레이션 합니다.

```typescript
@@filename(cats.e2e-spec)
import * as request from 'supertest';
import { Test } from '@nestjs/testing';
import { CatsModule } from '../../src/cats/cats.module';
import { CatsService } from '../../src/cats/cats.service';
import { INestApplication } from '@nestjs/common';

describe('Cats', () => {
  let app: INestApplication;
  let catsService = { findAll: () => ['test'] };

  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({
      imports: [CatsModule],
    })
      .overrideProvider(CatsService)
      .useValue(catsService)
      .compile();

    app = moduleRef.createNestApplication();
    await app.init();
  });

  it(`/GET cats`, () => {
    return request(app.getHttpServer())
      .get('/cats')
      .expect(200)
      .expect({
        data: catsService.findAll(),
      });
  });

  afterAll(async () => {
    await app.close();
  });
});
@@switch
import * as request from 'supertest';
import { Test } from '@nestjs/testing';
import { CatsModule } from '../../src/cats/cats.module';
import { CatsService } from '../../src/cats/cats.service';
import { INestApplication } from '@nestjs/common';

describe('Cats', () => {
  let app: INestApplication;
  let catsService = { findAll: () => ['test'] };

  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({
      imports: [CatsModule],
    })
      .overrideProvider(CatsService)
      .useValue(catsService)
      .compile();

    app = moduleRef.createNestApplication();
    await app.init();
  });

  it(`/GET cats`, () => {
    return request(app.getHttpServer())
      .get('/cats')
      .expect(200)
      .expect({
        data: catsService.findAll(),
      });
  });

  afterAll(async () => {
    await app.close();
  });
});
```

> 정보 **힌트** 만약 [Fastify](/techniques/performance) 를 HTTP adapter 로 사용하신다면, 약간 다른 구성이 필요하며, 내장된 테스트 기능이 있습니다.
>
> ```ts
> let app: NestFastifyApplication;
> 
> beforeAll(async () => {
> app = moduleRef.createNestApplication<NestFastifyApplication>(
>  new FastifyAdapter(),
> );
> 
> await app.init();
> await app.getHttpAdapter().getInstance().ready();
> });
> 
> it(`/GET cats`, () => {
> return app
>  .inject({
>    method: 'GET',
>    url: '/cats',
>  })
>  .then((result) => {
>    expect(result.statusCode).toEqual(200);
>    expect(result.payload).toEqual(/* expectedPayload */);
>  });
> });
> 
> afterAll(async () => {
> await app.close();
> });
> ```

이 예제에서는,앞에서 설명한 몇가지 개념을 기반으로 합니다. 이전에 사용한 `compile()` 메서드 외에도, `createNestApplication()` 메서드를 사용하여 전체 Nest 런타임 환경을 인스턴스화합니다. 실행중인 앱에 대한 참조를 `app` 변수에 저장하여 HTTP request 에 대한 요청을 시뮬레이션하는데 사용할 수 있습니다.

Supertest의  `request()` 함수를 사용하여 HTTP 테스트를 시뮬레이션 합니다. 이러한 HTTP 요청이 실행중인 Nest 앱으로 라우팅되기를 원하므로, `request()` 함수에 의해 HTTP 수신측에 참조를 전달합니다 (Express 플램폼에 의해 제공될 수 있음). 따라서 `request(app.getHttpServer())` 는 HTTP SERVER를 구성.  `request()`를 호출하면 이제 Nest 앱에 연결된 Wrapping 된 HTTP 서버가 제공되며, 이 서버는 실제 HTTP 요청을 시뮬레이션하는 메서드를 제공합니다. 예를 들어 `request(...).get('/cats')`을 사용하면 `get '/cats'` 같은 **실제**  인터넷을 통한  HTTP 요청과 동일한 Nest 앱에 대한 요청이 이루어집니다.

이 예에서는 테스트할 수 있는 하드 코딩된 값을 단순 히 반환하는 `CatsService`의 대체(이중 테스트) 구현도 제공합니다.  이러한 대체 구현을 사용하려면 `overrideProvider()` 를 사용하세요 . 마찬가지로, Nest는 각각 `overrideGuard()`, `overrideInterceptor()`, `overrideFilter()`, 및 `overridePipe()` 메서드를 사용하여 가드, 인터셉터, 필터 및 파이프를 재정의하는 메서드를 제공합니다..

각 재정의 메서드는 다음에 대해 설명된 메서드를 미러링하는 3가지 다른 메서드가 잇는 개체를 반환합니다 [custom providers](https://docs.nestjs.com/fundamentals/custom-providers):

- `useClass`: 개체를 재정의할 인스턴스를 제공하기 위해 인스턴스화 될 클래스 제공 (provider, guard, etc.).
- `useValue`: 개체를 재정의할 인스턴스 제공.
- `useFactory`: 객체를 재정의할 인스턴스를 반환하는 함수를 제공.

각 재정의 메서드 유형은 차례로 `TestingModule` 인스턴스를 반환하므로,  [fluent style](https://en.wikipedia.org/wiki/Fluent_interface). 의 다른 메서드와 연결할 수 있습니다. NEst가 모듈을 인스턴스화 하고 초기화하도록 하려면 이러한 체인의 끝에 `compile()` 을 사용해야 합니다.

또한, 때로는 사용자 정의 로거를 사용하고자 할 수 있습니다. 예를들어. 테스트가 실행될 때 (예) CI 서버에서). `setLogger()` 메서드를 사용하고 `LoggerService` 인터페이스를 충족하는 객체를 전달하고 `TestModuleBuilder` 에 테스트 중 로그 방법을 지시합니다. (기본적으로, 'error'만 콘솔창에 표시됨).

다음 표에 설명된 대로 컴파일된 모듈에는 몇 가지 유용한 메서드가 있습니다:

<table>
  <tr>
    <td>
      <code>createNestApplication()</code>
    </td>
    <td>
      Creates and returns a Nest application (<code>INestApplication</code> instance) based on the given module.
      Note that you must manually initialize the application using the <code>init()</code> method.
    </td>
  </tr>
  <tr>
    <td>
      <code>createNestMicroservice()</code>
    </td>
    <td>
      Creates and returns a Nest microservice (<code>INestMicroservice</code> instance) based on the given module.
    </td>
  </tr>
  <tr>
    <td>
      <code>get()</code>
    </td>
    <td>
      Retrieves a static instance of a controller or provider (including guards, filters, etc.) available in the application context. Inherited from the <a href="/fundamentals/module-ref">module reference</a> class.
    </td>
  </tr>
  <tr>
     <td>
      <code>resolve()</code>
    </td>
    <td>
      Retrieves a dynamically created scoped instance (request or transient) of a controller or provider (including guards, filters, etc.) available in the application context. Inherited from the <a href="/fundamentals/module-ref">module reference</a> class.
    </td>
  </tr>
  <tr>
    <td>
      <code>select()</code>
    </td>
    <td>
      Navigates through the module's dependency graph; can be used to retrieve a specific instance from the selected module (used along with strict mode (<code>strict: true</code>) in <code>get()</code> method).
    </td>
  </tr>
</table>

> info **Hint** Keep your e2e test files inside the `test` directory. The testing files should have a `.e2e-spec` suffix.

#### Overriding globally registered enhancers

If you have a globally registered guard (or pipe, interceptor, or filter), you need to take a few more steps to override that enhancer. To recap the original registration looks like this:

```typescript
providers: [
  {
    provide: APP_GUARD,
    useClass: JwtAuthGuard,
  },
],
```

This is registering the guard as a "multi"-provider through the `APP_*` token. To be able to replace the `JwtAuthGuard` here, the registration needs to use an existing provider in this slot:

```typescript
providers: [
  {
    provide: APP_GUARD,
    useExisting: JwtAuthGuard,
    // ^^^^^^^^ notice the use of 'useExisting' instead of 'useClass'
  },
  JwtAuthGuard,
],
```

> info **Hint** Change the `useClass` to `useExisting` to reference a registered provider instead of having Nest instantiate it behind the token.

Now the `JwtAuthGuard` is visible to Nest as a regular provider that can be overridden when creating the `TestingModule`:

```typescript
const moduleRef = await Test.createTestingModule({
  imports: [AppModule],
})
  .overrideProvider(JwtAuthGuard)
  .useClass(MockAuthGuard)
  .compile();
```

Now all your tests will use the `MockAuthGuard` on every request.

#### Testing request-scoped instances

[Request-scoped](/fundamentals/injection-scopes) providers are created uniquely for each incoming **request**. The instance is garbage-collected after the request has completed processing. This poses a problem, because we can't access a dependency injection sub-tree generated specifically for a tested request.

We know (based on the sections above) that the `resolve()` method can be used to retrieve a dynamically instantiated class. Also, as described <a href="https://docs.nestjs.com/fundamentals/module-ref#resolving-scoped-providers">here</a>, we know we can pass a unique context identifier to control the lifecycle of a DI container sub-tree. How do we leverage this in a testing context?

The strategy is to generate a context identifier beforehand and force Nest to use this particular ID to create a sub-tree for all incoming requests. In this way we'll be able to retrieve instances created for a tested request.

To accomplish this, use `jest.spyOn()` on the `ContextIdFactory`:

```typescript
const contextId = ContextIdFactory.create();
jest
  .spyOn(ContextIdFactory, 'getByRequest')
  .mockImplementation(() => contextId);
```

Now we can use the `contextId` to access a single generated DI container sub-tree for any subsequent request.

```typescript
catsService = await moduleRef.resolve(CatsService, contextId);
```
