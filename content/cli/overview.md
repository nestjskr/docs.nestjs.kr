### 시작하기

[Nest CLI](https://github.com/nestjs/nest-cli) 는 Nest application 초기화, 개발 및 유지보수 하는데 도움이 되는 명령줄-인터페이스 도구입니다. 프로젝트 스케폴딩, 개발모드 제공, 빌드 및 번들링 도구 등 다양한 방법을 지원 합니다. 이 도구는 잘 구성된 아키텍쳐 패턴들을 구현하여 여러분의 애플리케이션이 견고한 구조를 가질 수 있도록 합니다.

#### 설치

**참고**: 이 가이드에서는 [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) 을 사용하여 Nest CLI를 포함한 패키지를 설치하는 방법에 대해 설명합니다. 선호하는 다른 패키지 매니저를 사용해도 좋습니다. npm 을 사용하면 OS 명령줄에서 `nest` CLI 바이너리 파일의 위치를 찾는 데 몇 가지 옵션을 사용할 수 있습니다. 여기서는 `-g`옵션을 사용하여 `nest` 바이너리를 전역으로 설치하는 방법을 설명합니다. 이는 어느정도의 편의성을 제공하며, 해당 설치 방법을 전제로 문서를 작성 하였습니다. **어떠한** `npm` 패키지를 전역으로 설치하면 올바른 버전이 실행되고 있는지 확인하는 책임은 사용자에게 있습니다. 또한 다른 프로젝트 이더라도 **같은** 버전의 CLI 를 실행 한다는 걸 의미합니다. 합리적인 대안은 [npx](https://github.com/npm/npx) 프로그램(또는 다른 패키지 매니저와 유사한 기능)을 사용하여 Nest CLI의 **managed version**을(를) 실행하는 것입니다. 자세한 내용은 [npx 설명서](https://github.com/npm/npx) 및/또는 DevOps 지원 담당자에게 문의하세요.

`npm install -g` 명령을 사용하여 CLI를 전역으로 설치합니다. (전역 설치에 대한 자세한 내용은 위의 **참고** 항목을 참조하세요).

```bash
$ npm install -g @nestjs/cli
```

#### 기본 흐름

설치되면 `nest` 실행 파일을 통해 OS 명령줄에서 직접 CLI 명령을 호출할 수 있습니다. 다음을 입력하여 사용 가능한 `nest` 명령을 확인해 보세요

```bash
$ nest --help
```

다음과 같은 구문을 사용하여 개별적인 명령어에 대한 도움말을 볼 수 있습니다. `new`, `add`등과 같은 명령어에 대한 자세한 도움말을 보려면 아래의 예에서 `generate` 자리에 명령어를 입력하면 됩니다.

```bash
$ nest generate --help
```

개발 모드에서 새로운 Nest 프로젝트를 생성하거나 빌드, 실행 시키려면 프로젝트의 상위 폴더로 이동한 후 다음 명령어를 실행합니다.

```bash
$ nest new my-nest-project
$ cd my-nest-project
$ npm run start:dev
```

브라우저에서 [http://localhost:3000](http://localhost:3000)을 열어 실행 중인 애플리케션을 확인합니다. 소스 파일을 변경하면 앱이 자동으로 다시 컴파일하고 로드합니다.

#### 프로젝트 구조

`nest new` 를 실행하면 Nest는 새 폴더를 만들고 초기 설정파일들을 만들어 boilerplate 애플리케이션 구조를 생성합니다. 이 문서의 설명에 따라 기본 구조에서 새 컴포넌트를 추가하면서 작업을 이어나가면 됩니다. 우리는 `nest new` 로 생성한 프로젝트의 구조를 **standard mode** 라고 부릅니다. 또한 Nest는 **monorepo mode** 라고 하는 여러 프로젝트 및 라이브러리를 관리하기위한 대체 구조를 지원합니다.

**build** 프로세스가 작동하는 방식과 내장 [라이브러리](/cli/library) 지원에 대한 몇 가지 구체적인 고려 사항 외에도(기본적으로 monorepo mode 는 monorepo-style의 프로젝트 구조에서 발생할 수 있는 빌드 복잡성을 단순화합니다), 나머지 Nest 기능 과 이 문서는 standard mode 및 monorepo mode 프로젝트 구조 모두에 동일하게 적용됩니다. 나중에 언제든지 standard mode에서 monorepo mode로 쉽게 전환할 수 있으므로 Nest에 대해 배우는 동안 이 결정을 하지 않아도 괜찮습니다.

두 모드 중 하나를 사용하여 여러 프로젝트를 관리할 수 있습니다. 다음은 각 모드의 차이점을 간단히 요약한 것입니다.

| Feature                                               | Standard Mode                                                      | Monorepo Mode                                              |
| ----------------------------------------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------- |
| Multiple projects                                     | Separate file system structure                                     | Single file system structure                               |
| `node_modules` & `package.json`                       | Separate instances                                                 | Shared across monorepo                                     |
| Default compiler                                      | `tsc`                                                              | webpack                                                    |
| Compiler settings                                     | Specified separately                                               | Monorepo defaults that can be overridden per project       |
| Config files like `.eslintrc.js`, `.prettierrc`, etc. | Specified separately                                               | Shared across monorepo                                     |
| `nest build` and `nest start` commands                | Target defaults automatically to the (only) project in the context | Target defaults to the **default project** in the monorepo |
| Libraries                                             | Managed manually, usually via npm packaging                        | Built-in support, including path management and bundling   |

가장 적합한 모드를 결정하는 데 도움이 되는 자세한 내용은 [Workspaces](/cli/monorepo) 및 [Libraries](/cli/libraries) 섹션을 참조하세요.

<app-banner-courses></app-banner-courses>

#### CLI 명령어 문법

모든 `nest` 명령어들은 동일한 형식을 따릅니다

```bash
nest commandOrAlias requiredArg [optionalArg] [options]
```

예시:

```bash
$ nest new my-nest-project --dry-run
```

여기서 `new` 는 _commandOrAlias_ 입니다.`new` 의 별칭은 `n` 입니다. `my-nest-project` 는 _requiredArg_ 입니다. _requiredArg_ 가 입력되지 않은 경우, `nest` 에서 이를 입력하라고 합니다. 또한, `--dry-run` 의 약식 표현은 `-d` 입니다. 이 점을 참고하면, 다음 명령은 위의 명령과 동일합니다.

```bash
$ nest n my-nest-project -d
```

대부분의 명령과 일부 옵션에는 별칭이 있습니다. `nest new --help` 를 실행하여 이러한 옵션과 별칭들을 보고 위의 구조에 대해 이해했는지 확인하세요.

#### 명령어 정리

명령별 옵션을 보려면 다음 명령어들 중 하나에 대해 `nest <command> --help` 를 실행하세요.

각 명령에 대한 자세한 내용은 [usage](/cli/uses)를 참조하세요.

| Command    | Alias | Description                                                                                           |
| ---------- | ----- | ----------------------------------------------------------------------------------------------------- |
| `new`      | `n`   | Scaffolds a new _standard mode_ application with all boilerplate files needed to run.                 |
| `generate` | `g`   | Generates and/or modifies files based on a schematic.                                                 |
| `build`    |       | Compiles an application or workspace into an output folder.                                           |
| `start`    |       | Compiles and runs an application (or default project in a workspace).                                 |
| `add`      |       | Imports a library that has been packaged as a **nest library**, running its install schematic.        |
| `update`   | `u`   | Update `@nestjs` dependencies in the `package.json` `"dependencies"` list to their `@latest` version. |
| `info`     | `i`   | Displays information about installed nest packages and other helpful system info.                     |
