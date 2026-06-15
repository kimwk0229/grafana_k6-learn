# Setup 및 Teardown 함수

지금까지 대부분의 코드를 `default` 함수 내에 넣었습니다. default 함수 내의 모든 코드는 **VU 코드**라고 하며, 모든 가상 사용자가 테스트가 끝날 때까지 실행하고 반복하는 코드입니다.

[테스트 옵션](../II-k6-Foundations/06-k6-Load-Test-Options.md)에서 가상 사용자 수와 테스트 기간을 지정하지 않으면, k6는 한 번의 iteration과 하나의 가상 사용자로 스크립트를 한 번 실행합니다. 그런데 테스트 기간을 1분으로 설정하면 어떻게 될까요?

```js
import { sleep } from 'k6'

export const options = {
	duration: '1m',
}

export default function () {

	console.log("An iteration was executed");
	sleep(1);

}


```

위 테스트를 실행하면 출력에서 다음 줄들이 보입니다:

```plain
INFO[0000] An iteration was executed                     source=console
INFO[0001] An iteration was executed                     source=console
INFO[0002] An iteration was executed                     source=console
INFO[0003] An iteration was executed                     source=console
INFO[0004] An iteration was executed                     source=console
INFO[0005] An iteration was executed                     source=console
INFO[0006] An iteration was executed                     source=console
INFO[0007] An iteration was executed                     source=console
INFO[0008] An iteration was executed                     source=console
...
...
running (1m00.0s), 0/1 VUs, 60 complete and 0 interrupted iterations
```

각 줄은 하나의 iteration, 즉 VU 코드의 단일 실행을 나타냅니다.

k6가 default 함수 내의 모든 것을 반복적으로 실행하기 때문에, 이에 맞게 스크립트를 계획할 수 있습니다. 예를 들어, 전체 함수가 테스트 전반에 걸쳐 반복된다는 것을 알기 때문에 이제 default 함수 내에서 코드를 반복할 필요가 없습니다.

그러나 모든 코드를 default 함수 내에 넣으면 테스트에 해가 될 수 있습니다.

## 모든 코드를 반복하는 경우의 문제점

default 함수 내의 모든 코드를 반복하는 것은 특히 스크립트에 다음과 같은 것이 포함된 경우 불필요하게 리소스를 많이 사용할 수 있습니다:
- 대용량 데이터 파일 생성 또는 읽기
- 테스트가 사용할 사용자 계정 생성과 같은 테스트 데이터 준비
- 가져온 라이브러리 초기화

모든 코드를 default 함수에 넣는 것과 관련하여 발생할 수 있는 또 다른 문제는 테스트하려는 비즈니스 프로세스와 관련이 있습니다.

테스트 scenario의 초점이 새 사용자 포털 페이지와 같이 로그인이 필요한 작업인 경우, 모든 iteration에서 로그인을 반복할 필요가 없을 수 있습니다. 실제로 반복적으로 로그인하고 로그아웃하면 애플리케이션의 보안 정책에 의해 테스트 사용자 계정이 잠길 수 있습니다.

또한 전체 테스트 후에만 일부 코드를 실행하고 싶을 수도 있습니다. 이에 대한 일반적인 사용 사례는 테스트가 생성했을 수 있는 불필요한 데이터를 제거하는 것입니다. 예를 들어, 장바구니에 남아 있는 모든 항목을 제거하거나 계정을 삭제할 수 있습니다. 불충분한 테스트 데이터 관리는 시간이 지남에 따라 테스트 환경을 저하시키고 부하 테스트 결과를 오해하게 만들 수 있습니다.

## 어디에 어떤 코드를 넣어야 하나요?

k6에서 코드를 어디에 넣느냐에 따라 해당 코드가 실행되는 시기와 빈도가 달라집니다.

```js
// This is init code. It runs once per VU before VU code, once before setup, and once before teardown.

export function setup() {
    // This is setup code. It runs once at the beginning of the test, regardless of the number of VUs.
}

export default function () {

     // This is VU code. It runs repeatedly until the test is stopped.
}

export function teardown() {
    // This is teardown code. It runs once at the end of the test, regardless of the number of VUs.
}
```

### Init

