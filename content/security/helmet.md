### Helmet

[Helmet](https://github.com/helmetjs/helmet)은 HTTP 헤더를 적절하게 설정해 줘서 우리의 앱을 잘 알려진 웹 취약점으로부터 보호하는 데 도움을 줍니다. 일반적으로 Helmet은 보안과 관련된 HTTP 헤더를 설정하는 작은 미들웨어 함수의 집합체입니다(자세한 정보는 [여기](https://github.com/helmetjs/helmet#how-it-works)를 참조하세요).

> info **힌트** `helmet`을 전역적으로 등록하고 적용하는 일은 다른 `app.use()` 구문들 혹은 `app.use()`를 호출하는 설정 함수들보다 먼저 실행되어야 합니다. 이는 기본 플랫폼(즉, Express 또는 Fastify)이 동작할 때 미들웨어/라우트들이 정의된 순서가 중요하기 때문입니다. 만약 라우트를 정의한 이후에 `helmet`이나 `cors`같은 미들웨어를 사용한다면, 그 미들웨어는 해당 라우트들에게 적용되지 않고 미들웨어 다음에 정의된 라우트에만 적용될 것입니다.

#### Express와 함께 사용하기 (기본)

우선 필요한 패키지를 설치합니다.

```bash
$ npm i --save helmet
```

설치가 완료되면, 이를 전역 미들웨어로 적용합니다.

```typescript
import * as helmet from "helmet";
// somewhere in your initialization file
app.use(helmet());
```

> info **힌트** 만약 `Helmet`을 import할 때 `This expression is not callable`라는 에러가 발생한다면 프로젝트의 `tsconfig.json` 파일에 `allowSyntheticDefaultImports` 옵션과 `esModuleInterop` 옵션이 `true`로 설정되었을 가능성이 큽니다. 만약 그렇다면 import 구문을 이렇게 바꿔 주세요: `import helmet from 'helmet'`.

#### Fastify와 함께 사용하기

만약 `FastifyAdapter`를 사용한다면 [fastify-helmet](https://github.com/fastify/fastify-helmet) 패키지를 설치해야 합니다:

```bash
$ npm i --save fastify-helmet
```

[fastify-helmet](https://github.com/fastify/fastify-helmet)는 미들웨어가 아닌 [Fastify 플러그인](https://www.fastify.io/docs/latest/Plugins/)으로 사용합니다. 즉, `app.register()`을 사용합니다:

```typescript
import { fastifyHelmet } from "fastify-helmet";
// somewhere in your initialization file
await app.register(fastifyHelmet);
```

> warning **주의** `apollo-server-fastify`과 `fastify-helmet`를 사용한다면, GraphQL playground에서 [CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)와 관련된 문제가 있을 수도 있습니다. 이 충돌을 해결하기 위해 아래와 같이 CSP를 설정해 주세요:
>
> ```typescript
> await app.register(fastifyHelmet, {
>   contentSecurityPolicy: {
>     directives: {
>       defaultSrc: [`'self'`],
>       styleSrc: [
>         `'self'`,
>         `'unsafe-inline'`,
>         "cdn.jsdelivr.net",
>         "fonts.googleapis.com",
>       ],
>       fontSrc: [`'self'`, "fonts.gstatic.com"],
>       imgSrc: [`'self'`, "data:", "cdn.jsdelivr.net"],
>       scriptSrc: [`'self'`, `https: 'unsafe-inline'`, `cdn.jsdelivr.net`],
>     },
>   },
> });
>
> // If you are not going to use CSP at all, you can use this:
> await app.register(fastifyHelmet, {
>   contentSecurityPolicy: false,
> });
> ```
