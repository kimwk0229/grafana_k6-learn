# Constant Arrival Rate Executor

[executor로 부하 프로필 설정하기](../08-Setting-load-profiles-with-executors.md#Constant-Arrival-Rate)에서 언급한 바와 같이, _Constant Arrival Rate_ executor는 지정된 시간 동안 적용되는 _iteration rate_에 주된 초점을 둡니다.

이는 특정 API 엔드포인트에 대한 일정한 요청 비율을 시뮬레이션해야 할 때 [API 부하 테스트](https://k6.io/docs/testing-guides/api-load-testing/)에서 일반적으로 볼 수 있는 시나리오입니다.

## 연습 문제

이번 연습은 기본 스크립트로 시작합니다. 각 테스트 iteration에서 HTTP 요청을 실행하고 시간 단위당(기본값은 1초) 50개의 요청을 달성하려 합니다. 변경 사항에 따른 콘솔 출력도 확인할 수 있습니다.

### 스크립트 작성

최소한으로 필요한 설정으로 테스트 스크립트를 구현하는 것부터 시작하겠습니다. 다음 내용으로 _test.js_ 파일을 생성하세요:

```js
import http from 'k6/http';

export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'constant-arrival-rate',
      rate: 50,
      duration: '30s',
      preAllocatedVUs: 2,
    },
  },
};

export default function () {
  console.log(`[VU: ${__VU}, iteration: ${__ITER}] Starting iteration...`);
  http.get('https://test.k6.io/contacts.php');
}
```

executor를 사용하는 데 필요한 최소한의 설정으로 시작합니다. 이전 executor들과 비교했을 때, 일반적인 `executor` 자체 외에 조금 더 많은 필수 설정이 있습니다.

[`constant-arrival-rate` 문서](https://k6.io/docs/using-k6/scenarios/executors/constant-arrival-rate/)에서 언급한 바와 같이 `rate`와 `duration`이 이제 필수입니다. 이 executor의 초점이 제공된 시간 동안 지정된 _iteration rate_를 달성하고 유지하는 것이라는 점에서 당연합니다.

`preAllocatedVUs`에 대한 논의는 초기 테스트가 완료될 때까지 잠시 미뤄두겠습니다.

### 초기 테스트 실행

스크립트의 초기 실행을 시작해 보겠습니다:

```bash
k6 run test.js
```

테스트가 이제 터미널에 결과를 출력하기 시작합니다...

```bash
  execution: local
     script: test.js
     output: -

  scenarios: (100.00%) 1 scenario, 2 max VUs, 1m0s max duration (incl. graceful stop):
           * k6_workshop: 50.00 iterations/s for 30s (maxVUs: 2, gracefulStop: 30s)

INFO[0000] [VU: 1, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 2, iteration: 0] Starting iteration...   source=console
WARN[0000] Insufficient VUs, reached 2 active VUs and cannot initialize more  executor=constant-arrival-rate scenario=k6_workshop
INFO[0000] [VU: 1, iteration: 1] Starting iteration...   source=console
INFO[0000] [VU: 2, iteration: 1] Starting iteration...   source=console
...
INFO[0030] [VU: 2, iteration: 204] Starting iteration...  source=console
INFO[0030] [VU: 1, iteration: 204] Starting iteration...  source=console

running (0m30.1s), 0/2 VUs, 410 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 0/2 VUs  30s  50.00 iters/s

     data_received..................: 325 kB 11 kB/s
     data_sent......................: 47 kB  1.6 kB/s
     dropped_iterations.............: 1091   36.235918/s
     http_req_blocked...............: avg=3.48ms   min=1µs      med=6µs      max=256.93ms p(90)=10µs     p(95)=12µs    
     http_req_connecting............: avg=1.73ms   min=0s       med=0s       max=136.79ms p(90)=0s       p(95)=0s      
     http_req_duration..............: avg=132.35ms min=103.06ms med=115.38ms max=827.03ms p(90)=143.56ms p(95)=165.43ms
       { expected_response:true }...: avg=132.35ms min=103.06ms med=115.38ms max=827.03ms p(90)=143.56ms p(95)=165.43ms
     http_req_failed................: 0.00%  ✓ 0         ✗ 410
     http_req_receiving.............: avg=86.77µs  min=19µs     med=80µs     max=524µs    p(90)=142µs    p(95)=153.54µs
     http_req_sending...............: avg=26.88µs  min=7µs      med=24.5µs   max=194µs    p(90)=39.1µs   p(95)=43.54µs 
     http_req_tls_handshaking.......: avg=1.74ms   min=0s       med=0s       max=126.36ms p(90)=0s       p(95)=0s      
     http_req_waiting...............: avg=132.23ms min=102.98ms med=115.28ms max=826.93ms p(90)=143.43ms p(95)=165.28ms
     http_reqs......................: 410    13.617531/s
     iteration_duration.............: avg=136.06ms min=103.26ms med=115.77ms max=827.24ms p(90)=144.98ms p(95)=183ms   
     iterations.....................: 410    13.617531/s
     vus............................: 2      min=2       max=2
     vus_max........................: 2      min=2       max=2
```

테스트는 성공적으로 실행되었습니다. 그러나 자세히 살펴보면 결과가 의도한 대로 되지 않았습니다.

출력을 보면 다음과 같은 내용이 확인됩니다:

```bash
WARN[0000] Insufficient VUs, reached 2 active VUs and cannot initialize more  executor=constant-arrival-rate scenario=k6_workshop
```

무슨 일이 일어났을까요?

앞서 살짝 언급했던 `preAllocatedVUs` 설정을 기억하시나요? 이 설정으로 k6에게 2개의 virtual user로 테스트를 시작하도록 지시했습니다. 이렇게 적게 할당된 VU로는 k6가 원하는 iteration rate (50 iterations/s)를 **달성하지 못했습니다**.

이 사실은 테스트 요약의 `iterations` 값에서 확인할 수 있습니다. 테스트는 초당 13.61번의 iteration만 달성했으며 우리가 원했던 것은 50이었습니다.

### 사전 할당된 virtual user 조정

다시 말하지만, _Constant Arrival Rate_는 _iteration rate_에 초점을 맞춥니다. k6는 그러한 rate를 달성하는 데 필요한 실제 사용자 수에 대해 과도하게 신경 쓸 필요를 없애는 것을 목표로 합니다. 그러나 여전히 해당 rate를 달성하기에 충분한 VU를 스크립트에 제공해야 합니다.

앞의 예시에서 요청 duration 또는 지연 시간은 평균 132.35ms이며 95 퍼센타일은 약 165.43ms입니다. `preAllocatedVUs`가 2개뿐인 경우 매우 낙관적인 시나리오에서도 `2 VUs / 0.132 s = 15.15 iterations/s` 이상을 기대할 수 없습니다.

`preAllocatedVUs` 수를 조금씩 변경하면서 더 높은 값, 예를 들어 25로 업데이트할 수 있습니다.

```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: 'constant-arrival-rate',
            rate: 50,
            duration: '30s',
            preAllocatedVUs: 25,
        },
    },
};
```

테스트를 다시 실행해 보겠습니다:
```bash
k6 run test.js
```

```bash
running (0m30.5s), 00/25 VUs, 1501 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 00/25 VUs  30s  50.00 iters/s

     data_received..................: 1.2 MB 40 kB/s
     data_sent......................: 174 kB 5.7 kB/s
     http_req_blocked...............: avg=4.26ms   min=1µs      med=4µs      max=332.88ms p(90)=8µs      p(95)=14µs    
     http_req_connecting............: avg=2.05ms   min=0s       med=0s       max=167.98ms p(90)=0s       p(95)=0s      
     http_req_duration..............: avg=136.87ms min=101.78ms med=114.56ms max=862.77ms p(90)=146.68ms p(95)=184.36ms
       { expected_response:true }...: avg=136.87ms min=101.78ms med=114.56ms max=862.77ms p(90)=146.68ms p(95)=184.36ms
     http_req_failed................: 0.00%  ✓ 0         ✗ 1501
     http_req_receiving.............: avg=46.57µs  min=11µs     med=39µs     max=503µs    p(90)=82µs     p(95)=96µs    
     http_req_sending...............: avg=17.21µs  min=5µs      med=15µs     max=151µs    p(90)=28µs     p(95)=30µs    
     http_req_tls_handshaking.......: avg=2.19ms   min=0s       med=0s       max=203.88ms p(90)=0s       p(95)=0s      
     http_req_waiting...............: avg=136.81ms min=101.73ms med=114.5ms  max=862.74ms p(90)=146.57ms p(95)=184.31ms
     http_reqs......................: 1501   49.213783/s
     iteration_duration.............: avg=141.27ms min=101.88ms med=114.83ms max=862.9ms  p(90)=150.38ms p(95)=407.84ms
     iterations.....................: 1501   49.213783/s
     vus............................: 25     min=25      max=25
     vus_max........................: 25     min=25      max=25
```

이제 출력을 검토하면 원하는 rate가 전체 테스트에서 초당 49.21번의 iteration으로 더 가깝게 달성된 것을 확인할 수 있습니다.

[`preAllocatedVUs`에 낮은 값을 설정하고 `maxVUs` 옵션](https://k6.io/docs/using-k6/scenarios/executors/constant-arrival-rate/)을 사용하여 테스트를 자동으로 확장하고 싶을 수 있습니다. `maxVUs`는 테스트 실행 중 허용되는 최대 VU 수로, 설정하지 않으면 기본적으로 `preAllocatedVUs`와 같습니다. 그러나 일반적으로 `maxVUs`의 기본값을 유지하고 `preAllocatedVUs`를 늘리는 것이 가장 좋습니다. 이 방법으로 VU는 테스트 시작 전에 미리 할당되어 필요할 때만 사용됩니다.

테스트 도중에 VU를 할당하면 부하 생성기 인스턴스에서 CPU 리소스 측면에서 비용이 많이 들고, 테스트가 왜곡될 수 있습니다. VU를 사전 할당하면 테스트 실행 중간에 더 많이 할당하기를 기다릴 필요 없이 테스트가 모든 VU를 사용 가능한 상태로 시작됩니다.

### 기타 rate 옵션

지금까지 예시에서 초당 iteration을 기준으로 _iteration rate_를 설정했습니다. `timeUnit` 옵션을 사용하여 rate 분모를 변경할 수 있습니다. 다시 말하지만 기본 설정 값은 `1s`입니다.

예를 들어, 서비스의 로그 집계가 시간당 10,000개의 요청을 받는다고 가정해 보겠습니다. 이를 초당 iteration으로 직접 계산하는 대신 k6에게 작업을 맡길 수 있습니다:

```js
export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'constant-arrival-rate',
      rate: 10000,
      timeUnit: '1h',
      duration: '30s',
      preAllocatedVUs: 5,
    },
  },
};
```

스크립트를 실행하면 초당 2.78번의 iteration 페이스를 보여주며, 5개의 VU로 달성할 수 있습니다.

```bash
  scenarios: (100.00%) 1 scenario, 5 max VUs, 1m0s max duration (incl. graceful stop):
           * k6_workshop: 2.78 iterations/s for 30s (maxVUs: 5, gracefulStop: 30s)
...
INFO[0029] [VU: 1, iteration: 16] Starting iteration...  source=console
INFO[0030] [VU: 5, iteration: 16] Starting iteration...  source=console

running (0m30.0s), 0/5 VUs, 84 complete and 0 interrupted iterations
k6_workshop ✓ [======================================] 0/5 VUs  30s  2.78 iters/s
...
     iterations.....................: 83    2.682393/s
     vus............................: 4     min=3      max=4
```

### 마무리

이 연습을 통해 매우 기본적인 테스트를 실행하는 방법과 원하는 iteration rate를 달성하기 위해 k6가 자동으로 virtual user 수를 제어하도록 허용하는 방법을 확인했습니다.
