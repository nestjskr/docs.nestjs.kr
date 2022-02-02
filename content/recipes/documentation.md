### 문서화

**Compodoc**은 Angular 애플리케이션을 위한 문서화 도구입니다. Nest와 Angular는 유사한 프로젝트 코드 구조를 공유하므로 **Compodoc**은 Nest 애플리케이션에서도 작동합니다.

#### 설정

기존 Nest 프로젝트 내에서 Compodoc을 설정하는 것은 매우 간단합니다. OS 터미널에서 다음 명령을 사용하여 dev-dependency를 추가합니다.

```bash
$ npm i -D @compodoc/compodoc
```

#### 생성

다음 명령을 사용하여 프로젝트 문서를 생성합니다('npx' 지원을 위해서는 npm 6 필요합니다). 더 많은 옵션은 [공식 문서](https://compodoc.app/guides/usage.html)를 참조하세요.

```bash
$ npx @compodoc/compodoc -p tsconfig.json -s
```

브라우저를 열고 [http://localhost:8080](http://localhost:8080)으로 이동합니다. 다음과 같은 초기 Nest CLI 프로젝트가 표시됩니다:

<figure><img src="/assets/documentation-compodoc-1.jpg" /></figure>
<figure><img src="/assets/documentation-compodoc-2.jpg" /></figure>

#### 기여

Compodoc 프로젝트는 [여기](https://github.com/compodoc/compodoc)에서 참여하고 기여할 수 있습니다.