전역 범위의 코드는 init 코드라고 하며, 다음과 같이 실행됩니다:
- default 함수의 모든 것 이전에 각 VU가 초기화될 때 한 번
- `setup()` 함수 이전에 한 번
- `teardown()` 함수 이전에 한 번

이 섹션은 VU당 첫 번째 iteration을 실행하기 전에 필요한 준비를 하는 곳으로 생각할 수 있습니다.

init 코드에 넣을 수 있는 항목은 다음과 같습니다:
- 단일 VU의 iteration 간에 또는 VU 전반에 걸쳐 재사용하고 싶을 수 있는 변수 선언(예: [Shared Array](04-Adding-test-data.md#Shared-Array))
- 테스트의 매개변수를 설정할 수 있는 [k6 부하 테스트 옵션](../II-k6-Foundations/06-k6-Load-Test-Options.md) 객체

모든 k6 기능이 init 섹션에서 사용 가능한 것은 아닙니다. 예를 들어, init 코드 내에서 HTTP 요청을 할 수 없습니다.

### Setup

```js
export function setup() {
    // This is setup code. It runs once at the beginning of the test, regardless of the number of VUs.
}
```

setup 함수는 모든 VU 코드 이전에, 모든 VU 전체에 걸쳐 한 번만 실행되어야 하는 코드를 넣을 수 있는 헬퍼 함수입니다. init 코드 바로 뒤에 실행됩니다.

setup 중에 하고 싶을 수 있는 항목은 다음과 같습니다:
- 테스트 환경이 예상된 상태인지 확인
- 여러 VU가 사용할 테스트 데이터 생성(이 데이터는 다른 함수로 전달될 수 있음)
- [`setResponseCallback()`](https://k6.io/docs/javascript-api/k6-http/setresponsecallback-callback/)을 통해 나머지 테스트에 대한 예상 HTTP 응답 코드 설정

setup 함수에서 반환된 모든 것은 저장되고 다음과 같이 default 및 teardown 함수로 전달될 수 있습니다:

HTTP 요청 수행을 포함하여 전체 k6 API를 setup 중에 사용할 수 있습니다.

### Teardown

```js
export function teardown() {
    // This is teardown code. It runs once at the end of the test, regardless of the number of VUs.
}
```

teardown 함수는 테스트당 한 번만 실행된다는 점에서 setup 함수와 유사하지만(VU당이 아님), 테스트의 _끝_에 실행됩니다. 모든 teardown 코드는 테스트 결과가 생성되기 전에 k6가 마지막으로 실행합니다.

`setup()` 함수가 비정상적으로 종료되면 `teardown()` 함수는 호출되지 않습니다. 필요한 경우 적절한 정리를 위해 setup에 로직을 추가하는 것을 고려하세요.

teardown 중에 하고 싶을 수 있는 항목은 다음과 같습니다:
- 활성 사용자 로그아웃
- 테스트 실행 중 생성된 테스트 데이터 삭제
- 테스트 전 상태로 테스트 환경 복구

HTTP 요청 수행을 포함하여 전체 k6 API를 teardown 중에 사용할 수 있습니다.

```js
export function setup() {
  return { v: 1 };
}

export default function (data) {
  console.log(JSON.stringify(data));
}

export function teardown(data) {
  if (data.v != 1) {
    throw new Error('incorrect data: ' + JSON.stringify(data));
  }
}
```

자세한 내용은 [테스트 라이프사이클 문서](https://k6.io/docs/using-k6/test-lifecycle/)를 확인하세요.

## 디버깅 중 Setup 및 Teardown

스크립트를 디버깅하거나 데이터 생성이 필요하지 않은 소규모 점검 테스트를 실행하는 경우와 같이, 일부 상황에서는 setup과 teardown을 건너뛰고 싶을 수 있습니다.

다음과 같이 CLI를 사용하여 이 함수들을 건너뛸 수 있습니다:

```shell
k6 run test.js --no-setup --no-teardown
```

### VU당 Setup/Teardown

setup과 teardown 헬퍼 함수는 VU당이 아니라 테스트당 한 번 실행됩니다. 모든 VU의 시작이나 끝에 특정 코드를 실행하려면 어떻게 해야 할까요?

#### 각 VU 이전

다음과 같이 init 코드에서 변수를 선언하여 VU의 첫 번째 iteration에서만 코드를 실행할 수 있습니다:

```js
// This is init code. It runs once per VU before VU code, once before setup, and once before teardown.

let isFirstIteration = true;

export function setup() {
    // This is setup code. It runs once at the beginning of the test, regardless of the number of VUs.
}

export default function () {
    if (isFirstIteration) {
        // This code runs only once per VU, after the setup code but before the rest of the VU code.
		isFirstIteration = false;
    }

     // This is VU code. It runs repeatedly until the test is stopped.
}
```

이 접근 방식을 사용하면 사용자 계정으로 한 번만 로그인한 다음 인증 후 동작을 반복할 수 있습니다.

#### 각 VU 이후

테스트가 실행할 iteration 수를 정확히 알고 있다면, init 코드의 변수를 포함하는 동일한 접근 방식을 적용하여 teardown 함수로 가기 직전에 VU당 코드를 실행할 수 있습니다. 테스트가 실행된 상대적 시간에 조건부로 VU 코드를 포함할 수도 있습니다.

그러나 테스트가 몇 번의 iteration을 실행할지 또는 얼마나 오래 실행될지 항상 알 수는 없습니다. 그런 상황에서 가장 좋은 옵션은 대신 teardown 함수를 사용하는 것입니다.

아래는 무작위 사용자 계정을 선택하고, VU당 한 번 로그인하고, 그런 다음 모든 사용자 계정을 로그아웃하고 싶은 상황에서 이것이 어떻게 작동할 수 있는지에 대한 예입니다:

```js
import { sleep } from 'k6'
// This is init code. It runs once per VU before VU code, once before setup, and once before teardown.

let counter = 0;
let usernames = ['user1', 'user2', 'user3', 'user4', 'user5', 'user6', 'user7', 'user8', 'user9', 'user10'];
let passwords = ['password1', 'password2', 'password3', 'password4', 'password5', 'password6', 'password7', 'password8', 'password9', 'password10'];
let rand = Math.floor(Math.random() * usernames.length);
let username = usernames[rand];
let password = passwords[rand];
console.log(`VU#${__VU} - username: ${username}, password: ${password}`);

export function setup() {
    // This is setup code. It runs once at the beginning of the test, regardless of the number of VUs.
}

export default function () {
    if (counter === 0) {
        // This code runs only once per VU, after the setup code but before the rest of the VU code.
        console.log(`This is the first iteration of VU#${__VU} using username ${username} and password ${password}`);
    }
    counter++;

     // This is VU code. It runs repeatedly until the test is stopped.
     console.log(`This is iteration #${counter} of VU#${__VU} using username ${username} and password ${password}`);

     sleep(1);
}

export function teardown() {
    // This is teardown code. It runs once at the end of the test, regardless of the number of VUs.

    for (let i = 0; i < usernames.length; i++) {
        console.log(`Check if logged in, and if so, log out username ${usernames[i]} and password ${passwords[i]}`);
    }
}
```

## 지식 확인

### Question 1

전체 부하 테스트에서 가장 많은 횟수로 실행되는 함수는 무엇입니까?

A: default 함수

B: setup 함수

C: teardown 함수

### Question 2

k6 테스트가 100 VU로 30분 동안 실행됩니다. 테스트 스크립트에 setup 함수가 포함되어 있다고 가정할 때, setup 함수가 몇 번 실행될 것으로 예상합니까?

A: 100

B: 1

C: 30

### Question 3

테스트 스크립트에 setup, default, teardown 함수가 있고 1 VU로 1번의 iteration으로 실행됩니다. init 코드가 몇 번 실행될 것으로 예상합니까?

A: 1

B: 3

C: 5

### 정답

1. A. setup과 teardown 함수는 의도적으로 테스트당 한 번씩만 실행됩니다(각각 시작과 끝에). 한편 default 함수는 반복적으로 실행됩니다.
2. B. setup 함수는 테스트 시작 시 한 번만 실행됩니다.
3. B. Init 코드(전역 범위의 코드)는 VU당 한 번, setup 이전에 한 번, teardown 이전에 한 번 실행됩니다.
