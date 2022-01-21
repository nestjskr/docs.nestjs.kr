### 문서화

**Compodoc**은 Angulaer 애플리케이션을 위한 문서화 도구입니다. Nest와 Angular는 유사한 프로젝트 코드 구조를 공유하므로 **Compodoc**은 Nest 애플리케이션에서도 작동합니다.

#### 설정

기존 Nest 프로젝트 내에서 Compodoc을 설정하는 것은 매우 간단합니다. OS 터미널에서 다음 명령을 사용하여 dev-dependency를 추가합니다.

```bash
$ npm i -D @compodoc/compodoc
```

#### Generation

Generate project documentation using the following command (npm 6 is required for `npx` support). See [the official documentation](https://compodoc.app/guides/usage.html) for more options.

```bash
$ npx @compodoc/compodoc -p tsconfig.json -s
```

Open your browser and navigate to [http://localhost:8080](http://localhost:8080). You should see an initial Nest CLI project:

<figure><img src="/assets/documentation-compodoc-1.jpg" /></figure>
<figure><img src="/assets/documentation-compodoc-2.jpg" /></figure>

#### Contribute

You can participate and contribute to the Compodoc project [here](https://github.com/compodoc/compodoc).
