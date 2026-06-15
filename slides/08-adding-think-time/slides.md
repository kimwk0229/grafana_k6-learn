# Think time

---

## Think time이란 무엇인가?

테스트 실행 중 실제 사용자가 애플리케이션을 사용하는 과정에서 갖는 지연을 시뮬레이션하기 위해 스크립트가 일시 중지하는 시간입니다.

---

## Think time을 언제 사용해야 할까?

- 테스트가 사용자 흐름을 따를 때
- 수행하는 데 시간이 걸리는 작업을 시뮬레이션하려는 경우
- 테스트 실행 중 k6를 실행하는 부하 생성기 또는 머신에서 CPU 사용률이 높을 때 (> 80%)

---

## Think time을 **사용하지 말아야** 하는 경우?

- [스트레스 테스트](https://k6.io/docs/test-types/stress-testing/)를 수행하려는 경우
- 테스트 중인 API 엔드포인트가 프로덕션에서 지연 없이 초당 많은 수의 요청을 받는 경우
- 부하 생성기가 80% CPU 사용률을 초과하지 않고 테스트 스크립트를 실행할 수 있는 경우

---

## k6 Sleep

```js [2|11]
import http from 'k6/http';
import { check, sleep } from 'k6';

export default function() {
  let url = 'https://httpbin.test.k6.io/post';
  let response = http.post(url, 'Hello world!');
  check(response, {
      'Application says hello': (r) => r.body.includes('Hello world!')
  });

  sleep(1);
}
```

---

**Sleep은 응답 시간(`http_req_duration`)에 영향을 미치지 않습니다. 응답 시간은 항상 sleep을 제외하고 보고됩니다. 그러나 sleep은 `iteration_duration`에 포함됩니다.**

---

## 동적 think time

동적 think time은 더 현실적이며 실제 사용자를 더 정확하게 시뮬레이션합니다.

---

### 랜덤 sleep

동적 think time을 구현하는 한 가지 방법은 JavaScript의 `Math.random()` 함수를 사용하는 것입니다:

```js
sleep(Math.random() * 5);
```

---

### 범위 내 랜덤 sleep

```js [1|3]
import { randomIntBetween } from "https://jslib.k6.io/k6-utils/1.0.0/index.js";

sleep(randomIntBetween(1,5));
```

---

## think time을 얼마나 추가해야 할까?

정답은: 상황에 따라 다릅니다.

---

## 테스트 확장으로 넘어갑시다

- 이동: [09-load-test-options](?p=09-load-test-options)
