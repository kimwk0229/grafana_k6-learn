# k6 부하 테스트 옵션

---

## 스크립트 options

```js
export let options = {
  vus: 10,
  iterations: 40,
};
```

---

> 💡 VU만 정의하고 다른 테스트 옵션을 정의하지 않으면 다음과 같은 오류가 발생할 수 있습니다:

```shell
          /\      |‾‾| /‾‾/   /‾‾/   
     /\  /  \     |  |/  /   /  /    
    /  \/    \    |     (   /   ‾‾\  
   /          \   |  |\  \ |  (‾)  | 
  / __________ \  |__| \__\ \_____/ .io

WARN[0000] the `vus=10` option will be ignored, it only works in conjunction with `iterations`, `duration`, or `stages` 
  execution: local
     script: test.js
     output: -
```

---

## Iterations

```js
  vus: 10,
  iterations: 40,
```

> 테스트 options에서 iteration 수를 설정하면 **모든** 사용자에 대해 정의됩니다.

---

## Duration

```js
  vus: 10,
  duration: '2m'
```

> duration을 설정하면 k6에게 duration이 도달할 때까지 각 VU에 대해 스크립트를 반복하도록 지시합니다.

---

## Iterations와 duration

```js
  vus: 10,
  duration: '5m',
  iterations: 40,
```

> duration과 함께 iteration 수를 설정하면 먼저 종료되는 값이 사용됩니다.

---

## Stages

iterations와 duration을 정의하면 _단순한 부하 프로필_이 생성됩니다

![단순한 부하 프로필](../../images/load_profile-no_ramp-up_or_ramp-down.png)
<!-- .element class="stretch" -->

---

## 일정한 부하 프로필

ramp-up이나 ramp-down을 추가하여 프로필이 더 이렇게 보이도록 하려면 어떻게 할까요?

![ramp이 포함된 일정한 부하 프로필](../../images/load_profile-constant.png.png)
<!-- .element class="stretch" -->

---

그 경우 [stages](https://k6.io/docs/using-k6/options/#stages)를 사용하는 것이 좋습니다.

```js
export let options = {
  stages: [
    { duration: '30m', target: 100 },
    { duration: '1h', target: 100 },
    { duration: '5m', target: 0 },
  ],
};
```

---

## 지금까지의 전체 스크립트

stages를 사용하는 경우 지금까지의 스크립트는 다음과 같아야 합니다:

```js
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '30m', target: 100 },
    { duration: '1h', target: 100 },
    { duration: '5m', target: 0 },
  ],
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

## threshold를 설정해 봅시다

- 이동 [10-setting-test-criteria-with-thresholds](?p=10-setting-test-criteria-with-thresholds)
