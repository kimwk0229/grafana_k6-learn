# Per VU Iterations Executor

[executor로 부하 프로필 설정하기](../08-Setting-load-profiles-with-executors.md#Per-VU-Iterations)에서 언급한 바와 같이, _Per VU Iterations_는 _virtual user (VU)_가 수행하는 _iteration_에 초점을 맞춘 executor입니다.

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
      executor: 'per-vu-iterations',
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

executor를 사용하는 데 필요한 최소한의 설정으로 시작합니다. `executor` 자체만 지정하면 됩니다. 기본 스크립트를 작성했으니 k6를 실행해 보겠습니다:

```bash
k6 run test.js
```

결과를 보면 단일 virtual user로부터 단 하나의 테스트 iteration이 실행되었음을 확인할 수 있습니다.

```bash
INFO[0000] [VU: 1, iteration: 0] Starting iteration...   source=console

running (00m01.3s), 0/1 VUs, 1 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 1 VUs  00m01.3s/10m0s  1/1 iters, 1 per VU
```

### virtual user 수 늘리기

테스트에 virtual user 수를 지정하지 않으면 기본값은 단일 사용자입니다. 이름에서 알 수 있듯이, VU 수는 이 executor의 중요한 측면입니다. `vus` 옵션을 사용하여 virtual user 수를 늘려 10명의 사용자를 시뮬레이션해 보겠습니다. 테스트 스크립트의 `options` 섹션을 업데이트합니다:

```js
export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'per-vu-iterations',
      vus: 10,
    },
  },
};
```

스크립트를 다시 실행합니다:

```bash
k6 run test.js
```

출력을 보면 이제 테스트가 10번의 iteration을 실행한 것을 확인할 수 있습니다. 즉, _VU당 1번의 iteration_입니다.

```bash
INFO[0000] [VU: 7, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 2, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 1, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 5, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 3, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 4, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 8, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 9, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 6, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 10, iteration: 0] Starting iteration...  source=console

running (00m01.7s), 00/10 VUs, 10 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 10 VUs  00m01.7s/10m0s  10/10 iters, 1 per VU
```

### iteration 횟수 변경

이전 예시에서는 원하는 iteration 수를 지정하지 않았으므로 k6는 10개의 VU 각각에 기본값인 1을 사용하여 전체 10번의 iteration이 수행되었습니다. 이를 확장하여 `iterations` 옵션을 20으로 늘려 봅시다. iteration이 _VU당_이므로, 테스트 실행에서 총 200번(`10 VUs * 20 iterations`)의 iteration이 예상됩니다.

```js
export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'per-vu-iterations',
      vus: 10,
      iterations: 20,
    },
  },
};
```

다시 k6로 스크립트를 실행합니다:

```bash
k6 run test.js
```

예상대로 테스트는 총 200번의 iteration을 실행했습니다. 이 결과를 좀 더 자세히 살펴보겠습니다:

```bash
INFO[0022] [VU: 1, iteration: 19] Starting iteration...  source=console
INFO[0022] [VU: 8, iteration: 19] Starting iteration...  source=console
INFO[0022] [VU: 9, iteration: 18] Starting iteration...  source=console
INFO[0022] [VU: 2, iteration: 18] Starting iteration...  source=console
INFO[0022] [VU: 3, iteration: 19] Starting iteration...  source=console
INFO[0022] [VU: 5, iteration: 19] Starting iteration...  source=console
INFO[0022] [VU: 6, iteration: 19] Starting iteration...  source=console
INFO[0022] [VU: 10, iteration: 19] Starting iteration...  source=console
INFO[0022] [VU: 4, iteration: 19] Starting iteration...  source=console
INFO[0022] [VU: 7, iteration: 18] Starting iteration...  source=console
INFO[0023] [VU: 9, iteration: 19] Starting iteration...  source=console
INFO[0023] [VU: 2, iteration: 19] Starting iteration...  source=console
INFO[0024] [VU: 7, iteration: 19] Starting iteration...  source=console

running (00m25.1s), 00/10 VUs, 200 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 10 VUs  00m25.1s/10m0s  200/200 iters, 20 per VU

```

> :point_up: iteration 카운터는 0부터 시작하므로, 카운트가 19이면 실제로는 20번의 iteration을 의미합니다.

출력에서 _VU #1_이 _VU #7_보다 일찍 완료된 것을 알 수 있습니다. 실제로 _VU #7_은 _VU #1_이 이미 완료된 **이후에도** 2번의 iteration을 더 완료해야 했습니다. 이것은 _공정한 스케줄링(fair-share scheduling)_으로 간주할 수 있습니다. 각 VU는 동일한 양의 작업을 수행합니다.

학창 시절과 비슷하게, 시험을 일찍 끝내면 나머지 학생들이 끝나거나 종이 울릴 때까지 기다려야 했습니다. 그 종소리 이야기를 하자면...

### 시간 제한 설정

지금까지 예시에서는 원하는 iteration을 완료하기에 충분한 시간이 있었습니다. `maxDuration`을 지정하지 않았기 때문에 k6는 기본값인 10분을 사용합니다.

테스트 스크립트를 업데이트하여 `maxDuration`을 10초로 설정합니다:

```js
export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'per-vu-iterations',
      vus: 10,
      iterations: 20,
      maxDuration: '10s',
    },
  },
};
```

> :point_up: duration은 양의 정수와 시간 단위를 나타내는 접미사로 구성된 문자열 값으로 설정합니다. 예를 들어 초에는 "s", 분에는 "m"을 사용합니다.

이전과 같이 `k6 run test.js`로 스크립트를 실행합니다.

스크립트를 실행하면 시간 제한에 도달하여 모든 iteration을 완료하지 못한 것을 확인할 수 있습니다. _수업 종료, 연필을 내려놓으세요!_

```bash
INFO[0010] [VU: 9, iteration: 9] Starting iteration...   source=console
INFO[0010] [VU: 6, iteration: 9] Starting iteration...   source=console
INFO[0010] [VU: 10, iteration: 9] Starting iteration...  source=console

running (11.0s), 00/10 VUs, 95 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 10 VUs  10s  095/200 iters, 20 per VU
```
스크립트는 허용된 10초 내에 95번의 iteration만 완료할 수 있었습니다. 이전 결과에 따르면 전체 200번의 iteration은 보통 25초 안에 완료됩니다.

이 정보를 기준으로 테스트 중인 서비스에 대한 _서비스 수준 협약(SLA)_을 설정할 수 있습니다. 약간의 변동을 감안하여 `maxDuration`을 30초로 설정합니다. 그 시간 내에 테스트를 완료할 수 없다면 서비스 성능이 충분히 저하되어 조사가 필요할 수 있습니다.

### 마무리

이 연습을 통해 매우 기본적인 테스트를 실행하는 방법과 iteration 수, virtual user 수, 심지어 시간 제한을 설정하는 방법을 확인했습니다. 또한 _Per VU Iterations_를 사용할 때 VU 간의 테스트 분배가 _공정하게 스케줄링_된다는 것도 알 수 있었습니다.
