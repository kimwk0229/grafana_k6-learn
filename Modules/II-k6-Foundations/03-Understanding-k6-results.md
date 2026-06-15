이전 섹션에서 첫 번째 k6 테스트를 실행했습니다&mdash;그런데 테스트에서 무슨 일이 일어났는지 어떻게 알 수 있을까요?

이 섹션에서는 k6의 기본 출력을 이해하고 스크립트가 의도한 대로 실행되었는지 확인하는 방법을 배웁니다.

## 테스트 종료 요약 보고서

다시 한번 그 출력을 살펴봅시다:

```plain
$ k6 run test.js

          /\      |‾‾| /‾‾/   /‾‾/
     /\  /  \     |  |/  /   /  /
    /  \/    \    |     (   /   ‾‾\
   /          \   |  |\  \ |  (‾)  |
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: test.js
     output: -

  scenarios: (100.00%) 1 scenario, 1 max VUs, 10m30s max duration (incl. graceful stop):
           * default: 1 iterations for each of 1 VUs (maxDuration: 10m0s, gracefulStop: 30s)

INFO[0001] Hello world!                                  source=console

running (00m00.7s), 0/1 VUs, 1 complete and 0 interrupted iterations
default ✓ [======================================] 1 VUs  00m00.7s/10m0s  1/1 iters, 1 per VU

     data_received..................: 5.9 kB 9.0 kB/s
     data_sent......................: 564 B  860 B/s
     http_req_blocked...............: avg=524.18ms min=524.18ms med=524.18ms max=524.18ms p(90)=524.18ms p(95)=524.18ms
     http_req_connecting............: avg=123.28ms min=123.28ms med=123.28ms max=123.28ms p(90)=123.28ms p(95)=123.28ms
     http_req_duration..............: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
       { expected_response:true }...: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
     http_req_failed................: 0.00%  ✓ 0        ✗ 1
     http_req_receiving.............: avg=165µs    min=165µs    med=165µs    max=165µs    p(90)=165µs    p(95)=165µs
     http_req_sending...............: avg=80µs     min=80µs     med=80µs     max=80µs     p(90)=80µs     p(95)=80µs
     http_req_tls_handshaking.......: avg=399.48ms min=399.48ms med=399.48ms max=399.48ms p(90)=399.48ms p(95)=399.48ms
     http_req_waiting...............: avg=129.94ms min=129.94ms med=129.94ms max=129.94ms p(90)=129.94ms p(95)=129.94ms
     http_reqs......................: 1      1.525116/s
     iteration_duration.............: avg=654.72ms min=654.72ms med=654.72ms max=654.72ms p(90)=654.72ms p(95)=654.72ms
     iterations.....................: 1      1.525116/s

```

이것이 테스트 종료 요약 보고서입니다. k6가 테스트 결과를 표시하는 기본 방식입니다.

한 줄씩 살펴보겠습니다.

### 실행 파라미터

```plain
execution: local
```

k6 OSS를 사용하여 로컬(`local`)에서 또는 k6 Cloud(`cloud`)에서 테스트 스크립트를 실행할 수 있습니다.
이 테스트에서는 로컬 머신에서 테스트 스크립트가 실행되었습니다.

```plain
script: test.js`
```

이것은 실행된 스크립트의 파일명입니다.

```plain
output: -`
```

이것은 기본 동작을 나타냅니다: k6가 테스트 결과를 표준 출력으로 출력했습니다.

