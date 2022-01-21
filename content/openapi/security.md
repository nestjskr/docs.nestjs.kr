### 보안

특정 작업에 사용할 보안 메커니즘을 정의하려면 `@ApiSecurity()` 데코레이터를 사용하십시오.

```typescript
@ApiSecurity('basic')
@Controller('cats')
export class CatsController {}
```

어플리케이션을 실행하기 전에 `DocumentBuilder`를 사용하여 기본 문서에 보안 정의를 추가해야 합니다.

```typescript
const options = new DocumentBuilder().addSecurity('basic', {
  type: 'http',
  scheme: 'basic',
});
```

주로 사용되는 인증 방식 중 일부는 빌트인(예: `basic` 및 `bearer`)이므로 위와 같이 보안 메커니즘을 직접 정의할 필요가 없습니다.

#### Basic 인증

Basic 인증을 사용하려면 `@ApiBasicAuth()`를 사용하십시오.

```typescript
@ApiBasicAuth()
@Controller('cats')
export class CatsController {}
```

어플리케이션을 실행하기 전에 `DocumentBuilder`를 사용하여 기본 문서에 보안 정의를 추가해야 합니다.

```typescript
const options = new DocumentBuilder().addBasicAuth();
```

#### Bearer 인증

Bearer 인증을 사용하려면, `@ApiBearerAuth()`를 사용하십시오.

```typescript
@ApiBearerAuth()
@Controller('cats')
export class CatsController {}
```

어플리케이션을 실행하기 전에 `DocumentBuilder`를 사용하여 기본 문서에 보안 정의를 추가해야 합니다.

```typescript
const options = new DocumentBuilder().addBearerAuth();
```

#### OAuth2 인증

OAuth2를 사용하려면, `@ApiOAuth2()`를 사용하십시오.

```typescript
@ApiOAuth2(['pets:write'])
@Controller('cats')
export class CatsController {}
```

어플리케이션을 실행하기 전에 `DocumentBuilder`를 사용하여 기본 문서에 보안 정의를 추가해야 합니다.

```typescript
const options = new DocumentBuilder().addOAuth2();
```

#### Cookie 인증

Cookie 인증을 사용하려면, `@ApiCookieAuth()`를 사용하십시오.

```typescript
@ApiCookieAuth()
@Controller('cats')
export class CatsController {}
```

어플리케이션을 실행하기 전에 `DocumentBuilder`를 사용하여 기본 문서에 보안 정의를 추가해야 합니다.

```typescript
const options = new DocumentBuilder().addCookieAuth('optional-session-id');
```
