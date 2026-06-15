# k6 check

---

## 우리의 스크립트

```js
import http from 'k6/http';

export default function() {
  let url = 'https://httpbin.test.k6.io/post';
  let response = http.post(url, 'Hello world!');

  console.log(response.json().data);
}
```

---

## 스크립트에 check 추가하기

```js [1|3-6]
import { check } from 'k6';

check(response, {
    'Application says hello': (r) => r.body.includes('Hello world!')
  });
}
```

---

## 테스트를 다시 실행해 봅시다!

테스트를 실행하는 명령어를 기억하시나요? 👀

```shell
	✓ Application says hello

     checks.........................: 100.00% ✓ 1        ✗ 0
```

---

## 실패한 check

```js
check(response, {
      'Application says hello': (r) => r.body.includes('Bonjour!')
  });
}
```

---

## 테스트를 다시 실행해 봅시다!

```shell
     ✗ Application says hello
      ↳  0% — ✓ 0 / ✗ 1

     checks.........................: 0.00%  ✓ 0        ✗ 1
```

---

## 실패한 check는 오류가 아닙니다!

> 💡 실패한 check로 테스트를 중지하려면 [threshold와 결합](https://k6.io/docs/using-k6/thresholds/#failing-a-load-test-using-checks)할 수 있습니다.

---

## think time으로 스크립트를 현실적으로 만들기

- 이동: [08-adding-think-time](?p=08-adding-think-time)
