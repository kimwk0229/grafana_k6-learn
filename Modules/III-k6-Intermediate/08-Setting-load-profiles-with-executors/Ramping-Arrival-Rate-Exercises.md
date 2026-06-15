# Ramping Arrival Rate Executor

[executor로 부하 프로필 설정하기](Setting-load-profiles-with-executors.md#Ramping-Arrival-Rate)에서 언급한 바와 같이, _Ramping Arrival Rate_ executor는 _stage_ 내의 지정된 duration 동안 적용되는 _iteration rate_에 주된 초점을 둡니다.

이는 API를 한계점 이상으로 점진적으로 밀어붙이거나 매우 짧은 기간에 극도의 부하로 급증을 시뮬레이션해야 할 때 [stress 또는 spike 테스트](https://k6.io/docs/test-types/stress-testing/)에서 일반적으로 볼 수 있는 시나리오입니다.

## 연습 문제

이번 연습에서는 HTTP 요청을 실행하고 정의하는 유일한 _stage_ 동안 시간 단위당(기본값은 1초) 30개의 요청을 달성하려는 기본 스크립트로 시작합니다. 변경 사항에 따른 콘솔 출력도 확인할 수 있습니다.

### 스크립트 작성

테스트 스크립트를 구현하는 것부터 시작하겠습니다. 다음 내용으로 _test.js_ 파일을 생성하세요:

```js
import http from 'k6/http';

export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'ramping-arrival-rate',
      stages: [
        { target: 30, duration: "30s" },
      ],
      preAllocatedVUs: 2,
    },
  },
};

export default function () {
  console.log(`[VU: ${__VU}, iteration: ${__ITER}] Starting iteration...`);
  http.get('https://test.k6.io/contacts.php');
}
```

이 "기본" 스크립트는 이전 대부분의 예시보다 약간 더 복잡합니다. `executor` 자체 외에도 `stages`를 정의해야 하기 때문입니다. 최소 하나의 _stage_가 필요하며, 각각 `target`과 `duration`을 포함해야 합니다. 이 경우 `target`은 `stage` 내의 `duration` 종료 시 달성해야 할 원하는 _iteration rate_를 나타냅니다. 초기 테스트 실행이 완료될 때까지 필수 `preAllocatedVUs`에 대한 논의는 잠시 미뤄두겠습니다.

### 초기 테스트 실행

k6 CLI를 사용하여 스크립트를 실행해 보겠습니다:

```bash
k6 run test.js
```

테스트가 이제 터미널에 결과를 출력하기 시작합니다...

```bash
  execution: local
     script: test.js
     output: -

  scenarios: (100.00%) 1 scenario, 2 max VUs, 1m0s max duration (incl. graceful stop):
           * k6_workshop: Up to 30.00 iterations/s for 30s over 1 stages (maxVUs: 2, gracefulStop: 30s)

INFO[0001] [VU: 1, iteration: 0] Starting iteration...   source=console
INFO[0002] [VU: 2, iteration: 0] Starting iteration...   source=console
INFO[0002] [VU: 1, iteration: 1] Starting iteration...   source=console
...
INFO[0015] [VU: 1, iteration: 55] Starting iteration...  source=console
INFO[0015] [VU: 2, iteration: 56] Starting iteration...  source=console
WARN[0015] Insufficient VUs, reached 2 active VUs and cannot initialize more  executor=ramping-arrival-rate scenario=k6_workshop
INFO[0015] [VU: 1, iteration: 56] Starting iteration...  source=console
INFO[0015] [VU: 2, iteration: 57] Starting iteration...  source=console
...
INFO[0030] [VU: 2, iteration: 157] Starting iteration...  source=console
INFO[0030] [VU: 1, iteration: 156] Starting iteration...  source=console

running (0m30.1s), 0/2 VUs, 315 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 0/2 VUs  30s  29.95 iters/s

     data_received..................: 246 kB 8.2 kB/s
     data_sent......................: 36 kB  1.2 kB/s
     dropped_iterations.............: 134    4.455953/s
     http_req_blocked...............: avg=2.96ms   min=2µs      med=8µs      max=280.3ms  p(90)=12µs     p(95)=16µs    
     http_req_connecting............: avg=1.37ms   min=0s       med=0s       max=121.04ms p(90)=0s       p(95)=0s      
     http_req_duration..............: avg=116.57ms min=100.21ms med=112.53ms max=423.4ms  p(90)=133.78ms p(95)=151.35ms
       { expected_response:true }...: avg=116.57ms min=100.21ms med=112.53ms max=423.4ms  p(90)=133.78ms p(95)=151.35ms
     http_req_failed................: 0.00%  ✓ 0         ✗ 315
     http_req_receiving.............: avg=93.73µs  min=25µs     med=94µs     max=244µs    p(90)=146µs    p(95)=158.29µs
     http_req_sending...............: avg=31.07µs  min=8µs      med=30µs     max=470µs    p(90)=42.6µs   p(95)=47µs    
     http_req_tls_handshaking.......: avg=1.49ms   min=0s       med=0s       max=133.12ms p(90)=0s       p(95)=0s      
     http_req_waiting...............: avg=116.45ms min=100.05ms med=112.39ms max=423.24ms p(90)=133.68ms p(95)=151.19ms
     http_reqs......................: 315    10.474815/s
     iteration_duration.............: avg=119.81ms min=100.54ms med=112.83ms max=423.76ms p(90)=140.04ms p(95)=154.62ms
     iterations.....................: 315    10.474815/s
     vus............................: 2      min=2       max=2
     vus_max........................: 2      min=2       max=2
```

_성공_이긴 하지만, 결과를 자세히 살펴보면 테스트 동작이 의도한 대로 되지 않았습니다.

출력을 보면 다음과 같은 내용이 확인됩니다:

```bash
WARN[0015] Insufficient VUs, reached 2 active VUs and cannot initialize more  executor=ramping-arrival-rate scenario=k6_workshop
```

무슨 일이 일어났을까요?

앞서 살짝 언급했던 `preAllocatedVUs` 설정으로 k6에게 2개의 virtual user로 테스트를 시작하도록 지시했습니다. 이 사전 할당으로 인해 k6는 stage 내에서 원하는 iteration rate를 **달성하지 못했습니다**.

이 사실은 테스트 요약의 `iterations` 값에서 확인할 수 있습니다. 테스트는 초당 10.47번의 iteration만 달성했으며 우리가 원했던 것은 30이었습니다.

### 사전 할당된 virtual user 조정

다시 한번 말하지만, _Ramping Arrival Rate_ executor는 각 stage 내의 시간 동안 _iteration rate_에 초점을 맞춥니다. k6는 그러한 rate를 달성하는 데 필요한 실제 사용자 수에 대해 과도하게 신경 쓸 필요를 없애는 것을 목표로 합니다. 그러나 여전히 해당 rate를 달성하기에 충분한 VU를 스크립트에 제공해야 합니다.

위의 예시에서 요청 duration 또는 지연 시간은 평균 116.57ms이며 95 퍼센타일은 약 151.35ms입니다. `preAllocatedVUs`가 2개뿐인 경우 매우 낙관적인 시나리오에서도 `2 VUs / 0.116 s = 17.24 iterations/s` 이상을 기대하기 어렵습니다. 이것이 유일한 요인이 아니라는 것은 [다음 섹션](#ramping-effect)에서 확인할 수 있습니다.

`preAllocatedVUs` 수를 조금씩 변경하면서 더 높은 값, 예를 들어 10으로 업데이트할 수 있습니다.

```js
export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'ramping-arrival-rate',
      stages: [
        { target: 30, duration: "30s" },
      ],
      preAllocatedVUs: 10,
    },
  },
};
```

테스트를 다시 실행해 보겠습니다:
```bash
k6 run test.js
```

```bash
running (0m30.1s), 00/10 VUs, 449 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 00/10 VUs  30s  29.95 iters/s

     data_received..................: 374 kB 12 kB/s
     data_sent......................: 53 kB  1.8 kB/s
     http_req_blocked...............: avg=5.19ms   min=2µs      med=8µs      max=254.28ms p(90)=12µs     p(95)=13µs    
     http_req_connecting............: avg=2.46ms   min=0s       med=0s       max=122.56ms p(90)=0s       p(95)=0s      
     http_req_duration..............: avg=115.77ms min=101.27ms med=110.42ms max=182.21ms p(90)=134.9ms  p(95)=143.31ms
       { expected_response:true }...: avg=115.77ms min=101.27ms med=110.42ms max=182.21ms p(90)=134.9ms  p(95)=143.31ms
     http_req_failed................: 0.00%  ✓ 0         ✗ 449 
     http_req_receiving.............: avg=87.06µs  min=23µs     med=85µs     max=942µs    p(90)=130µs    p(95)=150.19µs
     http_req_sending...............: avg=31.32µs  min=9µs      med=29µs     max=386µs    p(90)=43µs     p(95)=49.19µs 
     http_req_tls_handshaking.......: avg=2.64ms   min=0s       med=0s       max=131.24ms p(90)=0s       p(95)=0s      
     http_req_waiting...............: avg=115.65ms min=101.19ms med=110.29ms max=182.12ms p(90)=134.75ms p(95)=143.19ms
     http_reqs......................: 449    14.929648/s
     iteration_duration.............: avg=121.23ms min=101.49ms med=110.94ms max=367.86ms p(90)=138.57ms p(95)=150.07ms
     iterations.....................: 449    14.929648/s
     vus............................: 10     min=10      max=10
     vus_max........................: 10     min=10      max=10
```

iterations/s가 증가했음을 확인할 수 있습니다. 하지만 아직 충분하지 않습니다.

### Ramping 효과

이전 요약을 보면 _iteration rate_가 초당 14.92번의 iteration으로 끝났고 우리가 원했던 것은 30이었습니다! VU 수만이 _iteration rate_를 제어할 때 작용하는 요인이 아닙니다.

이것이 이 executor의 _ramping_ 측면을 보여줍니다. k6는 stage 내에서 `target` rate를 달성하기 위해 iteration rate를 선형적으로 확장하거나 축소합니다. `duration`은 ramping up/down이 얼마나 오래 진행될지를 결정합니다.

`startRate` 옵션을 지정하지 않았으므로 k6는 기본값인 초당 0번의 iteration을 사용했습니다. 따라서 테스트는 0에서 시작하여 30초에 걸쳐 초당 30번의 iteration으로 _ramping up_되었습니다.

```text
Rate
   
30/s |                                .......
     |                        ......./
     |                ......./
     |        ......./
     |......./
 0/s +---------------------------------------+ 30s
                 S T A G E  # 1    
```

이 선형적인 진행 때문에 전체 rate가 14.92 iterations/s인 것이 타당합니다. 이는 대략 0 - 30의 중간 지점과 같습니다.

### 기울기 변경

_ramping_에서 각 stage는 `duration` 종료 시 달성할 `target`을 정의합니다. 첫 번째 stage의 경우, 시작 rate는 `startRate` 옵션으로 정의됩니다. 앞서 언급한 바와 같이 생략하면 기본 시작 rate는 0입니다. 다음 예시에서 `startRate`를 제공하여 기울기를 _평평하게_ 만들어 보겠습니다:

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: 'ramping-arrival-rate',
            startRate: 30,
            stages: [
                { target: 30, duration: "30s" },
            ],
            preAllocatedVUs: 10,
        },
    },
};
```

`k6 run test.js`로 테스트를 다시 실행하면 다음과 같은 결과가 나타납니다:

```bash
running (0m30.8s), 00/10 VUs, 897 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 00/10 VUs  30s  30.00 iters/s

