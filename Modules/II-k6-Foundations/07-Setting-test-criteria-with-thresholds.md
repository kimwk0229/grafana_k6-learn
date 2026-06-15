k6는 개발자와 테스터가 각 테스트에 대한 목표를 정의하도록 권장합니다. 이를 위해 테스트가 특정 기준에 맞게 수행되는지 평가하는 [임계값(thresholds)](https://k6.io/docs/using-k6/thresholds)을 설정할 수 있습니다. 예를 들어, 테스트가 실행되는 동안 시스템이 서비스 수준 목표(SLO) 내에서 성능을 발휘하는지 확인하기 위해 thresholds를 사용할 수 있습니다.

테스트 통과 또는 실패를 결정하기 위해 thresholds를 사용할 수도 있습니다. 부하 테스트 스크립트에 thresholds를 추가하는 것은 k6가 해당 thresholds가 위반될 때 경고하거나 테스트를 중단하도록 지시하기 때문에 유용합니다. 코드의 일부로 thresholds를 포함하면 다른 동료가 테스트를 실행하는 데 더 쉽게 참여할 수 있습니다.

CI/CD 파이프라인 내에서 부하 테스트를 실행하려는 경우, 실패가 명확하게 기록되도록 k6가 0이 아닌 종료 코드를 보내기를 원할 것입니다.

테스트 옵션 객체의 k6 스크립트에 thresholds를 추가할 수 있습니다:

```js
export let options = {
  stages: [
    { duration: '30m', target: 100 },
    { duration: '1h', target: 100 },
    { duration: '5m', target: 0 },
  ],
  thresholds: {
    http_req_failed: ['rate<=0.05'],
    http_req_duration: ['p(95)<=5000'],
  },
};
```

thresholds는 항상 메트릭을 기반으로 합니다. [내장 메트릭의 전체 목록](https://k6.io/docs/using-k6/metrics/#built-in-metrics)을 검토할 수 있습니다.

thresholds는 기대되는 것에 대한 진술로 표현됩니다:
- threshold 진술이 `true`로 평가되면, threshold가 충족되고 테스트가 통과됩니다. k6가 반환하는 종료 코드는 0입니다.
- threshold 진술이 `false`이면, threshold가 충족되지 않아 테스트가 실패하고, k6는 0이 아닌 종료 코드를 반환합니다.

## thresholds 유형

다음은 설정할 수 있는 가장 일반적인 thresholds 유형입니다. 단일 테스트에서 여러 유형의 thresholds를 설정할 수 있습니다:
- 오류율
- 응답 시간
- check

**테스트 모범 사례**: 가능한 경우 테스트에서 오류율, 응답 시간 및 check thresholds를 사용하세요.

### 오류율

```js
thresholds: {
    http_req_failed: ['rate<=0.05'],
},
```

오류율에 대한 threshold를 추가하려면, `http_req_failed` 메트릭을 사용하고 테스트가 충족되길 기대하는 오류율을 입력하세요. 기본적으로 `http_req_failed`는 모든 HTTP 4xx 및 HTTP 5xx 오류를 실패로 계산합니다. 이 동작은 [`setResponseCallback()`](https://k6.io/docs/javascript-api/k6-http/setresponsecallback-callback/)으로 변경할 수 있습니다.

위의 threshold는 테스트 중 오류율이 5% 이하인 경우에만 충족됩니다.

#### 경험칙: 오류율

좋은 오류율이란 무엇일까요? 이는 테스트, 스크립트, 테스트 데이터, 애플리케이션 및 사용자의 오류에 대한 허용 범위에 따라 다릅니다:
- 미션 크리티컬 애플리케이션의 경우 1%와 같이 낮은 오류율을 고려하세요.
- 보조적이거나 중요하지 않은 애플리케이션의 경우 5%와 같은 높은 오류율을 고려하세요.
- 재해 복구 상황 테스트의 경우, 의도적으로 서버 노드를 종료하거나 재시작하는 경우 10-15%의 오류율이 허용 가능할 수 있습니다.

### 응답 시간

```js
thresholds: {
    http_req_duration: ['p(95)<=5000'],
},
```

위의 예시에서 응답 시간 threshold는 5000밀리초의 95번째 백분위수 응답 시간으로 설정되어 있습니다. 이 진술은 모든 HTTP 요청의 95%가 5초 이하의 응답 시간을 가지면 참입니다.

[k6 테스트 종료 요약](03-Understanding-k6-results.md)에 표시된 메트릭 중 하나를 사용하여 응답 시간 threshold를 추가할 수 있습니다:

```plain
http_req_duration..............: avg=151.06ms min=151.06ms med=151.06ms max=151.06ms p(90)=151.06ms p(95)=151.06ms
```

여기에는 다음이 포함됩니다:
- 평균: `http_req_duration: ['avg<=5000'],`
- 최솟값: `http_req_duration: ['min<=1000'],`
- 중앙값: `http_req_duration: ['med<=3000'],`
- 최댓값: `http_req_duration: ['max<=6000'],`
- 백분위수: `http_req_duration: ['p(95)<=5000'],`

**테스트 모범 사례**: 이러한 메트릭들은 모두 극단적인 응답 시간 이상값의 존재에 의해 왜곡될 수 있지만, 어떤 것을 선택해야 할지 모르겠다면 95번째 백분위수 응답 시간으로 시작하는 것을 권장합니다.

#### 경험칙: 응답 시간

[![Google mobile-page-speed-new-industry-benchmarks](../../images/52qQi-marketing-strategies-app-and-mobile-page-load-time-statistics-downlo.jpg)](https://www.thinkwithgoogle.com/marketing-strategies/app-and-mobile/page-load-time-statistics/)

[Google의 연구에 따르면](https://www.thinkwithgoogle.com/marketing-strategies/app-and-mobile/mobile-page-speed-new-industry-benchmarks/), 응답 시간이 3초로 늘어날 때 잠재 고객이 사이트를 떠날 확률이 32% 증가합니다.

확신이 없다면, 시작점으로 2초의 95번째 백분위수 응답 시간을 사용하는 것을 권장합니다.

#### 여러 응답 시간 threshold 사용하기

동일한 메트릭에 대해 여러 thresholds를 배열로 연결할 수도 있으며, 응답 시간이 이에 적합한 후보입니다:

```js
thresholds: {
    http_req_duration: ['p(90) < 400', 'p(95) < 800', 'p(99.9) < 2000'],
},
```

위의 threshold는 다음을 진술합니다:
- 모든 HTTP 요청의 90%는 400ms 미만의 응답 시간을 가져야 합니다.
- 모든 HTTP 요청의 95%는 800ms 미만의 응답 시간을 가져야 합니다.
- 모든 HTTP 요청의 99.9%는 2000ms 미만의 응답 시간을 가져야 합니다.

> :warning: 동일한 메트릭에 대해 여러 thresholds 지정하기. `http_req_duration`과 같이 동일한 메트릭에 대해 하나 이상의 threshold를 포함하려면 배열로 선언해야 **합니다**. 아래 예시는 작동하지 **않으며**, 마지막 줄만 threshold로 사용될 것입니다:

```js
thresholds: {
  http_req_duration: ['p(90) < 400'],
  http_req_duration: ['p(95) < 800'],
  http_req_duration: ['p(99.9) < 2000'],
},
```

### check

[스크립트에 check 추가하기](04-Adding-checks-to-your-script.md)에서 배웠듯이, 실패한 check는 보고되지만 전체 테스트의 상태에 영향을 미치지 않습니다. 특정 check 오류율에 도달했을 때 테스트가 실패하게 하려면, check와 thresholds의 조합을 사용할 수 있습니다:

```js
thresholds: {
  checks: ['rate>=0.9'],
},
```

위의 threshold는 테스트의 모든 check 중 90% 이상이 성공해야 한다고 진술합니다. 그렇지 않으면 테스트가 실패합니다.

## 실패 시 테스트 중단

thresholds가 충족되지 않을 때 테스트를 중단하고 싶은 상황이 있을 수 있습니다. 예를 들어, 애플리케이션 구성 요소가 응답하지 않아 테스트에서 매우 높은 오류율이 발생하는 경우, 테스트를 중단하고 문제를 해결하는 것이 더 나을 수 있습니다.

이러한 상황에서는 관련 threshold를 확장하여 threshold가 충족되지 않으면 k6가 테스트를 중단하도록 지시할 수 있습니다. 다음 대신:

```js
thresholds: {
    http_req_failed: ['rate<=0.05'],
},
```

다음을 시도하세요:

```js
thresholds: {
    http_req_failed: [{
      threshold: 'rate<=0.05',
      abortOnFail: true,
    }],
},
```

추가 파라미터 `abortOnFail: true`는 threshold가 초과되는 즉시 k6가 테스트를 중단하도록([graceful stop](https://k6.io/docs/misc/glossary/#graceful-stop) 없이) 지시합니다. 이 경우 오류율이 5%를 초과하면 발생합니다. k6는 다음과 같은 오류와 함께 테스트 종료 요약 보고서를 표시합니다:

```plain
ERRO[0012] some thresholds have failed 
```

이것은 threshold 실패로 인해 테스트가 중단되었음을 알려줍니다.

## 직접 실행해보세요!

스크립트는 다음과 같아야 합니다:

```js
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '30m', target: 100 },
    { duration: '1h', target: 100 },
    { duration: '5m', target: 0 },
  ],
  thresholds: {
    http_req_failed: [{
      threshold: 'rate<=0.05',
      abortOnFail: true,
    }],
    http_req_duration: ['p(95)<=100'],
    checks: ['rate>=0.99'],
  },
};

export default function() {
  let url = 'https://httpbin.test.k6.io/post';
  let response = http.post(url, 'Hello world!');
  check(response, {
      'Application says hello': (r) => r.body.includes('Hello world!')
  });

  sleep(Math.random() * 5);
}
```

스크립트를 복사하고 저장한 후 `k6 run test.js`를 실행하여 thresholds가 있는 테스트를 실행해보세요.

## 지식 확인

### 문제 1

팀이 95번째 백분위수 응답 시간을 2초로 목표로 결정했습니다. 이를 threshold로 표현하는 가장 좋은 방법은?

A: `http_req_duration: ['p(95)>2000'],`

B: `http_response_time: ['avg<=2000'],`

C: `http_req_duration: ['p(95)<=2000'],`

### 문제 2

테스트 스크립트에 다음 threshold가 정의되어 있습니다:

```js
thresholds: {
    checks: ['rate>=0.99'],
},
```

테스트가 각각 1번의 HTTP 요청으로 100번 반복됩니다. 3번의 HTTP 요청이 check를 실패합니다. 다음 중 어떤 결과를 기대할 수 있나요?

A: check threshold 실패로 인해 테스트가 중단됩니다.

B: check 실패는 테스트를 실패시키지 않으므로 테스트가 완료되고 성공으로 표시됩니다.

C: 테스트가 완료되지만 일부 thresholds가 실패했다는 오류가 있을 것입니다.

### 문제 3

테스트 스크립트에 다음 threshold가 정의되어 있습니다:

```js
thresholds: {
	http_req_failure: ['rate<=0.05'],
	http_req_failure: ['rate<=0.03'],
	http_req_duration: ['p(90)<4000'],
}
```

다음 중 참인 진술은?

A: 오류율이 5% 이상이면 테스트가 실패합니다.

B: 오류율이 3%를 초과하면 테스트가 실패합니다.

C: 90번째 백분위수 응답 시간이 4초이면 테스트가 통과합니다.

## 임계값에 대한 참고 사항

thresholds는 테스트 실행의 성공 또는 실패에 대한 초기 지표로 유용합니다. 그러나 테스트를 평가하는 _유일한_ 방법으로 사용해서는 안 됩니다.

thresholds의 주요 한계는 메트릭을 기반으로 하며, 메트릭은 극도로 낮거나 높은 소수의 이상값 측정에 크게 영향을 받을 수 있다는 것입니다.

백분위수와 같은 메트릭은 측정값이 어디에 해당하는지 설명하는 데 약간 더 나은 역할을 하지만, 해당 측정값 각각을 그래프로 그리고 데이터 분포의 형태를 직접 보는 것을 대체할 수는 없습니다.

문제는 k6 OSS가 기본적으로 그래프를 생성하지 않는다는 것입니다. 다음 섹션에서는 선택한 결과 시각화 도구에서 사용하기 위해 결과를 출력하는 방법을 배웁니다.

### 정답

1. C. thresholds는 통과 원인이 되는 것으로 표현되므로, C가 정답입니다. 95번째 백분위수 응답 시간이 2000ms(2초) 이하(더 빠른)인 테스트를 설명하기 때문입니다. 이를 초과하면 threshold가 실패합니다.
2. C. thresholds는 [`abortOnFail`이 `true`로 설정되지 않으면](https://k6.io/docs/using-k6/thresholds/#aborting-a-test-when-a-threshold-is-crossed) 실패 시 테스트를 중단하지 않으므로, C만 올바릅니다. 테스트는 여전히 실행되지만 일부 thresholds가 실패했다는 메시지가 테스트 종료 요약에 표시됩니다.
3. B. A는 모순되는 thresholds가 정의될 때 마지막 것이 사용되므로, `http_req_failure`의 threshold는 5%가 아닌 3%로 설정되기 때문에 틀립니다. C는 threshold가 응답 시간이 4초 *미만*이면 통과로 설정되어 있어 4초 응답 시간은 실패하게 하기 때문에 틀립니다.
