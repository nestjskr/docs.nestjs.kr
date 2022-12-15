### 큐

큐는 일반적인 어플리케이션 확장 및 성능 문제를 처리하는 데 도움이 되는 강력한 디자인 패턴입니다. 큐가 해결할 수 있는 문제는 다음과 같습니다:

- 프로세스 피크를 완화할 수 있습니다. 예를 들어, 유저는 임의의 시간에 리소스 집약적인 작업을 시작할 수 있고 작업을 동기로 수행하는 대신 큐에 작업을 추가할 수 있습니다. 그러면 워커가 큐에서 작업을 꺼내 처리를 합니다. 어플리케이션이 확장됨에 따라 백엔드 작업 처리를 확장하기 위해 새 큐 컨슈머를 쉽게 추가할 수 있습니다.
- Node.js의 이벤트 루프를 차단할 수 있는 모놀리틱 태스크를 분리합니다. 예를 들어, 사용자 요청이 오디오 트랜스코딩과 같은 CPU 집약적 작업이 필요한 경우, 이 작업을 다른 프로세스에 위임할 수 있습니다. 반응형 상태를 유지하기 위해 사용자 대면 프로세스를 확보합니다. 
- 다양한 서비스에 걸쳐 안정적인 커뮤니케이션 채널을 제공합니다. 예를 들어, 한 프로세스나 서비스에서 태스크(작업)을 큐에 넣고 다른 프로세스나 서비스에서 사용할 수 있습니다. 모든 프로세스 또는 서비스에서 잡의 수명주기 완료, 오류 또는 기타 상태 변경 시 상태 이벤를 수신해 알림을 받을 수 있습니다. 큐 프로듀서 또는 컨슈머가 실패하면, 상태는 유지되고 노드가 재시작할 때 태스크 핸들링이 자동으로 재개됩니다.

