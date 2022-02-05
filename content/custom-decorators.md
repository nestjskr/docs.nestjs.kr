### 사용자 정의 라우트 데코레이터

Nest는 **데코레이터**라는 언어 기능을 활용하여 구축됩니다. 데코레이터는 많은 프로그래밍 언어들에서는 잘 알려진 개념이지만, 자바스크립트 세계에서는 아직까지는 상대적으로 신기능입니다. 데코레이터가 어떻게 동작하는지 더 잘 이해하고 싶다면, [이 자료](https://medium.com/google-developers/exploring-es7-decorators-76ecb65fb841)를 읽어보기를 추천합니다. 데코레이터에 대해 간단하게 정의하자면 이렇습니다.

<blockquote class="external">
  ES2016 데코레이터는 특정 대상이나 이름, 프로퍼티 설명자를 인자로 받으며 함수를 반환하는 표현식입니다. 데코레이터를 적용할 때는 <code>@</code>문자를 데코레이터의 접두사로 붙이고, 적용할 대상의 바로 위에 위치시킵니다. 데코레이터는 클래스나 메서드, 프로퍼티를 적용 대상으로 삼을 수 있습니다.
</blockquote>

#### 매개변수 데코레이터

Nest는 HTTP 라우트 핸들러와 함께 유용하게 사용할 수 있는 유용한 **매개변수 데코레이터**들을 제공합니다. 다음은 제공되는 데코레이터들의 종류와 각 데코레이터들이 나타내는 Express(또는 Fastify) 객체입니다.

<table>
  <tbody>
    <tr>
      <td><code>@Request(), @Req()</code></td>
      <td><code>req</code></td>
    </tr>
    <tr>
      <td><code>@Response(), @Res()</code></td>
      <td><code>res</code></td>
    </tr>
    <tr>
      <td><code>@Next()</code></td>
      <td><code>next</code></td>
    </tr>
    <tr>
      <td><code>@Session()</code></td>
      <td><code>req.session</code></td>
    </tr>
    <tr>
      <td><code>@Param(param?: string)</code></td>
      <td><code>req.params</code> / <code>req.params[param]</code></td>
    </tr>
    <tr>
      <td><code>@Body(param?: string)</code></td>
      <td><code>req.body</code> / <code>req.body[param]</code></td>
    </tr>
    <tr>
      <td><code>@Query(param?: string)</code></td>
      <td><code>req.query</code> / <code>req.query[param]</code></td>
    </tr>
    <tr>
      <td><code>@Headers(param?: string)</code></td>
      <td><code>req.headers</code> / <code>req.headers[param]</code></td>
    </tr>
    <tr>
      <td><code>@Ip()</code></td>
      <td><code>req.ip</code></td>
    </tr>
    <tr>
      <td><code>@HostParam()</code></td>
      <td><code>req.hosts</code></td>
    </tr>
  </tbody>
</table>

추가적으로, **사용자 정의 데코레이터**를 작성할 수도 있습니다. 이게 왜 유용할까요?

node.js 세계에서는 **요청**객체에 프로퍼티들을 붙이는 경험을 흔하게 합니다. 그러고는 아래처럼 각 라우트 핸들러에서 코드를 통해 해당 프로퍼티들을 추출할 수 있습니다.

```typescript
const user = req.user;
```

코드의 가독성을 높이고 더욱 투명하게 만들기 위해, `@User()` 데코레이터를 작성하여 모든 컨트롤러에서 재사용할 수 있습니다.

```typescript
@@filename(user.decorator)
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

그러고나서 필요하다면 어디서든 간단하게 이 데코레이터를 사용할 수 있습니다.

```typescript
@@filename()
@Get()
async findOne(@User() user: UserEntity) {
  console.log(user);
}
@@switch
@Get()
@Bind(User())
async findOne(user) {
  console.log(user);
}
```

#### 데이터 넘겨주기

작성한 데코레이터가 특정 조건에 의존하여 동작해야 한다면, 데코레이터를 만드는 팩토리 함수에 넘겨주기 위한 `data` 매개변수를 사용할 수 있습니다. 사용 사례로는 요청 객체에서 특정 키로 프로퍼티를 가져오는 사용자 정의 데코레이터가 있습니다. 예를 들어, <a href="techniques/authentication#implementing-passport-strategies">인증 계층</a>에서는 요청 내용에 대해 유효성 검사를 실시한 후에 요청 객체 속에 사용자 정의 정보를 붙입니다. 인증에 성공한 사용자 정의 정보는 다음과 같을 것입니다:

```json
{
  "id": 101,
  "firstName": "Alan",
  "lastName": "Turing",
  "email": "alan@email.com",
  "roles": ["admin"]
}
```

이제 키를 통해 이 프로퍼티를 가져와서 값이 있다면 그 값을 반환하는 데코레이터를 작성해 봅시다. (`user`객체가 없거나 객체에 값이 없다면 undefined를 반환합니다.)

```typescript
@@filename(user.decorator)
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    return data ? user?.[data] : user;
  },
);
@@switch
import { createParamDecorator } from '@nestjs/common';

