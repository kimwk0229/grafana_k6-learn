# 테스트 데이터 추가하기
이 섹션에서는 부하 테스트 스크립트에 테스트 데이터를 추가하는 가장 좋은 방법을 배워 스크립트를 더 동적이고 현실적으로 만드는 방법을 알아봅니다.

## 테스트 데이터를 추가하는 이유는 무엇인가요?

테스트 데이터는 실행 중에 테스트 스크립트가 사용하도록 설계된 정보입니다. 이 정보는 특정 매개변수에 다른 값을 사용하여 테스트를 더 현실적으로 만듭니다. 사용자 이름과 비밀번호 같은 것에 동일한 값을 반복해서 보내면 캐싱이 발생할 수 있습니다.

캐시는 애플리케이션 성능 향상을 목표로 저장된 콘텐츠를 나타냅니다. 캐싱은 서버 측이나 클라이언트 측에서 발생할 수 있습니다. 서버 측 캐시는 일반적으로 요청되는 리소스(예: 홈페이지의 리소스)를 저장하여 해당 리소스가 요청될 때 서버가 다운스트림 서버에서 가져오지 않고 즉시 리소스를 반환할 수 있도록 합니다.

클라이언트 측 캐시는 동일한 리소스를 저장하여 사용자의 브라우저가 해당 리소스가 변경되지 않는 한 요청을 보낼 필요가 없다는 것을 알 수 있습니다. 이 구현은 전송해야 하는 네트워크 요청 수를 줄이며, 주로 모바일 기기를 통해 접근되는 앱에 특히 중요합니다.

캐싱은 부하 테스트 결과에 크게 영향을 줄 수 있습니다. 캐싱이 활성화되어야 하는 상황(예: 재방문 고객의 행동을 시뮬레이션하려는 경우)이 있지만, 캐싱이 비활성화되어야 하는 상황(예: 신규 사용자를 시뮬레이션하는 경우)도 있습니다. 어느 쪽이든, 캐싱 전략을 의도적으로 결정하고 그에 맞게 스크립팅해야 합니다.

테스트 데이터를 추가하면 서버 측 캐싱을 방지하는 데 도움이 됩니다. 일반적인 테스트 데이터에는 다음이 포함됩니다:
- 애플리케이션에 로그인하고 인증된 작업을 수행하기 위한 사용자 이름과 비밀번호
- 계정 가입이나 연락처 양식 작성을 위한 이름, 주소, 이메일 및 기타 개인 정보
- 제품 페이지 검색을 위한 제품 이름
- 카탈로그 검색을 위한 키워드
- 업로드 테스트를 위한 PDF

## 배열(Array)

테스트 데이터를 추가하는 가장 간단한 방법은 배열(array)을 사용하는 것입니다. [k6에서의 동적 상관관계](02-Dynamic-correlation-in-k6.md)에서 다음과 같이 배열을 정의했습니다:

```js
let usernameArr = ['admin', 'test_user'];
let passwordArr = ['123', '1234'];
```

배열을 정의한 후, 난수를 생성하여 배열에서 무작위로 값을 선택할 수 있습니다:

```js
// Get random username and password from array
let rand = Math.floor(Math.random() * usernameArr.length);
let username = usernameArr[rand];
let password = passwordArr[rand];
console.log('username: ' + username, ' / password: ' + password);
```

배열은 이 예와 같이 매우 짧은 텍스트 목록이나 테스트 스크립트를 디버깅할 때 사용하기에 가장 적합합니다.

## CSV 파일

CSV 파일은 _쉼표로 구분된 값(comma-separated values)_으로 구성된 정보 목록으로, 스크립트와 별도로 저장됩니다.

아래는 사용자 이름과 비밀번호를 포함하는 `users.csv`라는 CSV 파일의 예입니다:
```plain
username,password
admin,123
test_user,1234
...
```

k6에서는 CSV 파일을 다음과 같이 스크립트에 추가할 수 있습니다:

```js
import papaparse from 'https://jslib.k6.io/papaparse/5.1.1/index.js';

const csvData = papaparse.parse(open('users.csv'), { header: true }).data;

export default function () {
    let rand = Math.floor(Math.random() * csvData.length);
    console.log('username: ', csvData[rand].username, ' / password: ', csvData[rand].password);
}
```

이 코드는 CSV 파일과 상호 작용할 수 있게 해주는 `papaparse`라는 라이브러리를 가져옵니다. CSV 파일은 새로운 사용자 이름과 계정을 생성하거나 스프레드시트 애플리케이션에서 테스트 데이터 파일을 열고 싶을 때 가장 적합합니다.