...
     iteration_duration.............: avg=130.68ms min=101.77ms med=112.27ms max=1.54s    p(90)=145.2ms  p(95)=165.15ms
     iterations.....................: 897    29.089145/s
     vus............................: 10     min=10      max=10
     vus_max........................: 10     min=10      max=10
```
이번에는 30의 iteration rate로 테스트를 시작하도록 요청했고, 10개의 VU를 사용했으며, 이것으로 해당 rate에 가깝게 도달하기에 충분했습니다.

```text
Rate
   
30/s |........................................
     |       
     |
 0/s +---------------------------------------+ 30s
                 S T A G E  # 1    
```

[`preAllocatedVUs`에 낮은 값을 설정하고 `maxVUs` 옵션](https://k6.io/docs/using-k6/scenarios/executors/ramping-arrival-rate/)을 사용하여 테스트를 자동으로 확장하고 싶을 수 있습니다. `maxVUs`는 테스트 실행 중 허용되는 최대 VU 수로, 설정하지 않으면 기본적으로 `preAllocatedVUs`와 같습니다. 그러나 일반적으로 `maxVUs`의 기본값을 유지하고 `preAllocatedVUs`를 늘리는 것이 가장 좋습니다. 이 방법으로 VU는 테스트 시작 전에 미리 할당되어 필요할 때만 사용됩니다.

테스트 도중에 VU를 할당하면 부하 생성기 인스턴스에서 CPU 리소스 측면에서 비용이 많이 들고, 테스트가 왜곡될 수 있습니다. VU를 사전 할당하면 테스트 실행 중간에 더 많이 할당하기를 기다릴 필요 없이 테스트가 모든 VU를 사용 가능한 상태로 시작됩니다.

단일 stage에 평탄하거나 _일정한_ rate를 원한다면 대신 _Constant Arrival Rate_ executor를 사용하면 됩니다! 이 executor는 다음 섹션에서 볼 수 있는 것처럼 [stress 또는 spike 테스트](https://k6.io/docs/test-types/stress-testing/)에 유용할 수 있습니다.

### 급증을 시뮬레이션하기 위한 stage 추가

_ramping_의 진정한 힘은 활동 급증을 시뮬레이션하는 능력에 있습니다. 이는 여러 stage를 정의함으로써 가능합니다.

오픈 직후 활동 급증이 예상되는 대규모 세일이 있다고 가정해 봅시다. 급증 후에는 잠시 머물다가 평소보다 높은 안정된 페이스로 수그러든다고 합시다:

```text
Rate
                  * Door-buster Savings!!
