# Constant VUs Executor

[executor로 부하 프로필 설정하기](../08-Setting-load-profiles-with-executors.md#Constant-VUs)에서 언급한 바와 같이, _Constant VUs_ executor는 지정된 시간 동안 실행되는 _virtual user (VU)_ 수에 주된 초점을 둡니다.

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
      executor: 'constant-vus',
      duration: '30s',
    },
  },
};

export default function () {
  console.log(`[VU: ${__VU}, iteration: ${__ITER}] Starting iteration...`);
  http.get('https://test.k6.io/contacts.php');
  sleep(1);
}
```

> :point_up: 워크샵 연습을 순서대로 진행해 왔다면, 이번에는 초기 스크립트에 `duration`이 포함되어 있다는 것을 알아챘을 것입니다. 이 옵션은 executor에서 **필수**입니다. 이 옵션 없이 스크립트를 실행하려고 하면 설정 오류가 발생합니다.

### 초기 테스트 실행

executor를 사용하는 데 필요한 최소한의 설정으로 시작합니다. `executor`와 테스트 `duration` 모두 필요합니다. 기본 스크립트를 작성했으니 k6를 실행해 보겠습니다:

```bash
k6 run test.js
```

실행 중인 스크립트의 출력을 보면 단일 virtual user (VU)가 테스트 iteration을 계속 수행하는 것을 확인할 수 있습니다. 설정된 30초 duration이 도달하면 테스트가 완료됩니다. 수행되는 iteration 수는 `default function ()` 코드 블록에서 실행되는 코드에 전적으로 의존합니다.

```bash
INFO[0026] [VU: 1, iteration: 24] Starting iteration...  source=console
INFO[0027] [VU: 1, iteration: 25] Starting iteration...  source=console
INFO[0028] [VU: 1, iteration: 26] Starting iteration...  source=console
INFO[0029] [VU: 1, iteration: 27] Starting iteration...  source=console

running (0m30.5s), 0/1 VUs, 28 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 1 VUs  30s
```

### 동시성 변경

다시 한번, 테스트는 단 하나의 virtual user 또는 VU만 실행했습니다. `vus` 옵션을 업데이트하여 동시에 수행되는 요청 수를 늘릴 수 있습니다. 테스트 스크립트의 `options` 섹션을 업데이트하여 `vus`를 10명의 사용자를 시뮬레이션하도록 변경해 보겠습니다:

```js
export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'constant-vus',
      duration: '30s',
      vus: 10,
    },
  },
};
```

스크립트를 한 번 더 실행합니다:

```bash
k6 run test.js
```

이제 설정된 duration이 도달할 때까지 10개의 VU가 반복적으로 iteration을 수행하므로 훨씬 더 많은 활동이 보여야 합니다.

```bash
INFO[0029] [VU: 4, iteration: 27] Starting iteration...  source=console
INFO[0029] [VU: 8, iteration: 27] Starting iteration...  source=console
INFO[0029] [VU: 3, iteration: 27] Starting iteration...  source=console
INFO[0029] [VU: 9, iteration: 27] Starting iteration...  source=console
INFO[0029] [VU: 6, iteration: 27] Starting iteration...  source=console

running (0m30.5s), 00/10 VUs, 280 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 10 VUs  30s
```

> :point_up: iteration 카운터는 0부터 시작하므로, 카운트가 27이면 실제로는 28번의 iteration을 의미합니다.

결과에서 요청이 교차하거나 일부 VU가 다른 VU보다 더 많은 iteration을 수행할 수 있습니다. 이는 개별 요청의 응답 시간에 따라 달라집니다.

### 마무리

이 연습을 통해 매우 기본적인 테스트를 실행하는 방법과 지정된 duration 동안 동시성 수를 제어하는 방법을 확인했습니다. 또한 수행되는 iteration 수는 `default function ()` 블록의 각 iteration을 완료하는 데 필요한 시간에 전적으로 의존한다는 것을 알았습니다.
