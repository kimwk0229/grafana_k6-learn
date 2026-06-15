# 실행 컨텍스트 변수 사용하기

이전 섹션에서 executor와 scenario를 사용하여 애플리케이션의 실제 사용자의 트래픽을 더 잘 시뮬레이션함으로써 k6 스크립트를 [더 현실적으로](03-Workload-modeling.md) 만드는 방법을 배웠습니다. 스크립트를 프로덕션 트래픽에 더 가깝게 만들면 테스트 결과가 더 정확해집니다.

그러나 여러 VU, scenario, 인스턴스에 걸쳐 스크립트를 확장하다 보면 결과를 이해하기가 더 어려워질 수 있다는 것을 금방 알게 될 것입니다. 예를 들어, 스크립트가 오류를 보고할 때 어떻게 그 오류를 만난 VU를 추적할 수 있을까요?

실행 컨텍스트 변수는 테스트가 실행되는 방법에 대한 정보를 저장합니다. 환경 변수처럼 작동하지만, k6에 의해 미리 결정되고 현재 실행 상태에 특화된 변수입니다. k6에서 이러한 변수는 `k6/execution` 모듈에 포함되어 있습니다.

## 실행 컨텍스트 변수의 유형

`k6/execution` 모듈에는 테스트에서 호출할 수 있는 많은 속성이 포함되어 있습니다. 다음 중 어느 것에 대한 정보를 제공하는지에 따라 분류할 수 있습니다:
- Instance
- Scenario
- Test
- VU

위의 네 가지 유형 각각은 `k6/execution` 모듈의 객체입니다.

**instance** 객체는 각 부하 생성기, 즉 k6 프로세스가 실행 중인 기계에 대한 정보를 저장합니다.

**scenario** 객체는 어떤 scenario와 executor가 실행 중인지와 같은 부하 프로필에 대한 정보를 보유합니다.

**test** 객체는 선택적 오류 메시지와 함께 테스트 실행을 중단할 수 있게 합니다.

**VU** 객체는 부하 생성기의 모든 iteration과 모든 VU에 고유한 번호를 할당합니다.

