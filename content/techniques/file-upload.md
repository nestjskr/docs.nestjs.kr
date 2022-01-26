### 파일 업로드

파일 업로드를 처리하기 위해, 네스트는 익스프레스의 미들웨어인 [multer](https://github.com/expressjs/multer) 에 기반한 빌트인 모듈을 제공합니다. Multer는 HTTP POST 요청을 통해 파일을 업로드 할 때 자주 사용되는 `multipart/form-data` 형식으로 전송된 데이터를 다룹니다. 이 모듈은 사용자의 어플리케이션에 맞게 옵션을 설정하여 사용할 수 있습니다.

> warning **주의** Multer는 (`multipart/form-data`) 형식을 지원하지 않는 데이터를 처리할 수 없습니다. 또한, 이 패키지는 `FastifyAdapter`와 호환이 되지 않는 것을 명심하세요.

더 나은 타입 안정성을 위해, Multer 타입 패키지를 설치합니다:

```shell
$ npm i -D @types/multer
```

이 패키지를 설치함으로써, 우리는 `Express.Multer.File` 타입을 사용할 수 있습니다. (다음과 같이 타입을 임포트할 수 있습니다 `import {{ '{' }} Express {{ '}' }} from 'express'`).

#### 기본 예제

단일 파일을 업로드하기 위해, `FileInterceptor()` 인터셉터를 라우트 핸들러에 연결하고 `@UploadedFile()` 데코레이터를 이용하여 `request` 로 부터 `파일`을 추출합니다.

```typescript
@@filename()
@Post('upload')
@UseInterceptors(FileInterceptor('file'))
uploadFile(@UploadedFile() file: Express.Multer.File) {
  console.log(file);
}
@@switch
@Post('upload')
@UseInterceptors(FileInterceptor('file'))
@Bind(UploadedFile())
uploadFile(file) {
  console.log(file);
}
```

> info **힌트** `FileInterceptor()` 데코레이터는 `@nestjs/platform-express` 패키지에서 export 됩니다. The `@UploadedFile()` 데코레이터는 `@nestjs/common` 패키지에서 export 됩니다.

`FileInterceptor()` 데코레이터는 2개의 인수를 받습니다:

- `fieldName`: 파일이 있는 HTML form에서 제공하는 필드 이름
- `options`: (선택적) `MulterOptions` 타입의 객체입니다. multer 생성자에서 사용되는 것과 같은 객체입니다. (자세히 알아보기 [여기](https://github.com/expressjs/multer#multeropts)).

> warning **주의** `FileInterceptor()` 는 Google Firebase 등의 타사 클라우드 공급자와 호환되지 않을 수 있습니다.

#### 파일 배열

파일로 이루어진 배열을 업로드 하기 위해 (하나의 필드 이름으로 되어있는), `FilesInterceptor()` 데코레이터를 사용합니다. (데코레이터의 이름이 Files인 것을 확인하세요). 이 데코레이터는 3개의 인수를 받습니다:

- `fieldName`: 위의 설명과 동일합니다.
- `maxCount`: (선택적) 허용 할 최대 파일의 수
- `options`: (선택적) `MulterOptions` 객체, 위의 설명과 동일합니다.

`FilesInterceptor()` 를 사용할 때 , `@UploadedFiles()` 데코레이터를 이용하여 `request` 로 부터 파일들을 추출합니다 .

```typescript
@@filename()
@Post('upload')
@UseInterceptors(FilesInterceptor('files'))
uploadFile(@UploadedFiles() files: Array<Express.Multer.File>) {
  console.log(files);
}
@@switch
@Post('upload')
@UseInterceptors(FilesInterceptor('files'))
@Bind(UploadedFiles())
uploadFile(files) {
  console.log(files);
}
```

> info **힌트** `FilesInterceptor()` 데코레이터는 `@nestjs/platform-express` 패키지에서 export 됩니다. The `@UploadedFiles()` 데코레이터는 `@nestjs/common` 패키지에서 export 됩니다.

#### Multiple files

To upload multiple fields (all with different field name keys), use the `FileFieldsInterceptor()` decorator. This decorator takes two arguments:

- `uploadedFields`: an array of objects, where each object specifies a required `name` property with a string value specifying a field name, as described above, and an optional `maxCount` property, as described above
- `options`: optional `MulterOptions` object, as described above

When using `FileFieldsInterceptor()`, extract files from the `request` with the `@UploadedFiles()` decorator.

```typescript
@@filename()
@Post('upload')
@UseInterceptors(FileFieldsInterceptor([
  { name: 'avatar', maxCount: 1 },
  { name: 'background', maxCount: 1 },
]))
uploadFile(@UploadedFiles() files: { avatar?: Express.Multer.File[], background?: Express.Multer.File[] }) {
  console.log(files);
}
@@switch
@Post('upload')
@Bind(UploadedFiles())
@UseInterceptors(FileFieldsInterceptor([
  { name: 'avatar', maxCount: 1 },
  { name: 'background', maxCount: 1 },
]))
uploadFile(files) {
  console.log(files);
}
```

#### Any files

To upload all fields with arbitrary field name keys, use the `AnyFilesInterceptor()` decorator. This decorator can accept an optional `options` object as described above.

When using `AnyFilesInterceptor()`, extract files from the `request` with the `@UploadedFiles()` decorator.

```typescript
@@filename()
@Post('upload')
@UseInterceptors(AnyFilesInterceptor())
uploadFile(@UploadedFiles() files: Array<Express.Multer.File>) {
  console.log(files);
}
@@switch
@Post('upload')
@Bind(UploadedFiles())
@UseInterceptors(AnyFilesInterceptor())
uploadFile(files) {
  console.log(files);
}
```

#### Default options

You can specify multer options in the file interceptors as described above. To set default options, you can call the static `register()` method when you import the `MulterModule`, passing in supported options. You can use all options listed [here](https://github.com/expressjs/multer#multeropts).

```typescript
MulterModule.register({
  dest: './upload',
});
```

> info **Hint** The `MulterModule` class is exported from the `@nestjs/platform-express` package.

#### Async configuration

When you need to set `MulterModule` options asynchronously instead of statically, use the `registerAsync()` method. As with most dynamic modules, Nest provides several techniques to deal with async configuration.

One technique is to use a factory function:

```typescript
MulterModule.registerAsync({
  useFactory: () => ({
    dest: './upload',
  }),
});
```

Like other [factory providers](https://docs.nestjs.com/fundamentals/custom-providers#factory-providers-usefactory), our factory function can be `async` and can inject dependencies through `inject`.

```typescript
MulterModule.registerAsync({
  imports: [ConfigModule],
  useFactory: async (configService: ConfigService) => ({
    dest: configService.getString('MULTER_DEST'),
  }),
  inject: [ConfigService],
});
```

Alternatively, you can configure the `MulterModule` using a class instead of a factory, as shown below:

```typescript
MulterModule.registerAsync({
  useClass: MulterConfigService,
});
```

The construction above instantiates `MulterConfigService` inside `MulterModule`, using it to create the required options object. Note that in this example, the `MulterConfigService` has to implement the `MulterOptionsFactory` interface, as shown below. The `MulterModule` will call the `createMulterOptions()` method on the instantiated object of the supplied class.

```typescript
@Injectable()
class MulterConfigService implements MulterOptionsFactory {
  createMulterOptions(): MulterModuleOptions {
    return {
      dest: './upload',
    };
  }
}
```

If you want to reuse an existing options provider instead of creating a private copy inside the `MulterModule`, use the `useExisting` syntax.

```typescript
MulterModule.registerAsync({
  imports: [ConfigModule],
  useExisting: ConfigService,
});
```

#### Example

A working example is available [here](https://github.com/nestjs/nest/tree/master/sample/29-file-upload).