네스트는 `@nestjs/bull` 패키지를 제공합니다. 이 패키지는 인기 많고 충분한 서포트를 받고 있는 고성능 Node.js 기반 대기열 시스템의 구현인 [불](https://github.com/OptimalBits/bull) 의 추상화/래퍼 입니다. 패키지를 사용하면 어플리케이션에 네스트 친화적인 방식으로 불 큐를 쉽게 통합할 수 있습니다.

불은 작업 데이터를 유지하는데 [레디스](https://redis.io/) 사용하기 때문에 시스템에 레디스가 설치되어 있어야 합니다. 레디스를 백엔드로 사용하기 때문에, 큐 아키텍쳐는 완전히 분산되고 플랫폼에 독립적일 수 있습니다. 예를 들어, 몇개의 큐가 <a href="techniques/queues#producers">프로듀서</a> 와 <a href="techniques/queues#consumers">컨슈머</a> 그리고 <a href="techniques/queues#event-listeners">리스너</a> 가 네스트의 한 (또는 여러개) 노드에서 실행중일 때, 다른 프로듀서, 컨슈머 그리고 리스너가 다른 Node.js 플랫폼 또는 다른 네트워크 노드에서 실행할 수 있습니다.

이 챕터는 `@nestjs/bull` 패키지에 대해 설명합니다. 자세한 배경 및 구체적인 구현 세부 정보를 얻고 싶으면 [불 공식문서](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md) 를 권장합니다.

#### 설치

먼저 필요한 패키지를 설치합니다.

```bash
$ npm install --save @nestjs/bull bull
$ npm install --save-dev @types/bull
```

설치가 완료되면, 루트 `AppModule` 에 `BullModule` 를 임포트 할 수 있습니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { BullModule } from '@nestjs/bull';

@Module({
  imports: [
    BullModule.forRoot({
      redis: {
        host: 'localhost',
        port: 6379,
      },
    }),
  ],
})
export class AppModule {}
```

`forRoot()` 메소드는 `bull` 패키지 구성 옵션을 등록하는데 사용하며, 특별한 명시가 없는 한 어플리케이션에 등록된 모든 큐에서 사용됩니다. 구성 객체는 다음 속성으로 구성됩니다:

- `limiter: RateLimiter` - 대기열의 작업이 처리되는 속도를 제어하는 옵션입니다. 자세한 내용은 [RateLimiter](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queue) 참조하세요. 선택 옵션.
- `redis: RedisOpts` - 레디스 연결을 구성하는 옵션. 자세한 내용은 [RedisOpts](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queue) 참조하세요. 선택 옵션.
- `prefix: string` - 모든 큐 키의 프리픽스. 선택 옵션.
- `defaultJobOptions: JobOpts` - 새 작업의 기본 설정을 제어하는 옵션. 자세한 내용은 [JobOpts](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queueadd) 참조하세요. 선택 옵션.
- `settings: AdvancedSettings` - 고급 큐 구성 설정. 일반적으로는 이 설정을 변경하면 안됩니다. 자세한 내용은 [AdvancedSettings](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queue) 참조사헤요. 선택 옵션.

모든 옵션은 선택 사항이며, 대기열 동작을 세부적으로 제어할 수 있습니다. 이 옵션들은 불 `Queue` 생성자에 직접 전달됩니다. 옵션들에 대애서는 [여기](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queue) 더 알아볼 수 있습니다.

큐를 등록하려면, 아래와 같이 다이나믹 모듈에 `BullModule#registerQueue()` 임포트 합니다:

```typescript
BullModule.registerQueue({
  name: 'audio',
});
```

> 정보 **힌트** `registerQueue()` 메소드에 콤마로 구분된 여러개의 구성 객체를 전달해 여러개의 큐를 생성합니다. 

`registerQueue()` 메소드는 큐를 인스턴스화 하거나 등록하는 데 사용합니다. 큐는 동일한 크레덴셜을 사용해 레디스에 연결하는 모듈 및 프로세스 간에 공유됩니다. 각 큐는 이름 속성에 따라 고유합니다. 큐의 이름 주입 토큰(컨트롤러/프로바이더에 큐를 주입)과 컨슈머 클래스 그리고 리스너를 큐와 연결하는 데코레이터의 인수로 사용합니다.

다음과 같이 특정 대기열에 대해 미리 구성된 일부 옵션을 오버라이드 할 수도 있습니다:

```typescript
BullModule.registerQueue({
  name: 'audio',
  redis: {
    port: 6380,
  },
});
```

잡은 레디스에서 지속되기 때문에 특정 이름으로 명명된 큐가 인스턴스화 될 때 마다(예: 앱이 시작/재시작), 이전의 완료되지 않은 세션에 남이있을 수 있는 오래된 잡을 진행하려고 시도합니다.

각 큐는 한개 또는 여러개의 프로듀서, 컨슈머를 가질 수 있습니다. 컨슈머는 큐에서 특정 순서로 잡을 갖고옵니다: FIFO (기본), LIFO, 또는 우선 순위. 큐의 작업 순서에 대해서는 <a href="techniques/queues#consumers">여기</a>에서 더 알아볼 수 있습니다.

<app-banner-enterprise></app-banner-enterprise>

#### 명명된 구성

큐가 여러개의 각기 다른 레디스 인스턴스와 연결되어 있는 경우, **명명된 구성**이란 테크닉을 사용할 수 있습니다. 이 기능을 사용하면 지정된 키 아래에 여러 구성을 등록할 수 있으며, 큐 옵션에서 참조할 수 있습니다.

예를 들어, 추가 레디스 인스턴스가 있다고 가정하고 (기본 인스턴스 제외) 어플리케이션에 등록된 몇 개의 큐가 사용하고 있다면, 아래와 같이 구성에 등록할 수 있습니다: 

```typescript
BullModule.forRoot('alternative-config', {
  redis: {
    port: 6381,
  },
});
```

위의 에시에서, `'alternative-config'` 는 구성 키 입니다. (임의의 문자열일 수 있음).

이제 `registerQueue()` 옵션 객체에서 이 구성을 가리킬 수 있습니다:

```typescript
BullModule.registerQueue({
  configKey: 'alternative-queue'
  name: 'video',
});
```

#### 프로듀서

잡 프로듀서는 큐에 잡을 추가합니다. 프로듀서는 일반적으로 어플리케이션 서비스입니다. (네스트 [프로바이더](/providers)). 큐에 잡을 추가하려면, 아래와 같이 서비스에 큐를 주입합니다:

```typescript
import { Injectable } from '@nestjs/common';
import { Queue } from 'bull';
import { InjectQueue } from '@nestjs/bull';

@Injectable()
export class AudioService {
  constructor(@InjectQueue('audio') private audioQueue: Queue) {}
}
```

> 정보 **힌트** `@InjectQueue()` 데코레이터는 `registerQueue()` 메소드 호출시 제공된 이름으로 큐를 식별합니다 (예., `'audio'`).

큐의 `add()` 메소드를 호출해 잡을 추가하고, 유저가 정의한 잡 객체를 전달합니다. 잡은 직렬화 가능한 자바스크립트 객체입니다. (레디스에 저장되는 형태). 잡의 모양은 임의적이며, 잡 객체의 의미를 나타내는 데 사용합니다.

```typescript
const job = await this.audioQueue.add({
  foo: 'bar',
});
```

#### 명명된 잡

잡은 고유한 이름을 가질 수 있습니다. 이를 통해 지정된 이름의 작업만 처리하는 특수한 <a href="techniques/queues#consumers">컨슈머</a> 를 만들 수 있습니다.

```typescript
const job = await this.audioQueue.add('transcode', {
  foo: 'bar',
});
```

> 경고 **경고** 명명된 잡을 사용할 때는 큐에 추가된 각각의 고유한 이름을 위한 프로세스를 생성해야 합니다. 그렇지 않으면 큐에서는 주어진 잡을 위한 프로세스가 없다고 할 것입니다. <a href="techniques/queues#consumers">여기</a> 에서 더 자세한 정보를 읽어보세요. 

#### 잡 옵션

잡에는 옵션을 추가할 수 있습니다. `Queue.add()` 메소드에서 `job` 인수 뒤에 옵션 객체를 전달합니다. 잡 옵션 속성은 다음과 같습니다.

- `priority`: `number` - 선택적 우선 순위 값. 범위는 1(가장 높은 우선 순위) 부터 MAX_INT(가장 낮은 우선 순위)까지 입니다. 우선 순위를 사용하면 성능에 약간 영향을 미치므로 사용할 때 주의해야 합니다. 
- `delay`: `number` - 이 작업을 처리할 수 있을 때 까지 대기하는 시간(밀리초) 입니다. 정확한 지연을 위해 서버와 클라이언트 모두 시게를 동기화해야 합니다.
- `attempts`: `number` - 잡이 완료될 때 까지 시도할 총 시도 횟수입니다.
- `repeat`: `RepeatOpts` - 크론의 사양에 따라 작업을 반복합니다. [RepeatOpts](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queueadd).
- `backoff`: `number | BackoffOpts` - 잡이 실패할 경우 자동으로 재시도 하기 위한 백오프 설정입니다. [BackoffOpts](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queueadd).
- `lifo`: `boolean` - true일 경우, 큐의 왼쪽 대신 오른쪽 끝에 잡을 추가합니다. (기본값은 false).
- `timeout`: `number` - 시간 초과 오류와 함께 작업이 실패해야 하는 시간(밀리초) 입니다. 
- `jobId`: `number` | `string` - 잡 ID 오버라이드 - 기본적으로 잡 ID 는 고유합니다.
   이 설정을 오버라이드 해서 integer를 사용할 수 있습니다. 이 옵션을 사용하고 싶으면, 잡 id가 고유해야 합니다. 이미 존재한느 id를 사용애 잡을 추가하려 시도하면 추가가 되지 않습니다. 
- `removeOnComplete`: `boolean | number` - true일 경우, 성공적으로 완료한 잡을 삭제합니다. 유지할 잡의 양을 숫자로 지정합니다. 기본 동작은 작업을 완료된 세트로 유지합니다.
- `removeOnFail`: `boolean | number` - true일 경우, 시도 후에 실패한 작업을 제거합니다. 유지할 작업의 양을 숫자로 지정합니다. 기본 동작은 작업을 실패한 세트로 유지하는 것입니다. 
- `stackTraceLimit`: `number` - 스택트레이스에 기록할 스택트레이스 라인의 수를 제한합니다.

다음은 잡 옵션을 사용자 지정하는 몇 가지 예제입니다.

작업 시작을 지연하려면 `delay` 구성 속성을 사용합니다.

```typescript
const job = await this.audioQueue.add(
  {
    foo: 'bar',
  },
  { delay: 3000 }, // 3 seconds delayed
);
```

큐의 오른쪽 끝에 잡ㅇ르 추가하려면 (잡을 **LIFO** (후입선출)로 처리), 구성 객체의 `lifo` 속성을 `true`로 설정합니다.

```typescript
const job = await this.audioQueue.add(
  {
    foo: 'bar',
  },
  { lifo: true },
);
```

작업의 우선 순위를 지정하려면 `priority` 속성을 사용합니다.

```typescript
const job = await this.audioQueue.add(
  {
    foo: 'bar',
  },
  { priority: 2 },
);
```

#### 컨슈머

컨슈머는 **클래스** 로 큐에 추가된 작업을 처리하는 메소드를 정의하거나, 큐에서 이벤트 수신을 대기하거나, 둘 다를 수행 합니다. 다음과 같이 `@Processor()` 데코레이터를 사용해서 컨슈머를 선언합니다:

```typescript
import { Processor } from '@nestjs/bull';

@Processor('audio')
export class AudioConsumer {}
```

> 정보 **힌트** 컨슈머는 `providers` 로 등록해야 `@nestjs/bull` 패키지가 사용할 수 있습니다.

여기서 데코레이터의 문자열 인수 (예: `'audio'`) 는 클래스 메소드에 연결할 큐의 이름입니다.

컨슈머 클래스 내부에서는 `@Process()` 데코레이터를 핸들러 메소드에 사용해 잡 핸들러를 선언할 수 있습니다.

```typescript
import { Processor, Process } from '@nestjs/bull';
import { Job } from 'bull';

@Processor('audio')
export class AudioConsumer {
  @Process()
  async transcode(job: Job<unknown>) {
    let progress = 0;
    for (i = 0; i < 100; i++) {
      await doSomething(job.data);
      progress += 10;
      await job.progress(progress);
    }
    return {};
  }
}
```

데코레이터 메소드 (예: `transcode()`) 는 워커가 유휴상태이고 큐에 처리할 잡이 있을때 호출됩니다. 이 핸들러 메소드는 `job` 객체를 유일한 인수로 받습니다. 핸들러 메소드에서 반환한 값은 잡 객체에 저장되며, 나중에, 예를 들면 완료된 이벤트에 대한 리스너에서 접근할 수 있습니다.

`Job` 객체는 상태와 상호작용 할 수 있게 하는 여러 메소드가 있습니다. 예를 들어, 위의 코드는 `progress()` 메소드를 사용해서 잡의 진행상태를 업데이트 합니다. [여기](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#job) 에서 `Job` 객체의 API 레퍼런스를 볼 수 있습니다.

아래와 같이 `name` 을 `@Process()` 데코레이터에 전달해 잡 핸들러 메서드가 **오직** 특정 유형의 잡만 (특정 `name`의 잡)을 처리하도록 지정할 수 있습니다. 주어진 컨슈머 클래스에 각 잡 유형 (`name`)에 해당하는 여러 `@Process()` 핸들러를 가질 수 있습니다. 명명된 잡을 사용할 때, 각 이름에 해당하는 핸들러가 있어야 합니다.

```typescript
@Process('transcode')
async transcode(job: Job<unknown>) { ... }
```

#### 리퀘스트 스쿠프 컨슈머

컨슈머가 리퀘스트 스푸크로 지정될 경우 (스쿠프 주입에 대해서 더 알아보려면 [여기](/fundamentals/injection-scopes#provider-scope)를 참조하세요), 클래스의 새 인스턴스는 각 잡에 독점적으로 생성됩니다. 인스턴스는 잡이 완료된 후 가비지 콜렉팅 됩니다.

```typescript
@Processor({
  name: 'audio',
  scope: Scope.REQUEST,
})
```

리퀘스트 스코프 컨슈머 클래스는 동적으로 인스턴스화 되고, 단일 작업으로 범위가 지정되므로 표준 접근 방식을 사용해 생성자를 통해 `JOB_REF`를 주입할 수 있습니다.

```typescript
constructor(@Inject(JOB_REF) jobRef: Job) {
  console.log(jobRef);
}
```

> 정보 **힌트** `JOB_REF` 토큰은 `@nestjs/bull` 패키지에서 임포트 합니다.

#### 이벤트 리스너

불은 큐 또는 작업 상태 변경이 발생할 때 유용한 이벤트 집합을 생성합니다. 네스트는 표준 이벤트의 핵심 세트를 구독할 수 있는 데코레이터 세트를 제공합니다. 이 모든 것은 `@nestjs/bull` 패키지에서 익스포트 합니다.

이벤트 리스너는 반드시 <a href="techniques/queues#consumers">컨슈머</a> 클래스 내부에 선언되어야 합니다. (예: `@Processor()` 데코레이더를 사용하는 클래스 내). 이벤트를 구독하려면, 아래 테이블에 있는 데코레이터 중 하나를 사용해 이벤트를 위한 핸들러를 선언합니다. 예를 들어, `audio` 큐에서 잡이 활성상태가 될 때 발생하는 이벤트를 수신하려면 다음과 같은 구성을 사용합니다:

```typescript
import { Processor, Process } from '@nestjs/bull';
import { Job } from 'bull';

@Processor('audio')
export class AudioConsumer {

  @OnQueueActive()
  onActive(job: Job) {
    console.log(
      `Processing job ${job.id} of type ${job.name} with data ${job.data}...`,
    );
  }
  ...
```

불이 분산 (멀티 노드) 환경에서 작동하기 때문에, 이벤트에 로컬 개념을 정의합니다. 이 개념은 이벤트가 단일 프로세스 내에서만 트리거 되거나, 다른 프로세스의 공유된 큐에서 트리거 될 수 있음을 인식합니다. **로컬** 이벤트는 로컬 프로세스의 큐에서 작업 또는 상태 변경이 트리거 될 때 생성되는 것입니다. 즉, 이벤트 프로듀서와 컨슈머가 단일 프로세스 로컬인 경우 큐에서 발생하는 모든 이벤트는 로컬입니다. 

큐가 여러 프로세스에서 공유되면, **글로벌** 이벤트가 발생할 가능성이 있습니다. 한 프로세스의 리스너가 다른 프로세스에 의해 트리거된 이벤트 알림을 수신하려면 전역 이벤트로 등록해야 합니다.

이벤트 핸들러는 해당 이벤트가 발생할 때마다 호출됩니다. 핸들러는 아래 테이블에 나와있는 시그니처로 호출 할 수 있으며, 이벤트와 관련된 정보에 접근할 수 있게 합니다. 아래에서 로컬 이벤트 핸들러 시그니처와 글로벌 이벤트 시그니처 간의 한가지 중요한 차이점에 대해 설명합니다. 

<table>
  <tr>
    <th>로컬 이벤트 리스터</th>
    <th>글로벌 이벤트 리스너</th>
    <th>핸들러 메소드 시그니처 / 호출될 때</th>
  </tr>
  <tr>
    <td><code>@OnQueueError()</code></td><td><code>@OnGlobalQueueError()</code></td><td><code>handler(error: Error)</code> - 에러 발생. <code>error</code> 는 트리거 에러도 포함.</td>
  </tr>
  <tr>
    <td><code>@OnQueueWaiting()</code></td><td><code>@OnGlobalQueueWaiting()</code></td><td><code>handler(jobId: number | string)</code> - 워커가 유휴상태인 즉시 잡을 처리하기 위해 대기중. <code>jobId</code> 는 잡의 id와 상태를 담고있음.</td>
  </tr>
  <tr>
    <td><code>@OnQueueActive()</code></td><td><code>@OnGlobalQueueActive()</code></td><td><code>handler(job: Job)</code> - 잡 <code>job</code>이 시작됨. </td>
  </tr>
  <tr>
    <td><code>@OnQueueStalled()</code></td><td><code>@OnGlobalQueueStalled()</code></td><td><code>handler(job: Job)</code> - 잡 <code>job</code> 이 정지된 것으로 표시됨. 이벤트 루프가 충돌하거나 중단되어 잡 워커를 디버깅 할 때 유용함.</td>
  </tr>
  <tr>
    <td><code>@OnQueueProgress()</code></td><td><code>@OnGlobalQueueProgress()</code></td><td><code>handler(job: Job, progress: number)</code> - 잡 <code>job</code> 의 진행상황이 <code>progress</code> 값에 업데이트.</td>
  </tr>
  <tr>
    <td><code>@OnQueueCompleted()</code></td><td><code>@OnGlobalQueueCompleted()</code></td><td><code>handler(job: Job, result: any)</code> 잡 <code>job</code> 이 결과 <code>result</code>와 함께 성공적으로 작업 완료.</td>
  </tr>
  <tr>
    <td><code>@OnQueueFailed()</code></td><td><code>@OnGlobalQueueFailed()</code></td><td><code>handler(job: Job, err: Error)</code> 잡 <code>job</code> 이 <code>err</code> 를 이유로 실패.</td>
  </tr>
  <tr>
    <td><code>@OnQueuePaused()</code></td><td><code>@OnGlobalQueuePaused()</code></td><td><code>handler()</code> 큐가 일시 중지됨.</td>
  </tr>
  <tr>
    <td><code>@OnQueueResumed()</code></td><td><code>@OnGlobalQueueResumed()</code></td><td><code>handler(job: Job)</code> 큐가 재개됨.</td>
  </tr>
  <tr>
    <td><code>@OnQueueCleaned()</code></td><td><code>@OnGlobalQueueCleaned()</code></td><td><code>handler(jobs: Job[], type: string)</code> 오래된 잡이 큐에서 제거됨. <code>jobs</code> 제거된 잡의 어레이 형태. <code>type</code> 은 제거된 잡의 타임.</td>
  </tr>
  <tr>
    <td><code>@OnQueueDrained()</code></td><td><code>@OnGlobalQueueDrained()</code></td><td><code>handler()</code> 큐가 모든 대기 잡을 처리할 때 발생 (몇몇의 지연된 잡이 아직 처리되지 않았을 때도 발생).</td>
  </tr>
  <tr>
    <td><code>@OnQueueRemoved()</code></td><td><code>@OnGlobalQueueRemoved()</code></td><td><code>handler(job: Job)</code> 잡 <code>job</code> 이 성공적으로 제거됨.</td>
  </tr>
</table>

글로벌 이벤트를 구독할때는, 메소드 시그니처가 로컬과는 약간 다를 수 있습니다. 특히, 메소드 시그니처는 로컬 버전일 때 `job` 객체를 수신하지만 글로벌 버전일 때 `jobId` (`number`) 를 수신받습니다. 이러한 경우 실제 `job` 객체의 참조를 얻으려면, `Queue#getJob` 메소드를 사용합니다. 이 호출은 대기해야 하기 때문에 핸들러는 `async` 로 선언되어야 합니다. 예:

```typescript
@OnGlobalQueueCompleted()
async onGlobalCompleted(jobId: number, result: any) {
  const job = await this.immediateQueue.getJob(jobId);
  console.log('(Global) on completed: job ', job.id, ' -> result: ', result);
}
```

> 정보 **힌트**  `Queue` 객체에 접근하려면 (`getJob()` 호출하려면), 당연히 주입해야 합니다. 또한 큐는 주입하려는 모듈에 등록되어 있어야 합니다.

특정 이벤트 리스너 데코레이터 외에도, 제네릭 `@OnQueueEvent()` 데코레이터를 in combination with  `BullQueueEvents` 또는 `BullQueueGlobalEvents` 이넘과 함께 사용할 수 있습니다. 이벤트에 대한 자세한 내용은 [여기](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#events) 를 참조하세요.

#### 큐 관리

큐에는 일시 중지 및 재개, 다양한 상태의 잡 수 검색 등과 같이 관리 기능을 수행하는 API가 있습니다. 모든 큐 API [여기](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queue) 에서 볼 수 있습니다. `Queue` 객체에서 이러한 메소드를 직접 호출해, 아래와 같이 일시 중지/재개를 할 수 있습니다. 

`pause()` 메소드를 호출해 큐를 일시 중지 합니다. 일시 중지된 큐는 재개되지 전에는 잡을 실행하지 않지만, 실행 중이던 잡은 해당 잡이 완료될 때 까지 실행합니다.

```typescript
await audioQueue.pause();
```

일시 중지된 큐를 재개하려면, 아래와 같이 `resume()` 메소드를 사용합니다:

```typescript
await audioQueue.resume();
```

#### 별도의 프로세스

잡 핸들러는 별도의(포크)된 프로세스에서 실행될 수 있습니다. ([소스](https://github.com/OptimalBits/bull#separate-processes)). 이 방법에는 몇 가지 장점이 있습니다:

- 프로세스가 샌드박스 처리되어, 충돌이 발생해도 워커에 영향을 미치지 않음. 
- 큐에 영향을 주지 않고 차단 코드를 실행 가능(작업이 중단되지 않음).
- 더 높은h 멀티 코어 CPU 활용도.
- 적은 레디스 연결.

```ts
@@filename(app.module)
import { Module } from '@nestjs/common';
import { BullModule } from '@nestjs/bull';
import { join } from 'path';

@Module({
  imports: [
    BullModule.registerQueue({
      name: 'audio',
      processors: [join(__dirname, 'processor.js')],
    }),
  ],
})
export class AppModule {}
```

펑션이 포크된 프로세스에서 실행되고 있기 때문에, 종속성 주입 Dependency Injection (그리고 IoC 컨테이너) 를 사용할 수 없습니다. 즉, 프로세서 기능은 필요한 외부 종속성의 모든 인스턴스를 포함(또는 생성) 해야 합니다.

```ts
@@filename(processor)
import { Job, DoneCallback } from 'bull';

export default function (job: Job, cb: DoneCallback) {
  console.log(`[${process.pid}] ${JSON.stringify(job.data)}`);
  cb(null, 'It works');
}
```

#### 비동기 구성

`bull` 옵션을 정적으로 전달하는 대신 비동기적으로 전달할 수 있습니다. 이런 경우, 비동기 구성을 처리하는 여러가지 방법을 제공하는 `forRootAsync()` 메소드를 사용합니다. 이와 비슷하게, `registerQueueAsync()` 메소드를 사용하면 큐에 옵션을 비동기로 전달할 수 있습니다.

한 가지 접근 방식은 팩토리 펑션을 사용입니다:

```typescript
BullModule.forRootAsync({
  useFactory: () => ({
    redis: {
      host: 'localhost',
      port: 6379,
    },
  }),
});
```

이 팩토리는 다른 [비동기 프로바이더](https://docs.nestjs.com/fundamentals/async-providers) (예: `async` 일 수 있고 `inject` 를 사용해 종속성을 주입할 수 있음.) 처럼 작동합니다.

```typescript
BullModule.forRootAsync({
  imports: [ConfigModule],
  useFactory: async (configService: ConfigService) => ({
    redis: {
      host: configService.get('QUEUE_HOST'),
      port: +configService.get('QUEUE_PORT'),
    },
  }),
  inject: [ConfigService],
});
```

또는 `useClass` 구문을 사용할 수 있습니다:

```typescript
BullModule.forRootAsync({
  useClass: BullConfigService,
});
```

위의 구성은 `BullConfigService` 내부에서 `BullModule` 를 인스턴스화 하고 `createSharedConfiguration()` 을 호출해 옵션 객체를 제공하는데 사용합니다. 아래와 같이 `BullConfigService` 가 `SharedBullConfigurationFactory` 인터페이스를 구현해야 함을 의미합니다:

```typescript
@Injectable()
class BullConfigService implements SharedBullConfigurationFactory {
  createSharedConfiguration(): BullModuleOptions {
    return {
      redis: {
        host: 'localhost',
        port: 6379,
      },
    };
  }
}
```

`BullConfigService` 내부에서 `BullModule` 가 생성되는 걸 방지하고 다른 모듈에서 임포트한 프로바이더를 사용하려면 `useExisting` 구문을 사용합니다.

```typescript
BullModule.forRootAsync({
  imports: [ConfigModule],
  useExisting: ConfigService,
});
```

이 구성은 `useClass` 와 동일하게 작동하지만 단 한가지 큰 차이점이 있습니다. `BullModule` 는 새 객체를 생성하는 대신 기존의 `ConfigService` 객체를 재사용 하기 위해 객체를 조회합니다.

#### 예제

예시는 [여기](https://github.com/nestjs/nest/tree/master/sample/26-queues) 에서 볼 수 있습니다.
