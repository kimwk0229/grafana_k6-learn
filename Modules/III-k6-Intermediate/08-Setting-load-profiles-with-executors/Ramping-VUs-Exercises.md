# Ramping VUs Executor

[executor로 부하 프로필 설정하기](../08-Setting-load-profiles-with-executors.md#Ramping-VUs)에서 언급한 바와 같이, _Ramping VUs_ executor는 _stage_ 내의 _virtual user (VU)_ 수에 주된 초점을 둡니다.

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
      executor: 'ramping-vus',
      stages: [
        { target: 10, duration: "30s" },
      ],
    },
  },
};

export default function () {
  console.log(`[VU: ${__VU}, iteration: ${__ITER}] Starting iteration...`);
  http.get('https://test.k6.io/contacts.php');
  sleep(1);
}
```

이 _"기본"_ 스크립트는 이전 예시들보다 약간 더 복잡합니다. `executor` 자체 외에도 `stages`를 정의해야 하기 때문입니다. 최소 하나의 _stage_가 필요하며, 각각 `target`과 `duration` 옵션을 포함해야 하고 둘 다 필수입니다.

### 초기 테스트 실행

k6 CLI를 사용하여 스크립트를 실행해 보겠습니다:

```bash
k6 run test.js
```

스크립트 출력을 보면 여러 virtual user (VU)가 30초 동안 테스트 iteration을 수행한 것을 확인할 수 있습니다.

```bash
INFO[0029] [VU: 2, iteration: 18] Starting iteration...  source=console
INFO[0029] [VU: 7, iteration: 2] Starting iteration...   source=console
INFO[0029] [VU: 4, iteration: 21] Starting iteration...  source=console
INFO[0029] [VU: 6, iteration: 15] Starting iteration...  source=console
INFO[0030] [VU: 5, iteration: 12] Starting iteration...  source=console

running (0m31.0s), 00/10 VUs, 141 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 00/10 VUs  30s
```

> :point_up: iteration 카운터는 0부터 시작하므로, 카운트가 12이면 실제로는 13번의 iteration을 의미합니다.

iteration 카운트를 자세히 살펴보면 이상해 보일 수 있습니다. 카운트가 다양하게 나타납니다: _VU #7_은 3번의 iteration만 수행했지만, _VU #4_는 22번을 수행했습니다. [Constant VUs](../08-Setting-load-profiles-with-executors.md#Constant-VUs) executor와 마찬가지로 각 virtual user가 `default function ()`을 계속 실행하는데, 어떻게 이렇게 큰 차이가 날 수 있을까요?

이 iteration 카운트 차이는 executor의 _ramping_ 측면 때문입니다. k6는 stage 내에서 정의된 `target` VU 수를 달성하기 위해 실행 중인 VU 수를 선형적으로 확장하거나 축소합니다. `duration`은 확장이 얼마나 오래 진행될지를 결정합니다.

`startVUs` 옵션을 지정하지 않았으므로 k6는 기본값인 1을 사용했습니다. 따라서 테스트는 단일 VU로 시작하여 30초에 걸쳐 10개의 VU로 _ramping up_되었습니다. 그렇기 때문에 _VU #7_이 _VU #4_보다 stage에서 훨씬 늦게 활성화되었을 것이라고 추론할 수 있습니다.

```text
VUs
   
10 |                                .......
   |                        ......./
 5 |                ......./
   |        ......./
 1 |......./
   +---------------------------------------+ 30s
                 S T A G E  # 1    
```

### 기울기 반전

`startVUs`와 stage의 `target`을 변경하여 테스트의 기울기를 반전시켜 보겠습니다. 테스트 스크립트의 `options`를 다음과 같이 업데이트합니다:

```js
export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'ramping-vus',
      stages: [
        { target: 1, duration: "30s" },
      ],
      startVUs: 10,
    },
  },
};
```
```bash
INFO[0026] [VU: 3, iteration: 24] Starting iteration...  source=console
INFO[0027] [VU: 6, iteration: 25] Starting iteration...  source=console
INFO[0027] [VU: 7, iteration: 25] Starting iteration...  source=console
INFO[0028] [VU: 6, iteration: 26] Starting iteration...  source=console
INFO[0028] [VU: 7, iteration: 26] Starting iteration...  source=console
INFO[0029] [VU: 6, iteration: 27] Starting iteration...  source=console
INFO[0029] [VU: 7, iteration: 27] Starting iteration...  source=console

running (0m30.5s), 00/10 VUs, 168 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 00/10 VUs  30s
```

이번에는 높은 iteration 카운트를 수행하는 VU 수가 duration이 지남에 따라 줄어드는 것을 확인할 수 있습니다.

```text
VUs
   
10 |.......
   |       \.......
 5 |               \.......
   |                       \.......
 1 |                               \.......
   +---------------------------------------+ 30s
                 S T A G E  # 1    
```

마지막 연습에서처럼 VU를 _ramping down_할 때, k6는 리소스를 회수할 때 어느 정도 유연성을 제공합니다. 기본적으로 제거 대상인 VU에게는 현재 작업 중인 iteration을 완료할 수 있도록 30초의 유예 기간이 주어집니다. 이 유예 기간은 `gracefulRampDown` 옵션을 사용하여 필요에 따라 조정할 수 있습니다.

### 활성 사용자 급증 시뮬레이션

_ramping_의 진정한 힘은 활동 급증을 시뮬레이션하는 능력에 있습니다. 이는 여러 stage를 정의함으로써 가능합니다.

```text
VUs
                   
10 |                 .  
   |                / \ 
 5 |               /   \
   |              /     \............      
 1 |............./                              
   +------------+---+---+------------+ 30s
        #1        #2  #3       #4
```

이 _ASCII 아트_ 다이어그램에서, 4개의 stage를 실행하여 virtual user 급증을 시뮬레이션하도록 스크립트를 수정하겠습니다. `startVUs`, 첫 번째 stage를 변경하고 새로운 stage를 시뮬레이션에 추가합니다:

```js
export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'ramping-vus',
      stages: [
        { target: 1, duration: "12s" },
        { target: 10, duration: "3s" },
        { target: 3, duration: "3s" },
        { target: 3, duration: "12s" },
      ],
      startVUs: 1,
    },
  },
};
```

스크립트를 한 번 더 실행합니다:

```bash
k6 run test.js
```

이제 테스트가 천천히 시작되다가 급증이 발생하면서 빠르게 ramping up되고, 테스트가 끝날 때까지 적당한 속도로 ramping down되는 것을 확인할 수 있습니다.

```bash
INFO[0028] [VU: 3, iteration: 26] Starting iteration...  source=console
INFO[0028] [VU: 2, iteration: 14] Starting iteration...  source=console
INFO[0028] [VU: 7, iteration: 15] Starting iteration...  source=console
INFO[0029] [VU: 3, iteration: 27] Starting iteration...  source=console
INFO[0029] [VU: 2, iteration: 15] Starting iteration...  source=console
INFO[0029] [VU: 7, iteration: 16] Starting iteration...  source=console

running (0m30.7s), 00/10 VUs, 80 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 00/10 VUs  30s
```

### 마무리

이 연습을 통해 테스트 활동을 모델링하기 위해 VU 수를 ramping up 및 down하는 것의 장점을 확인할 수 있었습니다.