k6는 [다른 형식으로도 결과를 출력](https://k6.io/docs/getting-started/results-output/#external-outputs)할 수 있습니다. 이러한 옵션이 사용될 때 `output`에 표시됩니다.

```plain
scenarios: (100.00%) 1 scenario, 1 max VUs, 10m30s max duration (incl. graceful stop):
```

실행 [시나리오](https://k6.io/docs/misc/glossary/#scenario)는 테스트 실행에 관한 지침 집합입니다: 어떤 코드가 실행되어야 하는지, 언제 그리고 얼마나 자주 실행되어야 하는지, 그리고 다른 구성 가능한 파라미터들입니다. 이 경우 첫 번째 테스트는 기본 파라미터를 사용하여 실행되었습니다: 1개의 시나리오, 1개의 [가상 사용자(VU)](https://k6.io/docs/misc/glossary/#virtual-user), 그리고 최대 10분 30초의 duration.

최대 duration은 실행 시간 제한입니다; 이 시간을 초과하면 테스트가 강제로 중단됩니다. 이 경우 k6는 이 시간이 경과하기 훨씬 전에 스크립트의 요청에 대한 응답을 받았습니다.

[graceful stop](https://k6.io/docs/misc/glossary/#graceful-stop)은 테스트 끝에 k6가 가능하면 실행 중인 [반복(iterations)](https://k6.io/docs/misc/glossary/#iteration)을 완료하는 기간입니다. 기본적으로 k6는 최대 duration인 10분 30초 내에 30초의 graceful stop을 포함합니다.

```plain
* default: 1 iterations for each of 1 VUs (maxDuration: 10m0s, gracefulStop: 30s)
```

여기서 `default`는 시나리오 이름을 나타냅니다. 테스트 스크립트에 명시적으로 설정된 것이 없었기 때문에, k6는 기본 이름을 사용했습니다.

반복(iteration)은 테스트의 단일 실행 루프입니다. 부하 테스트는 일반적으로 요청이 지속적으로 이루어지도록 일정 시간 내에 반복 실행 루프를 포함합니다. 별도로 지정하지 않으면, k6는 default 함수를 한 번 실행합니다.

가상 사용자는 애플리케이션의 실제 최종 사용자를 시뮬레이션하려는 단일 스레드 또는 인스턴스로 생각할 수 있습니다. 이 경우 k6는 테스트를 실행하기 위해 하나의 가상 사용자를 시작했습니다.

### 콘솔 출력

테스트 종료 요약의 이 섹션은 일반적으로 비어 있지만, 테스트 스크립트에는 응답 본문의 일부를 콘솔에 저장하는 줄(`console.log(response.json().data);`)이 포함되어 있었습니다. 보고서에서 어떻게 보이는지 살펴봅시다:

```plain
INFO[0001] Hello world!                                  source=console`
```

테스트 스크립트의 대상 엔드포인트인 `https://httpbin.test.k6.io/post`는 POST 본문에 보낸 것을 그대로 반환하므로, 이것은 좋은 신호입니다! 대상 엔드포인트가 스크립트에서 보낸 `Hello world!`를 받고 같은 본문을 되돌려 보냈습니다.

테스트 스크립트에서 여러 `console.log()` 문을 사용했다면, 모두 이 섹션에 표시됩니다.

### 실행 요약

실행 요약은 테스트 실행 중 일어난 일의 개요를 보여줍니다.

```plain
running (00m00.7s), 0/1 VUs, 1 complete and 0 interrupted iterations
default ✓ [======================================] 1 VUs  00m00.7s/10m0s  1/1 iters, 1 per VU
```

이 경우 테스트는 1개의 VU로 0.7초 동안 실행되었습니다. 단일 반복이 실행되어 완전히 완료되었습니다(즉, 중단되지 않았습니다). VU당 1번의 반복이 실행되었습니다(총 1번).

### k6 내장 메트릭

이제 [메트릭](https://k6.io/docs/misc/glossary/#metric)을 살펴봅시다! k6에는 [많은 내장 메트릭](https://k6.io/docs/using-k6/metrics/#built-in-metrics)이 포함되어 있습니다.

```plain
     data_received..................: 5.9 kB 9.0 kB/s
     data_sent......................: 564 B  860 B/s
     http_req_blocked...............: avg=524.18ms min=524.18ms med=524.18ms max=524.18ms p(90)=524.18ms p(95)=524.18ms
     http_req_connecting............: avg=123.28ms min=123.28ms med=123.28ms max=123.28ms p(90)=123.28ms p(95)=123.28ms
     http_req_duration..............: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
       { expected_response:true }...: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
     http_req_failed................: 0.00%  ✓ 0        ✗ 1
     http_req_receiving.............: avg=165µs    min=165µs    med=165µs    max=165µs    p(90)=165µs    p(95)=165µs
     http_req_sending...............: avg=80µs     min=80µs     med=80µs     max=80µs     p(90)=80µs     p(95)=80µs
     http_req_tls_handshaking.......: avg=399.48ms min=399.48ms med=399.48ms max=399.48ms p(90)=399.48ms p(95)=399.48ms
     http_req_waiting...............: avg=129.94ms min=129.94ms med=129.94ms max=129.94ms p(90)=129.94ms p(95)=129.94ms
     http_reqs......................: 1      1.525116/s
     iteration_duration.............: avg=654.72ms min=654.72ms med=654.72ms max=654.72ms p(90)=654.72ms p(95)=654.72ms
     iterations.....................: 1      1.525116/s
```

다음 메트릭들은 일반적으로 테스트 분석에 가장 중요합니다.

#### 응답 시간

```plain
http_req_duration..............: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
```

"응답 시간"은 여러 구성 요소로 나눌 수 있어 모호할 수 있습니다. 그러나 대부분의 경우 `http_req_duration`이 원하는 메트릭입니다. 여기에 포함되는 항목:
- `http_req_sending` (대상 호스트로 데이터를 보내는 데 걸린 시간)
- `http_req_waiting` (TTFB 또는 "Time to First Byte"; 대상 서버가 요청에 응답하기 시작하기 전까지 걸린 시간)
- `http_req_receiving` (대상 서버가 k6에 완전히 응답하는 데 걸린 시간)

이 줄의 응답 시간은 평균, 최솟값, 중앙값, 최댓값, 90번째 백분위수, 95번째 백분위수로 밀리초(ms) 단위로 보고됩니다. 어느 것을 사용해야 할지 모르겠다면 95번째 백분위수 값을 사용하세요.

95번째 백분위수 응답 시간이 130.19ms라는 것은 요청의 95%가 130.19ms 이하의 응답 시간을 가졌다는 의미입니다. 그러나 이 특정 상황에서는 테스트 스크립트가 단 하나의 요청만 했기 때문에, 이 줄의 모든 메트릭이 같은 값인 130.19ms를 보고합니다. 여러 요청으로 테스트를 실행하면 이 값들에 변화가 생깁니다.

여기서 주의해야 할 점은 `http_req_duration`이 통과 여부와 관계없이 *모든* 요청에 대한 값이라는 것입니다. 이 동작은 실패한 요청이 종종 성공한 것보다 짧거나 긴 응답 시간을 가질 수 있기 때문에 응답 시간을 해석할 때 오해를 일으킬 수 있습니다.

아래 줄은 성공한 요청에 대한 응답 시간만 보고합니다.

```plain
  { expected_response:true }...: avg=130.19ms min=130.19ms med=130.19ms max=130.19ms p(90)=130.19ms p(95)=130.19ms
```

정확성을 높이고 실패한 요청이 결과를 왜곡하지 않도록 하려면, 성공한 요청의 95번째 백분위수 값을 응답 시간으로 사용하세요.

#### 오류율

`http_req_failed` 메트릭은 테스트의 오류율을 설명합니다. 오류율은 테스트 중 실패한 요청 수를 전체 요청의 비율로 나타낸 것입니다.

```plain
http_req_failed................: 0.00%  ✓ 0        ✗ 1
```

`http_req_failed`는 200에서 399 사이의 HTTP 응답 코드를 자동으로 표시합니다. 이는 HTTP 4xx 및 HTTP 5xx 응답 코드가 k6에서 기본적으로 오류로 간주된다는 것을 의미합니다. (참고: 이 동작은 [`setResponseCallback`](https://k6.io/docs/javascript-api/k6-http/setresponsecallback-callback)을 사용하여 변경할 수 있습니다.)

이 테스트 실행에서는 오류율이 `0.00%`였는데, 실행한 단일 요청이 성공했기 때문입니다. 반직관적으로 보일 수 있지만, 이 줄의 `✓`는 실제로 `http_req_failed = true`인 요청이 없었다는 것을 의미합니다. 즉, 실패가 없었다는 뜻입니다. 반대로 `✗`는 1개의 요청이 `http_req_failed = false`를 가졌다는 것을 의미하며, 즉 성공했다는 뜻입니다.

#### 요청 수

테스트 중 모든 VU가 전송한 총 요청 수는 아래 줄에 설명되어 있습니다.

```plain
http_reqs......................: 1      1.525116/s
```

또한 숫자 `1.525116/s`는 테스트 전체에 걸쳐 테스트가 실행한 **초당 요청 수(rps)**입니다. 일부 도구에서는 이를 "테스트 처리량"이라고 설명합니다. 이를 통해 테스트 중 애플리케이션이 얼마나 많은 부하를 경험했는지 더 잘 정량화할 수 있습니다.

#### 반복 duration

`http_req_duration`은 스크립트 내의 HTTP 요청이 서버에서 응답을 받는 데 걸리는 시간을 측정합니다. 그런데 사용자 흐름에 여러 HTTP 요청이 연결되어 있고, 전체 흐름이 사용자에게 얼마나 걸릴지 알고 싶다면 어떻게 해야 할까요?

그런 경우에는 반복 duration이 살펴볼 메트릭입니다.

```plain
iteration_duration.............: avg=654.72ms min=654.72ms med=654.72ms max=654.72ms p(90)=654.72ms p(95)=654.72ms
```

반복 duration은 k6가 VU 코드의 단일 루프를 수행하는 데 걸린 시간입니다. 스크립트에 로그인, 제품 페이지 탐색, 장바구니 추가, 결제 정보 입력 같은 단계가 포함되어 있다면, 반복 duration은 애플리케이션 사용자 중 한 명이 제품을 구매하는 데 얼마나 걸릴지에 대한 아이디어를 줍니다.

이 메트릭은 각 HTTP 요청에 대해 허용 가능한 응답 시간을 결정하려고 할 때 유용할 수 있습니다. 예를 들어, 결제 요청이 2초가 걸리더라도 총 반복 duration이 여전히 3초에 불과하다면 괜찮다고 판단할 수 있습니다.

다른 메트릭과 마찬가지로, 반복 duration은 평균, 최솟값, 중앙값, 최댓값, 90번째 백분위수, 95번째 백분위수로 밀리초 단위로 표현됩니다.

#### 반복 수

반복 수는 k6가 모든 VU에 대해 스크립트를 총 몇 번 루프했는지를 설명합니다. 이 메트릭은 계정 가입과 같은 각 반복과 관련된 출력을 확인하려는 경우 유용할 수 있습니다.

```plain
iterations.....................: 1      1.525116/s
```

같은 줄의 숫자 `1.525116/s`는 **초당 반복 수**입니다. k6가 스크립트를 완전히 반복한 속도를 나타냅니다. 이것은 [초당 요청 수](03-Understanding-k6-results.md#Number-of-requests)와 마찬가지로, k6가 애플리케이션 서버에 메시지를 보낸 속도 또는 비율의 측정입니다.

## 다음 단계

요청 응답 본문을 콘솔에 로깅하는 것은 문제 해결 시 유용할 수 있지만, 로그를 확인하지 않고도 응답을 자동으로 확인하려면 어떻게 해야 할까요? 다음 섹션에서 check에 대해 배울 것입니다.

## 지식 확인

다음 질문에 답하려면 아래의 샘플 테스트 종료 요약 보고서를 참조하세요.

```plain
          /\      |‾‾| /‾‾/   /‾‾/
     /\  /  \     |  |/  /   /  /
    /  \/    \    |     (   /   ‾‾\
   /          \   |  |\  \ |  (‾)  |
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: test.js
     output: -

  scenarios: (100.00%) 1 scenario, 10 max VUs, 2m30s max duration (incl. graceful stop):
           * default: 10 looping VUs for 2m0s (gracefulStop: 30s)


running (2m00.1s), 00/10 VUs, 9463 complete and 0 interrupted iterations
default ✓ [======================================] 10 VUs  2m0s

     data_received..................: 6.3 MB 52 kB/s
     data_sent......................: 1.5 MB 12 kB/s
     http_req_blocked...............: avg=4.46ms   min=1µs      med=5µs      max=647.67ms p(90)=9µs      p(95)=11µs
     http_req_connecting............: avg=1.4ms    min=0s       med=0s       max=255.11ms p(90)=0s       p(95)=0s
     http_req_duration..............: avg=122.22ms min=111.57ms med=120.95ms max=282.39ms p(90)=127.78ms p(95)=131.04ms
       { expected_response:true }...: avg=122.22ms min=111.57ms med=120.95ms max=282.39ms p(90)=127.78ms p(95)=131.04ms
     http_req_failed................: 0.00%  ✓ 0         ✗ 9463
     http_req_receiving.............: avg=107.08µs min=15µs     med=85µs     max=13.35ms  p(90)=163µs    p(95)=197µs
     http_req_sending...............: avg=35.86µs  min=5µs      med=30µs     max=2.7ms    p(90)=59µs     p(95)=72µs
     http_req_tls_handshaking.......: avg=2.99ms   min=0s       med=0s       max=501.24ms p(90)=0s       p(95)=0s
     http_req_waiting...............: avg=122.08ms min=111.47ms med=120.81ms max=282.16ms p(90)=127.64ms p(95)=130.87ms
     http_reqs......................: 9463   78.808231/s
     iteration_duration.............: avg=126.85ms min=111.75ms med=121.14ms max=803.13ms p(90)=128.46ms p(95)=132.55ms
     iterations.....................: 9463   78.808231/s
     vus............................: 10     min=10      max=10
     vus_max........................: 10     min=10      max=10
```


### 문제 1

다음 중 모든 HTTP 요청의 응답 시간으로 사용하기에 가장 좋은 값은?

A: 131.04 ms

B: 122.08 ms

C: 4.46 ms

### 문제 2

이 테스트는 몇 명의 가상 사용자로 실행되었나요?

A: 9463

B: 1

C: 10


### 문제 3

몇 개의 요청이 실패했나요?

A: 0

B: 9463

C: 10

### 정답

1. A. B는 응답 시간의 구성 요소일 뿐인 `http_req_waiting`을 가리킵니다. C는 요청을 보내기 전에 TCP 연결이 설정될 때까지 기다리는 시간을 나타내는 `http_req_blocked`를 가리킵니다. A는 응답 시간으로 사용하기에 가장 좋은 메트릭인 `http_req_duration`의 95번째 백분위수 값입니다.
2. C. VU 수는 결과의 아래에서 두 번째 행에 나열되어 있으며, 이 경우 10입니다.
3. A. `http_req_failed` 메트릭이 있는 줄에서 `0.00%  ✓ 0`은 오류가 있는 응답의 수를 나타냅니다. 즉, 요청 중 `http_req_failed`가 `true`로 설정된 것이 얼마나 되는지입니다. 9463은 통과한 요청, 즉 `http_req_failed`가 `false`로 설정된 요청의 수입니다. 정답은 A, 0입니다. 요청이 실패하지 않았으며 테스트의 오류율은 0%입니다.
