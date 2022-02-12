### 동적 모듈

[모듈 챕터](/modules)에서 모듈에 대한 기본적인 내용을 다루면서 [동적 모듈](https://docs.nestjs.com/modules#dynamic-modules)에 대해 간단히 소개하기도 했습니다. 이번 챕터에서는 동적 모듈에 대해 더 자세히 다룹니다. 이번 챕터를 마치고 나면 그것들이 무엇이며 언제 어떻게 사용하는지 제대로 파악할 수 있을 것입니다.

#### 소개

본 문서의 **개요** 섹션에 있는 대부분의 예제 코드에서는 일반 모듈이나 정적 모듈을 사용합니다. 모듈은 [프로바이더](/providers)와 [컨트롤러](/controllers)같이 전체 어플리케이션의 모듈적인 부분이 되어 함께 어울리는 컴포넌트들의 그룹을 나타냅니다. 모듈은 이러한 컴포넌트들에게 실행 컨텍스트나 스코프 등을 주입해 줍니다. 예를 들어, 어떤 모듈 안에 정의된 프로바이더를 따로 내보내지 않아도 해당 모듈의 다른 다른 멤버들이 그 프로바이더를 찾을 수 있습니다. 프로바이더가 모듈 바깥에 노출될 필요가 있다면, 우선 해당 프로바이더의 호스트 모듈로부터 내보낸 다음, 사용하고자 하는 모듈에 import 되어야 합니다.

익숙한 예시를 통해 다시 들여다 봅시다.

우선, `UsersModule`에서 `UsersService`를 공급받고 내보냅니다. `UsersModule`은 `UsersService`의 **호스트** 모듈입니다.

```typescript
import { Module } from "@nestjs/common";
import { UsersService } from "./users.service";

@Module({
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

그 다음, `UsersModule`을 import하는 `AuthModule`을 정의하고, `UsersModule`에서 내보낸 프로바이더를 `AuthModule` 내부에서 사용할 수 있게 만듭니다:

```typescript
import { Module } from "@nestjs/common";
import { AuthService } from "./auth.service";
import { UsersModule } from "../users/users.module";

@Module({
  imports: [UsersModule],
  providers: [AuthService],
  exports: [AuthService],
})
export class AuthModule {}
```

이렇게 작성하면 `AuthModule`을 호스트로 두는 가령 `AuthService`같은 곳에 `UsersService`를 주입할 수 있게 됩니다:

```typescript
import { Injectable } from "@nestjs/common";
import { UsersService } from "../users/users.service";

@Injectable()
export class AuthService {
  constructor(private usersService: UsersService) {}
  /*
    Implementation that makes use of this.usersService
  */
}
```

이와 같은 일을 **정적** 모듈 바인딩이라고 합니다. Nest가 엮어야 할 모듈에 대한 정보는 호스트 모듈과 그 모듈을 사용하는 모듈에 이미 정의되어 있습니다. 이러한 작업이 내부적으로 어떻게 이루어지는지 살펴봅시다. Nest는 아래의 과정들을 통해 `AuthModule` 내부에서 `UsersService`를 사용할 수 있게 만듭니다:

1. `UsersModule`을 인스턴스화 하는데, 이 때 이 모듈이 사용하는 다른 모듈들을 가져오면서 의존성을 해소합니다. ([사용자 정의 프로바이더](/fundamentals/custom-providers)를 참조해 주세요)
2. `AuthModule`을 인스턴스화 하면서, `UsersModule`에서 내보낸 프로바이더들을 `AuthModule` 속 컴포넌트들이 사용할 수 있게 만듭니다. (마치 그 프로바이더들이 `AuthModule` 안에 선언되듯이)
3. `AuthService`에 `UsersService` 인스턴스를 주입합니다.

#### 동적 모듈 사용 사례

정적 모듈 바인딩을 할 때, 사용하는 모듈 쪽에서는 호스트 모듈에서 제공하는 프로바이더를 설정할 수 있도록 영향을 미칠 기회가 없습니다. 그게 왜 문제가 될까요? 다양한 유스케이스에서 다르게 동작할 필요가 있도록 범용적으로 사용하는 모듈이 있다고 가정해 봅시다. 이것은 많은 시스템에서 "플러그인" 이라 칭하는 컨셉과 유사할 테고, 사용자가 이를 사용하기 전에 몇가지 설정을 해야 합니다.

Nest에서 이와 관련한 좋은 예시가 바로 **환경설정 모듈**입니다. 많은 애플리케이션들이 세부설정을 외부로 빼기 위해 환경설정 모듈을 유용하게 사용합니다. 이것은 다양한 배포환경에서 동적으로 애플리케이션 설정을 쉽게 바꿀 수 있게 해줍니다: 예를 들면, 개발자가 사용하는 개발용 데이터베이스, 스테이징과 테스팅 환경에서 쓰이는 스테이징 데이터베이스 등이 있습니다. 설정값들의 관리를 환경설정 모듈에게 위임함으로써, 애플리케이션의 소스코드를 설정값으로부터 독립적으로 유지할 수 있습니다.

해야 할 일은, 환경설정 모듈 자체는 "플러그인과"처럼 범용적이기 때문에 이를 사용하는 모듈에서 커스터마이징 하는 것입니다. 이곳이 바로 *동적 모듈*이 동작하는 곳입니다. 동적 모듈의 기능으로 환경설정 모듈을 **동적**으로 만들었기 때문에 사용하는 모듈에서는 환경설정 모듈을 import할 때 API를 호출할 수 있게 되며, 해당 API를 통해 환경설정 모듈이 어떻게 커스터마이징될지를 제어합니다.

달리 말하면, 동적 모듈은 어떤 모듈을 다른 여러 모듈에서 import한다거나 해당 모듈을 import할 때 그 모듈의 프로퍼티와 동작을 커스터마이징할 수 있도록 API를 제공합니다. 이는 이제껏 보았던 정적 바인딩과 아주 대조적입니다.

<app-banner-shop></app-banner-shop>

#### 설정 모듈 예제

이번 섹션에서는 [환경설정 챕터](https://docs.nestjs.com/techniques/configuration#service)의 예제 코드를 기반으로 설명하겠습니다. 이번 챕터를 통해 완성된 코드는 [이 예제](https://github.com/nestjs/nest/tree/master/sample/25-dynamic-modules)에서 동작을 확인할 수 있습니다.

우리의 요구사항은 `ConfigModule`을 만들고 이를 커스터마이징 하기 위해 `options` 객체를 사용하는 것입니다. 제공하려는 기능에 대해 말씀 드리겠습니다. 기본 예제에서는 `.env`파일의 위치가 프로젝트 폴더 최상단으로 하드코딩 되어 있습니다. 이것을 `.env` 파일들이 어느 폴더에 있든 관리할 수 있게끔 설정 가능하게 만든다고 가정해 봅시다. 예를 들어, 프로젝트 최상단의 `config`폴더(`src`폴더의 형제 위치) 안에 다양한 `.env` 파일들을 저장해 두었다고 한다면, 다른 프로젝트들에서는 `ConfigModule`을 사용할 때 다른 폴더를 지정하고 싶어집니다.

동적 모듈은 해당 모듈을 import할 때 인자를 전달할 수 있게 해주어 우리가 그 동작을 변경할 수 있게 됩니다. 이것이 어떻게 동작하는지 살펴봅시다. 사용하는 모듈의 관점에서 어떻게 보이는지부터 시작해서 거꾸로 파악해 나가면 더 이해하기 쉽습니다. 우선, `ConfigModule`을 _정적으로_ import하던 예제(모듈을 import할 때 그 모듈의 동작에 영향을 줄 수 없던 접근방식이었습니다)를 다시금 빠르게 살펴보고 지나갑시다. `@Module()` 데코레이터의 `imports` 배열을 집중적으로 봐주세요:

```typescript
import { Module } from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { ConfigModule } from "./config/config.module";

@Module({
  imports: [ConfigModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

우리가 설정 객체를 전달할 *동적 모듈*을 import하는 모습은 어떨지 생각해 봅시다. 위와 아래 두 예제의 `import` 배열에 어떤 차이가 있는지 비교해 보세요:

```typescript
import { Module } from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { ConfigModule } from "./config/config.module";

@Module({
  imports: [ConfigModule.register({ folder: "./config" })],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

위의 동적 모듈 예제에 무슨 일이 일어나는지 봅시다. 어떤 부분이 달라졌나요?

1. `ConfigModule`은 평범한 클래스이므로, `register()`이라는 **정적 메서드**를 가진다고 짐작할 수 있습니다. 이 메서드는 클래스의 **인스턴스**가 아니라 `ConfigModule` 클래스 자체에서 불러오기 때문에 정적입니다. 곧 구현할 이 메서드는 임의의 이름을 가질 수 있지만, 관례적으로 `forRoot()`나 `register()`라고 부릅니다.
2. `register()` 메서드는 우리가 만들 것이기 때문에 원하는 인자를 넣을 수 있습니다. 이번에는, 일반적인 경우에 적합한 프로퍼티들을 가지는 `options` 객체를 받도록 하겠습니다.
3. `register()` 메서드는 `module`같은 무언가를 반환해야 한다고 짐작할 수 있습니다. 왜냐면 그 반환 값이 `imports` 리스트에 들어가고 있고, 지금까지 이 리스트에 모듈들이 들어가는 것을 보았기 때문입니다.

사실, `register()`메서드는 `DynamicModule`을 반환합니다. 동적 모듈은 런타임에서 만들어진다는 것 말고는 아무것도 없습니다. 정적 모듈이 가지는 모든 프로퍼티를 동일하게 가지고, 거기에 `module`이라는 프로퍼티만 추가로 가집니다. 정적 모듈을 선언할 때 데코레이터에 전달되는 모듈 옵션을 다시금 빠르게 살펴봅시다:

```typescript
@Module({
  imports: [DogsModule],
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
```

동적 모듈은 인터페이스와 동일한 객체에 `module`이라는 프로퍼티를 추가하여 반환해야 합니다. `module` 프로퍼티는 모듈의 이름을 제공하며 아래 예시에서 볼 수 있듯 모듈의 클래스 이름과 같아야 합니다.

> info **힌트** 동적 모듈에서는 `module`프로퍼티를 **제외**하고 모듈 옵션 객체의 모든 프로퍼티를 선택적으로 가집니다.

`register()`라는 정적 메서드는 무엇일까요? 우리는 이제 이 메서드가 하는 일이 `DynamicModule` 인터페이스를 가지는 객체를 반환하는 것이라는 걸 압니다. 이 메서드를 호출함으로써, `imports` 리스트에 정적으로 모듈 클래스 이름을 적어주었던 것과 유사한 방법으로 모듈을 효과적으로 제공합니다. 달리 말하면, 동적 모듈의 API는 단순하게 어떤 모듈을 반환하되, `@Module` 데코레이터의 프로퍼티들을 고정시키지 않고 프로그래밍적으로 명시합니다.

그림을 완성하려면 몇 가지 세부사항들이 더 필요합니다:

1. 이제는 `@Module()` 데코레이터의 `imports` 프로퍼티에는 모듈 클래스 이름(예: `imports: [UsersModule]`)뿐만 아니라 동적 모듈을 반환하는 함수(예: `imports: [ConfigModule.register(...)]`)도 사용할 수 있다고 말할 수 있습니다.
2. 동적 모듈은 스스로 다른 모듈들을 import할 수 있습니다. 이번 예제에서는 그러지 않았지만 만약 동적 모듈이 다른 모듈의 프로바이더에 의존한다면, `imports` 프로퍼티를 사용하여 그 모듈들을 import하면 됩니다. 다시 말씀 드리지만, 정적 모듈에서 `@Module()` 데코레이터를 사용하여 메타데이터를 선언하던 방식과 완전히 동일합니다.

이러한 이해를 바탕으로, 이제 우리의 동적 모듈인 `ConfigModule`의 선언이 어떻게 생겼는지 확인할 수 있습니다. 다음과 같이 시도해 봅시다.

```typescript
import { DynamicModule, Module } from "@nestjs/common";
import { ConfigService } from "./config.service";

@Module({})
export class ConfigModule {
  static register(): DynamicModule {
    return {
      module: ConfigModule,
      providers: [ConfigService],
      exports: [ConfigService],
    };
  }
}
```

이제는 지금까지 다룬 조각들이 서로 어떻게 연결되는지 명확히 이해할 차례입니다. `ConfigModule.register(...)`을 호출하면, 지금까지 `@Module()` 데코레이터를 통해 메타데이터를 제공했던 것과 본질적으로 동일한 프로퍼티를 가지는 `DynamicModule` 객체를 반환합니다.

> info **힌트** `DynamicModule`는 `@nestjs/common`에서 import합니다.

우리의 동적 모듈은 아직까지는 그다지 흥미롭지 않은데, 이는 우리가 하고 싶다고 말했던 그 **설정** 기능을 아직 다루지 않았기 때문입니다. 이는 다음 섹션에서 다루도록 하겠습니다.

#### 모듈 설정

위에서 추측한 대로, `ConfigModule`의 동작을 커스터마이징하는 분명한 방법은 정적 메서드인 `register()`에 `options` 객체를 넘기는 것입니다. 사용하는 모듈의 `imports` 프로퍼티를 다시 한 번 봅시다:

```typescript
import { Module } from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { ConfigModule } from "./config/config.module";

@Module({
  imports: [ConfigModule.register({ folder: "./config" })],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

동적 모듈에 `options` 객체를 넘기는 것이 좋은 조작입니다. 이후에 그 `options` 객체를 `ConfigModule`에서 어떻게 사용할 수 있을까요? 잠시 스스로 생각해 봅시다. `ConfigModule`은 기본적으로 다른 프로바이더에 주입할 수 있는 서비스(`ConfigService`)를 포함하고 내보내는 호스트 모듈입니다. 실제로 `options` 객체를 읽어서 동작을 커스터마이징 해야 하는 건 `ConfigService`입니다. 이러한 가정을 바탕으로, `options` 객체의 프로퍼티들을 기반으로 서비스의 동작을 커스터마이징 하기 위해 서비스를 약간 수정할 수 있습니다. (**참고**: 실제로 어떻게 전달하는지는 아직 다루지 않았기 때문에, 당분간은 `options`를 하드코딩하겠습니다. 이는 잠시 후에 해결하겠습니다.)

```typescript
import { Injectable } from "@nestjs/common";
import * as dotenv from "dotenv";
import * as fs from "fs";
import { EnvConfig } from "./interfaces";

@Injectable()
export class ConfigService {
  private readonly envConfig: EnvConfig;

  constructor() {
    const options = { folder: "./config" };

    const filePath = `${process.env.NODE_ENV || "development"}.env`;
    const envFile = path.resolve(__dirname, "../../", options.folder, filePath);
    this.envConfig = dotenv.parse(fs.readFileSync(envFile));
  }

  get(key: string): string {
    return this.envConfig[key];
  }
}
```

이제 `ConfigService`는 `options`에 명시한 폴더에서 `.env`파일을 찾습니다.

이제 남은 작업은 어떻게든 `options` 객체를 `register()`를 통해 `ConfigService`에 주입하는 것입니다. 이번에도 당연히 *의존성 주입*을 사용합니다. 이것이 중요한 포인트이니 꼭 숙지하시길 바랍니다. `ConfigModule`은 `ConfigService`를 제공합니다. `ConfigService`는 `options` 객체에 의존하며 이는 오직 런타임 환경에서만 제공됩니다. 따라서 런타임에서는, 우선 `options` 객체를 Nest의 IoC 컨테이너에 바인딩하고, Nest가 이를 `ConfigService`에 주입해야 합니다. **사용자 정의 프로바이더** 챕터에서 말씀 드렸듯 프로바이더는 [어떤 값이든 포함할 수 있다](https://docs.nestjs.com/fundamentals/custom-providers#non-service-based-providers)는 점을 기억하시길 바라며, 이는 단순히 서비스만 해당되는 내용이 아닙니다. 따라서 우리는 `options` 객체를 쉽게 다루기 위해 의존성 주입을 사용하기만 하면 됩니다.

options 객체를 IoC 컨테이너에 바인딩하는 문제부터 해결해 봅시다. 이 작업은 우리의 정적 메서드인 `register()`에서 이루어집니다. 우리는 지금 모듈을 동적으로 구성하고 있고, 모듈의 프로퍼티 중 하나는 프로바이더 리스트라는 점을 기억해 보세요. 그렇다면 우리가 해야 할 일은 우리의 options 객체를 프로바이더라고 정의하는 것입니다. 이렇게 하면 다음 단계에서 활용할 `ConfigModule`에 options 객체를 주입할 수 있게 됩니다. 아래 코드에서 `providers` 배열을 집중적으로 살펴보세요:

```typescript
import { DynamicModule, Module } from "@nestjs/common";
import { ConfigService } from "./config.service";

@Module({})
export class ConfigModule {
  static register(options): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: "CONFIG_OPTIONS",
          useValue: options,
        },
        ConfigService,
      ],
      exports: [ConfigService],
    };
  }
}
```

이제 `ConfigService`에 `'CONFIG_OPIONS'` 프로바이더를 주입하면 완성입니다. 클래스가 아닌 토큰을 사용하는 프로바이더를 정의하려면 `@Inject()` 데코레이터를 사용해야 하며, 이에 대한 내용은 [여기에서 설명하고 있습니다](https://docs.nestjs.com/fundamentals/custom-providers#non-class-based-provider-tokens).

```typescript
import * as dotenv from "dotenv";
import * as fs from "fs";
import { Injectable, Inject } from "@nestjs/common";
import { EnvConfig } from "./interfaces";

@Injectable()
export class ConfigService {
  private readonly envConfig: EnvConfig;

  constructor(@Inject("CONFIG_OPTIONS") private options) {
    const filePath = `${process.env.NODE_ENV || "development"}.env`;
    const envFile = path.resolve(__dirname, "../../", options.folder, filePath);
    this.envConfig = dotenv.parse(fs.readFileSync(envFile));
  }

  get(key: string): string {
    return this.envConfig[key];
  }
}
```

마지막으로 기억해야 할 것이 있습니다. 위에서는 단순하게 문자열 기반의 주입 토큰(`'CONFIG_OPTIONS'`)을 사용했지만, 이러한 토큰을 정의하는 가장 좋은 방법은 별도의 파일에 상수(또는 `Symbol`)로 정의해 두고 다른 파일에서 가져다 사용하는 것입니다. 아래는 그 예시입니다:

```typescript
export const CONFIG_OPTIONS = "CONFIG_OPTIONS";
```

### 예제

이번 챕터의 예제 코드 최종본은 [여기](https://github.com/nestjs/nest/tree/master/sample/25-dynamic-modules)에 있습니다.
