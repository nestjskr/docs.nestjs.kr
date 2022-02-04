### First steps

이 공식 문서들을 통해, 여러분은 Nest의 **핵심 기본**에 대해 배울 것입니다. Nest 애플리케이션에서 필수 구성 요소를 숙지할 수 있도록, 기본 CRUD 애플리케이션을 구축해 다양한 분야를 포괄하는 기능을 도입하겠습니다.

#### 프로그래밍 언어

우린 [TypeScript](https://www.typescriptlang.org/)를 좋아하지만, 무엇보다 [Node.js](https://nodejs.org/en/)를 사랑합니다. 그것이 Nest가 Typescript와 **순수 Javascript** 모두 호환되는 이유입니다. Nest는 최신 언어의 기능의 이점을 활용하기 때문에 vanilla Javascript와 함께 사용하려면 [Babel](https://babeljs.io/) 컴파일러가 필요합니다.

제공하는 예제에서는 주로 Typescript를 사용하지만 예제 코드를 vanilla Javascript로 전환할 수도 있습니다(각 code snippet의 오른쪽 상단 모서리에 있는 language 버튼을 클릭해 전환).

#### 전제 조건

운영 체제에 [Node.js](https://nodejs.org/) (>= 10.13.0, v13 제외)가 설치되어 있는지 확인하세요.

#### Setup

[Nest CLI](/cli/overview)를 사용하면 새 프로젝트를 설정하는 것이 매우 간단합니다. [npm](https://www.npmjs.com/)이 설치되어 있다면, 다음 명령을 사용해 새 Nest 프로젝트를 생성할 수 있습니다.:

```bash
$ npm i -g @nestjs/cli
$ nest new project-name
```

`project-name` 폴더가 생성되고 노드 모듈과 몇 가지 다른 boilerplate 파일들이 설치되며 `src/` 디렉토리가 생성되고 여러 개의 core 파일로 채워집니다.

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

다음은 core 파일의 간략한 개요입니다:

|                          |                                                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| `app.controller.ts`      | 단일 경로를 가진 기본 controller입니다.                                                                             |
| `app.controller.spec.ts` | controller의 unit test입니다.                                                                                       |
| `app.module.ts`          | 애플리케이션의 root 모듈입니다.                                                                                     |
| `app.service.ts`         | 단일 메서드를 사용하는 기본 service입니다.                                                                          |
| `main.ts`                | core 함수인 `NestFactory`를 사용해 Nest 애플리케이션 인스턴스를 만드는 응용 프로그램의 엔트리 파일입니다.           |

`main.ts` 파일은 async 함수를 포함하며, 이를 통해 응용 프로그램을 **bootstrap** 합니다:

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

Nest 애플리케이션 인스턴스를 생성하기 위해, core 클래스인 `NestFactory`를 사용합니다. `NestFactory`는 애플리케이션 인스턴스를 만들 수 있는 몇 가지 정적 메서드를 노출합니다. `create()` 메서드는 `INestApplication`를 충족하는 애플리케이션 객체를 응답합니다. 이 객체는 다음 장에서 설명하는 메서드 집합을 제공합니다. 위의 `main.ts` 예제에서, 우리는 응용 프로그램이 인바운드 HTTP 요청을 기다릴 수 있도록 HTTP 리스너를 실행합니다.

Nest CLI로 구성된 프로젝트는 개발자가 각 모듈을 자체 전용 디렉토리에 보관하는 컨벤션을 따르도록 권장하는 초기 프로젝트 구조를 생성한다는 점에 유의하셔야 합니다.

<app-banner-courses></app-banner-courses>

#### Platform

Nest aims to be a platform-agnostic framework. Platform independence makes it possible to create reusable logical parts that developers can take advantage of across several different types of applications. Technically, Nest is able to work with any Node HTTP framework once an adapter is created. There are two HTTP platforms supported out-of-the-box: [express](https://expressjs.com/) and [fastify](https://www.fastify.io). You can choose the one that best suits your needs.

|                    |                                                                                                                                                                                                                                                                                                                                    |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `platform-express` | [Express](https://expressjs.com/) is a well-known minimalist web framework for node. It's a battle tested, production-ready library with lots of resources implemented by the community. The `@nestjs/platform-express` package is used by default. Many users are well served with Express, and need take no action to enable it. |
| `platform-fastify` | [Fastify](https://www.fastify.io/) is a high performance and low overhead framework highly focused on providing maximum efficiency and speed. Read how to use it [here](/techniques/performance).                                                                                                                                  |

Whichever platform is used, it exposes its own application interface. These are seen respectively as `NestExpressApplication` and `NestFastifyApplication`.

When you pass a type to the `NestFactory.create()` method, as in the example below, the `app` object will have methods available exclusively for that specific platform. Note, however, you don't **need** to specify a type **unless** you actually want to access the underlying platform API.

```typescript
const app = await NestFactory.create<NestExpressApplication>(AppModule);
```

#### Running the application

Once the installation process is complete, you can run the following command at your OS command prompt to start the application listening for inbound HTTP requests:

```bash
$ npm run start
```

This command starts the app with the HTTP server listening on the port defined in the `src/main.ts` file. Once the application is running, open your browser and navigate to `http://localhost:3000/`. You should see the `Hello World!` message.
