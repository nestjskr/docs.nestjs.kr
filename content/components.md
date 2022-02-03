### Providers

프로바이더는 Nest의 기본 개념입니다. 기본 Nest 클래스의 대부분은 서비스, 리포지토리, 팩토리, 도우미 등 프로바이더로 취급될 수 있습니다. 프로바이더의 주요 아이디어는 종속성으로 **주입**할 수 있다는 것입니다. 즉, 객체는 서로 다양한 관계를 생성할 수 있으며 객체의 인스턴스를 "연결"하는 기능은 대부분 Nest 런타임 시스템에 위임될 수 있습니다.

<figure><img src="/assets/Components_1.png" /></figure>

이전 장에서 우리는 간단한 `CatsController`를 만들었습니다. 컨트롤러는 HTTP 요청을 처리하고 더 복잡한 작업을 **프로바이더**에 위임해야 합니다. 공급자는 [모듈](/modules)에서 `프로바이더'로 선언된 일반 자바스크립트 클래스입니다.

> 정보 **힌트** Nest를 사용하면 종속성을 보다 OO-방식으로 설계하고 구성할 수 있으므로 [SOLID](https://en.wikipedia.org/wiki/SOLID) 원칙을 따르는 것이 좋습니다.

#### Services

간단한 'CatsService'를 만드는 것으로 시작하겠습니다. 이 서비스는 데이터 저장 및 검색을 담당하며 `CatsController`에서 사용하도록 설계되었으므로 프로바이더로 정의하기에 좋은 후보입니다.

```typescript
@@filename(cats.service)
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
@@switch
import { Injectable } from '@nestjs/common';

@Injectable()
export class CatsService {
  constructor() {
    this.cats = [];
  }

  create(cat) {
    this.cats.push(cat);
  }

  findAll() {
    return this.cats;
  }
}
```

> 정보 **힌트** CLI를 사용하여 서비스를 생성하려면 `$ nest g service cats` 명령을 실행하면 됩니다.

우리의 `CatsService`는 하나의 속성과 두 개의 메소드가 있는 기본 클래스입니다. 유일한 새로운 기능은 `@Injectable()` 데코레이터를 사용한다는 것입니다. `@Injectable()` 데코레이터는 `CatsService`가 Nest IoC 컨테이너에서 관리할 수 있는 클래스임을 선언하는 메타데이터를 첨부합니다. 그건 그렇고, 이 예제는 'Cat' 인터페이스도 사용하는데, 아마도 다음과 같을 것입니다:

```typescript
@@filename(interfaces/cat.interface)
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```

이제 고양이를 검색하는 서비스 클래스가 있으므로 `CatsController` 내부에서 사용하겠습니다.

```typescript
@@filename(cats.controller)
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
@@switch
import { Controller, Get, Post, Body, Bind, Dependencies } from '@nestjs/common';
import { CatsService } from './cats.service';

@Controller('cats')
@Dependencies(CatsService)
export class CatsController {
  constructor(catsService) {
    this.catsService = catsService;
  }

  @Post()
  @Bind(Body())
  async create(createCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll() {
    return this.catsService.findAll();
  }
}
```

`CatsService`는 클래스 생성자를 통해 **주입**됩니다. `private` 구문의 사용에 주목하십시오. 이 약식을 사용하면 같은 위치에서 즉시 `catsService` 멤버를 선언하고 초기화할 수 있습니다.

#### Dependency injection

Nest is built around the strong design pattern commonly known as **Dependency injection**. We recommend reading a great article about this concept in the official [Angular](https://angular.io/guide/dependency-injection) documentation.

In Nest, thanks to TypeScript capabilities, it's extremely easy to manage dependencies because they are resolved just by type. In the example below, Nest will resolve the `catsService` by creating and returning an instance of `CatsService` (or, in the normal case of a singleton, returning the existing instance if it has already been requested elsewhere). This dependency is resolved and passed to your controller's constructor (or assigned to the indicated property):

```typescript
constructor(private catsService: CatsService) {}
```

#### Scopes

Providers normally have a lifetime ("scope") synchronized with the application lifecycle. When the application is bootstrapped, every dependency must be resolved, and therefore every provider has to be instantiated. Similarly, when the application shuts down, each provider will be destroyed. However, there are ways to make your provider lifetime **request-scoped** as well. You can read more about these techniques [here](/fundamentals/injection-scopes).

<app-banner-courses></app-banner-courses>

#### Custom providers

Nest has a built-in inversion of control ("IoC") container that resolves relationships between providers. This feature underlies the dependency injection feature described above, but is in fact far more powerful than what we've described so far.  There are several ways to define a provider: you can use plain values, classes, and either asynchronous or synchronous factories. More examples are provided [here](/fundamentals/dependency-injection).

#### Optional providers

Occasionally, you might have dependencies which do not necessarily have to be resolved. For instance, your class may depend on a **configuration object**, but if none is passed, the default values should be used. In such a case, the dependency becomes optional, because lack of the configuration provider wouldn't lead to errors.

To indicate a provider is optional, use the `@Optional()` decorator in the constructor's signature.

```typescript
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(@Optional() @Inject('HTTP_OPTIONS') private httpClient: T) {}
}
```

Note that in the example above we are using a custom provider, which is the reason we include the `HTTP_OPTIONS` custom **token**. Previous examples showed constructor-based injection indicating a dependency through a class in the constructor. Read more about custom providers and their associated tokens [here](/fundamentals/custom-providers).

#### Property-based injection

The technique we've used so far is called constructor-based injection, as providers are injected via the constructor method. In some very specific cases, **property-based injection** might be useful. For instance, if your top-level class depends on either one or multiple providers, passing them all the way up by calling `super()` in sub-classes from the constructor can be very tedious. In order to avoid this, you can use the `@Inject()` decorator at the property level.

```typescript
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  @Inject('HTTP_OPTIONS')
  private readonly httpClient: T;
}
```

> warning **Warning** If your class doesn't extend another provider, you should always prefer using **constructor-based** injection.

#### Provider registration

Now that we have defined a provider (`CatsService`), and we have a consumer of that service (`CatsController`), we need to register the service with Nest so that it can perform the injection. We do this by editing our module file (`app.module.ts`) and adding the service to the `providers` array of the `@Module()` decorator.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```

Nest will now be able to resolve the dependencies of the `CatsController` class.

This is how our directory structure should look now:

<div class="file-tree">
<div class="item">src</div>
<div class="children">
<div class="item">cats</div>
<div class="children">
<div class="item">dto</div>
<div class="children">
<div class="item">create-cat.dto.ts</div>
</div>
<div class="item">interfaces</div>
<div class="children">
<div class="item">cat.interface.ts</div>
</div>
<div class="item">cats.controller.ts</div>
<div class="item">cats.service.ts</div>
</div>
<div class="item">app.module.ts</div>
<div class="item">main.ts</div>
</div>
</div>

#### Manual instantiation

Thus far, we've discussed how Nest automatically handles most of the details of resolving dependencies.  In certain circumstances, you may need to step outside of the built-in Dependency Injection system and manually retrieve or instantiate providers. We briefly discuss two such topics below.

 To get existing instances, or instantiate providers dynamically, you can use [Module reference](https://docs.nestjs.com/fundamentals/module-ref).
 
 To get providers within the `bootstrap()` function (for example for standalone applications without controllers, or to utilize a configuration service during bootstrapping) see [Standalone applications](https://docs.nestjs.com/standalone-applications).
