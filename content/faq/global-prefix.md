### Global prefix

HTTP 어플리케이션에 등록된 **모든 경로에** 접두사(prefix)를 붙이기 위해서는, `INestApplication` 인스턴스의 `setGlobalPrefix()` 메서드를 사용합니다.

```typescript
const app = await NestFactory.create(AppModule);
app.setGlobalPrefix('v1');
```

다음 구성을 사용해 전역 접두사(prefix)에서 경로를 제외할 수 있습니다.

```typescript
app.setGlobalPrefix('v1', {
  exclude: [{ path: 'health', method: RequestMethod.GET }],
});
```

대안으로, 경로를 문자열로 지정할 수도 있습니다 (모든 요청 메서드에 적용됩니다).

```typescript
app.setGlobalPrefix('v1', { exclude: ['cats'] });
```
