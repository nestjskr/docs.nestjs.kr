### Libraries

많은 애플리케이션들은 동일한 일반적인 문제들을 해결하거나, 몇몇 다른 컨텍스트에서 모듈러 컴포넌트들을 재사용할 필요가 있습니다. Nest는 이러한 문제를 해결하는 몇 가지 방법이 있지만, 각 방법들은 다른 구조적, 조직적 목표를 충족시키는데 도움을 주는 방식으로 해당 문제를 해결하기 위해 서로 다른 수준에서 동작합니다.

Nest [모듈](/modules)들은 하나의 애플리케이션 내에서 컴포넌트들을 공유를 가능하게하는 실행 컨텍스트를 제공하는데 유용합니다. 모듈들은 또한 다른 프로젝트에 설치할 수 있는 재사용가능한 라이브러리를 생성하기 위해 [npm](https://npmjs.com)으로 패키징될 수 있습니다. 이것은 서로 다르거나, 느슨하게 연결되거나 독립적인 조직에서 사용되는 설정가능하고, 재사용가능한 라이브러리를 배포하는데 효과적인 방법이 될 수 있습니다 (예: 서드파티 라이브러리 배포/설치).

밀접하게 조직된 그룹들 사이에서(예: 회사/프로젝트 경계 내에서) 코드를 공유하기 위해서, 컴포넌트 공유에 대해 더 가볍게 접근하는 것은 유용할 수 있습니다. Monorepos는 이것을 가능하게 하는 구조로 생겨났고, Monorepo 내에서 **라이브러리**는 코드를 쉽고, 가벼운 방식으로 공유하는 방법을 제공합니다. Nest monorepo에서, 라이브러리를 사용하는 것은 컴포넌트를 공유하는 애플리케이션의 쉬운 조립을 가능하게 합니다. 실제로, 이것은 모듈러 컴포넌트들을 구축하고 구성하는데 초점을 맞추기 위해 모놀리식 애플리케이션과 개발 프로세스의 분해를 권장합니다.

#### Nest libraries

A Nest library is a Nest project that differs from an application in that it cannot run on its own. A library must be imported into a containing application in order for its code to execute. The built-in support for libraries described in this section is only available for **monorepos** (standard mode projects can achieve similar functionality using npm packages).

For example, an organization may develop an `AuthModule` that manages authentication by implementing company policies that govern all internal applications. Rather than build that module separately for each application, or physically packaging the code with npm and requiring each project to install it, a monorepo can define this module as a library. When organized this way, all consumers of the library module can see an up-to-date version of the `AuthModule` as it is committed. This can have significant benefits for coordinating component development and assembly, and simplifying end-to-end testing.

#### Creating libraries

Any functionality that is suitable for re-use is a candidate for being managed as a library. Deciding what should be a library, and what should be part of an application, is an architectural design decision. Creating libraries involves more than simply copying code from an existing application to a new library. When packaged as a library, the library code must be decoupled from the application. This may require **more** time up front and force some design decisions that you may not face with more tightly coupled code. But this additional effort can pay off when the library can be used to enable more rapid application assembly across multiple applications.

To get started with creating a library, run the following command:

```bash
nest g library my-library
```

When you run the command, the `library` schematic prompts you for a prefix (AKA alias) for the library:

```bash
What prefix would you like to use for the library (default: @app)?
```

This creates a new project in your workspace called `my-library`.
A library-type project, like an application-type project, is generated into a named folder using a schematic. Libraries are managed under the `libs` folder of the monorepo root. Nest creates the `libs` folder the first time a library is created.

The files generated for a library are slightly different from those generated for an application. Here is the contents of the `libs` folder after executing the command above:

<div class="file-tree">
  <div class="item">libs</div>
  <div class="children">
    <div class="item">my-library</div>
    <div class="children">
      <div class="item">src</div>
      <div class="children">
        <div class="item">index.ts</div>
        <div class="item">my-library.module.ts</div>
        <div class="item">my-library.service.ts</div>
      </div>
      <div class="item">tsconfig.lib.json</div>
    </div>
  </div>
</div>

The `nest-cli.json` file will have a new entry for the library under the `"projects"` key:

```javascript
...
{
    "my-library": {
      "type": "library",
      "root": "libs/my-library",
      "entryFile": "index",
      "sourceRoot": "libs/my-library/src",
      "compilerOptions": {
        "tsConfigPath": "libs/my-library/tsconfig.lib.json"
      }
}
...
```

There are two differences in `nest-cli.json` metadata between libraries and applications:

- the `"type"` property is set to `"library"` instead of `"application"`
- the `"entryFile"` property is set to `"index"` instead of `"main"`

These differences key the build process to handle libraries appropriately. For example, a library exports its functions through the `index.js` file.

As with application-type projects, libraries each have their own `tsconfig.lib.json` file that extends the root (monorepo-wide) `tsconfig.json` file. You can modify this file, if necessary, to provide library-specific compiler options.

You can build the library with the CLI command:

```bash
nest build my-library
```

#### Using libraries

With the automatically generated configuration files in place, using libraries is straightforward. How would we import `MyLibraryService` from the `my-library` library into the `my-project` application?

First, note that using library modules is the same as using any other Nest module. What the monorepo does is manage paths in a way that importing libraries and generating builds is now transparent. To use `MyLibraryService`, we need to import its declaring module. We can modify `my-project/src/app.module.ts` as follows to import `MyLibraryModule`.

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { MyLibraryModule } from '@app/my-library';

@Module({
  imports: [MyLibraryModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

Notice above that we've used a path alias of `@app` in the ES module `import` line, which was the `prefix` we supplied with the `nest g library` command above. Under the covers, Nest handles this through tsconfig path mapping. When adding a library, Nest updates the global (monorepo) `tsconfig.json` file's `"paths"` key like this:

```javascript
"paths": {
    "@app/my-library": [
        "libs/my-library/src"
    ],
    "@app/my-library/*": [
        "libs/my-library/src/*"
    ]
}
```

So, in a nutshell, the combination of the monorepo and library features has made it easy and intuitive to include library modules into applications.

This same mechanism enables building and deploying applications that compose libraries. Once you've imported the `MyLibraryModule`, running `nest build` handles all the module resolution automatically and bundles the app along with any library dependencies, for deployment. The default compiler for a monorepo is **webpack**, so the resulting distribution file is a single file that bundles all of the transpiled JavaScript files into a single file. You can also switch to `tsc` as described <a href="https://docs.nestjs.com/cli/monorepo#global-compiler-options">here</a>.
