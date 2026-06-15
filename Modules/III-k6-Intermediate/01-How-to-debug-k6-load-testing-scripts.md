# k6 부하 테스트 스크립트 디버깅 방법

k6 메트릭은 [k6 테스트에서 무슨 일이 일어났는지 이해하는 데](../II-k6-Foundations/03-Understanding-k6-results.md) 도움이 되지만, 부하 테스트 스크립트를 작성하는 과정에서는 더 자세한 정보가 필요할 수 있습니다. 이 섹션에서는 스크립트가 실패하는 이유를 파악하기 위해 할 수 있는 방법들을 알아봅니다.

[![이 k6 Office Hours에서 Tom Miseur와 Nicole van der Hoeven가 k6 부하 테스트 스크립트를 디버깅하는 다양한 방법을 시연합니다](https://img.youtube.com/vi/Zln_TWOuoho/0.jpg)](https://www.youtube.com/embed/Zln_TWOuoho)

*이 k6 Office Hours에서 Tom Miseur와 Nicole van der Hoeven가 k6 부하 테스트 스크립트를 디버깅하는 다양한 방법을 시연합니다.*

스크립트의 어느 시점에서 무슨 일이 일어나고 있는지 파악하는 데 도움이 되는 몇 가지 항목을 스크립트에 추가할 수 있습니다.

### check 추가하기

[스크립트에 check를 추가](../II-k6-Foundations/04-Adding-checks-to-your-script.md)하여 텍스트 실행 중 문제를 식별하는 방법은 이미 배웠지만, 디버깅 시에도 유용하게 활용할 수 있습니다.
스크립트가 수행하는 모든 주요 동작에 check를 포함하는 것이 좋은 관행이며, 이를 통해 문제가 어디에 있는지 명확하게 파악할 수 있습니다.

아래 스크립트를 살펴보세요.

```js
import http from 'k6/http';

let usernameArr = ['admin', 'test_user', 'guest'];
let passwordArr = ['123', '1234', '12345'];

export default function() {
    // Get random username and password from array
    let rand = Math.floor(Math.random() * 3);
    let username = usernameArr[rand];
    let password = passwordArr[rand];

    http.post('http://test.k6.io/login.php', { login: username, password: password });
}
```

이 스크립트는 두 개의 간단한 배열을 선언합니다: 하나는 사용자 이름을 포함하고, 다른 하나는 비밀번호를 포함합니다.
스크립트는 그 배열에서 무작위로 한 쌍을 선택한 다음, HTTP POST 요청에 전달합니다.

이 스크립트를 실행하면 오류가 반환되지 않는 것을 알 수 있습니다:

```shell

          /\      |‾‾| /‾‾/   /‾‾/   
     /\  /  \     |  |/  /   /  /    
    /  \/    \    |     (   /   ‾‾\  
   /          \   |  |\  \ |  (‾)  | 
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: testdata-simple.js
     output: -

  scenarios: (100.00%) 1 scenario, 1 max VUs, 10m30s max duration (incl. graceful stop):
           * default: 1 iterations for each of 1 VUs (maxDuration: 10m0s, gracefulStop: 30s)


running (00m01.1s), 0/1 VUs, 1 complete and 0 interrupted iterations
default ✓ [======================================] 1 VUs  00m01.1s/10m0s  1/1 iters, 1 per VU

     data_received..................: 6.3 kB 5.5 kB/s
     data_sent......................: 839 B  743 B/s
     http_req_blocked...............: avg=424.56ms min=248.87ms med=424.56ms max=600.26ms p(90)=565.12ms p(95)=582.69ms
     http_req_connecting............: avg=156.1ms  min=131.71ms med=156.1ms  max=180.49ms p(90)=175.61ms p(95)=178.05ms
     http_req_duration..............: avg=139.17ms min=130.49ms med=139.17ms max=147.84ms p(90)=146.1ms  p(95)=146.97ms
       { expected_response:true }...: avg=130.49ms min=130.49ms med=130.49ms max=130.49ms p(90)=130.49ms p(95)=130.49ms
     http_req_failed................: 50.00% ✓ 1        ✗ 1  
     http_req_receiving.............: avg=84.99µs  min=74µs     med=84.99µs  max=96µs     p(90)=93.8µs   p(95)=94.9µs  
     http_req_sending...............: avg=203µs    min=91µs     med=203µs    max=315µs    p(90)=292.59µs p(95)=303.79µs
     http_req_tls_handshaking.......: avg=209.81ms min=0s       med=209.81ms max=419.63ms p(90)=377.66ms p(95)=398.65ms
     http_req_waiting...............: avg=138.88ms min=130.08ms med=138.88ms max=147.67ms p(90)=145.91ms p(95)=146.79ms
     http_reqs......................: 2      1.771779/s
     iteration_duration.............: avg=1.12s    min=1.12s    med=1.12s    max=1.12s    p(90)=1.12s    p(95)=1.12s   
     iterations.....................: 1      0.885889/s
     vus............................: 1      min=1      max=1
     vus_max........................: 1      min=1      max=1
```

아무것도 오류를 나타내지 않지만, 실제로 오류가 있습니다.

이를 찾으려면 응답으로 무엇이 반환되는지 확인하는 check를 추가하세요.

```js
import http from 'k6/http';
import { check } from 'k6';

let usernameArr = ['admin', 'test_user', 'guest'];
let passwordArr = ['123', '1234', '12345'];

export default function() {
    // Get random username and password from array
    let rand = Math.floor(Math.random() * 3);
    let username = usernameArr[rand];
    let password = passwordArr[rand];

    let response = http.post('http://test.k6.io/login.php', { login: username, password: password });
    check(response, {
        'is status 200': (r) => r.status === 200,
    })
}
```

위 스크립트는 이제 반환된 응답이 HTTP 200인지 확인합니다.

해당 스크립트를 실행하면 테스트 종료 요약에서 상당한 변화를 볼 수 있습니다:

```shell
     ✗ is status 200
      ↳  0% — ✓ 0 / ✗ 1
```

스크립트가 예상대로 HTTP 200을 반환하지 않는 것으로 밝혀졌지만, check 없이는 이를 찾기 어려웠습니다. check를 추가하면 문제가 있다는 것을 식별할 수 있습니다.

그런데 무슨 일이 일어나고 있는 걸까요?

### 로깅 추가하기

스크립트가 테스트 데이터를 사용하기 때문에, 사용자 이름과 비밀번호가 잘못되었을 가능성이 있습니다. 조합이 인증 오류를 일으키는 것일 수도 있습니다. 이 경우, `username`과 `password` 배열에는 세 가지 요소만 있으므로 수동으로 모두 테스트하는 것이 어렵지 않을 것입니다. 하지만 수백 개가 있다면 어떻게 할까요?

그 경우, `console.log()`를 사용하여 스크립트의 특정 부분에 로깅을 추가해 볼 수 있습니다. 아래 스크립트는 이 구문이 실제로 작동하는 것을 보여줍니다:

```js
import http from 'k6/http';
import { check } from 'k6';

let usernameArr = ['admin', 'test_user', 'guest'];
let passwordArr = ['123', '1234', '12345'];

export default function() {
    // Get random username and password from array
    let rand = Math.floor(Math.random() * 3);
    let username = usernameArr[rand];
    let password = passwordArr[rand];
    console.log('username: ' + username, ' / password: ' + password);

    let response = http.post('http://test.k6.io/login.php', { login: username, password: password });
    check(response, {
        'is status 200': (r) => r.status === 200,
    })
}
```

`console.log()` 구문은 사용된 정확한 조합을 다음과 같이 출력합니다:

```shell
INFO[0000] username: guest  / password: 12345          source=console
```

이렇게 하면 어떤 조합을 시도해야 하는지 정확히 알 수 있습니다. 사용자 이름이나 비밀번호가 잘못되었거나, 아니면 둘 다 잘못되었을 수 있습니다. 어쨌든, 스크립트에 로그를 추가하면 스크립트가 사용하는 주요 변수의 값을 이해하는 데 도움이 됩니다.

> :warning: **부하 테스트 중에는 로깅을 비활성화하세요**. `console.log()`는 리소스를 많이 소모할 수 있으며, 너무 많은 로깅은 테스트 결과에 영향을 줄 수 있습니다. 테스트 실행 중 높은 리소스 사용률을 피하기 위해 가능한 한 로깅을 주석 처리하세요.

이제 스크립트가 선택한 사용자 이름과 비밀번호(`guest`와 `12345`)를 사용하여 앱에 로그인을 시도하면 작동하지 않는다고 상상해 보세요. 이런! 자격 증명 자체가 잘못되어 있었습니다. 가지고 있던 다른 두 계정은 작동한다는 것을 확인합니다.

세 번째 `guest` 자격 증명을 제거하고, 난수가 `0` 또는 `1`만 반환하도록 수정하세요:

```js
import http from 'k6/http';
import { check } from 'k6';

let usernameArr = ['admin', 'test_user'];
let passwordArr = ['123', '1234'];

export default function() {

    // Get random username and password from array
    let rand = Math.floor(Math.random() * 2);
    let username = usernameArr[rand];
    let password = passwordArr[rand];
	console.log('username: ' + username, ' / password: ' + password);

    let response = http.post('http://test.k6.io/login.php', { login: username, password: password });
    check(response, {
        'is status 200': (r) => r.status === 200,
    })
}
```

해당 스크립트를 다시 실행하면, 수정된 자격 증명에도 불구하고 여전히 실패한다는 것을 알 수 있습니다. 이것은 좋은 일입니다! 첫 번째 오류를 해결했고 이제 두 번째 오류를 작업할 수 있다는 것을 의미합니다.

다른 무엇이 잘못되었을까요? HTTP 200이 아니라면 어떤 응답이 반환되고 있는 걸까요?

### HTTP debug 플래그 사용하기

요청이 정확히 무엇을 반환하는지 알아내려면 `--http-debug` 플래그를 사용할 수 있습니다. 다음과 같이 테스트를 실행하세요:

```shell
k6 run test.js --http-debug
```

이번에는 출력에 새로운 정보가 포함됩니다:

```shell
          /\      |‾‾| /‾‾/   /‾‾/   
     /\  /  \     |  |/  /   /  /    
    /  \/    \    |     (   /   ‾‾\  
   /          \   |  |\  \ |  (‾)  | 
  / __________ \  |__| \__\ \_____/ .io

  execution: local
     script: testdata-simple.js
     output: -

  scenarios: (100.00%) 1 scenario, 1 max VUs, 10m30s max duration (incl. graceful stop):
           * default: 1 iterations for each of 1 VUs (maxDuration: 10m0s, gracefulStop: 30s)

INFO[0000] Request:
POST /login.php HTTP/1.1
Host: test.k6.io
User-Agent: k6/0.36.0 (https://k6.io/)
Content-Length: 26
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip

  group= iter=0 request_id=34a024e9-d51d-41e6-5695-a4a465b12978 scenario=default source=http-debug vu=1
INFO[0000] Response:
HTTP/1.1 308 Permanent Redirect
Content-Length: 164
Connection: keep-alive
Content-Type: text/html
Date: Tue, 01 Feb 2022 14:29:13 GMT
Location: https://test.k6.io/login.php

  group= iter=0 request_id=34a024e9-d51d-41e6-5695-a4a465b12978 scenario=default source=http-debug vu=1
INFO[0000] Request:
POST /login.php HTTP/1.1
Host: test.k6.io
User-Agent: k6/0.36.0 (https://k6.io/)
Content-Length: 26
Content-Type: application/x-www-form-urlencoded
Referer: http://test.k6.io/login.php
Accept-Encoding: gzip

  group= iter=0 request_id=d33e02d3-0ac5-46cd-4437-45650f2c052e scenario=default source=http-debug vu=1
INFO[0001] Response:
HTTP/1.1 403 Forbidden
Transfer-Encoding: chunked
Connection: keep-alive
Content-Type: text/plain;charset=UTF-8
Date: Tue, 01 Feb 2022 14:29:14 GMT
Set-Cookie: uid=bad; expires=Tue, 01-Feb-2022 15:29:14 GMT; Max-Age=3600; path=/; domain=test.k6.io; httponly
Set-Cookie: sid=bad; expires=Tue, 01-Feb-2022 15:29:14 GMT; Max-Age=3600; path=/; domain=test.k6.io; httponly
X-Powered-By: PHP/5.6.40

  group= iter=0 request_id=d33e02d3-0ac5-46cd-4437-45650f2c052e scenario=default source=http-debug vu=1
```

> :warning: **디버깅 중에만 HTTP-debug를 사용하세요** `http-debug`는 상당히 많은 출력을 생성할 수 있으므로, 더 길거나 큰 테스트에서는 사용을 피하세요. 스크립트 작성 중 문제 해결에 가장 적합합니다.

테스트 종료 요약에는 이제 2개의 요청과 2개의 응답에 대한 정보가 포함됩니다. 잠깐, 스크립트에는 요청이 하나만 있지 않나요?

출력을 더 분석하면 무슨 일이 일어났는지 알 수 있습니다:
- k6가 초기 POST 요청을 보냈습니다.
- 서버가 HTTP 308 리디렉션으로 응답했습니다.
- k6가 리디렉션에 의해 또 다른 POST를 보냈습니다.
- 서버가 HTTP 403 Forbidden으로 응답했습니다.

스크립트에는 단일 요청만 포함되어 있지만, 서버의 응답이 k6로 하여금 리디렉션된 요청을 보내게 했으며, 이 요청이 결국 실패했습니다. [HTTP 403 Forbidden](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403)은 인증 문제와 관련된 오류입니다. 이 오류는 인증 오류의 이론을 확인합니다.

그런데 *왜* 인증 오류가 발생하는 걸까요?

더 많은 정보를 얻으려면, 이번에는 전체 HTTP debug를 요청하여 스크립트를 다시 실행하세요:

```shell
k6 run test.js --http-debug=full
```

이번에는 응답 및 요청 헤더뿐만 아니라 요청 및 응답 *바디*도 볼 수 있습니다:

```shell
INFO[0000] Request:
POST /login.php HTTP/1.1
Host: test.k6.io
User-Agent: k6/0.36.0 (https://k6.io/)
Content-Length: 26
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip

login=user&password=userpw  group= iter=0 request_id=9b3c2dd9-8c2a-49bf-7a92-713c757d1a3a scenario=default source=http-debug vu=1
INFO[0000] Response:
HTTP/1.1 308 Permanent Redirect
Content-Length: 164
Connection: keep-alive
Content-Type: text/html
Date: Tue, 01 Feb 2022 14:35:34 GMT
Location: https://test.k6.io/login.php

<html>
<head><title>308 Permanent Redirect</title></head>
<body>
<center><h1>308 Permanent Redirect</h1></center>
<hr><center>nginx</center>
</body>
</html>
  group= iter=0 request_id=9b3c2dd9-8c2a-49bf-7a92-713c757d1a3a scenario=default source=http-debug vu=1
INFO[0000] Request:
POST /login.php HTTP/1.1
Host: test.k6.io
User-Agent: k6/0.36.0 (https://k6.io/)
Content-Length: 26
Content-Type: application/x-www-form-urlencoded
Referer: http://test.k6.io/login.php
Accept-Encoding: gzip

login=user&password=userpw  group= iter=0 request_id=3d0d9247-b93e-47f0-6dee-2f0a3420ca4e scenario=default source=http-debug vu=1
INFO[0001] Response:
HTTP/1.1 403 Forbidden
Transfer-Encoding: chunked
Connection: keep-alive
Content-Type: text/plain;charset=UTF-8
Date: Tue, 01 Feb 2022 14:35:35 GMT
Set-Cookie: uid=bad; expires=Tue, 01-Feb-2022 15:35:35 GMT; Max-Age=3600; path=/; domain=test.k6.io; httponly
Set-Cookie: sid=bad; expires=Tue, 01-Feb-2022 15:35:35 GMT; Max-Age=3600; path=/; domain=test.k6.io; httponly
X-Powered-By: PHP/5.6.40

34
error: invalid csrf token
token: <not set>
session: 
0
```

HTTP 403의 응답 바디가 문제를 드러냅니다: `error: invalid csrf token`.

스크립트가 사용자 이름과 비밀번호를 POST하지만 CSRF 토큰은 없습니다! 맞습니다.

CSRF([Cross-Site Request Forgery](https://owasp.org/www-community/attacks/csrf)) 토큰은 공격자가 사용자의 세션을 탈취하는 것을 방지하여 보안을 향상시키는 데 사용됩니다. 애플리케이션이 스크립트가 아직 가지고 있지 않은 CSRF 토큰을 요구하는 것으로 보입니다. CSRF 토큰을 얻으려면, `https://test.k6.io/`에 대한 초기 요청을 만들고 응답에서 해당 토큰을 파싱할 수 있습니다. 이것은 다음 섹션에서 다룰 예정입니다.

## 지식 확인

### Question 1

스크립트가 보낸 요청에 대한 응답으로 애플리케이션이 PDF 파일을 반환하는지 확인하는 가장 좋은 방법은 무엇입니까?

A: 응답 바디 크기를 기반으로 check를 추가합니다.

B: `http-debug` 플래그를 사용하여 PDF 내용을 읽으려고 합니다.

C: 요청 후 `console.log()` 구문을 추가합니다.

### Question 2

이 페이지에서 사용된 스크립트에서는 배열에서 사용자 계정을 선택하기 위해 난수를 사용합니다. 스크립트를 실행할 때마다 난수의 값을 어떻게 알 수 있습니까?

A: 

```js
check(response, {
        'random number selected': (r) => r.rand === 0,
    })
```

B: `k6 run test.js --http-debug="rand"`

C: `console.log('rand', rand);`

### Question 3

램프 업 부하 테스트 중에 가능한 한 많은 로깅을 비활성화해야 하는 이유는 무엇입니까?

A: 제대로 된 테스트를 실행할 때는 절대 로깅을 하면 안 됩니다.

B: 불필요한 로깅은 부하 생성기의 리소스를 소모합니다.

C: `http-debug`를 사용하는 것이 부하 테스트 중에 더 나은 선택입니다.

### 정답

1. A. `http-debug`와 `console.log` 방식은 불필요하게 장황하고 부하 생성기 리소스에 부담이 될 수 있습니다. PDF 파일이 PDF 없이 반환되는 응답 크기보다 훨씬 크다는 것을 알고 있다면, 응답 바디 크기를 사용하여 PDF를 확인할 수 있습니다.
2. C. `console.log()`는 이런 사용 사례, 특히 스크립트를 문제 해결하거나 디버깅할 때 훌륭합니다.
3. B. 로그에 너무 많은 정보를 쓰는 것 자체가 부하 생성기 내의 성능 병목이 될 수 있으므로, 출력하고 저장하는 정보에 대해 신중하게 판단하세요.
