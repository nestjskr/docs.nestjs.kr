### 마이그레이션 가이드

`@nestjs/swagger@3.*` 를 사용하고 있다면, 버전 4.0 에서는 하단의 중요한 변화들이 있다는 점을 유념해야 합니다.

#### 중요한 변화들

다음의 decorator 들은 기능 혹은 이름이 변경되었습니다.

- `@ApiModelProperty` 는 `@ApiProperty` 로 이름이 변경되었습니다.
- `@ApiModelPropertyOptional` 은 `@ApiPropertyOptional` 로 이름이 변경되었습니다.
- `@ApiResponseModelProperty` 는 `@ApiResponseProperty` 로 이름이 변경되었습니다.
- `@ApiImplicitQuery` 는 `@ApiQuery` 로 이름이 변경되었습니다.
- `@ApiImplicitParam` 은 `@ApiParam` 로 이름이 변경되었습니다.
- `@ApiImplicitBody` 는 `@ApiBody` 로 이름이 변경되었습니다.
- `@ApiImplicitHeader` 는 `@ApiHeader` 로 이름이 변경되었습니다.
- `@ApiOperation({{ '{' }} title: 'test' {{ '}' }})` 은 `@ApiOperation({{ '{' }} summary: 'test' {{ '}' }})` 로 이름이 변경되었습니다.
- `@ApiUseTags` 는 `@ApiTags` 이름이 변경되었습니다.

`DocumentBuilder` 의 중요한 변화들 (변경된 method signature들):

- `addTag`
- `addBearerAuth`
- `addOAuth2`
- `setContactEmail` 은 `setContact` 로 이름이 변경되었습니다.
- `setHost` 는 삭제되었습니다.
- `setSchemes` 는 삭제되었습니다. (`addServer` 를 사용하십시오. 예를 들어, `addServer('http://')`와 같이 사용할 수 있습니다.)

#### 새로운 method들

다음의 method 들은 추가되었습니다.

- `addServer`
- `addApiKey`
- `addBasicAuth`
- `addSecurity`
- `addSecurityRequirements`

<app-banner-shop></app-banner-shop>
