### Introduction

Nest(NestJS)는 효율적이고 확장 가능한 [Node.js](https://nodejs.org/) 서버 어플리케이션을 구축하기 위한 프레임워크입니다. 프로그레시브 자바스크립트를 사용하고, [타입스크립트](http://www.typescriptlang.org/)를 완벽히 지원하며 빌드에 사용합니다. (하지만 여전히 순수한 자바스크립트로도 코딩할 수 있습니다.) 또한 OOP(Object Oriented Programming), FP(Functional Reactive Programming) 및 FRP(Functional Reactive Programming)의 요소들을 결합하고 있습니다.

내부적으로 Nest는 [Express](https://expressjs.com/)(기본값)와 같은 강력한 HTTP 서버 프레임워크를 사용하며, 선택적으로 [Fastify](https://github.com/fastify/fastify)를 사용하도록 구성할 수도 있습니다!

Nest는 일반적인 Node.js 프레임워크(Express/Fastify) 위에 일정 수준의 추상화를 제공하지만, 또한 개발자에게 직접 프레임워크 API를 노출하기도 합니다. 이것을 통해 개발자들이 기반 플랫폼에서 이용 가능한 무수한 서드 파티 모듈을 자유롭게 사용할 수 있게 해줍니다.

#### Philosophy

최근에 들어서 Node.js 덕분에 자바스크립트는 프론트엔드 및 백엔드 어플리케이션 모두에서 웹의 "공용어"가 되었습니다. 이로 인해 [Angular](https://angular.io/), [React](https://github.com/facebook/react) 및 [Vue](https://github.com/vuejs/vue)와 같이 개발자의 생산성을 높여주며, 빠르고, 테스트 할 수 있고, 확장성 있는 프론트엔드 어플리케이션을 만들 수 있는 멋진 프로젝트들이 생겨났습니다. 그러나 백엔드는 Node(및 서버측 JavaScript)의 우수한 라이브러리, 헬퍼 및 도구가 많이 존재함에도, **구조적인** 주요 문제를 효과적으로 해결하는 프로젝트가 없습니다.

Nest는 개발자와 팀에게 테스트 가능하며, 확장성 있고, 느슨하게 결합되고, 유지 및 보수가 용이한 어플리케이션을 만들 수 있는, 즉시 사용 가능한 어플리케이션 아키텍처를 제공합니다. 이 아키텍처는 Angular에서 많은 영감을 받았습니다.

#### Installation

Nest를 시작하기 위해 [Nest CLI](/cli/overview)를 사용해 프로젝트를 스캐폴드 하거나, 또는 스타터 프로젝트를 클론할 수 있습니다. (두 가지 방법 모두 같은 결과물을 도출합니다.)

Nest CLI를 사용하여 프로젝트를 스캐폴드 하려면 다음 명령을 실행합니다. 이는 새 프로젝트 디렉토리를 생성하고, 디렉토리 내부에 Nest 초기 핵심 파일들과 지원 모듈을 생성하여 프로젝트에 대한 기본 구조를 잡아줍니다. 처음 사용자는 **Nest CLI**를 사용하여 새 프로젝트를 생성하는 것을 추천합니다. 이 방법은 [First Steps](first-steps)에서 계속 진행하겠습니다.

```bash
$ npm i -g @nestjs/cli
$ nest new project-name
```

#### Alternatives

**Git**을 이용해 타입스크립트 스타터 프로젝트를 설치하려면:

```bash
$ git clone https://github.com/nestjs/typescript-starter.git project
$ cd project
$ npm install
$ npm run start
```

> info **힌트** 만약 git history 없이 clone 하고 싶다면, [degit](https://github.com/Rich-Harris/degit)을 사용할 수 있습니다.

브라우저를 열고 [`http://localhost:3000/`](http://localhost:3000/)로 이동합니다.

자바스크립트 버전의 스타터 프로젝트를 설치하고자 한다면, 위의 커맨드에 `javascript-starter.git`를 사용하십시오.

또한 **npm** (또는 **yarn**)을 사용하여 Nest 코어와 지원 파일을 설치하면 처음부터 수동으로 새 프로젝트를 생성할 수도 있습니다. 물론 이 경우에는 프로젝트의 보일러 플레이트 파일을 직접 생성해야 합니다.

```bash
$ npm i --save @nestjs/core @nestjs/common rxjs reflect-metadata
```