[`k6/execution` 모듈에 대해 더 알아보기](https://k6.io/docs/javascript-api/k6-execution/).

## 고유한 데이터 생성하기

웹 애플리케이션에 로그인하는 k6 스크립트를 작성하고 싶다고 상상해 보세요. 그런데 애플리케이션에는 동일한 계정에 대한 여러 동시 로그인을 허용하지 않는 보안 조치가 있습니다. 이 기능은 두 번째 k6 VU가 첫 번째 VU가 여전히 로그인한 상태에서 로그인할 때 HTTP 403 Forbidden 오류를 발생시킵니다. 하나 또는 두 VU 모두 로그아웃될 수 있습니다.

이를 어떻게 방지할 수 있을까요? 각 VU에 고유한 로그인을 할당하여 해당 VU만이 해당 사용자 계정에 로그인을 시도하도록 할 수 있습니다. 고유한 로그인을 만들려면, 전체 테스트에서 VU당 고유하다고 보장할 수 있는 식별자인 `vu.idInTest`, 즉 실행 컨텍스트 변수를 활용할 수 있습니다.

먼저, 이 형식으로 사용자 이름과 비밀번호의 CSV 파일을 만드세요:

```plain
username,password
user01,123
user02,234
user03,345
user04,456
user05,567
user06,678
user07,789
user08,890
user09,901
user10,012
```

파일을 `/data/users-unique.csv`로 저장하세요. 이것들이 테스트 중인 애플리케이션의 유효한 자격 증명인지 확인하세요.

다음으로, `users-unique.csv`를 사용하는 k6 스크립트를 작성하세요. 아래 예시 스크립트는 실제 로그인 대신 로그 구문을 포함하고 있어 무슨 일이 일어나고 있는지 이해하기 더 쉽습니다.

```js
import papaparse from 'https://jslib.k6.io/papaparse/5.1.1/index.js';
import { SharedArray } from 'k6/data';
import { vu } from 'k6/execution';
import { sleep } from 'k6';

const users = new SharedArray("Logins", function() {
    let data = papaparse.parse(open('./data/users-unique.csv'), { header: true }).data;
    return data;
});

export const options = {
    scenarios: {
        login: {
            executor: 'per-vu-iterations',
            vus: users.length,
            iterations: 1,
        },
    },
};

export default function () {
    console.log('VU: ' + vu.idInTest + ' / username: ', users[vu.idInTest - 1].username, ' / password: ', users[vu.idInTest - 1].password);
    sleep(1);
}
```

스크립트가 단계별로 무엇을 하는지 살펴보겠습니다.

스크립트가 가져오는 세 가지 객체가 있습니다:
- `papaparse`는 [CSV 파일을 테스트 데이터로](04-Adding-test-data.md#CSV-Files) 사용할 수 있게 합니다
- `SharedArray`는 리소스를 [절약하기 위해](04-Adding-test-data.md#Shared-Array) 각 VU가 필요로 하는 만큼만 CSV 파일의 줄을 배포합니다
- `vu`는 각 VU의 고유 식별자에 접근할 수 있게 합니다
- `sleep`은 스크립트에서 지연을 사용할 수 있게 합니다

이 줄들은 CSV 파일, 즉 앞서 만든 `users-unique.csv` 파일을 사용하도록 shared array를 설정합니다:

```js
const users = new SharedArray("Logins", function() {
    let data = papaparse.parse(open('./data/users-unique.csv'), { header: true }).data;
    return data;
});
```

> :warning: **header를 true로 설정하세요**. 이 경우에 생성한 CSV 파일의 첫 번째 줄이 데이터 파일의 열 이름(`username,password`)을 정의하기 때문에 `header: true` 옵션이 필요합니다.

다음 몇 줄은 테스트에 대한 [옵션과 매개변수](../II-k6-Foundations/06-k6-Load-Test-Options.md)를 정의합니다:

```js
export const options = {
    scenarios: {
        login: {
            executor: 'per-vu-iterations',
            vus: users.length,
            iterations: 1,
        },
    },
};
```

VU 수는 CSV 파일의 줄당 하나의 VU가 있도록 `users.length`로 설정됩니다. 이 설정은 헤더 줄을 자동으로 무시합니다. CSV에 11개의 줄이 있고 첫 번째가 헤더이므로, 10 VU가 각각 한 번씩 실행될 것으로 예상할 수 있습니다.

이제 테스트의 핵심 부분이 나옵니다:

```js
export default function () {
    console.log('VU: ' + vu.idInTest + ' / username: ', users[vu.idInTest - 1].username, ' / password: ', users[vu.idInTest - 1].password);
    sleep(1);
}
```

이 함수는 로그 구문과 sleep으로 구성됩니다. 로그 구문은 로그인 함수의 대역입니다. 사용자로 로그인하는 대신, 스크립트는 k6에게 전역적으로 고유한 식별자인 현재 VU의 `idInTest`를 출력하도록 지시합니다. k6는 그런 다음 해당 VU에 선택된 사용자 이름과 비밀번호를 출력합니다.

> :point_up: **왜 `idInTest - 1`인가요?**
> 식별자를 출력할 때 `vu.idInTest`가 사용되지만, CSV 파일에서 사용자 이름과 비밀번호를 선택할 때는 `vu.idInTest - 1`이 사용된다는 것을 알아챘을 것입니다.
> 이 차이는 배열이 0으로 시작하지만 `idInTest`는 1로 시작한다는 사실 때문입니다. 이 예에서, CSV의 행에 해당하는 배열 요소는 `0,1,2,3,4,5,6,7,8,9`이고 각 VU의 `idInTest`는 `1,2,3,4,5,6,7,8,9,10`입니다.

마지막으로, 스크립트를 `script.js`로 저장하고 실행하세요: `k6 run script.js`.

```plain
INFO[0000] VU: 10 / username:  user10  / password:  012  source=console
INFO[0000] VU: 5 / username:  user05  / password:  567   source=console
INFO[0000] VU: 6 / username:  user06  / password:  678   source=console
INFO[0000] VU: 9 / username:  user09  / password:  901   source=console
INFO[0000] VU: 4 / username:  user04  / password:  456   source=console
INFO[0000] VU: 1 / username:  user01  / password:  123   source=console
INFO[0000] VU: 8 / username:  user08  / password:  890   source=console
INFO[0000] VU: 2 / username:  user02  / password:  234   source=console
INFO[0000] VU: 3 / username:  user03  / password:  345   source=console
INFO[0000] VU: 7 / username:  user07  / password:  789   source=console
```

위의 스크립트 출력을 바탕으로 다음을 확인할 수 있습니다:
- CSV 파일의 헤더가 아닌 줄 수에 따라 10 VU가 각각 한 번씩 실행됩니다
- 각 VU는 할당된 사용자 이름과 비밀번호만 사용합니다
- 각 VU의 `idInTest`는 전역적으로 고유합니다.

데이터 충돌은 k6 테스트를 여러 VU, scenario, 인스턴스에 걸쳐 확장할 때 큰 문제가 될 수 있습니다. 여기서 시연된 것처럼 실행 컨텍스트 변수를 사용하면 테스트가 원하는 수준의 부하에 도달하는 것을 방해할 수 있는 스크립팅으로 인한 오류를 방지하는 데 도움이 됩니다.

## 실행 컨텍스트 변수의 다른 사용 사례

실행 컨텍스트 변수를 사용하여 다음과 같은 작업도 수행할 수 있습니다:
- [scenario에 대한 조건부 로직 추가](https://k6.io/docs/javascript-api/k6-execution/#script-naming)
- [테스트를 중단할 시기 정의](https://k6.io/docs/javascript-api/k6-execution/#test-abort)
- [오류에 대한 개선된 로깅](https://k6.io/docs/javascript-api/k6-execution/#timing-operations)
- [VU에 태그 추가 및 검색](https://k6.io/docs/javascript-api/k6-execution/#tags)


## 지식 확인

### Question 1

어떤 상황에서 실행 컨텍스트 변수를 사용하는 것이 도움이 될 수 있습니까?

A: 대용량 데이터 파일이 있지만 VU 전체에 걸쳐 중복 데이터가 선택되지 않도록 하고 싶습니다.

B: k6 스크립트를 실행하지만 명령줄에서 VU 수를 설정하고 싶습니다.

C: scenario가 사용하는 executor를 지정하고 싶습니다.

### Question 2

테스트 스크립트가 10 VU를 정의하고 각각 2번의 iteration을 수행합니다. 이 테스트에서 `vu.idInTest`의 고유한 값이 몇 개 존재할 것으로 예상합니까?

A: 20.

B: 10.

C: 테스트가 실행되는 인스턴스 수에 따라 다릅니다.

### Question 3

현재 실행 중인 executor에 대한 정보를 검색하는 데 어떤 실행 컨텍스트 객체를 사용하겠습니까?

A: `scenario`

B: `vu`

C: `test`

### 정답

1. A. `vu.idInTest`와 같은 고유 식별자를 사용하여 데이터 충돌을 방지할 수 있습니다. B는 [CLI 플래그](../II-k6-Foundations/02-The-k6-CLI.md)를 사용하여 명령줄에서 VU 수를 설정하기 때문에 틀렸으며, C는 executor를 선택하는 방법이 [스크립트에서](08-Setting-load-profiles-with-executors.md) 이루어지기 때문에 틀렸습니다.
2. B. `vu.idInTest`는 iteration당이 아니라 VU당 할당됩니다.
3. A. `scenario`는 사용 중인 executor에 대한 정보를 보유합니다.
