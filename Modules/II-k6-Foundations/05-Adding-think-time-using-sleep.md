# sleep을 사용하여 think time 추가하기

부하 테스트를 확장하기 전에 추가해야 할 한 가지가 더 있습니다: think time입니다.

_Think time_은 실제 사용자가 애플리케이션을 사용하는 과정에서 발생하는 지연을 시뮬레이션하기 위해 테스트 실행 중 스크립트가 일시 정지하는 시간입니다.

### 언제 think time을 사용해야 하나요?

일반적으로 최종 사용자의 동작을 정확하게 시뮬레이션하기 위해 think time을 사용하면 부하 테스트 스크립트가 더 현실적이 됩니다. 현실성이 테스트 목표 달성에 도움이 된다면, think time을 사용하는 것이 좋습니다.

다음과 같은 상황에서 think time 추가를 고려해야 합니다:
- 테스트가 특정 순서로 애플리케이션의 다른 부분에 접근하는 것과 같은 사용자 흐름을 따르는 경우
- 페이지에서 텍스트를 읽거나 양식을 작성하는 것과 같이 시간이 걸리는 작업을 시뮬레이션하려는 경우
- 부하 생성기(k6를 실행하는 머신)가 테스트 실행 중 높은(> 80%) CPU 사용률을 보이는 경우.

