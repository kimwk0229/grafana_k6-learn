# Externally Controlled Executor

[executor로 부하 프로필 설정하기](../08-Setting-load-profiles-with-executors.md#Externally-Controlled)에서 언급한 바와 같이, 이 특정 executor는 VU 제어와 테스트의 실행 상태를 외부 프로세스에 위임합니다. Bash, Python 또는 어떤 자동화 컴포넌트든 자유롭게 사용할 수 있습니다. 이러한 프로세스의 출처는 executor에 관계없습니다.

executor의 초점은 테스트 시나리오를 설정하고 전체 duration 및 허용 가능한 virtual user 수에 대한 제약을 제공하는 것입니다. 이 시점부터 실행 중인 테스트는 추가 명령을 기다리는 _대기 상태_에 있게 됩니다.

실행 중인 테스트와의 상호 작용은 k6 프로세스가 노출하는 [REST API](https://k6.io/docs/misc/k6-rest-api/)나 `k6` 명령줄 인터페이스([CLI](https://k6.io/blog/how-to-control-a-live-k6-test/))를 사용하여 이루어집니다.

## 연습 문제
이번 연습에서는 HTTP 요청을 수행한 뒤 테스트 iteration이 완료되기 전 3초를 대기하는 매우 기본적인 스크립트를 사용하여 시작합니다. 변경 사항에 따른 콘솔 출력도 확인할 수 있습니다.

설정된 `options`는 `externally-controlled` executor에 필요한 절대 최소한의 것입니다.

### 테스트 스크립트 작성
다음 내용으로 _test.js_ 파일을 생성하세요:
```js
import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
  scenarios: {
    k6_workshop: {
      executor: 'externally-controlled',
      duration: '5m',
    },
  },
};

export default function () {
  console.log(`[VU: ${__VU}, iteration: ${__ITER}] Starting iteration...`);
  http.get('https://test.k6.io/contacts.php');
  sleep(3);
}
```

### 초기 테스트 실행
`test.js` 스크립트가 있는 동일한 디렉토리 내에서 _터미널_ 창을 엽니다. 이제 k6 바이너리를 사용하여 스크립트를 실행합니다:
```bash
k6 run test.js
```
이제 k6가 시작되어야 합니다. 타이머가 테스트 실행 중에 계속 올라가는 것이 보이지만, 실제로는 아무 일도 일어나지 않습니다. 스크립트에 포함된 예상 콘솔 메시지가 표시되지 않습니다! 타이머가 스크립트 options에서 설정된 `duration`에 도달할 때까지 이 대기 상태가 계속됩니다.

### VU 확장
스크립트에 의해 시작된 virtual user (VU)가 `0`이기 때문에 실제 테스트가 수행되지 않았습니다. k6는 외부 프로세스가 VU를 _확장_하기를 기다립니다. 확장하기 위해 'k6' 명령줄을 사용합니다.

다른 _터미널_ 창을 엽니다. 이번에는 디렉토리가 중요하지 않습니다. 그 사이에 스크립트가 시간 초과되었다면 `k6 run test.js`를 사용하여 다시 시작합니다.
```bash
k6 scale --vus 2 --max 10
```
> :point_up: 스크립트 options에 `maxVUs`를 지정하지 않으면, 확장 요청이 `--max`와 함께 제공되지 않는 한 모든 확장 요청이 실패합니다!

이제 VU가 확장되었으므로 각 virtual user가 테스트 코드를 실행하고 있음을 보여주는 콘솔 출력이 표시되어야 합니다.

### 실행 엔진 제어
이전 단계에서 현재 실행 중인 테스트를 계속하면서, 전체 테스트 실행을 제어하는 데 사용 가능한 다른 옵션들을 살펴보겠습니다.

유휴 상태의 _터미널_ 창(현재 k6 테스트를 실행하지 않는 창)을 사용하여 다음 명령을 실행합니다:
```bash
# Halt the currently running test; console output should stop
k6 pause

# Go ahead and scale up the desired number of VUs
k6 scale --vus 5

# Resume the test; output should confirm the test is running with additional VUs
k6 resume
```
 
### 상태 확인
언제든지 `k6` 명령을 사용하여 실행 중인 인스턴스의 상태를 조회할 수 있습니다:

```bash
$ k6 status
status: 7
paused: "false"
vus: "5"
vus-max: "10"
stopped: false
running: true
tainted: false
```

### 메트릭 스냅샷 수집
스크립트가 실행 중일 때, 일시 중지되었든 활성화되었든 실행 중인 스크립트에서 현재 메트릭을 폴링할 수 있습니다. 이는 현재 메트릭을 YAML 형식으로 콘솔에 덤프하며, 출력 파일로 _파이프_할 수 있습니다.

```bash
$ k6 stats
...
- name: http_req_duration
  type:
      type: trend
      valid: true
  contains:
      type: time
      valid: true
  tainted: ""
  sample:
      avg: 60.06574999999998
      max: 71.202
      med: 59.515
      min: 55.203
      p(90): 63.559000000000005
      p(95): 65.21095
...
```

### 테스트 종료
`duration` 기간이 종료되기 전에 테스트를 완료하려면 테스트를 실행하는 _터미널_에서 `Ctrl+C` 키보드 명령을 사용하거나 REST API를 사용해야 합니다:
```bash
curl -X PATCH \
  http://localhost:6565/v1/status \
  -H 'Content-Type: application/json' \
  -d '{
    "data": {
        "attributes": {
            "stopped": true
        },
        "id": "default",
        "type": "status"
    }
}'
```

> `k6` CLI는 동등한 `k6 stop` 명령을 제공하지 않습니다. 테스트를 종료하려면 단순히 `Ctrl+C`를 사용하세요.

### 스크립트 options
초기 스크립트는 테스트를 시작하는 데 필요한 최소한의 것을 제공합니다.

`maxVUs` 지정을 고려하세요. 앞서 언급한 바와 같이 `maxVUs`는 필수가 아닙니다. 그러나 초기 확장 요청에 `--max`가 지정되지 않으면 모든 확장 시도가 오류로 응답받게 됩니다. `maxVUs` 값은 필요하다고 판단될 경우 `--max` 인수를 사용하여 재정의할 수 있습니다.

마지막 스크립트 옵션은 `vus` 설정입니다. 제공하면 스크립트가 시작되자마자 이 수의 VU가 처리를 시작하여 초기 확장 필요성을 없애줍니다.

_test.js_ 스크립트의 _options_ 선언을 업데이트하여 이러한 추가 설정을 포함해 보겠습니다:
```js
export const options = {
    scenarios: {
        k6_workshop: {
            executor: 'externally-controlled',
            duration: '5m',
            maxVUs: 10,
            vus: 2,
        },
    },
};
```

이제 스크립트를 실행하면 즉시 2개의 virtual user에 의해 테스트가 실행되고 있음을 보여줍니다:
```bash
$ k6 run test.js

          /\      |‾‾| /‾‾/   /‾‾/   
     /\  /  \     |  |/  /   /  /    
    /  \/    \    |     (   /   ‾‾\  
   /          \   |  |\  \ |  (‾)  | 
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: scripts/test.js
     output: -

  scenarios: (100.00%) 1 scenario, 10 max VUs, 5m0s max duration (incl. graceful stop):
           * k6_workshop: Externally controlled execution with 2 VUs, 10 max VUs, 5m0s duration

INFO[0000] [VU: 4, iteration: 0] Starting iteration...   source=console
INFO[0000] [VU: 1, iteration: 0] Starting iteration...   source=console
INFO[0003] [VU: 1, iteration: 1] Starting iteration...   source=console
INFO[0003] [VU: 4, iteration: 1] Starting iteration...   source=console
INFO[0006] [VU: 1, iteration: 2] Starting iteration...   source=console
INFO[0006] [VU: 4, iteration: 2] Starting iteration...   source=console
```

### 마무리
이상입니다! 외부 소스에서 k6 부하 테스트를 제어하는 추가적인 힘을 확인할 수 있기를 바랍니다!
