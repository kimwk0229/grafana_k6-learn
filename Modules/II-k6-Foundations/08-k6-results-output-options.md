k6는 부하 테스트 결과를 기본적으로 그래프로 표시하는 방법이 없습니다. 하지만 결과를 다양한 형식으로 저장하는 많은 옵션이 있습니다. 이 [블로그](https://k6.io/blog/ways-to-visualize-k6-results/)가 좋은 출발점입니다. 결과 시각화 통합 또는 튜토리얼의 전체 목록은 [여기](https://k6.io/docs/integrations/#result-store-and-visualization)에서 찾을 수 있습니다.

이 섹션에서는 두 가지 일반적인 테스트 결과 형식인 CSV와 JSON에 대해 설명합니다.

참고: CSV와 JSON 형식 모두 자체 시각화 도구가 필요합니다. [Google Sheets](https://sheets.google.com), [Grafana](https://grafana.com), [Tableau](https://tableau.com) 등 어떤 것이든 사용할 수 있습니다.

## 테스트 종료 결과와 시계열 결과의 차이점은?

## CSV

### k6 결과를 CSV로 저장하기

k6 결과를 CSV 파일로 저장하면 선택한 데이터 시각화 소프트웨어에서 추가 분석하기에 좋습니다. CSV는 스프레드시트로 열거나 그래프 및 요약 테이블을 생성하는 데 사용할 수 있습니다.

k6 테스트 결과를 CSV 파일로 출력하려면 테스트를 실행할 때 다음 명령을 사용하세요:

```plain
k6 run test.js -o csv=results.csv
```

`-o` 대신 `--out`을 사용할 수도 있습니다.

### CSV 결과 출력 형식

위의 명령은 k6 결과를 다음 형식의 CSV로 저장합니다:

```csv
metric_name,timestamp,metric_value,check,error,error_code,expected_response,group,method,name,proto,scenario,service,status,subproto,tls_version,url,extra_tags
http_reqs,1641298536,1.000000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_duration,1641298536,114.365000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_blocked,1641298536,589.667000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_connecting,1641298536,117.517000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_tls_handshaking,1641298536,415.043000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_sending,1641298536,0.251000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_waiting,1641298536,114.010000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_receiving,1641298536,0.104000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
http_req_failed,1641298536,0.000000,,,,true,,POST,https://httpbin.test.k6.io/post,HTTP/1.1,default,,200,,tls1.2,https://httpbin.test.k6.io/post,
checks,1641298536,1.000000,Application says hello,,,,,,,,default,,,,,,
vus,1641298536,2.000000,,,,,,,,,,,,,,,
vus_max,1641298536,100.000000,,,,,,,,,,,,,,,
```

파일의 각 줄은 테스트 실행 중 기록된 단일 측정값입니다.

결과 파일은 다음 열을 사용합니다:

- **`metric_name`**: 값이 기록되는 메트릭의 이름입니다. 기본적으로 k6에는 [이러한 내장 메트릭](https://k6.io/docs/using-k6/metrics/#built-in-metrics)이 포함되어 있습니다. 모든 메트릭의 모든 값은 더 쉬운 분석을 위해 동일한 결과 파일에 저장됩니다.
- **`timestamp`**: 각 측정이 이루어진 로컬 날짜 및 시간으로, [Epoch 시간](https://www.epochconverter.com/)으로 표시됩니다.
- **`metric_value`**: 제공된 타임스탬프에서 주어진 메트릭의 측정값입니다. 이 값의 측정 단위는 메트릭에 따라 다릅니다. 예를 들어 `http_req_duration` 값은 밀리초 단위입니다.
- **`check`**: 검증되는 check에 부여된 고유한 이름입니다. 아래 check 예시에서 check 이름은 `Application says hello`입니다:

  ```js
  check(response, {
  'Application says hello': (r) => r.body.includes('Hello world!')
  });
  ```
      
- **`error`**: 네트워크 또는 DNS 오류와 같이 비 HTTP 오류가 발생한 경우 오류 텍스트입니다. 오류가 없으면 이 값은 비어 있습니다.
- **`error_code`**: k6 오류 코드입니다. 오류가 없으면 이 값은 비어 있습니다. 가능한 오류 코드의 [전체 목록](https://k6.io/docs/javascript-api/error-codes)이 있습니다.
- **`expected_response`**: 반환된 응답이 예상한 것인지(기본적으로 400 미만의 HTTP 코드)를 나타내는 불리언(`true` 또는 `false`)입니다.
- **`group`**: 메트릭이 속하는 [요청 그룹](https://k6.io/docs/using-k6/tags-and-groups/#groups)의 이름입니다.
- **`method`**: 사용된 HTTP 메서드의 이름(예: `GET` 또는 `POST`) 또는 gRPC의 RPC 메서드 이름입니다.
- **`name`**: 전송된 요청의 이름입니다. 기본값은 URL이지만, [태그를 사용하여 변경](https://k6.io/docs/using-k6/http-requests#url-grouping)할 수 있습니다.
- **`proto`**: 사용되는 프로토콜의 이름(예: `HTTP/1.1`)입니다.
- **`scenario`**: 측정이 이루어진 테스트 시나리오의 이름입니다. 표준 시나리오는 `default`입니다.
- **`service`**: gRPC의 경우 RPC 서비스 이름입니다.
- **`subproto`**: 웹소켓의 경우 서브프로토콜 이름입니다.
- **`tls_version`**: 연결을 암호화하는 데 사용된 전송 계층 보안(TLS)의 유형입니다.
- **`url`**: 전송된 요청의 URL입니다. 이름이 변경되지 않는 한 `name`과 동일합니다.
- **`extra_tags`**: 이 측정에 적용되는 태그입니다. 스크립트 내에서 [태그](https://k6.io/docs/using-k6/tags-and-groups/)가 사용되지 않으면 비어 있습니다.

## JSON

### k6 결과를 JSON으로 저장하기

k6 결과를 JSON 파일로 출력하려면 다음 명령으로 테스트를 실행하세요:

```plain
k6 run test.js -o json=results.json
```

### JSON 결과 출력 형식

JSON 파일은 다음과 같이 보입니다:

```plain
{"type":"Metric","data":{"name":"http_reqs","type":"counter","contains":"default","tainted":null,"thresholds":[],"submetrics":null,"sub":{"name":"","parent":"","suffix":"","tags":null}},"metric":"http_reqs"}
{"type":"Point","data":{"time":"2022-01-05T12:46:23.603474+01:00","value":1,"tags":{"expected_response":"true","group":"","method":"POST","name":"https://httpbin.test.k6.io/post","proto":"HTTP/1.1","scenario":"default","status":"200","tls_version":"tls1.2","url":"https://httpbin.test.k6.io/post"}},"metric":"http_reqs"}
{"type":"Metric","data":{"name":"http_req_duration","type":"trend","contains":"time","tainted":null,"thresholds":["p(95)<=100"],"submetrics":null,"sub":{"name":"","parent":"","suffix":"","tags":null}},"metric":"http_req_duration"}
{"type":"Point","data":{"time":"2022-01-05T12:46:23.603474+01:00","value":118.96,"tags":{"expected_response":"true","group":"","method":"POST","name":"https://httpbin.test.k6.io/post","proto":"HTTP/1.1","scenario":"default","status":"200","tls_version":"tls1.2","url":"https://httpbin.test.k6.io/post"}},"metric":"http_req_duration"}
{"type":"Metric","data":{"name":"http_req_blocked","type":"trend","contains":"time","tainted":null,"thresholds":[],"submetrics":null,"sub":{"name":"","parent":"","suffix":"","tags":null}},"metric":"http_req_blocked"}
```

각 줄은 메트릭 또는 포인트입니다. **메트릭(metric)**은 [내장 메트릭](https://k6.io/docs/using-k6/metrics/#built-in-metrics) 또는 커스텀 메트릭을 유형, 관련 thresholds, 또는 어떤 thresholds가 실패했는지의 관점에서 정의합니다. **포인트(point)**는 메트릭에 대한 측정값으로, 특정 타임스탬프에서의 메트릭 실제 값을 포함합니다.

## 지식 확인

### 문제 1

기본 k6 CSV 결과 형식에 대한 다음 진술 중 참인 것은?

A: 응답 시간을 그래프로 그리려면 CSV 파일의 모든 줄에서 `metric_value` 열의 숫자를 가져와야 합니다.

B: 각 줄에는 특정 타임스탬프에서의 메트릭과 해당 메트릭의 값이 포함됩니다.

C: CSV 파일의 `metric_name` 열은 전송된 HTTP 요청의 URL을 가리킵니다.

### 문제 2

다음은 k6 테스트 결과가 포함된 JSON의 한 줄입니다:

```plain
{"type":"Point","data":{"time":"2022-01-05T12:46:26.893633+01:00","value":100,"tags":null},"metric":"vus_max"}
```

이 줄에서 무엇을 알 수 있나요?

A: 그 시점의 최대 VU 수는 100이었습니다.

B: 이 측정값은 `http_req_duration` 메트릭에 대한 것입니다.

C: 테스트는 2022년 1월 5일 12:46에 최대 VU 수에 도달했습니다.

### 문제 3

다음 중 k6 결과를 다른 형식으로 출력하는 올바른 명령은?

A: `k6 run test.js --out json=myresults.json`

B: `k6 run test -o csv=results.csv`

C: `k6 output csv=results.csv`

## k6 Cloud

이전 두 가지 옵션은 k6 결과를 다른 형식으로 출력할 수 있게 해주지만, 별도의 결과 시각화 소프트웨어에서 분석을 수행해야 합니다.

이에 대한 대안은 k6 Cloud를 사용하는 것입니다. k6 Cloud는 k6 OSS를 기반으로 구축된 유료 SaaS 플랫폼입니다. k6 Cloud를 사용하지 않고도 k6를 사용할 수 있지만, k6 Cloud는 꽤 유용한 추가 기능을 제공하며, 그 중 하나가 결과 시각화입니다.

다음 섹션에서는 k6 Cloud, k6 OSS와 함께 사용하는 방법, 그리고 결과 분석에 어떻게 도움이 될 수 있는지에 대해 설명합니다.

### 정답

1. B. A는 CSV 출력에 `http_req_duration`만이 아닌 여러 메트릭이 포함되어 있기 때문에 틀립니다. C는 `url`이 요청의 URL을 포함하는 열이기 때문에 틀립니다.
2. A. 메트릭은 그 시점에 실행 중인 가상 사용자 수(이 경우 100)인 `vus_max`입니다. 그러나 이것이 반드시 테스트의 최대 VU 수에 해당하는 것은 아닙니다. 테스트가 이 시점을 넘어 더 많이 램프업될 수 있기 때문입니다.
3. A. A만이 결과를 출력하기 위한 올바른 구문을 가지고 있습니다.
