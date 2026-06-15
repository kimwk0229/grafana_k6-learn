# scenario를 사용한 워크로드 모델링

[워크로드 모델링](03-Workload-modeling.md)에서 사용자 흐름이 복잡할 수 있다는 것이 도전 과제 중 하나라고 언급했습니다. 단일 테스트가 동시에 여러 엔드 투 엔드 흐름을 다루어야 할 수도 있습니다. 예를 들어, 쇼핑몰 애플리케이션에서 일부 회원들이 제품을 탐색하는 동안 많은 다른 사람들이 (바라건대!) 장바구니에 항목을 추가하고 있을 수 있습니다. 이러한 다양한 흐름을 구성하는 데 도움이 되도록, k6 팀은 [scenarios](https://k6.io/docs/using-k6/scenarios/)를 만들었습니다.

_scenarios_를 사용하면 스크립트 작성자가 서로 다른 실행 패턴과 환경 설정을 가진 별개의 흐름을 만들 수 있습니다. Scenario는 `startTime` 옵션으로 시간 오프셋을 사용하여 병렬로 또는 순차적으로 실행될 수 있습니다.

각 흐름을 다른 scenario로 분리하면, 전체 스크립트가 더 간소화되고 논리가 따라가기 쉬워질 수 있습니다. 쇼핑몰 예에서, 한 scenario는 장바구니에 항목을 채우고 주문을 하는 사용자가 될 것입니다. 또 다른 scenario는 특정 제품의 가용성을 검색하는 다른 고객들을 나타낼 것입니다. 각 워크플로우는 복잡할 수 있습니다. 이를 분리하면 스크립트 유지 관리가 더 쉬워집니다.

## Scenario 정의하기

각 _scenario_에는 `executor`, `startTime`, 그리고 선택적으로 _scenario_에만 해당하는 `env` 변수와 `tags` 외에 [워크로드 모델의 동일한 요소](03-Workload-modeling.md#Elements-of-a-workload-model)가 포함됩니다. 전체 scenario 옵션 목록은 [문서](https://k6.io/docs/using-k6/scenarios/)에서 확인하세요.

다음 스크립트를 `playground.js`로 저장하세요.

```javascript
import { sleep } from 'k6';

export const options = {
    scenarios: {
        // Scenario 1: Users browsing through our product catalog
        users_browsing_products: {
            executor: 'shared-iterations', // name of the executor to use
            env: { WORKFLOW: 'browsing' }, // environment variable for the workflow

            // executor-specific configuration
            vus: 3,
            iterations: 10,
            maxDuration: '10s',
        },
        // Scenario 2: Users adding items to their cart and checking out
        users_buying_products: {
            executor: 'per-vu-iterations', // name of the executor to use
            exec: 'usersBuyingProducts', // js function for the purchasing workflow
            env: { WORKFLOW: 'buying' }, // environment variable for the workflow
            startTime: '2s', // delay start of purchasing workflow
            
            // executor-specific configuration
            vus: 3,
            iterations: 1,
            maxDuration: '10s',
        },
    },
};

export default function () {
  // Model your workflow for searching through products
  console.log(`[VU: ${__VU}, iteration: ${__ITER}, workflow: ${__ENV.WORKFLOW}] Just looking!!!`);
  sleep(1);
}

export function usersBuyingProducts() {
  // Model your workflow for adding items to cart and checking out
  console.log(`[VU: ${__VU}, iteration: ${__ITER}, workflow: ${__ENV.WORKFLOW}] CHA-CHING!!!`);
  sleep(1);
}
```

스크립트를 `k6 run playground.js`로 실행하면 다음과 유사한 결과가 생성됩니다:

```plain
$ k6 run playground.js

          /\      |‾‾| /‾‾/   /‾‾/   
     /\  /  \     |  |/  /   /  /    
    /  \/    \    |     (   /   ‾‾\  
   /          \   |  |\  \ |  (‾)  | 
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: playground.js
     output: -

  scenarios: (100.00%) 2 scenarios, 6 max VUs, 42s max duration (incl. graceful stop):
           * users_browsing_products: 10 iterations shared among 3 VUs (maxDuration: 10s, gracefulStop: 30s)
           * users_buying_products: 1 iterations for each of 3 VUs (maxDuration: 10s, exec: usersBuyingProducts, startTime: 2s, gracefulStop: 30s)

INFO[0000] [VU: 4, iteration: 0, workflow: browsing] Just looking!!!  source=console
INFO[0000] [VU: 1, iteration: 0, workflow: browsing] Just looking!!!  source=console
...
INFO[0002] [VU: 1, iteration: 2, workflow: browsing] Just looking!!!  source=console
INFO[0002] [VU: 5, iteration: 0, workflow: buying] CHA-CHING!!!  source=console
INFO[0003] [VU: 4, iteration: 3, workflow: browsing] Just looking!!!  source=console

running (04.0s), 0/6 VUs, 13 complete and 0 interrupted iterations
users_browsing_products ✓ [======================================] 3 VUs  04.0s/10s  10/10 shared iters
users_buying_products   ✓ [======================================] 3 VUs  01.0s/10s  3/3 iters, 1 per VU
```

결과를 보면, 사용자들이 제품을 탐색하다가 초기 지연 후 다른 사용자가 구매를 하고 있다는 것을 알 수 있습니다.

## 구성 옵션

우리의 예는 _scenarios_에 대한 최소한의 구성 옵션을 사용했지만, 사용 가능한 옵션에는 다음이 포함됩니다:

> 지정된 `executor`에 따라 추가 옵션이 필요할 수 있다는 것을 기억하세요. _executor_에 대한 자세한 내용은 [executor로 부하 프로필 설정하기](08-Setting-load-profiles-with-executors.md)를 참조하세요.

| 옵션           | 설명                                                                                       | 기본값        |
|----------------|--------------------------------------------------------------------------------------------|--------------|
| **`executor`** | 트래픽 패턴을 "형성"하는 특정 [executor](https://k6.io/docs/using-k6/scenarios/executors/) | - (필수)     |
| `env`          | scenario에만 해당하는 환경 변수                                                             | `{}`         | 
| `exec`         | scenario의 각 iteration에 대해 실행할 내보낸 JS 함수의 이름                                  | `"default"`  | 
| `gracefulStop` | 강제 종료 전에 iteration이 완료될 수 있도록 허용하는 기간                                    | `"30s"`      |
| `startTime`    | 테스트 내에서 scenario를 시작하는 시간 오프셋                                                | `"0s"`       |
| `tags`         | scenario에만 해당하는 태그                                                                   | `{}`         | 

우리의 예를 더 자세히 살펴보면, `users_browsing_products`와 `users_buying_products`라는 두 가지 scenario를 만들었습니다.

`users_browsing_products` scenario의 아이디어는 테스트에 대한 _배경 활동_을 만드는 것입니다. `shared-iterations` executor를 사용하여 세 명의 사용자가 제품 카탈로그를 탐색하는 것을 시뮬레이션합니다. 이 활동을 10번의 iteration으로 제한하고 스크립트가 10초를 초과하지 않도록 합니다. `env` 옵션은 `WORKFLOW` 환경 변수를 `browsing`으로 설정하는데, 이는 JavaScript 내에서 일부 실행 컨텍스트를 제공하는 데 사용될 수 있습니다. `exec` 옵션을 _지정하지 않기_ 때문에, 우리의 scenario는 `default`인 JavaScript 함수를 실행합니다.

`users_buying_products`는 세 명의 다른 사용자가 구매를 시뮬레이션하는 두 번째 scenario입니다. 이 scenario는 활동을 제어하기 위해 `per-vu-iterations`를 사용합니다. scenario는 각 scenario iteration에 대해 구매 로직을 호출하기 위해 `exec: 'usersBuyingProducts'`를 지정합니다. 이전 scenario와 마찬가지로, `WORKFLOW` 환경 변수를 주입하여 스크립트에 scenario에 대한 일부 컨텍스트를 제공합니다. 이번에는 `startTime: '2s'`를 지정하여 scenario에 대한 지연을 정의합니다. 이렇게 하면 `users_browsing_products`가 "헤드 스타트"를 얻어 사용자가 구매를 시작하기 전에 배경 활동을 생성합니다.

## 지식 확인

### Question 1
_참_ 또는 _거짓_, scenario를 더 광범위한 테스트 내의 활동 레이어로 생각할 수 있습니까?

### Question 2
_참_ 또는 _거짓_, 각 scenario에 대해 `exec` 옵션을 지정해야 합니까?

### Question 3
_참_ 또는 _거짓_, `exec` 옵션이 scenario의 트래픽 _형태_를 결정합니까?

### 정답
1. 참. 각 scenario는 독립적으로 작동하지만 다른 scenario의 영향을 받을 수 있습니다. 우리의 예에서, _배경_에 사용자 탐색 활동을 정의하고, _전경_에 구매하는 사용자를 두었습니다.
2. 거짓. `exec` 옵션을 명시적으로 구성하는 것이 권장되지만, _필수_는 아닙니다. `exec` 옵션 없이 여러 scenario를 구성하면 각각이 동일한 `default` 함수를 실행하게 되는데, 이는 혼란스러울 수 있습니다.
3. 거짓. `executor`가 궁극적으로 scenario의 부하 프로필 패턴을 정의합니다. `exec`는 scenario의 각 iteration에서 어떤 JavaScript 함수가 실행되는지를 나타냅니다.
