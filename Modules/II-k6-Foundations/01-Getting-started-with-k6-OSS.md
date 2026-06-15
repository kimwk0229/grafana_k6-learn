# k6 OSS 시작하기

k6로 스크립팅을 시작하는 방법은 여러 가지가 있지만, 몇 가지 이유로 [k6 OSS](https://github.com/grafana/k6)부터 시작합니다:
- 그 자체로 완전한 기능을 갖춘 부하 테스트 도구이며, 구독이나 결제 없이 사용할 수 있습니다.
- SaaS 플랫폼인 k6 Cloud도 k6 OSS를 사용하므로, 이 섹션에서 배운 기술은 나중에 k6 Cloud를 사용하기로 결정하더라도 적용됩니다.
- k6 OSS 스크립트에 고급 시나리오와 기능을 추가할 수 있습니다. 나중에 다룰 다른 스크립트 생성 방법은 기능이 제한적입니다.

시작해봅시다!

## 설치

먼저, 운영 체제에 맞는 [설치 지침](https://k6.io/docs/getting-started/installation/)을 따라 k6를 설치하세요.

다음으로, 좋아하는 IDE나 텍스트 편집기를 선택하세요. 많은 분들이 [VS Code](https://code.visualstudio.com/)를 사용하고 권장하지만, [Sublime Text](https://www.sublimetext.com/), [Atom](https://atom.io/), 또는 텍스트 파일을 생성할 수 있는 이미 사용 중인 다른 편집기도 사용할 수 있습니다.

## 첫 번째 k6 스크립트 작성하기

이제 스크립트를 작성할 시간입니다!

k6는 여러 프로토콜을 지원하지만, 지금은 HTTP에 집중하겠습니다. 첫 번째 스크립트는 보낸 내용을 그대로 되돌려 주는 테스트 API에 기본 HTTP POST 요청을 수행합니다.

k6 테스트를 만드는 가장 빠른 방법은 k6 버전 0.48.0에서 도입된 `k6 new [filename]` 명령을 사용하는 것입니다. 이 명령은 빠르게 시작하는 데 필요한 기본 보일러플레이트가 포함된 파일을 자동으로 생성합니다.

그러나 k6 학습의 일환으로, 테스트를 수동으로 만드는 방법도 가르쳐 드립니다.

`test.js`라는 새 파일을 만들고, 좋아하는 IDE에서 열어보세요. 이 파일이 바로 테스트 스크립트입니다. k6 자체는 Go로 작성되었지만, k6 스크립트는 항상 JavaScript로 작성됩니다. 여기서 단계별로 스크립트를 함께 만들겠습니다. 코드 스니펫을 필요에 따라 복사하여 붙여넣어 스크립트가 이와 같이 보이도록 하세요.

내장 모듈 `k6/http`에서 HTTP Client를 임포트합니다:

```js
import http from 'k6/http';
```

이제 default 함수를 만들고 내보냅니다:

```js
export default function() {
}
```

`default` 함수 내의 코드는 테스트가 실행될 때 각 k6 가상 사용자에 의해 실행됩니다.

실제 HTTP 호출을 위한 로직을 추가합니다:

```js
import http from 'k6/http';

export default function() {
  let url = 'https://httpbin.test.k6.io/post';
  let response = http.post(url, 'Hello world!');
}
```

여기서 `Hello world!`라는 본문과 함께 API 엔드포인트 `https://httpbin.test.k6.io/post`에 HTTP POST 요청을 보내도록 k6에 지시합니다.

이 스크립트를 실제로 이미 실행할 수 있지만, 제대로 작동했는지 어떻게 알 수 있을까요? 응답을 콘솔에 기록하는 방법은 다음과 같습니다:

```js
import http from 'k6/http';

export default function() {
  let url = 'https://httpbin.test.k6.io/post';
  let response = http.post(url, 'Hello world!');

  console.log(response.json().data);
}
```

나중에 테스트 결과를 확인하는 더 많은 방법을 배울 것이지만, 지금은 첫 번째 테스트를 실행해봅시다!

## Hello World: k6 스크립트 실행하기

편집기에서 스크립트를 저장하세요. 그런 다음 터미널을 열고 k6 스크립트를 저장한 디렉토리로 이동하세요. 이제 테스트를 실행하세요:

```js
k6 run test.js
```

다음과 같은 결과가 나와야 합니다:

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

많은 메트릭이 있네요! 다음 섹션에서 각 줄이 의미하는 바를 살펴보겠습니다.

## 기타 리소스

[![Week of Load testing day 3: Installing k6 and running load test](../../images/week-of-testing-youtube.png)](https://www.youtube.com/embed/y5tteMKZUqk)

## 지식 확인

### 문제 1

HTTP 응답의 JSON 본문에 접근하는 가장 좋은 방법은?

A: `response.json()`

B: `response.body()`

C: `response.content`

### 문제 2

HTTP 클라이언트가 포함된 내장 모듈의 이름은?

A: `http`

B: `k6/http-client`

C: `k6/http`

### 문제 3

가상 사용자가 실행하게 하려면 HTTP 호출을 테스트 스크립트의 어느 곳에 배치해야 하는가?

A: 전역 스코프

B: export된 default 함수 안

C: `exec()`라는 함수 안

### 정답

1. A. `response.json()`을 사용하면 응답 본문을 가져올 뿐만 아니라 JSON도 파싱합니다.
2. C. `k6/http`는 HTTP를 사용하려면 k6 스크립트에서 임포트해야 하는 모듈입니다. 다른 두 옵션은 존재하지 않습니다.
3. B. export된 함수(default 함수 또는 [시나리오](https://k6.io/docs/using-k6/scenarios/#common-options)의 `exec` 옵션에 명명된 함수) 내에 배치된 코드만 가상 사용자에 의해 실행됩니다.
