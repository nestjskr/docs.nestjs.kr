### Lifecycle Events

모든 애플리케이션 요소들과 마찬가지로, Nest 애플리케이션에도 Nest가 관리하는 라이프사이클이 있습니다. Nest는 주요 라이프사이클 이벤트에 가시성을 부여하는 **lifecycle hooks**를 제공하고, 이벤트가 발생할 때 작동할 수 있도록(`module`, `injectable` 또는 `controller`에 등록된 코드를 실행하는 것) 합니다.

#### 라이프사이클 순서
아래 다이어그램은 애플리케이션이 부트스트랩 되고 node 프로세스가 존재할 때까지의 주요 라이프사이클 이벤트 순서를 나타냅니다. 전체 라이프사이클을 3단계로 나눌 수 있습니다: **initializing(초기화)**, **running(실행)** and **terminating(종료)**. 이 라이프사이클을 통해 모듈 및 서비스의 적절한 초기화를 계획하고, 활성화된 연결을 관리할 수 있으며, 종료 신호를 받을 경우 정상적으로 애플리케이션을 종료할 수 있습니다.

<figure><img src="/assets/lifecycle-events.png" /></figure>

#### 라이프사이클 이벤트

라이프사이클 이벤트는 애플리케이션 부트스트래핑과 종료 시 발생합니다. Nest는 각 라이프사이클 이벤트 순서대로([아래](https://docs.nestjs.com/fundamentals/lifecycle-events#application-shutdown) 제시된 대로 **shutdown hooks**가 가장 먼저 활성화 되어야 합니다.) `modules`, `injectables` 그리고 `controllers`에 등록된 라이프사이클 훅 메서드를 호출합니다. 위 다이어그램에 소개된 것처럼 Nest는 연결 수신과 종료를 위해 적합한 기본 메서드도 호출합니다.

다음 표에서 `onModuleDestroy`, `beforeApplicationShutdown` 그리고 `onApplicationShutdown`는, `app.close()`를 확실하게 호출하거나 프로세스가 특수 시스템 신호(예: SIGTERM)를 수신하고, 애플리케이션 부트스트랩 시 `enableShutdownHooks`를 올바르게 호출할 경우에만 작동합니다. (아래 **Application shutdown** 부분 참고)

| 라이프사이클 훅 메서드                    | 훅 메서드를 호출하는 라이프사이클 이벤트                                                                                                                  |
|---------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| `onModuleInit()`                | 호스트 모듈의 종속성이 해결되면 호출                                                                                                                    |
| `onApplicationBootstrap()`      | 모든 모듈이 초기화된 후, 연결을 수신 대기하기 전에 호출                                                                                                        |
| `onModuleDestroy()`\*           | 종료 신호(예: `SIGTERM`)가 수신된 후 호출                                                                                                           |
| `beforeApplicationShutdown()`\* | 모든 `onModuleDestroy()` 핸들러가 완료(Promise resolved 또는 rejected)된 후 호출; 한 번 완료(Promise resolved 또는 rejected)되면 기존의 모든 연결 종료(`app.close()` 호출) |
| `onApplicationShutdown()`\*     | 연결 종료 후 호출(`app.close()` resolves)                                                                                                      |

\* 이러한 이벤트의 경우, `app.close()`를 명확하게 호출하지 않으면, `SIGTERM`과 같은 시스템 신호와 함께 작동하기 위해 옵트인을 해야 합니다. 아래 [Application shutdown](fundamentals/lifecycle-events#application-shutdown)을 참고하세요. 

> **경고**
> 
> 위에 나열된 라이프사이클 훅은 **request-scoped** 클래스에 관해 작동하지 않습니다. Requested-scoped 클래스는 애플리케이션 라이프사이클과 연결되어 있지 않으며 수명은 예측 불가합니다. 전적으로 각 요청을 위해 생성되며, 응답 후 자동적으로 가비지 컬렉션에 추가됩니다.

#### 사용법

각 라이프사이클은 인터페이스로 표현됩니다. 인터페이스는 Typescript 컴파일 이후에는 존재하지 않기 때문에 기술적 관점에서 보면 선택 사항입니다. 그럼에도 불구하고 엄격한 타입 제어와 에디터 도구에 이점이 있기 때문에 사용하는 것이 좋습니다. 라이프사이클 훅을 등록하기 위해서는 적합한 인터페이스를 실행하세요. 예를 들어, 특정 클래스에서 모듈 초기화 동안 호출되는 함수를 등록하기 위해서는, 아래 제시된 것처럼 `onModuleInit()` 함수를 주입하여 `OnModuleInit` 인터페이스를 실행해야 합니다.

```typescript
@@filename()
import { Injectable, OnModuleInit } from '@nestjs/common';

@Injectable()
export class UsersService implements OnModuleInit {
  onModuleInit() {
    console.log(`The module has been initialized.`);
  }
}
@@switch
import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
  onModuleInit() {
    console.log(`The module has been initialized.`);
  }
}
```

#### 비동기 초기화

`OnModuleInit`과 `OnApplicationBootstrap` 훅 모두 애플리케이션 초기화 과정을 연기할 수 있게 해줍니다.(`Promise`를 반환하거나 메서드 바디에서 비동기 메서드 완료를 처리하는 `async`와 `await`로 메서드를 작성합니다.)
```typescript
@@filename()
async onModuleInit(): Promise<void> {
  await this.fetch();
}
@@switch
async onModuleInit() {
  await this.fetch();
}
```

#### 애플리케이션 종료

`onModuleDestroy()`, `beforeApplicationShutdown()` 그리고 `onApplicationShutdown()` 훅은 종료 단계(`app.close()` 호출에 대한 응답이나 옵트인일 경우 SIGTERM과 같은 시스템 신호 수신 즉시)에서 호출됩니다. 이 기능은 dynos나 유사 서비스를 위한 [Heroku](https://www.heroku.com/)에서 컨테이너의 라이프사이클을 관리하기 위해 [Kubernetes](https://kubernetes.io/)와 함께 자주 사용됩니다.

종료(Shutdown) 훅 리스너는 시스템 리소스를 사용하기 때문에 기본적으로 비활성화 되어 있습니다. 종료 훅을 사용하려면 `enableShutdownHooks()` 호출을 통해 반드시 리스너를 활성화해야 합니다.

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Starts listening for shutdown hooks
  app.enableShutdownHooks();

  await app.listen(3000);
}
bootstrap();
```

> **경고**
>
> 플랫폼에 내재되어 있는 제한으로 인해, NestJS는 Windows의 애플리케이션 종료 훅에 대한 지원을 제한합니다. `SIGINT`는 물론, `SIGBREAK`와 함께 `SIGHUP`도 어느 정도 작동할 수 있습니다 - [자세히 보기](https://nodejs.org/api/process.html#process_signal_events). 하지만 task manager에서 프로세스를 강제 종료(kill)하는 것은 무조건적이기 때문에 `SIGTERM`은 Windows에서 절대 작동하지 않습니다. 즉, "애플리케이션이 이를 감지하거나 방지할 수 있는 방법은 없습니다". `SIGINT`, `SIGBREAK` 그리고 다른 시그널들은 Windows에서 어떻게 처리하는지 libuv의 [관련 문서](https://docs.libuv.org/en/v1.x/signal.html)를 참고하세요. 더불어, Node.js의 [Process Signal Events](https://nodejs.org/api/process.html#process_signal_events) 문서도 참고하세요.

> **정보**
> 
> `enableShutdownHooks`는 리스너를 작동시키며 메모리를 소모합니다. 단일 Node 프로세스에서 Nest 앱을 복수로 실행할 경우(예: Jest로 병렬 테스트를 실행할 경우)에는, Node는 과도한 리스너 프로세스를 감당하지 못할 수 있습니다. 이러한 이유로, `enableShutdownHooks`는 기본적으로 비활성화 되어 있습니다. 단일 Node 프로세스에서 복수 인스턴스를 실행할 경우 이 점을 유의하세요.

애플리케이션이 종료 신호를 받으면 해당하는 신호를 첫 번째 매개변수로 하여 등록된 `onModuleDestroy()`, `beforeApplicationShutdown()`, 그리고 `onApplicationShutdown()` 함수(위에서 설명한 순서에 맞춰)를 호출합니다. 만약 등록된 함수가 비동기 호출(promise 반환)을 기다리면, Nest는 promise가 resolve 또는 reject 되기 전까지 시퀀스를 진행하지 않습니다.

```typescript
@@filename()
@Injectable()
class UsersService implements OnApplicationShutdown {
  onApplicationShutdown(signal: string) {
    console.log(signal); // e.g. "SIGINT"
  }
}
@@switch
@Injectable()
class UsersService implements OnApplicationShutdown {
  onApplicationShutdown(signal) {
    console.log(signal); // e.g. "SIGINT"
  }
}
```

> **정보**
> 
> `app.close()`를 호출하는 것은 Node 프로세스를 중지시키지 않고 `onModuleDestroy()`와 `onApplicationShutdown()` 훅을 작동시키기 때문에, 만약 오래 실행되고 있는 백그라운드 작업 등 인터벌이 있는 경우 프로세스가 자동으로 종료되지 않습니다.