think time을 제거하거나 줄이는 주요 위험은 요청이 전송되는 속도가 빨라져 CPU 사용률이 높아질 수 있다는 것입니다. CPU 사용률이 너무 높으면 부하 생성기 자체가 요청을 *전송*하는 것으로 어려움을 겪을 수 있어, 거짓 음성과 같은 부정확한 결과가 나올 수 있습니다. think time을 추가하는 것은 [높은 CPU 사용률을 줄이는](https://k6.io/docs/cloud/analyzing-results/performance-insights/#high-load-generator-cpu-usage) 방법 중 하나입니다.

### 언제 think time을 사용하지 않아야 하나요?

think time을 사용하면 테스트에서 VU당 달성할 수 있는 최대 요청 속도가 줄어듭니다. 요청이 전송되는 속도가 느려집니다.

다음과 같은 상황에서는 think time이 불필요합니다:
- 애플리케이션이 처리할 수 있는 초당 요청 수를 알아내기 위한 [스트레스 테스트](https://k6.io/docs/test-types/stress-testing/)를 수행하려는 경우
- 테스트 중인 API 엔드포인트가 지연 없이 프로덕션에서 높은 초당 요청 수를 경험하는 경우
- 부하 생성기가 CPU 사용률 80% 선을 넘지 않고 테스트 스크립트를 실행할 수 있는 경우.

마지막 포인트는 think time이 이전 섹션에서 논의한 것처럼 부하 생성기의 상태에 영향을 줄 만큼 테스트 처리량을 증가시키지 않도록 하기 위한 요구 사항입니다.

보시다시피, think time 사용 여부는 테스트 목표에 따라 달라집니다. 확신이 없다면 think time을 사용하세요.

## Sleep

k6에서는 [`sleep()`](https://k6.io/docs/javascript-api/k6/sleep-t/)을 사용하여 think time을 추가할 수 있습니다. 사용하려면 sleep을 임포트한 다음, 테스트 실행을 일시 정지하려는 default 함수 내의 스크립트 부분에 sleep을 추가해야 합니다:

```js
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

`sleep(1);`은 실행될 때 스크립트가 1초 동안 일시 정지됨을 의미합니다.

sleep을 포함해도 응답 시간(`http_req_duration`)에는 영향을 미치지 않습니다; 응답 시간은 항상 sleep이 제외된 상태로 보고됩니다. 그러나 sleep은 [반복 duration](03-Understanding-k6-results.md#Iteration-duration)에는 포함됩니다.

### 동적 think time

스크립트에 지연을 하드코딩하는 문제점은 테스트에 인위적인 패턴을 도입하여 나중에 애플리케이션에 대한 부하가 프로덕션에서보다 더 예측 가능하게 될 수 있다는 것입니다.

**테스트 모범 사례:** 동적 think time을 사용하세요.

동적 think time은 더 현실적이며, 실제 사용자를 더 정확하게 시뮬레이션하여 테스트 결과의 정확성과 신뢰성을 향상시킵니다.

#### 랜덤 sleep

동적 think time을 구현하는 한 가지 방법은 JavaScript의 `Math.random()` 함수를 사용하는 것입니다:

```js
sleep(Math.random() * 5);
```

위의 줄은 k6가 0부터 5 사이의 랜덤한 시간 동안 sleep하도록 지시합니다(0 포함, 5 미포함). 선택된 값이 정수임을 보장하지 않습니다.

#### 정수 사이 랜덤 sleep

think time을 정수로 정의하는 것을 선호한다면, [jslib](https://jslib.k6.io/)이라는 k6의 유용한 함수 라이브러리에서 `randomIntBetween` 함수를 사용해보세요.

먼저 관련 함수를 임포트하세요:

```js
import { randomIntBetween } from "https://jslib.k6.io/k6-utils/1.0.0/index.js";
```

그런 다음 default 함수 내에 다음을 추가하세요:

```js
sleep(randomIntBetween(1,5));
```

스크립트는 1에서 5 사이(1과 5 모두 포함)의 초 동안 일시 정지합니다.

## 얼마나 많은 think time을 추가해야 하나요?

실제 답은: 경우에 따라 다릅니다.

추가하는 think time의 기간에 영향을 미치는 요소로는 테스트 목표, 프로덕션 트래픽의 모습, 사용 가능한 컴퓨팅 리소스 등이 있습니다. 테스트 스크립트를 프로덕션 환경에서 발생하는 것과 최대한 유사하게 모델링하는 것이 가장 좋습니다.

그러나 프로덕션 트래픽에 대한 데이터가 없는 경우, *직접* 사용자 흐름을 통과하는 데 걸리는 시간을 측정하고 이를 시작점으로 사용할 수 있습니다.

## 지식 확인

### 문제 1

사용자가 이름, 이메일 주소, 문제 설명을 입력하고 응답이 애플리케이션 팀에 전송되는 새로운 "티켓 열기" 페이지를 테스트하고 있습니다. think time을 사용해야 하나요?

A: 네, 애플리케이션 팀이 티켓에 응답하는 데 시간이 걸리기 때문입니다.

B: 네, 사용자가 문제를 입력하는 데 시간이 걸리기 때문입니다.

C: 아니요, 부하 생성기의 CPU 사용률이 너무 높기 때문입니다.

### 문제 2

다음 줄에서 숫자 3은 무엇을 나타내나요?

`sleep(3)`

A: 3밀리초의 think time

B: think time을 받을 반복 횟수

C: 3초의 think time

### 문제 3

think time이 없는 스크립트가 단일 반복으로 실행되었을 때 반복 duration이 5초였습니다. 스크립트에 1초의 sleep이 포함되었다면 반복 duration이 얼마였을까요?

A: 5초

B: 6초

C: 4초

### 정답

1. B. A는 think time이 애플리케이션 팀이 응답하는 데 걸리는 시간이 아니라 사용자가 애플리케이션과 상호작용하는 데 걸리는 시간을 시뮬레이션하기 때문에 틀립니다. C는 think time을 추가해도 CPU 사용률이 증가하지 않기 때문에 틀립니다; 실제로 반대로 요청 간격을 두어 CPU 사용률을 줄입니다.
2. C. `sleep()`의 파라미터는 초 단위로 주어지므로, `sleep(3)`은 스크립트 실행을 3초 동안 일시 정지합니다.
3. B. 스크립트가 sleep 없이 실행하는 데 5초가 걸리고 1초의 sleep이 포함된다면, 스크립트는 실행하는 데 6초가 걸립니다.
