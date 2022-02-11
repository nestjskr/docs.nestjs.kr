### RabbitMQ

[RabbitMQ](https://www.rabbitmq.com/)는 다양한 메시징 프로토콜을 지원하는 가벼운 오픈소스 메시지 브로커이며, 대규모 또는 고가용성에 대한 요구사항을 충족시키기 위해 분산 또는 연합 구성으로 배포할 수 있습니다. 추가로, 세계적으로 작은 스타트업이나 대기업 등 어느곳에서든 보편적으로 사용하는 메시지 브로커입니다.

#### 설치

RabbitMQ 기반의 마이크로서비스를 구축하기 위해서는 우선 필요한 패키지들을 설치해야 합니다:

```bash
$ npm i --save amqplib amqp-connection-manager
```

#### 개요

RabbitMQ 전송매체를 사용하려면 `createMicroservice()` 메서드에 다음의 옵션 객체를 넘겨야 합니다:

```typescript
@@filename(main)
const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
  transport: Transport.RMQ,
  options: {
    urls: ['amqp://localhost:5672'],
    queue: 'cats_queue',
    queueOptions: {
      durable: false
    },
  },
});
@@switch
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.RMQ,
  options: {
    urls: ['amqp://localhost:5672'],
    queue: 'cats_queue',
    queueOptions: {
      durable: false
    },
  },
});
```

> info **힌트** `Transport` enum은 `@nestjs/microservices` 패키지에서 import합니다.

#### 옵션

`options`의 프로퍼티는 선택한 전송매체에 따라 다른 값을 가집니다. <strong>RabbitMQ</strong> 전송매체에 대한 프로퍼티들은 아래와 같습니다.

<table>
  <tr>
    <td><code>urls</code></td>
    <td>연결할 URL들</td>
  </tr>
  <tr>
    <td><code>queue</code></td>
    <td>현재 서버가 귀 기울이고 있을 큐 이름</td>
  </tr>
  <tr>
    <td><code>prefetchCount</code></td>
    <td>채널에서 미리 불러올 횟수</td>
    <td></td>
  </tr>
  <tr>
    <td><code>isGlobalPrefetchCount</code></td>
    <td>prefetch를 채널별로 실행할지 여부</td>
  </tr>
  <tr>
    <td><code>noAck</code></td>
    <td><code>false</code>일 때, 승인을 수동으로 보내게 됩니다</td>
  </tr>
  <tr>
    <td><code>queueOptions</code></td>
    <td>큐에 대한 추가적인 옵션 (자세한 내용은 <a href="https://www.squaremobius.net/amqp.node/channel_api.html#channel_assertQueue" rel="nofollow" target="_blank">여기</a>를 참조하세요)</td>
  </tr>
  <tr>
    <td><code>socketOptions</code></td>
    <td>소켓에 대한 추가적인 옵션 (자세한 내용은 <a href="https://www.squaremobius.net/amqp.node/channel_api.html#socket-options" rel="nofollow" target="_blank">여기</a>를 참조하세요)</td>
  </tr>
  <tr>
    <td><code>headers</code></td>
    <td>모든 메시지에 붙일 헤더 정보</td>
  </tr>
</table>

#### 클라이언트

다른 마이크로서비스 전송매체를 다룰 때처럼, RabbitMQ `ClientProxy` 인스턴스를 만들기 위해 <a href="https://docs.nestjs.com/microservices/basics#client">몇가지 옵션</a>이 필요합니다.

인스턴스를 만드는 방법 중 하나는 `ClientsModule`을 사용하는 것입니다. `ClientsModule`로 클라이언트 인스턴스를 만들기 위해서는, import하고 위의 `createMicroservice()` 메서드에 넘겼던 내용 그대로 `register()` 메서드에 옵션 객체를 넘깁니다. 여기에 추가로 `name` 프로퍼티에 주입 토큰을 기입합니다. `ClientsModule`에 대한 자세한 내용은 <a href="https://docs.nestjs.com/microservices/basics#client">여기</a>를 확인해 주세요.

```typescript
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'MATH_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://localhost:5672'],
          queue: 'cats_queue',
          queueOptions: {
            durable: false
          },
        },
      },
    ]),
  ]
  ...
})
```

클라이언트를 생성하기 위한 다른 옵션들(`ClientProxyFactory`나 `@Client()`) 또한 사용해도 됩니다. 이에 대한 내용은 <a href="https://docs.nestjs.com/microservices/basics#client">여기</a>에서 설명합니다.

#### 컨텍스트

더 복잡한 시나리오에서는 들어오는 요청에 대해 더 자세한 정보가 필요할 것입니다. RabbitMQ 전송매체를 사용하면 `RmqContext` 객체에 접근할 수 있습니다.

```typescript
@@filename()
@MessagePattern('notifications')
getNotifications(@Payload() data: number[], @Ctx() context: RmqContext) {
  console.log(`Pattern: ${context.getPattern()}`);
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('notifications')
getNotifications(data, context) {
  console.log(`Pattern: ${context.getPattern()}`);
}
```

> info **힌트** `@Payload()`와 `@Ctx()`, `RmqContext`는 `@nestjs/microservices` 패키지에서 import합니다.

`properties`와 `fields`, `content`를 가지는 본래의 RabbitMQ 메시지에 접근하고 싶다면 아래와 같이 `RmqContext` 객체의 `getMessage()` 메서드를 사용하면 됩니다:

```typescript
@@filename()
@MessagePattern('notifications')
getNotifications(@Payload() data: number[], @Ctx() context: RmqContext) {
  console.log(context.getMessage());
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('notifications')
getNotifications(data, context) {
  console.log(context.getMessage());
}
```

RabbitMQ [채널]을 참조하고 싶다면 아래와 같이 `RmqContext` 객체의 `getChannerlRef` 메서드를 사용하면 됩니다:

```typescript
@@filename()
@MessagePattern('notifications')
getNotifications(@Payload() data: number[], @Ctx() context: RmqContext) {
  console.log(context.getChannelRef());
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('notifications')
getNotifications(data, context) {
  console.log(context.getChannelRef());
}
```

#### 메시지 승인

메시지를 유실되는 일을 방지하기 위해 RabbitMQ에서는 [메시지 승인](https://www.rabbitmq.com/confirms.html)을 지원합니다. 승인이란 소비자가 RabbitMQ에게 특정 메시지를 정상적으로 수신하고 처리했으니 마음대로 지워도 된다고 알리는 것입니다. 소비자가 승인을 보내지 않고 없어지면(채널이 닫히거나, 연결이 끊기거나, TCP 연결이 사라지는 등), RabbitMQ는 메시지가 제대로 처리되지 않았다고 이해하여 해당 메시지를 다시 큐에 담습니다.

수동 승인 모드를 활성화 하려면 `noAck` 프로퍼티를 `false`로 설정합니다:

```typescript
options: {
  urls: ['amqp://localhost:5672'],
  queue: 'cats_queue',
  noAck: false,
  queueOptions: {
    durable: false
  },
},
```

소비자가 승인을 수동으로 보내도록 전환 되었다면, 수신한 메시지를 처리하는 곳에서 적절한 승인을 보내어 작업이 완료되었음을 알립니다.

```typescript
@@filename()
@MessagePattern('notifications')
getNotifications(@Payload() data: number[], @Ctx() context: RmqContext) {
  const channel = context.getChannelRef();
  const originalMsg = context.getMessage();

  channel.ack(originalMsg);
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('notifications')
getNotifications(data, context) {
  const channel = context.getChannelRef();
  const originalMsg = context.getMessage();

  channel.ack(originalMsg);
}
```

#### 레코드 빌더

메시지에 대한 옵션을 설정하려면 `RmqRecordBuilder` 클래스(참고: 이 클래스는 이벤트 기반 흐름에서도 사용할 수 있습니다)를 사용합니다. 예를 들어 `headers`와 `priority` 프로퍼티를 설정하려면, 다음과 같이 `setOptions` 메서드를 사용합니다.

```typescript
const message = ':cat:';
const record = new RmqRecordBuilder(message)
  .setOptions({
    headers: {
      ['x-version']: '1.0.0',
    },
    priority: 3,
  })
  .build();

this.client.send('replace-emoji', record).subscribe(...);
```

> info **힌트** `RmqRecordBuilder` 클래스는 `@nestjs/microservices` 패키지에서 import합니다.

이제 다음과 같이 서버 사이드에서도 `RmqContext`를 통해 이 값들을 확인할 수 있습니다:

```typescript
@@filename()
@MessagePattern('replace-emoji')
replaceEmoji(@Payload() data: string, @Ctx() context: RmqContext): string {
  const { properties: { headers } } = context.getMessage();
  return headers['x-version'] === '1.0.0' ? '🐱' : '🐈';
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('replace-emoji')
replaceEmoji(data, context) {
  const { properties: { headers } } = context.getMessage();
  return headers['x-version'] === '1.0.0' ? '🐱' : '🐈';
}
```
