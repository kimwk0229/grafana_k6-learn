# k6에서의 동적 상관관계

이전 섹션에서 [k6에서 스크립트를 디버깅하는 다양한 방법](01-How-to-debug-k6-load-testing-scripts.md)을 배웠고, 모든 부하 테스터가 직면하는 문제인 상관관계(correlation)를 만났습니다.

## 상관관계란 무엇인가요?

데이터 상관관계(correlation)는 이전 HTTP 응답에서 값을 추출하고 그 값을 다음 HTTP 요청에 사용하는 프로세스입니다.

애플리케이션에 접근하는 사용자의 요청 흐름은 다음과 같을 수 있습니다:
- 1단계. *내 메시지* 페이지에 대한 HTTP GET 요청.
- 2단계. 사용자 이름과 비밀번호를 포함한 HTTP POST 요청

[이전 섹션](./01-How-to-debug-k6-load-testing-scripts.md)에서 스크립트가 즉시 로그인을 시도했지만(1단계를 건너뛰고 바로 2단계로 이동), 일부 디버깅 기법을 적용한 후, 응답 코드가 HTTP 403 Forbidden이고 응답 바디에 `error: invalid csrf token`이 포함되어 있다는 것을 발견했습니다.

이 오류를 수정하고 사용자 행동을 정확하게 모방하려면, 스크립트가 다음을 수행해야 합니다:
- 1단계의 요청에 대한 응답을 저장합니다.
- 1단계에서 [Cross-Site Request Forgery (CSFR)](https://owasp.org/www-community/attacks/csrf) 토큰을 추출합니다.
- 2단계를 요청할 때 CSFR 토큰을 전달합니다.

스크립트가 2단계에서 시작하기 때문에, 1단계에 대한 요청을 추가해야 합니다.

## 이전 요청 스크립팅

지금은 `default` 함수 안의 모든 것을 주석 처리하고 이 코드를 붙여 넣으세요:

```js
let response = http.get('https://test.k6.io/my_messages.php');
    check(response, {
        'is Unauthorized': r => r.body.includes('Unauthorized'),
    })
```

이 스니펫은 *내 메시지* 페이지를 요청하고 바디에 "Unauthorized"라는 단어가 포함되어 있는지 확인합니다. 실제로 작동하는 것을 보려면 전체 HTTP 디버깅으로 스크립트를 실행하세요:

```shell
k6 run test.js --http-debug=full
```

출력은 다음과 비슷하게 보여야 합니다:

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
GET /my_messages.php HTTP/1.1
Host: test.k6.io
User-Agent: k6/0.36.0 (https://k6.io/)
Accept-Encoding: gzip

  group= iter=0 request_id=aae39c79-6ad4-47de-631a-6213e3b218be scenario=default source=http-debug vu=1
INFO[0001] Response:
HTTP/1.1 200 OK
Transfer-Encoding: chunked
Connection: keep-alive
Content-Type: text/html; charset=UTF-8
Date: Thu, 03 Feb 2022 14:01:13 GMT
Set-Cookie: csrf=NjI1NjkwOTkw; expires=Thu, 03-Feb-2022 15:01:13 GMT; Max-Age=3600; path=/; domain=test.k6.io; httponly
X-Powered-By: PHP/5.6.40

3bb
 

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>
<title>My messages</title>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<link rel="icon" href="/static/favicon.ico" sizes="32x32">
</head>
<body>

<p><a href="/">&lt; Back</a></p>


<h2>Unauthorized</h2>
<p><small>Hint: try <code>admin</code>/<code>123</code> or
<code>test_user</code>/<code>1234</code></small></p>

<form method="POST" action="/login.php">
<input type="hidden" name="redir" value="1"> 
<input type="hidden" name="csrftoken" value="NjI1NjkwOTkw">
<table cellpadding="5" cellspacing="0" border="0" width="1%">
<tr>
  <td>Login:</td><td><input type="text" name="login" autocomplete="off"></td>
</tr>
<tr>
  <td>Password:</td><td><input type="password" name="password" autocomplete="off"></td>
</tr>
</table>
<input type="submit" value="Go!">
</form>


<p><small>Imitation page. Copyright &copy; 2022, k6.io</small></p>

</body>
</html>

0

  group= iter=0 request_id=aae39c79-6ad4-47de-631a-6213e3b218be scenario=default source=http-debug vu=1

running (00m00.6s), 0/1 VUs, 1 complete and 0 interrupted iterations
default ✓ [======================================] 1 VUs  00m00.6s/10m0s  1/1 iters, 1 per VU

     ✓ is Unauthorized

     checks.........................: 100.00% ✓ 1        ✗ 0
     data_received..................: 6.7 kB  11 kB/s
     data_sent......................: 526 B   866 B/s
     http_req_blocked...............: avg=493.46ms min=493.46ms med=493.46ms max=493.46ms p(90)=493.46ms p(95)=493.46ms
     http_req_connecting............: avg=112.34ms min=112.34ms med=112.34ms max=112.34ms p(90)=112.34ms p(95)=112.34ms
     http_req_duration..............: avg=112.26ms min=112.26ms med=112.26ms max=112.26ms p(90)=112.26ms p(95)=112.26ms
       { expected_response:true }...: avg=112.26ms min=112.26ms med=112.26ms max=112.26ms p(90)=112.26ms p(95)=112.26ms
     http_req_failed................: 0.00%   ✓ 0        ✗ 1
     http_req_receiving.............: avg=180µs    min=180µs    med=180µs    max=180µs    p(90)=180µs    p(95)=180µs   
     http_req_sending...............: avg=66µs     min=66µs     med=66µs     max=66µs     p(90)=66µs     p(95)=66µs    
     http_req_tls_handshaking.......: avg=378.53ms min=378.53ms med=378.53ms max=378.53ms p(90)=378.53ms p(95)=378.53ms
     http_req_waiting...............: avg=112.02ms min=112.02ms med=112.02ms max=112.02ms p(90)=112.02ms p(95)=112.02ms
     http_reqs......................: 1       1.646844/s
     iteration_duration.............: avg=606.35ms min=606.35ms med=606.35ms max=606.35ms p(90)=606.35ms p(95)=606.35ms
     iterations.....................: 1       1.646844/s
```

이제 뭔가 진전이 있습니다! 응답의 전체 HTML 바디가 변수 [`response`](https://k6.io/docs/javascript-api/k6-http/response/)에 저장됩니다.

## 응답에서 값 추출하기

이전 출력의 로그인 폼 HTML에는 CSRF 토큰이 포함되어 있습니다:

```html
<input type="hidden" name="csrftoken" value="NjI1NjkwOTkw">
```

이 값은 동적이므로, 스크립트는 제시된 형식에 따라 이 값을 추출해야 합니다.

스크립트에 다음 줄을 추가하세요:

```js
let csrfToken = response.html().find("input[name=csrftoken]").attr("value");
console.log(csrfToken);
```

첫 번째 줄은 HTML 응답 바디를 파싱하고 `name`이 `csrftoken`과 같은 `input` 요소의 값을 찾습니다. 스크립트를 실행하여 `csrfToken`의 값이 올바른지 확인하세요. 다음과 비슷한 결과가 나와야 합니다:

```js
INFO[0001] NDExODkxOTcz                                  source=console
```

작동하는 것 같습니다! 스크립트가 작동하도록 노력해 보세요:
- 로그인 요청의 주석을 해제하고 추출된 CSRF 토큰이 요청과 함께 전달되도록 수정하세요
- 로그인 요청 후 "successfully authorized" 문구에 대한 check를 추가하여 스크립트 사용자가 로그인했는지 확인하세요.

막히면, 이 섹션 끝의 스크립트와 비교해 보세요.

## 지식 확인

### Question 1

이전 응답에서 동적 값을 상관관계(correlation)로 처리하여 요청과 함께 보내야 할 수도 있다고 의심됩니다. 이 의심을 확인하는 가장 좋은 방법은 무엇입니까?

A: DevTools를 사용하여 작업을 수행하면서 네트워크 트래픽을 살펴보고, 성공적인 요청과 함께 전달되는 매개변수를 확인합니다.

B: 스크립트를 기록하고 k6로 실행합니다.

C: `csrftoken`이라는 단어에 대한 check를 추가하여 응답에서 반환되는 것이 있는지 확인합니다.

### Question 2

다음 중 상관관계(correlation)가 필요한 경우를 식별하는 데 도움이 될 수 있는 것은 무엇입니까?

A: 
```js
check(response, {
        'is Unauthorized': r => r.body.includes('Unauthorized'),
    })
```
B:  `let rand = Math.floor(Math.random() * 2);`

C: `--http-debug=full`

### Question 3

기록된 스크립트를 재생하면 오류가 발생할 수 있는 이유는 무엇입니까?

A: 일부 단계에서 이전 응답에서 추출된 동적 값이 필요할 수 있습니다.

B: 스크립트에 check가 정의되어 있지 않습니다.

C: 여러 사용자로 실행하면 애플리케이션 서버에 불필요한 부하를 줄 수 있습니다.

## 스크립트
```js
import http from 'k6/http';
import { check } from 'k6';

let usernameArr = ['admin', 'test_user'];
let passwordArr = ['123', '1234'];

export default function() {

    let response = http.get('https://test.k6.io/my_messages.php');
    check(response, {
        'is Unauthorized': r => r.body.includes('Unauthorized'),
    })

    let csrfToken = response.html().find("input[name=csrftoken]").attr("value");

    // Get random username and password from array
    let rand = Math.floor(Math.random() * 2);
    let username = usernameArr[rand];
    let password = passwordArr[rand];
    console.log('username: ' + username, ' / password: ' + password);

    response = http.post('http://test.k6.io/login.php', { login: username, password: password, csrftoken: csrfToken });
    check(response, {
        'is status 200': (r) => r.status === 200,
        'Successful login': r => r.body.includes('successfully authorized'),
    })
}
```

### 정답

1. A. DevTools는 웹 애플리케이션의 요청 및 응답 쌍을 단계별로 살펴보는 훌륭한 방법입니다. B는 오류 외에는 유용한 정보를 제공하지 않을 가능성이 높으며, C는 너무 구체적입니다. 올바르게 처리되지 않으면 오류를 일으킬 수 있는 토큰 외에도 다른 동적 매개변수가 있습니다.
2. C. `http-debug` 플래그를 사용하면 모든 응답 및 요청 정보가 출력되어 유용할 수 있습니다. 정보가 너무 많아 유용하지 않은 경우, DevTools 또는 [프록시](https://k6.io/blog/k6-load-testing-debugging-using-a-web-proxy/)를 사용하는 것을 고려하세요.
3. A.
