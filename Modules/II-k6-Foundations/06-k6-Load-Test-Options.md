# k6 부하 테스트 옵션

지금까지 단일 VU와 단일 반복으로 동일한 스크립트를 실행해왔습니다. 이 섹션에서는 이를 확장하여 애플리케이션에 대해 전체 규모의 부하 테스트를 실행하는 방법을 배웁니다.

테스트 옵션은 VU 수 또는 반복 횟수, 테스트 duration 등 테스트 스크립트가 실행되는 방식에 영향을 미치는 구성 값입니다. "테스트 파라미터"라고도 부릅니다.

k6에는 기본 테스트 옵션이 있지만, 스크립트의 테스트 파라미터를 변경하는 네 가지 다른 방법이 있습니다:
1. k6 스크립트를 실행할 때 명령줄 플래그를 포함할 수 있습니다(예: `k6 run --vus 10 --iterations 30`).
2. 스크립트에 전달되는 명령줄에서 [환경 변수](https://k6.io/docs/using-k6/environment-variables/)를 정의할 수 있습니다.
3. 테스트 스크립트 내에서 직접 정의할 수 있습니다.
4. 구성 파일을 포함할 수 있습니다.

지금은 세 번째 옵션인 스크립트 내에서 테스트 파라미터를 정의하는 방법을 배우겠습니다. 이 접근 방식의 장점은:
- 단순성: 추가 파일이나 명령이 필요하지 않습니다.
- 반복 가능성: 이러한 파라미터를 스크립트에 추가하면 동료가 작성한 테스트를 더 쉽게 실행할 수 있습니다.
- 버전 관리 가능성: 테스트 파라미터의 변경 사항을 테스트 코드와 함께 추적할 수 있습니다.

스크립트 내에서 테스트 옵션을 사용하려면 스크립트에 다음 줄을 추가하세요. 관례상 스크립트를 열었을 때 옵션을 쉽게 읽을 수 있도록 import 문 뒤, default 함수 앞에 추가하는 것이 가장 좋습니다:

```js
export let options = {
  vus: 10,
  iterations: 40,
};
```

여러 옵션을 설정하는 경우 각 옵션 끝에 `,`를 붙이세요.

## VU

```js
vus: 10,
```

이 줄에서 k6가 실행할 가상 사용자 수를 변경할 수 있습니다.

VU만 정의하고 다른 테스트 옵션을 정의하지 않으면 다음 오류가 발생할 수 있습니다:

```plain
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

VU 수를 설정한 경우, 해당 사용자를 실행할 기간을 다음 옵션 중 하나를 사용하여 추가로 지정해야 합니다:
- iterations
- durations
- stages

## Iterations

```js
  vus: 10,
  iterations: 40,
```

테스트 옵션에서 반복 횟수를 설정하면 *모든* 사용자에 대해 정의됩니다. 위의 예시에서 테스트는 총 40번의 반복으로 실행되며, 10명의 사용자 각각이 스크립트를 정확히 4번 실행합니다.

## Duration

```js
  vus: 10,
  duration: '2m'
```

duration을 설정하면 k6가 지정된 사용자 수 각각에 대해 duration에 도달할 때까지 스크립트를 반복(iterate)하도록 지시합니다.

Duration은 시간에 `h`, 분에 `m`, 초에 `s`를 사용하여 설정할 수 있습니다. 예시:
- `duration: '1h30m'`
- `duration: '30s'`
- `duration: '5m30s'`

duration을 설정하되 VU 수를 지정하지 않으면, k6는 기본 VU 수인 1을 사용합니다.

duration을 설정하고 반복 횟수도 함께 설정하면, 먼저 끝나는 값이 사용됩니다. 예를 들어, 다음과 같은 옵션이 있다면:

```js
  vus: 10,
  duration: '5m',
  iterations: 40,
```

k6는 40번의 반복 또는 5분 중 *먼저 끝나는 것*까지 테스트를 실행합니다. 40번의 총 반복을 완료하는 데 1분이 걸린다면, 테스트는 1분 후에 종료됩니다. 40번의 총 반복을 완료하는 데 10분이 걸린다면, 테스트는 5분 후에 종료됩니다.

### Stages

iterations와 durations를 모두 정의하면 k6가 [단순 부하 프로필](../XX-Future-Ideas/Parameters-of-a-load-test.md#Simple-load-profile)을 사용하여 테스트 스크립트를 실행하게 됩니다: VU가 시작되고, 일정 시간 또는 반복 횟수 동안 유지되다가 종료됩니다.

![단순 부하 프로필](../../images/load_profile-no_ramp-up_or_ramp-down.png)

_단순 부하 프로필_

[램프업 또는 램프다운](../XX-Future-Ideas/Parameters-of-a-load-test.md#ramp-up-and-ramp-down-periods)을 추가하여 프로필이 다음과 같이 보이게 하려면 어떻게 해야 할까요?

![상수 부하 프로필, 램프 포함](../../images/load_profile-constant.png.png)

_상수 부하 프로필, 램프 포함_

그런 경우 [stages](https://k6.io/docs/using-k6/options/#stages)를 사용할 수 있습니다.

```js
export let options = {
  stages: [
    { duration: '30m', target: 100 },
    { duration: '1h', target: 100 },
    { duration: '5m', target: 0 },
  ],
};
```

stages 옵션을 사용하면 부하 테스트의 여러 단계나 단계를 정의할 수 있으며, 각각 VU 수와 duration으로 구성할 수 있습니다. 위의 예시는 세 단계로 구성됩니다(더 추가할 수 있습니다).

1. 첫 번째 단계는 0 VU에서 100 VU로의 점진적인 램프업입니다.
2. 두 번째 단계는 [정상 상태(steady state)](../XX-Future-Ideas/Parameters-of-a-load-test.md#Steady-state)를 정의합니다. 부하는 1시간 동안 100 VU로 일정하게 유지됩니다.
3. 그런 다음 세 번째 단계는 100 VU에서 다시 0으로의 점진적인 램프다운이며, 이 시점에서 테스트가 종료됩니다.

stages는 단일 시나리오에 대한 테스트 파라미터를 정의하는 가장 다양한 방법입니다. 시뮬레이션하려는 프로덕션의 상황과 일치하도록 테스트의 부하를 형성하는 데 유연성을 제공합니다.

## 지금까지의 전체 스크립트

stages를 사용하는 경우, 스크립트는 지금 다음과 같아야 합니다:

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

## 지식 확인

### 문제 1

동일한 HTTP 요청을 정확히 100번 전송하는 스크립트를 만들도록 지시받았습니다. 이 작업을 수행하는 가장 좋은 방법은 다음 테스트 옵션 중 어느 것인가요?

A: Iterations

B: Stages

C: Duration


### 문제 2

아래에 지정된 테스트 옵션에 따라 테스트는 얼마 동안 실행될까요?

```js
export let options = {
  vus: 10,
  iterations: 3,
  duration: '1h',
};
```

A: 10시간

B: 3번의 반복을 완료하는 데 걸리는 시간 또는 1시간 중 짧은 쪽

C: 1시간에 3번의 반복을 완료하는 데 걸리는 시간을 더한 시간


### 문제 3

다음 테스트 옵션 중 10분 내에 100명의 사용자를 추가하고, 30분 동안 그 부하를 유지하며, 300 VU가 30분 동안 실행될 때까지 이 패턴을 계속하는 단계적 부하 패턴을 생성하는 것은?

A: 
```js
export let options = {
  stages: [
    { duration: '10m', target: 100 },
    { duration: '30m', target: 100 },
    { duration: '10m', target: 200 },
    { duration: '30m', target: 200 },
    { duration: '10m', target: 300 },
    { duration: '30m', target: 300 },	  
  ],
};
```

B: 

```js
export let options = {
  stages: [
    { duration: '30m', target: 300 },
};
```

C: 

```js
export let options = {
  vus: 300,
  duration: '30m',
};
```


### 정답

1. A. 반복 횟수를 100으로 설정하는 것이 이 작업을 수행하는 가장 좋은 방법입니다. [일부 executor](https://k6.io/docs/using-k6/scenarios/executors/ramping-arrival-rate)의 stages도 이를 수행할 수 있지만 모든 executor에서는 그렇지 않습니다. Duration은 테스트가 실행되는 시간만 변경하며, 구체적으로 몇 번 반복되는지는 변경하지 않습니다.
2. B. 모순되는 파라미터의 경우, k6는 더 짧은 시간 동안 실행됩니다. 이 경우 3번의 반복은 1시간보다 적은 시간이 걸릴 가능성이 높으므로, 테스트는 1시간보다 짧은 시간에 완료됩니다.
3. A. B와 C는 둘 다 300 VU가 될 때까지 가상 사용자를 점진적이고 균등하게 추가하는 테스트를 설명하기 때문에 틀립니다. 즉, 증가 속도가 일정합니다. A만이 램프업 기간 사이에 정상 상태(VU 증가가 없는 기간)를 포함하는 유일한 것이기 때문에 올바릅니다.
