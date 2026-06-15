# Shared Iterations Executor

[executor로 부하 프로필 설정하기](../08-Setting-load-profiles-with-executors.md#Shared-Iterations)에서 언급한 바와 같이, _Shared Iterations_는 executor 중 가장 기본적인 것입니다. 이름에서 유추할 수 있듯이, 주된 초점은 테스트의 _iteration_ 횟수에 맞춰져 있습니다. 이는 테스트 함수가 실행되는 횟수를 의미합니다.

## 연습 문제
이번 연습에서는 HTTP 요청을 수행한 뒤 테스트 iteration이 완료되기 전 1초를 대기하는 매우 기본적인 스크립트를 사용하여 시작합니다. 변경 사항에 따른 콘솔 출력도 확인할 수 있습니다.

### 스크립트 작성
테스트 스크립트를 구현하는 것부터 시작하겠습니다. 다음 내용으로 _test.js_ 파일을 생성하세요:
```js
import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'shared-iterations',
    },
  },
};

export default function () {
  console.log(`[VU: ${__VU}, iteration: ${__ITER}] Starting iteration...`);
  http.get('https://test.k6.io/contacts.php');
  sleep(1);
}
```

### 초기 테스트 실행
executor를 사용하는 데 필요한 최소한의 설정으로 시작합니다. `executor` 자체만 지정하면 됩니다. 기본 스크립트를 작성했으니, k6를 실행해 보겠습니다:

```bash
k6 run test.js
```

결과를 보면 실제로 단 하나의 테스트 iteration이 실행되었음을 확인할 수 있습니다.

```bash
INFO[0000] [VU: 1, iteration: 0] Starting iteration...   source=console

running (00m02.3s), 0/1 VUs, 1 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 1 VUs  00m02.3s/10m0s  1/1 shared iters
```

### iteration 횟수 변경
요청 하나만으로는 충분하지 않으므로, 수정해 보겠습니다. 테스트의 `options`를 수정하여 테스트에서 수행할 `iterations` 수를 늘려보겠습니다.
```js
export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'shared-iterations',
      iterations: 200,
    },
  },
};
```
다시 k6로 스크립트를 실행합니다:
```bash
k6 run test.js
```

시간이 꽤 걸립니다. 뭔가 잘못된 건 아닐까요? 아닙니다. 아직 완료되지 않았다면 `Ctrl+C`를 사용하여 테스트를 중단해 봅시다.

`sleep(1)` 등을 통해 테스트에 인위적으로 지연을 추가하고 있습니다. 이는 설명을 위한 것입니다. `vus` (virtual user, VU) 수를 지정하지 않았기 때문에 k6는 단일 사용자로 테스트를 순차적으로 수행합니다. 200번의 iteration 동안 인위적인 지연을 포함하면, 테스트를 완료하는 데 3분 이상이 걸릴 수 있습니다!

### 시간 제한 추가
로컬에서 스크립트를 실행할 때는 예상보다 오래 걸리는 경우 쉽게 알아차리고 프로세스를 종료할 수 있습니다. 하지만 자동화된 파이프라인에서는 시스템이 대기의 필요성이나 비용에 관계없이 프로세스가 완료될 때까지 기꺼이 대기할 수 있습니다.

이를 위해 executor에는 테스트에 시간 제한을 설정하는 `maxDuration` 옵션이 있습니다. 지정하지 않으면 executor는 기본적으로 10분 제한 시간을 사용하는데, 이는 너무 많을 수도 있고 너무 적을 수도 있습니다. 가장 좋은 방법은 최선의 추정값이나 이전 테스트 결과를 기반으로 `maxDuration`을 지정하는 것입니다.

테스트 스크립트를 업데이트하여 `maxDuration`을 30초로 설정합니다:
```js
export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'shared-iterations',
      iterations: 200,
      maxDuration: '30s',
    },
  },
};
```
> :point_up: duration은 양의 정수와 시간 단위를 나타내는 접미사로 구성된 문자열 값으로 설정합니다. 예를 들어 초에는 "s", 분에는 "m"을 사용합니다.

이전과 같이 `k6 run test.js`로 스크립트를 실행합니다.

스크립트를 실행하면 시간 제한에 도달하여 모든 iteration을 완료하지 못한 것을 확인할 수 있습니다.
```bash
INFO[0029] [VU: 1, iteration: 25] Starting iteration...  source=console
INFO[0030] [VU: 1, iteration: 26] Starting iteration...  source=console

running (0m30.9s), 0/1 VUs, 27 complete and 0 interrupted iterations
k6_workshop ✗ [====>---------------------------------] 1 VUs  30.9s/30s  027/200 shared iters
```
스크립트는 허용된 30초 내에 27번의 iteration만 완료할 수 있었습니다.

### 동시성 변경
지금까지 테스트는 단 하나의 virtual user, 즉 VU만 사용했습니다. `vus` 옵션을 업데이트하여 동시에 수행되는 요청 수를 늘릴 수 있습니다. 10명의 사용자를 시뮬레이션하도록 `vus`를 변경해 보겠습니다. 테스트 스크립트의 `options` 섹션을 다시 업데이트합니다:
```js
export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'shared-iterations',
      iterations: 200,
      maxDuration: '30s',
      vus: 10,
    },
  },
};
```
스크립트를 다시 실행합니다:
```bash
k6 run test.js
```
이제 테스트는 동시 요청 수가 증가했으므로 훨씬 짧은 시간에 동일한 수의 iteration을 수행해야 합니다.
```bash
INFO[0020] [VU: 7, iteration: 17] Starting iteration...  source=console
...
INFO[0020] [VU: 2, iteration: 17] Starting iteration...  source=console
...
INFO[0020] [VU: 6, iteration: 19] Starting iteration...  source=console
INFO[0020] [VU: 1, iteration: 18] Starting iteration...  source=console
INFO[0021] [VU: 3, iteration: 19] Starting iteration...  source=console
INFO[0021] [VU: 5, iteration: 20] Starting iteration...  source=console
INFO[0021] [VU: 4, iteration: 20] Starting iteration...  source=console
INFO[0021] [VU: 8, iteration: 20] Starting iteration...  source=console
INFO[0021] [VU: 9, iteration: 20] Starting iteration...  source=console
INFO[0021] [VU: 10, iteration: 20] Starting iteration...  source=console

running (0m22.6s), 00/10 VUs, 200 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 10 VUs  22.6s/30s  200/200 shared iters
```
이 결과를 좀 더 자세히 살펴보겠습니다. 위의 결과(간결함을 위해 일부 생략)는 실행 중인 각 VU의 최종 보고서를 보여줍니다.

> :point_up: iteration 카운터는 0부터 시작하므로, 카운트가 20이면 실제로는 21번의 iteration을 의미합니다.

`shared-iterations` executor의 동작이 분명해 질 것입니다: _일부 VU는 다른 VU보다 더 많은 테스트를 수행했습니다_. 이것이 executor 이름의 "shared"(공유) 측면입니다. 각 VU가 테스트 iteration을 완료하면, 남아 있는 iteration이 있는 동안 즉시 다음 iteration을 예약합니다. VU가 테스트 중인 서비스에서 빠른 응답을 계속 받는 경우, 이 예시처럼 VU가 _공정한 몫_ 이상을 가져갈 수 있습니다.

### 마무리
이 연습을 통해 매우 기본적인 테스트를 실행하는 방법과 iteration 수, 동시성, 심지어 시간 제한을 설정하는 방법을 확인했습니다. 또한 VU 간 테스트 분배가 _공평하지_ 않을 수 있다는 것도 알 수 있었습니다.
