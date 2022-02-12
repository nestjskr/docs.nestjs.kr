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

#### Config module example

We'll be using the basic version of the example code from the [configuration chapter](https://docs.nestjs.com/techniques/configuration#service) for this section. The completed version as of the end of this chapter is available as a working [example here](https://github.com/nestjs/nest/tree/master/sample/25-dynamic-modules).

Our requirement is to make `ConfigModule` accept an `options` object to customize it. Here's the feature we want to support. The basic sample hard-codes the location of the `.env` file to be in the project root folder. Let's suppose we want to make that configurable, such that you can manage your `.env` files in any folder of your choosing. For example, imagine you want to store your various `.env` files in a folder under the project root called `config` (i.e., a sibling folder to `src`). You'd like to be able to choose different folders when using the `ConfigModule` in different projects.

Dynamic modules give us the ability to pass parameters into the module being imported so we can change its behavior. Let's see how this works. It's helpful if we start from the end-goal of how this might look from the consuming module's perspective, and then work backwards. First, let's quickly review the example of _statically_ importing the `ConfigModule` (i.e., an approach which has no ability to influence the behavior of the imported module). Pay close attention to the `imports` array in the `@Module()` decorator:

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

Let's consider what a _dynamic module_ import, where we're passing in a configuration object, might look like. Compare the difference in the `imports` array between these two examples:

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

Let's see what's happening in the dynamic example above. What are the moving parts?

1. `ConfigModule` is a normal class, so we can infer that it must have a **static method** called `register()`. We know it's static because we're calling it on the `ConfigModule` class, not on an **instance** of the class. Note: this method, which we will create soon, can have any arbitrary name, but by convention we should call it either `forRoot()` or `register()`.
2. The `register()` method is defined by us, so we can accept any input arguments we like. In this case, we're going to accept a simple `options` object with suitable properties, which is the typical case.
3. We can infer that the `register()` method must return something like a `module` since its return value appears in the familiar `imports` list, which we've seen so far includes a list of modules.

In fact, what our `register()` method will return is a `DynamicModule`. A dynamic module is nothing more than a module created at run-time, with the same exact properties as a static module, plus one additional property called `module`. Let's quickly review a sample static module declaration, paying close attention to the module options passed in to the decorator:

```typescript
@Module({
  imports: [DogsModule],
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
```

Dynamic modules must return an object with the exact same interface, plus one additional property called `module`. The `module` property serves as the name of the module, and should be the same as the class name of the module, as shown in the example below.

> info **Hint** For a dynamic module, all properties of the module options object are optional **except** `module`.

What about the static `register()` method? We can now see that its job is to return an object that has the `DynamicModule` interface. When we call it, we are effectively providing a module to the `imports` list, similar to the way we would do so in the static case by listing a module class name. In other words, the dynamic module API simply returns a module, but rather than fix the properties in the `@Module` decorator, we specify them programmatically.

There are still a couple of details to cover to help make the picture complete:

1. We can now state that the `@Module()` decorator's `imports` property can take not only a module class name (e.g., `imports: [UsersModule]`), but also a function **returning** a dynamic module (e.g., `imports: [ConfigModule.register(...)]`).
2. A dynamic module can itself import other modules. We won't do so in this example, but if the dynamic module depends on providers from other modules, you would import them using the optional `imports` property. Again, this is exactly analogous to the way you'd declare metadata for a static module using the `@Module()` decorator.

Armed with this understanding, we can now look at what our dynamic `ConfigModule` declaration must look like. Let's take a crack at it.

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

It should now be clear how the pieces tie together. Calling `ConfigModule.register(...)` returns a `DynamicModule` object with properties which are essentially the same as those that, until now, we've provided as metadata via the `@Module()` decorator.

> info **Hint** Import `DynamicModule` from `@nestjs/common`.

Our dynamic module isn't very interesting yet, however, as we haven't introduced any capability to **configure** it as we said we would like to do. Let's address that next.

#### Module configuration

The obvious solution for customizing the behavior of the `ConfigModule` is to pass it an `options` object in the static `register()` method, as we guessed above. Let's look once again at our consuming module's `imports` property:

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

That nicely handles passing an `options` object to our dynamic module. How do we then use that `options` object in the `ConfigModule`? Let's consider that for a minute. We know that our `ConfigModule` is basically a host for providing and exporting an injectable service - the `ConfigService` - for use by other providers. It's actually our `ConfigService` that needs to read the `options` object to customize its behavior. Let's assume for the moment that we know how to somehow get the `options` from the `register()` method into the `ConfigService`. With that assumption, we can make a few changes to the service to customize its behavior based on the properties from the `options` object. (**Note**: for the time being, since we _haven't_ actually determined how to pass it in, we'll just hard-code `options`. We'll fix this in a minute).

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

Now our `ConfigService` knows how to find the `.env` file in the folder we've specified in `options`.

Our remaining task is to somehow inject the `options` object from the `register()` step into our `ConfigService`. And of course, we'll use _dependency injection_ to do it. This is a key point, so make sure you understand it. Our `ConfigModule` is providing `ConfigService`. `ConfigService` in turn depends on the `options` object that is only supplied at run-time. So, at run-time, we'll need to first bind the `options` object to the Nest IoC container, and then have Nest inject it into our `ConfigService`. Remember from the **Custom providers** chapter that providers can [include any value](https://docs.nestjs.com/fundamentals/custom-providers#non-service-based-providers) not just services, so we're fine using dependency injection to handle a simple `options` object.

Let's tackle binding the options object to the IoC container first. We do this in our static `register()` method. Remember that we are dynamically constructing a module, and one of the properties of a module is its list of providers. So what we need to do is define our options object as a provider. This will make it injectable into the `ConfigService`, which we'll take advantage of in the next step. In the code below, pay attention to the `providers` array:

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

Now we can complete the process by injecting the `'CONFIG_OPTIONS'` provider into the `ConfigService`. Recall that when we define a provider using a non-class token we need to use the `@Inject()` decorator [as described here](https://docs.nestjs.com/fundamentals/custom-providers#non-class-based-provider-tokens).

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

One final note: for simplicity we used a string-based injection token (`'CONFIG_OPTIONS'`) above, but best practice is to define it as a constant (or `Symbol`) in a separate file, and import that file. For example:

```typescript
export const CONFIG_OPTIONS = "CONFIG_OPTIONS";
```

### Example

A full example of the code in this chapter can be found [here](https://github.com/nestjs/nest/tree/master/sample/25-dynamic-modules).
