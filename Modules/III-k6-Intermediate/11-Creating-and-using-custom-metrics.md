# 사용자 지정 메트릭 만들기 및 사용하기

[k6 결과 이해하기](../II-k6-Foundations/03-Understanding-k6-results.md)에서 k6가 기본적으로 보고하는 메트릭을 해석하는 방법을 배웠습니다. 하지만 k6가 추적하지 않는 다른 것을 측정하고 싶다면 어떻게 해야 할까요?

스크립트 내에서 자신만의 사용자 지정 메트릭을 만들 수 있습니다. 측정하려는 것을 정의하면, k6는 테스트 중에 측정값을 수집하고 테스트 종료 요약 보고서에서 기본 메트릭과 함께 사용자 지정 메트릭을 보고합니다.

## 메트릭의 유형

만들 수 있는 [사용자 지정 메트릭](https://k6.io/docs/using-k6/metrics/#custom-metrics)에는 [네 가지 유형](https://k6.io/docs/using-k6/metrics/#metric-types)이 있으며, 각각은 다른 종류의 데이터를 측정합니다.

### Counter

Counter는 특정 이벤트가 발생할 때마다 증가할 수 있는 정수입니다. Counter는 다음과 같은 발생 횟수를 추적하는 데 좋습니다:
- 특정 오류가 발생한 횟수
- Admin 기능에 접근할 수 있는 사용자 계정
- 요청이 리디렉션되는 빈도
- 스크립트가 무작위로 결정된 페이지를 접근하는 경우 특정 페이지의 방문 수

### Gauge

Gauge 메트릭은 최솟값, 최댓값, _마지막_ 값의 세 가지 숫자를 저장합니다. Gauge는 테스트가 실행되는 동안만 관련이 있는 것들을 추적하기 위해 실시간으로 감시하는 데 유용합니다. Gauge를 사용하여 다음을 측정할 수 있습니다:
- 반환된 응답 바디의 크기
- 응답 시간의 사용자 지정 버전(예: 세 개의 그룹화되지 않은 URL의 응답 시간)
- 애플리케이션을 생성하고 동시에 처리하는 scenario를 포함하는 테스트에서 처리되지 않은 애플리케이션 수

### Rate

Rate 메트릭은 수행된 총 측정 수에 대한 boolean 값을 추적합니다. 테스트가 끝나면, rate는 `true`인 모든 측정의 비율과 `false`인 측정의 비율을 표현합니다. Rate 메트릭은 다음을 측정하는 데 사용할 수 있습니다:
- VU의 몇 퍼센트가 제품에 대한 결제 프로세스를 수행했는지
- 광고가 나타난 페이지의 비율
- 권한이 충분하지 않은 사용자 계정의 비율

### Trend

Trend 메트릭은 최솟값, 최댓값, 평균 및 백분위 통계를 측정합니다. Gauge와 달리, trend는 _모든_ 값을 유지하고 테스트가 끝날 때 모든 값의 분포를 보고합니다. Trend는 k6에서 `http_req_duration`(응답 시간)이 측정되는 방식과 유사하며, 다음을 추적하는 데 사용할 수 있습니다:
- 응답 시간에 think time 또는 sleep이 추가되는 것과 같은 사용자 지정 타이머
- 올바른 응답이 반환되기 전에 실행된 재시도 수(폴링 동작 시뮬레이션과 같이)
- 관리자 사용자 계정에 보고된 활성 사용자 계정 수

## 예시: 사용자 지정 타이머

사용자 지정 메트릭을 사용하는 일반적인 이유는 지정하는 특정 동작에 대한 타이머를 설정하는 것입니다. 예를 들어, k6는 응답 시간 계산에서 sleep 시간을 자동으로 제거하지만, VU가 스크립트의 일부에 소비하는 총 시간(sleep 시간 포함)을 추적해야 하는 경우가 있습니다.

다음은 사용자 지정 타이머를 만드는 데 사용할 수 있는 짧은 스크립트입니다:

```js
import http from 'k6/http';
import { sleep, check } from 'k6';
import { Trend } from 'k6/metrics';

const transactionDuration = new Trend('transaction_duration');

export default function () {

  // This is a transaction
    let timeStart = Date.now();
    let res = http.get('https://test.k6.io', {tags: { name: '01_Home' }});
    check(res, {
      'is status 200': (r) => r.status === 200,
    });
    sleep(Math.random() * 5);
    let timeEnd = Date.now();
    transactionDuration.add(timeEnd - timeStart);
    console.log('The transaction took', timeEnd - timeStart, 'ms to complete.');

  // This is another transaction
  res = http.get('https://test.k6.io/about', {tags: { name: '02_About' }});
}
```

첫째, 필요한 메트릭 유형을 가져옵니다.

```js
import { Trend } from 'k6/metrics';
```

이 경우, Trend 메트릭이 가장 적합해 보이므로 k6가 새 타이머에 대한 전체 통계를 보고합니다.

둘째, 타이머를 선언합니다.

```js
const transactionDuration = new Trend('transaction_duration');
```

셋째, 타이머를 시작하고 중지할 위치를 결정합니다.

```js
let timeStart = Date.now();
let timeEnd = Date.now();
```

이 줄들을 스크립트에 전략적으로 배치하여 `timeStart`와 `timeEnd` 사이의 모든 코드 실행이 측정하려는 경과 시간에 포함되도록 합니다.

마지막으로, 경과 시간을 타이머에 추가합니다.

```js
transactionDuration.add(timeEnd - timeStart);
```

스크립트를 실행하면 다음과 같은 결과가 나타납니다:

```plain
checks.........................: 100.00% ✓ 1        ✗ 0  
     data_received..................: 17 kB   5.3 kB/s
     data_sent......................: 621 B   194 B/s
     http_req_blocked...............: avg=320.48ms min=15µs     med=320.48ms max=640.95ms p(90)=576.86ms p(95)=608.9ms 
     http_req_connecting............: avg=59.06ms  min=0s       med=59.06ms  max=118.12ms p(90)=106.31ms p(95)=112.21ms
     http_req_duration..............: avg=221.24ms min=128.58ms med=221.24ms max=313.89ms p(90)=295.36ms p(95)=304.63ms
       { expected_response:true }...: avg=128.58ms min=128.58ms med=128.58ms max=128.58ms p(90)=128.58ms p(95)=304.63ms
     http_req_failed................: 50.00%  ✓ 1        ✗ 1  
     http_req_receiving.............: avg=71µs     min=55µs     med=71µs     max=87µs     p(90)=83.8µs   p(95)=85.4µs  
     http_req_sending...............: avg=546µs    min=12µs     med=546µs    max=1.08ms   p(90)=973.2µs  p(95)=1.02ms  
     http_req_tls_handshaking.......: avg=258.3ms  min=0s       med=258.3ms  max=516.6ms  p(90)=464.94ms p(95)=490.77ms
     http_req_waiting...............: avg=220.62ms min=127.42ms med=220.62ms max=313.83ms p(90)=295.19ms p(95)=304.51ms
     http_reqs......................: 2       0.623378/s
     iteration_duration.............: avg=3.2s     min=3.2s     med=3.2s     max=3.2s     p(90)=3.2s     p(95)=3.2s    
     iterations.....................: 1       0.311689/s
     transaction_duration...........: avg=2893     min=2893     med=2893     max=2893     p(90)=2893     p(95)=2893    
     vus............................: 1       min=1      max=1
     vus_max........................: 1       min=1      max=1
```

사용자 지정 메트릭인 `transaction_duration`이 다른 기본 k6 메트릭과 함께 보고됩니다.

## 지식 확인

### Question 1

다음 중 k6 스크립트에서 사용자 지정 메트릭을 사용하는 것이 적합한 상황은 무엇입니까?

- [ ] A: 특정 URL의 응답 시간을 알고 싶을 때

- [ ] B: 테스트 중에 총 몇 건의 요청이 실행되었는지 보고해야 할 때

- [ ] C: 기본 메트릭이 다루지 않는 것을 측정하고 싶을 때

### Question 2

특정 체크박스가 선택된 열린 양식이 몇 개인지 추적하는 데 가장 적합한 사용자 지정 메트릭 유형은 무엇입니까?

- [ ] A: Trend

- [ ] B: Counter

- [ ] C: Gauge

### Question 3

테스트 후 사용자 지정 메트릭에 대한 보고서를 어떻게 얻을 수 있습니까?

- [ ] A: 모든 사용자 지정 메트릭은 테스트 종료 요약 보고서에 표시됩니다.

- [ ] B: 결과를 보려면 k6 Cloud에 로그인해야 합니다.

- [ ] C: rate와 trend 메트릭 결과만 테스트 종료 요약 보고서에 표시됩니다.

### 정답

1. **C.** A와 B 모두 이미 기본적으로 존재하는 메트릭을 설명합니다. 사용자 지정 메트릭은 k6에서 기본적으로 제공하는 것을 넘어서는 것들을 만들고 측정하기 위한 것입니다.

2. **B.** Counter는 테스트 중 이벤트의 발생 횟수를 추적하는 데 가장 적합합니다.

3. **A.** 사용자 지정 메트릭은 테스트 종료 요약 보고서에 표시되지만, k6 Cloud를 사용하는 경우 *그곳에서도* 사용 가능합니다. C는 거짓입니다.