> k6는 default 함수 내에 로컬 파일 시스템에서 읽는 코드를 배치하는 것을 허용하지 않습니다. 이 제한은 파일 읽기를 init 컨텍스트(default 함수 외부)에 배치하는 모범 사례를 강제합니다. 이에 대한 자세한 내용은 [테스트 라이프사이클](https://k6.io/docs/using-k6/test-life-cycle/)에서 확인할 수 있습니다.

## JSON 파일

테스트 데이터를 JSON 파일에 저장할 수도 있습니다.

아래는 `users.json`이라는 JSON 파일입니다:

```plain
{
  "users": [
    { "username": "admin", "password": "123" },
    { "username": "test_user", "password": "1234" }
  ]
}
```

그런 다음 k6 스크립트에서 다음과 같이 사용할 수 있습니다:

```js
const jsonData = JSON.parse(open('./users.json')).users;

export default function () {
    let rand = Math.floor(Math.random() * jsonData.length);
    console.log('username: ', jsonData[rand].username, ' / password: ', jsonData[rand].password);
}
```

JSON 파일은 테스트에 사용할 정보가 이미 JSON 형식으로 내보내진 경우 또는 데이터가 CSV 파일이 처리할 수 있는 것보다 더 계층적인 경우에 가장 적합합니다.

## Shared Array

Shared Array는 이전 세 가지 방식(단순 배열, CSV 파일, JSON 파일)의 일부 요소를 결합하면서 테스트 실행 중 테스트 데이터와 관련된 일반적인 문제인 높은 리소스 사용률을 해결합니다.

이전 방식에서는 파일이 테스트에 사용될 때, 여러 복사본이 생성되어 부하 생성기로 전송됩니다. 파일이 매우 클 경우, 이는 부하 생성기의 리소스를 불필요하게 사용하여 테스트 결과를 덜 정확하게 만들 수 있습니다.

이를 방지하려면 `SharedArray`를 사용하세요:

```js
import papaparse from 'https://jslib.k6.io/papaparse/5.1.1/index.js';
import { SharedArray } from "k6/data";

const sharedData = new SharedArray("Shared Logins", function() {
    let data = papaparse.parse(open('users.csv'), { header: true }).data;
    return data;
});

export default function () {
    let rand = Math.floor(Math.random() * sharedData.length);
    console.log('username: ', sharedData[rand].username, ' / password: ', sharedData[rand].password);
}
```

SharedArray는 다른 방식과 결합되어야 한다는 점에 유의하세요―이 경우 CSV 파일.

SharedArray를 사용하는 것이 k6 테스트 내에서 데이터 목록을 추가하는 가장 효율적인 방법입니다.

## 기타 테스트 데이터

테스트 데이터는 테스트 중인 scenario의 일부로 업로드해야 하는 파일을 가리킬 수도 있습니다. 예를 들어, 사용자가 프로필 사진을 업로드하는 것을 시뮬레이션하기 위해 이미지를 업로드해야 할 수도 있습니다. 이러한 유형의 데이터 업로드 스크립팅에 대한 자세한 내용은 [여기의 문서를 확인하세요.](https://k6.io/docs/examples/data-uploads/)

## 지식 확인

### Question 1

다음 중 어떤 상황에서 부하 테스트 스크립트에 테스트 데이터를 추가하는 것이 좋습니까?

A: 짧은 시간 안에 세 번 로그인하면 사용자 계정이 잠기는 애플리케이션이 있습니다.

B: 동일한 사용자가 페이지를 반복해서 새로 고침할 때 애플리케이션이 어떻게 동작하는지 확인하고 싶습니다.

C: A와 B 모두.

### Question 2

테스트 데이터로 사용하고 싶은 개인 정보가 담긴 100 MB의 CSV 파일이 있습니다. 다음 중 가장 좋은 방법은 무엇입니까?

A: SharedArray

B: 단순 배열

C: CSV가 JSON으로 더 잘 변환되기 때문에 JSON 파일

### Question 3

이 페이지의 모든 예제에는 데이터 파일에서 무작위로 요소를 선택하는 `Math.random()` 함수가 포함되어 있습니다. 어떤 상황에서 이 무작위화를 제거하고 싶을 수 있습니까?

A: 서버 측 캐싱을 방지하고 싶을 때.

B: 테스트 데이터의 각 요소가 스크립트에 의해 순차적으로 활용되었음을 보장하고 싶을 때.

C: 테스트를 가능한 한 현실적으로 만들고 싶을 때.

### 정답

1. A. A에서 설명한 상황은 스크립트가 사용할 수 있는 다른 로그인의 테스트 데이터를 추가하여 해결할 수 있습니다. B는 잘못되었습니다. 왜냐하면 이는 사실 테스트 데이터를 포함하지 *않는* 것이 더 나은 옵션일 수 있는 사용 사례의 좋은 예이기 때문입니다.
2. A. 매우 큰 데이터 파일은 모든 부하 생성기에 반복적으로 복사되고 전송될 때 부하 테스트 결과에 영향을 줄 수 있습니다. SharedArray는 이를 처리하는 더 좋은 방법이지만, 테스트 데이터를 데이터베이스에 저장하는 것도 고려할 가치가 있습니다.
3. B. 테스트 데이터를 무작위로 선택하면 캐싱을 방지하고 테스트를 더 현실적으로 만들 수 있습니다. 그러나 무작위 선택은 데이터 파일이 순차적으로 파싱되지 않으므로 테스트가 어떤 테스트 데이터를 활용했는지 결정하기 더 어렵게 만들 수 있습니다.