export const User = createParamDecorator((data, ctx) => {
  const request = ctx.switchToHttp().getRequest();
  const user = request.user;

  return data ? user && user[data] : user;
});
```

이제 컨트롤러에서 `@User()`데코레이터를 통해 특정 프로퍼티에 접근할 수 있습니다.

```typescript
@@filename()
@Get()
async findOne(@User('firstName') firstName: string) {
  console.log(`Hello ${firstName}`);
}
@@switch
@Get()
@Bind(User('firstName'))
async findOne(firstName) {
  console.log(`Hello ${firstName}`);
}
```

다른 프로퍼티에 접근하기 위해서는 똑같은 데코레이터에 다른 키를 사용하면 됩니다. 만약 `user`객체의 깊이가 깊거나 복잡하다면, 이 방법은 요청 핸들러를 더욱 쉽고 가독성 높게 구현하도록 만들어줄 수 있습니다.

> info **힌트** 타입스크립트 사용자 정의들은 `createParamDecorator<T>()`함수가 제네릭 타입을 취한다는 것을 염두해 두시길 바랍니다. 예를 들어 `createParamDecorator<string>((data, ctx) => ...)`와 같이 사용하여 명시적으로 Type Safety를 강제할 수 있습니다. 또한 `createParamDecorator((data: string, ctx) => ...)`와 같이 팩토리 함수의 매개변수 타입을 정할 수도 있습니다. 두 군데 모두 타입을 명시하지 않으면 `data`의 타입은 `any`가 됩니다.

#### 파이프와 함께 사용

Nest는 사용자 정의 파라미터 데코레이터를 이미 구현되어 있는 파라미터 데코레이터들(`@Body()`와 `@Param()`, `@Query()`)과 똑같은 방식으로 처리합니다. 즉, 파이프는 사용자 정의 데코레이터가 붙은 인자(아래 예제에서는 `user`)에 대해서도 실행됩니다. 게다가 사용자 정의 데코레이터에 직접적으로 파이프를 적용시킬 수도 있습니다:

```typescript
@@filename()
@Get()
async findOne(
  @User(new ValidationPipe({ validateCustomDecorators: true }))
  user: UserEntity,
) {
  console.log(user);
}
@@switch
@Get()
@Bind(User(new ValidationPipe({ validateCustomDecorators: true })))
async findOne(user) {
  console.log(user);
}
```

> info **힌트** `validateCustomDecorators` 옵션을 꼭 true로 지정해야 합니다. `ValidationPipe`는 기본적으로 사용자 정의 데코레이터가 붙은 인자에 대해 유효성 검사를 수행하지 않습니다.

#### 데코레이터 조합

Nest는 여러 데코레이터를 조합하는 헬퍼 메서드를 제공합니다. 예를 들어, 인증과 관련된 모든 데코레이터들을 모아 하나의 데코레이터로 사용하고 싶다면 다음과 같이 구성할 수 있습니다:

```typescript
@@filename(auth.decorator)
import { applyDecorators } from '@nestjs/common';

export function Auth(...roles: Role[]) {
  return applyDecorators(
    SetMetadata('roles', roles),
    UseGuards(AuthGuard, RolesGuard),
    ApiBearerAuth(),
    ApiUnauthorizedResponse({ description: 'Unauthorized' }),
  );
}
@@switch
import { applyDecorators } from '@nestjs/common';

export function Auth(...roles) {
  return applyDecorators(
    SetMetadata('roles', roles),
    UseGuards(AuthGuard, RolesGuard),
    ApiBearerAuth(),
    ApiUnauthorizedResponse({ description: 'Unauthorized' }),
  );
}
```

그 다음엔 아래와 같이 `@Auth()`라는 사용자 정의 데코레이터를 사용합니다:

```typescript
@Get('users')
@Auth('admin')
findAllUsers() {}
```

이렇게 하면 하나의 데코레이터로 네 개의 데코레이터를 모두 적용시킬 수 있는 효과를 얻을 수 있습니다.

> warning **주의** `@nestjs/swagger` 패키지의 `@ApiHideProperty()` 데코레이터는 조합할 수 없으며 `applyDecorators` 함수와 함께 사용하면 제대로 동작하지 않습니다.
