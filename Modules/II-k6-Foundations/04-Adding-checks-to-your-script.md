check는 요청에서 반환된 응답을 검증하는 데 사용할 수 있는 테스트 기준의 한 유형입니다.

지금까지 `console.log()`를 사용하여 응답 본문을 터미널에 출력해왔습니다. 이 접근 방식은 디버깅에 유용하지만, 응답 본문이 크거나 1명 이상의 VU로 테스트를 실행하기로 결정한 경우 금세 걷잡을 수 없게 됩니다. 대신 이 섹션에서는 check를 사용하는 방법을 배웁니다.

## 스크립트에 check 추가하는 방법

다시 스크립트로 돌아가봅시다!

스크립트가 보내는 요청에 대한 대상 서버의 예상 응답은 이미 알고 있습니다: 스크립트가 보낸 것을 그대로 되돌려 보내야 합니다. 이 경우 그것은 `Hello world!`입니다.

따라서 `console.log()` 문을 제거하고 이 코드 스니펫을 복사하여 check를 추가하세요:

```js
import http from 'k6/http';
import { check } from 'k6';

export default function() {
  let url = 'https://httpbin.test.k6.io/post';
  let response = http.post(url, 'Hello world!');
  check(response, {
      'Application says hello': (r) => r.body.includes('Hello world!')
  });
}
```

k6 라이브러리에서 `check`를 임포트해야 합니다:

```js
import { check } from 'k6';
```

그리고 실제 check를 default 함수에 넣어야 합니다:

```js
check(response, {
      'Application says hello': (r) => r.body.includes('Hello world!')
  });
}
```

방금 추가한 check는 *모든* 요청의 응답 본문에서 `Hello world!` 문자열을 찾습니다.

### 스크립트 실행

테스트 종료 요약에서 어떻게 보일까요? 스크립트를 저장하고 다음을 실행하세요:

```plain
k6 run test.js
```

다음과 유사한 출력을 볼 수 있습니다:

```plain
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


running (00m00.7s), 0/1 VUs, 1 complete and 0 interrupted iterations
default ✓ [======================================] 1 VUs  00m00.7s/10m0s  1/1 iters, 1 per VU

     ✓ Application says hello

     checks.........................: 100.00% ✓ 1        ✗ 0
     data_received..................: 5.9 kB  8.4 kB/s
     data_sent......................: 564 B   801 B/s
     http_req_blocked...............: avg=582.6ms  min=582.6ms  med=582.6ms  max=582.6ms  p(90)=582.6ms  p(95)=582.6ms 
     http_req_connecting............: avg=121.14ms min=121.14ms med=121.14ms max=121.14ms p(90)=121.14ms p(95)=121.14ms
     http_req_duration..............: avg=120.62ms min=120.62ms med=120.62ms max=120.62ms p(90)=120.62ms p(95)=120.62ms
       { expected_response:true }...: avg=120.62ms min=120.62ms med=120.62ms max=120.62ms p(90)=120.62ms p(95)=120.62ms
     http_req_failed................: 0.00%   ✓ 0        ✗ 1
     http_req_receiving.............: avg=72µs     min=72µs     med=72µs     max=72µs     p(90)=72µs     p(95)=72µs    
     http_req_sending...............: avg=292µs    min=292µs    med=292µs    max=292µs    p(90)=292µs    p(95)=292µs   
     http_req_tls_handshaking.......: avg=405.16ms min=405.16ms med=405.16ms max=405.16ms p(90)=405.16ms p(95)=405.16ms
     http_req_waiting...............: avg=120.26ms min=120.26ms med=120.26ms max=120.26ms p(90)=120.26ms p(95)=120.26ms
     http_reqs......................: 1       1.419825/s
     iteration_duration.............: avg=703.46ms min=703.46ms med=703.46ms max=703.46ms p(90)=703.46ms p(95)=703.46ms
     iterations.....................: 1       1.419825/s
```

새 check는 다음 줄에 표시됩니다:

```plain
	✓ Application says hello

     checks.........................: 100.00% ✓ 1        ✗ 0
```

`✓`는 실행된 모든 요청(이 테스트 실행에서는 단 하나)이 check를 통과했음을 의미합니다. 이 테스트 실행의 check 통과율은 100%였습니다.

### 실패한 check

check가 실패하면 어떻게 보일까요?

다음과 같이 응답에서 찾을 수 없는 텍스트를 검색하도록 스크립트를 수정하세요:

```js
import http from 'k6/http';
import { check } from 'k6';

export default function() {
  let url = 'https://httpbin.test.k6.io/post';
  let response = http.post(url, 'Hello world!');
  check(response, {
      'Application says hello': (r) => r.body.includes('Bonjour!')
  });
}
```

그것을 실행하면 다음 결과를 얻어야 합니다:

```plain
     ✗ Application says hello
      ↳  0% — ✓ 0 / ✗ 1

     checks.........................: 0.00%  ✓ 0        ✗ 1
```

이번에는 `✗ 1`이 하나의 check가 실패했음을 나타냅니다.

### 실패한 check는 오류가 아닙니다

마지막 예시에서 `http_req_failed`, 즉 HTTP 오류율이 실패한 check에 영향을 받지 않은 것을 알아챘을 것입니다. 이는 check가 스크립트를 성공적으로 실행하는 것을 중단하지 않으며, 실패한 종료 상태를 반환하지도 않기 때문입니다.

> :bulb: 실패한 check가 테스트를 중단하게 하려면 [thresholds와 결합](https://k6.io/docs/using-k6/thresholds/#failing-a-load-test-using-checks)할 수 있습니다.

## 다른 유형의 check

[checks](https://k6.io/docs/using-k6/checks/) 페이지에는 HTTP 응답 코드 및 응답 본문 크기를 포함하여 수행할 수 있는 다른 유형의 check가 있습니다. 단일 응답에 대해 여러 check를 포함할 수도 있습니다.

## 다음 단계

이제 테스트를 여러 사용자로 확장할 준비가 거의 되었습니다! 그 전에 다음 섹션에서 think time을 사용하여 테스트를 현실적으로 만드는 방법에 대해 설명합니다.

## 지식 확인

### 문제 1

다음 중 check로 검증할 수 있는 것은?

A: 요청의 95번째 백분위수 응답 시간이 1초 이상인지 여부

B: 반환된 응답 본문의 크기

C: 테스트의 오류율


### 문제 2

check는 테스트의 어떤 부분을 평가합니까?

A: 요청의 응답 시간

B: 애플리케이션에 전송된 요청의 구문

C: 애플리케이션 서버의 응답


### 문제 3

테스트 종료 요약의 다음 스니펫에서 몇 개의 check가 실패했나요?

```plain
     ✗ Application says hello
      ↳  51% — ✓ 1215 / ✗ 1144

     checks.........................: 51.50% ✓ 1215      ✗ 1144
```

A: 51.50%

B: 1215

C: 1144

### 정답

1. B. 응답 시간의 95번째 백분위수는 집계된 메트릭으로, 해당 시점까지의 모든 요청 측정에 의존합니다. 이것은 check를 만들 수 있는 것이 아니지만, 스크립트에 [threshold](https://k6.io/docs/using-k6/thresholds/)로 포함할 수는 있습니다. 테스트의 오류율도 마찬가지로 집계됩니다. B, 즉 응답 본문의 크기가 정답입니다.
2. C. check는 k6가 보낸 요청이 아닌 애플리케이션 서버의 응답을 파싱합니다.
3. C. 실패한 check 수는 `✗ 1144`에 표시됩니다. 이 요청의 check는 51.50%의 시간 동안 통과했습니다.
