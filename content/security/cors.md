### CORS

Cross-origin resource sharing (CORS)은 다른 도메인에서 리소스를 요청하는 것을 허용하는 메커니즘입니다. Nest에서는 내부적으로 Express의 [cors](https://github.com/expressjs/cors) 패키지를 사용합니다. 이 패키지는 요구사항에 따라 커스터마이징 할 수 있도록 다양한 옵션을 제공합니다.

#### 시작하기

CORS를 활성화하려면, Nest 애플리케이션 객체의 `enableCors()`를 호출해야 합니다.

```typescript
const app = await NestFactory.create(AppModule);
app.enableCors();
await app.listen(3000);
```

`enableCors()` 메서드에는 선택적으로 설정 객체를 인자로 넘길 수 있습니다. 이 객체에서 사용할 수 있는 프로퍼티들은 [CORS](https://github.com/expressjs/cors#configuration-options) 공식 문서에 명시되어 있습니다. 또 다른 방법은 요청을 기반하여(그 즉시) 비동기적으로 설정 객체를 정의하는 [콜백 함수](https://github.com/expressjs/cors#configuring-cors-asynchronously)를 넘기는 것입니다.

`create()` 메서드의 옵션 객체를 통해서도 CORS를 활성화할 수 있습니다. `cors` 프로퍼티를 `true`로 설정하면 기본 세팅값으로 COSR가 활성화됩니다. 아니면 [CORS 설정 객체](https://github.com/expressjs/cors#configuration-options)나 [콜백 함수](https://github.com/expressjs/cors#configuring-cors-asynchronously)를 `cors` 프로퍼티의 값으로 넘겨 동작을 커스터마이징 할 수도 있습니다.

```typescript
const app = await NestFactory.create(AppModule, { cors: true });
await app.listen(3000);
```

위 메서드는 REST 엔드포인트에만 적용됩니다.

CORS를 GraphQ에서 활성화하려면, GraphQL 모듈을 import할 때 `cors` 프로퍼티를 `true`로 설정하거나 [CORS 설정 객체](https://github.com/expressjs/cors#configuration-options) 또는 [콜백 함수](https://github.com/expressjs/cors#configuring-cors-asynchronously)를 `cors` 프로퍼티의 값으로 넘깁니다.

> wargin **주의** `apollo-server-fastify` 패키지에서는 아직 `CorsOptionsDelegate` 솔루션을 지원하지 않습니다.

```typescript
GraphQLModule.forRoot({
  cors: {
    origin: 'http://localhost:3000',
    credentials: true,
  },
}),
```