50/s |               .....
     |              /     \.....
30/s |             /            \............
     |             |
10/s |............/
     +------------+--+---+-------+-----------+ 30s
           #1      #2  #3   #4         #5
```
다음과 같이 스크립트의 `options` 섹션을 업데이트하여 이를 시뮬레이션할 수 있습니다:
```js
export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'ramping-arrival-rate',
      startRate: 10,
      stages: [
        // Level at 10 iters/s for 6 seconds
        { target: 10, duration: "6s" },
        // Spike from 10 iters/s to 50 iters/s in 3 seconds!
        { target: 50, duration: "3s" },
        // Level at 50 iters/s for 5 seconds
        { target: 50, duration: "5s" },
        // Slowing down from 50 iters/s to 30 iters/s over 8 seconds
        { target: 30, duration: "8s" },
        // Leveled off at 30 iters/s for remainder
        { target: 30, duration: "8s" },
      ],
      preAllocatedVUs: 50,
    },
  },
};
```

### 기타 rate 옵션

지금까지 예시에서 초당 iteration을 기준으로 _iteration rate_를 설정했습니다. `timeUnit` 옵션을 사용하여 rate 분모를 변경할 수 있습니다. 다시 말하지만 기본 설정 값은 `1s`입니다.

> :point_up: `timeUnit`은 모든 stage 내의 `target` rate와 `startRate`에 적용됩니다.

예시에서 rate를 초당 iteration에서 분당 iteration으로 변경하여 속도를 늦춰 보겠습니다.
```js
export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'ramping-arrival-rate',
      startRate: 10,
      timeUnit: '1m',
      stages: [
        // Level at 10 iters/minute for 6 seconds
        { target: 10, duration: "6s" },
        // Spike from 10 iters/minute to 50 iters/minute in 3 seconds!
        { target: 50, duration: "3s" },
        // Level at 50 iters/minute for 5 seconds
        { target: 50, duration: "5s" },
        // Slowing down from 50 iters/minute to 30 iters/minute over 8 seconds
        { target: 30, duration: "8s" },
        // Leveled off at 30 iters/minute for remainder
        { target: 30, duration: "8s" },
      ],
      preAllocatedVUs: 50,
    },
  },
};
```

### 마무리

이 연습을 통해 iteration rate를 ramping up 및 down하여 테스트 활동을 보다 현실적으로 모델링하는 능력의 장점을 확인할 수 있었습니다.
