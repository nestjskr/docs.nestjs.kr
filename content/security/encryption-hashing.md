### 암호화와 해싱

**암호화**는 정보를 인코딩하는 작업입니다. 이 작업은 평문이라 불리는 정보의 원래 표현을 암호문이라 불리는 또다른 형태로 변환합니다. 이상적으로는 승인된 당사자만이 암호문을 다시 평문으로 해독하여 원본 정보에 액세스할 수 있습니다. 암호화는 그 자체로 간섭을 막지는 못하지만, 잠재적인 인터셉터가 이해할 수 있는 내용은 거부합니다. 암호화는 양방향 기능입니다. 이는 암호화된 내용을 적절한 키를 사용하여 복호화할 수 있다는 것입니다.

**해싱**은 주어진 키를 다른 값으로 변환하는 작업입니다. 이 때 수학적인 알고리즘에 따라 새로운 값을 생성하는 역할을 하는 것이 해시 함수입니다. 해싱되고 나면 그 결과를 가지고 다시 입력값을 구해낼 수 없습니다.

#### 암호화

Node.js는 문자열, 숫자, 버퍼, 스트림 등을 암호화 및 복호화할 수 있는 [crypto 모듈](https://nodejs.org/api/crypto.html)을 자체적으로 제공합니다. Nest 자체는 불필요한 추상화를 피하기 위해 이 모듈 위에서 동작하는 추가적인 패키지를 제공하지는 않습니다.

예를 들어, AES (Advanced Encryption System) 알고리즘의 CTR 암호화 모드인 `'aes-256-ctr'`을 사용해 보겠습니다.

```typescript
import { createCipheriv, randomBytes, scrypt } from "crypto";
import { promisify } from "util";

const iv = randomBytes(16);
const password = "Password used to generate key";

// The key length is dependent on the algorithm.
// In this case for aes256, it is 32 bytes.
const key = (await promisify(scrypt)(password, "salt", 32)) as Buffer;
const cipher = createCipheriv("aes-256-ctr", key, iv);

const textToEncrypt = "Nest";
const encryptedText = Buffer.concat([
  cipher.update(textToEncrypt),
  cipher.final(),
]);
```

이제 `encryptedText`값을 복호화 해보겠습니다:

```typescript
import { createDecipheriv } from "crypto";

const decipher = createDecipheriv("aes-256-ctr", key, iv);
const decryptedText = Buffer.concat([
  decipher.update(encryptedText),
  decipher.final(),
]);
```

#### 해싱

해생을 위해서는 [bcrypt](https://www.npmjs.com/package/bcrypt) 패키지나 [argon2](https://www.npmjs.com/package/argon2) 패키지를 사용하기를 권장합니다. Nest 자체는 불필요한 추상화를 피하기 위해 (러닝커브를 완만하게 만들기 위해) 이 모듈들 위에 추가 래퍼를 제공하지 않습니다.

예를 들어, 랜덤한 비밀번호를 해싱하기 위해 `bcrypt`를 사용해 보겠습나다.

우선 필요한 패키지들을 설치합니다:

```shell
$ npm i bcrypt
$ npm i -D @types/bcrypt
```

설치가 완료되면, 다음과 같이 `hash` 함수를 사용할 수 있습니다:

```typescript
import * as bcrypt from "bcrypt";

const saltOrRounds = 10;
const password = "random_password";
const hash = await bcrypt.hash(password, saltOrRounds);
```

salt를 생성하려면 `genSalt` 함수를 사용합니다:

```typescript
const salt = await bcrypt.genSalt();
```

비밀번호를 확인 및 비교하려면 `compare` 함수를 사용합니다:

```typescript
const isMatch = await bcrypt.compare(password, hash);
```

관련하여 [여기](https://www.npmjs.com/package/bcrypt)에서 더 많은 함수들을 확인할 수 있습니다.
