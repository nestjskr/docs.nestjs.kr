### CSRF 보호

CSRF나 XSRF로 잘 알려진 Cross-site request forgery는 웹 에플리케이션이 신뢰하는 사용자로부터 **승인되지 않은**명령이 전송되게 만들도록 웹사이트를 악용하는 수법입니다. [csurf](https://github.com/expressjs/csurf) 패키지를 이용하면 이러한 유형의 공격을 완화시킬 수 있습니다.

#### Express와 함께 사용하기 (기본)

먼저 필요한 패키지를 설치합니다:

```bash
$ npm i --save csurf
```

> warning **주의** [`csurf` 문서](https://github.com/expressjs/csurf#csurf)에서 설명하는 바에 따르면, 이 미들웨어는 세션 미들웨어나 `cookie-parser`를 먼저 설치해야 사용할 수 있습니다. 문서를 참조하시어 추가 지침을 확인하시길 바랍니다.

설치가 완료되었으면 `csurf` 미들웨어를 전역 미들웨어로 적용합니다.

```typescript
import * as csurf from "csurf";
// ...
// somewhere in your initialization file
app.use(csurf());
```

#### Fastify와 함께 사용하기

먼저 필요한 패키지를 설치합니다:

```bash
$ npm i --save fastify-csrf
```

설치가 완료되었다면, 다음과 같이 `fastify-csrf` 플러그인을 등록합니다:

```typescript
import fastifyCsrf from "fastify-csrf";
// ...
// somewhere in your initialization file after registering some storage plugin
app.register(fastifyCsrf);
```

> warning **주의** [여기](https://github.com/fastify/fastify-csrf#usage) `fastify-csrf` 문서에서 설명하는 바에 따르면, 이 플러그인은 저장소 플러그인을 먼저 설치해야 사용할 수 있습니다. 문서를 참조하시어 추가 지침을 확인하시길 바랍니다.
