### First steps

이 문서에서는 Nest의 **핵심 원리** 에 대해 알아봅니다. Nest 애플리케이션의 필수 구성 요소를 숙지하려면,
기초적인 수준에서 많은 분야를 다루며 기본적인 CRUD 애플리케이션을 구축해 봐야 합니다.

#### 언어

우리는 [TypeScript](https://www.typescriptlang.org/)와 사랑에 빠졌지만, 무엇보다도 [Node.js](https://nodejs.org/en/)를 사랑합니다. 이것이 Nest가 TypeScript, **pure JavaScript** 모두와 호환되는 이유입니다. Nest는 최신 언어 기능을 활용하기 때문에 바닐라 자바스크립트와 함께 사용하려면 [Babel](https://babeljs.io/) 컴파일러가 필요합니다.

우리가 제공하는 예제 에서는 대부분 타입스크립트를 사용하겠지만, 언제든 바닐라 자바스크립트로 **코드 스니펫을 전환** 할 수 있습니다 (각 스니펫의 오른쪽 상단 모서리에 있는 언어 버튼을 클릭하여 전환).

#### 전제조건

운영 체제에 [Node.js](https://nodejs.org/)(>= 10.13.0, v13 제외)가 설치되어 있는지 확인하세요.

#### 설정하기

[Nest CLI](/cli/overview) 로 새 프로젝트를 설정하는 것은 매우 간단합니다. [npm](https://www.npmjs.com/)이 설치되어 있다면, OS 터미널에서 다음 명령을 사용해 새 Nest 프로젝트를 생성할 수 있습니다.

```bash
$ npm i -g @nestjs/cli
$ nest new project-name
```

`project-name` 디렉토리가 생성되고, node modules 와 몇가지 boilerplate 파일들이 생성되며, `src/` 디렉토리와 함께 여러개의 코어 파일들로 채워집니다.

<div class="file-tree">
  <div class="item">src</div>
  <div class="children">
    <div class="item">app.controller.spec.ts</div>
    <div class="item">app.controller.ts</div>
    <div class="item">app.module.ts</div>
    <div class="item">app.service.ts</div>
    <div class="item">main.ts</div>
  </div>
</div>

다음은 코어 파일들의 간략한 설명 입니다.

|                          |                                                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| `app.controller.ts`      | A basic controller with a single route.                                                                             |
| `app.controller.spec.ts` | The unit tests for the controller.                                                                                  |
| `app.module.ts`          | The root module of the application.                                                                                 |
| `app.service.ts`         | A basic service with a single method.                                                                               |
| `main.ts`                | The entry file of the application which uses the core function `NestFactory` to create a Nest application instance. |

`main.ts`는 우리의 어플리케이션의 **시작점**이 되는 비동기 함수를 포함하고 있습니다.

```typescript
@@filename(main)

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
@@switch
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

Nest 애플리케이션 인스턴스를 생성하기 위해 우리는 `NestFactory` 클래스를 사용합니다. `NestFactory` 는 애플리케이션 인스턴스를 만들 수 있는 몇가지 정적 메소드를 제공합니다. `create()` 메소드는 `INestApplication` 인터페이스를 따르는 애플리케이션 객체를 반환합니다. 이 객체는 다음 장에서 설명하는 메소드 집합을 제공합니다. 우리는 위의 `main.ts` 예제에서 HTTP 요청을 받을 수 있도록 HTTP 리스너를 실행 했습니다.

Nest CLI로 생성된 프로젝트는 개발자가 각 모듈을 독립된 디렉토리에 작성하는 관례를 따르도록 권장하기 위해 초기 프로젝트 구조를 생성합니다.

<app-banner-courses></app-banner-courses>

#### 플랫폼

Nest는 플랫폼에 구애받지 않는 프레임워크가 되는 것을 목표로 합니다. 플랫폼 독립성을 통해 개발자가 여러 유형의 애플리케이션에 걸쳐 활용할 수 있는 재사용 가능한 논리적 모듈을 만들 수 있습니다. 기술적으로 Nest는 어댑터가 만들어지면 어떤 Node HTTP 프레임워크에서도 동작 합니다. 기본적으로 지원되는 HTTP 플랫폼은 [express](https://expressjs.com/)와 [fastify](https://www.fastify.io) 두 가지가 있습니다. 요구사항에 맞는 적합한 프레임워크를 선택하여 사용하면 됩니다.

|                    |                                                                                                                                                                                                                                                                                                                                    |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `platform-express` | [Express](https://expressjs.com/) is a well-known minimalist web framework for node. It's a battle tested, production-ready library with lots of resources implemented by the community. The `@nestjs/platform-express` package is used by default. Many users are well served with Express, and need take no action to enable it. |
| `platform-fastify` | [Fastify](https://www.fastify.io/) is a high performance and low overhead framework highly focused on providing maximum efficiency and speed. Read how to use it [here](/techniques/performance).                                                                                                                                  |

어떤 플랫폼을 사용하더라도 자체적인 애플리케이션 인터페이스를 제공합니다. 각각 `NestExpressApplication` 과 `NestFastifyApplication` 입니다.

아래 예제와 같이 타입을 `NestFactory.create()` 메소드에 전달하면 `app` 객체는 해당 특정 플랫폼에서만 사용할 수있는 메소드를 갖게 됩니다. 하지만 참고로, 실제로 기본 플랫폼 API에 액세스하려는 경우가 **아니라면** 타입을 지정할 **필요**가 없습니다.

```typescript
const app = await NestFactory.create<NestExpressApplication>(AppModule);
```

#### 애플리케이션 실행

설치가 끝났다면 OS 명령 프롬프트에서 다음 명령을 실행하여 HTTP 요청을 기다리는 애플리케이션을 시작할 수 있습니다.

```bash
$ npm run start
```

이 명령은 `src/main.ts` 파일에 정의된 포트에서 HTTP 서버 앱을 시작합니다. 애플리케이션이 실행되면 브라우저를 열고 `http://localhost:3000/`로 이동하세요. `Hello World!` 라는 메시지를 볼 수 있습니다.
