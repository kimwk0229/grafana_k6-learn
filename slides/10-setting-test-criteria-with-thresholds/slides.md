# k6 Threshold

---

## 스크립트에 threshold 추가하기

```js [7-10]
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

---

> 💡 Threshold는 **항상** 메트릭을 기반으로 합니다.

---

## Threshold 유형

- 오류율
- 응답 시간
- Check

> 💡 권장 사항: 가능한 경우 테스트에 오류율, 응답 시간, check threshold를 사용하세요.

---

### 오류율

```js
thresholds: {
    http_req_failed: ['rate<=0.05'],
},
```

---

### 응답 시간

```js
thresholds: {
    http_req_duration: ['p(95)<=5000'],
},
```

> 💡 권장 사항: 95번째 퍼센타일 응답 시간부터 시작하세요.

---

#### 여러 응답 시간 threshold 사용

```js
thresholds: {
    http_req_duration: ['p(90) < 400', 'p(95) < 800', 'p(99.9) < 2000'],
},
```

---

### Check

```js
thresholds: {
  checks: ['rate>=0.9'],
},
```

---

## 실패 시 테스트 중단

```js
thresholds: {
    http_req_failed: [{
      threshold: 'rate<=0.05',
      abortOnFail: true,
    }],
},
```

---

## 전체 스크립트

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

---

## 다양한 방법으로 k6 결과 출력하기

- 이동 [11-k6-results-output-options](?p=11-k6-results-output-options)
