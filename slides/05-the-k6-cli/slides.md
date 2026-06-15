# k6 CLI

---

## 명령어

가장 일반적인 세 가지 명령어:

| 명령어    | 설명                    | 사용법            |
| --------- | ------------------------------ | ---------------- |
| `help`    | 가능한 모든 명령어 표시 | `k6 help`        |
| `run`     | k6 스크립트 실행           | `k6 run test.js` |
| `version` | 설치된 k6 버전 표시  | `k6 version`     |

---

## 플래그

| 플래그                   | 설명                                                    | 사용법                                    |
| ---------------------- | -------------------------------------------------------------- | ---------------------------------------- |
| `--help`               | 주어진 명령어에 대한 가능한 플래그 표시                  | `k6 run --help`                          |
| `--vus` 또는 `-u`        | virtual user 수 설정                                   | `k6 run test.js --vus 10 --duration 30s` |
| `--duration`           | 테스트 duration 설정                                  | `k6 run test.js --duration 10m`          |

---

| 플래그                   | 설명                                                    | 사용법                                    |
| ---------------------- | -------------------------------------------------------------- | ---------------------------------------- |
| `--iterations` 또는 `-i` | k6가 기본 함수를 지정 횟수만큼 반복하도록 지시 | `k6 run test.js -i 3`                    |
| `-e`                   | 스크립트에 전달할 환경 변수 설정             | `k6 run test.js -e DOMAIN=test.k6.io`                           

---

## 도움말 가져오기: `help`

`k6 help` 실행 시 다음이 반환됩니다:

```shell
          /\      |‾‾| /‾‾/   /‾‾/
     /\  /  \     |  |/  /   /  /
    /  \/    \    |     (   /   ‾‾\
   /          \   |  |\  \ |  (‾)  |
  / __________ \  |__| \__\ \_____/ .io

Usage:
  k6 [command]

Available Commands:
  archive     Create an archive
  cloud       Run a test on the cloud
  convert     Convert a HAR file to a k6 script
  help        Help about any command
  inspect     Inspect a script or archive
  login       Authenticate with a service
  pause       Pause a running test
  resume      Resume a paused test
  run         Start a load test
  scale       Scale a running test
  stats       Show test metrics
  status      Show test status
  version     Show application version

Flags:
  -a, --address string      address for the api server (default "localhost:6565")
  -c, --config string       JSON config file (default "/Users/nic/Library/Application Support/loadimpact/k6/config.json")
  -h, --help                help for k6
      --log-output string   change the output for k6 logs, possible values are stderr,stdout,none,loki[=host:port] (default "stderr")
      --log-format string   log output format
      --no-color            disable colored output
  -q, --quiet               disable progress updates
  -v, --verbose             enable verbose logging

Use "k6 [command] --help" for more information about a command.
```

---

## 실행 및 실행 옵션: `run`

또 다른 일반적인 k6 명령어는 `k6 run [filename].js`입니다.

---

### duration 플래그

duration은 테스트가 실행되는 시간을 지정합니다. `--duration` 플래그로 명령줄에서 설정할 수 있습니다:

```shell
k6 run test.js --duration 30s
```

`s`, `h`, `m`을 사용하여 duration을 정의할 수 있습니다.

---

### iterations 플래그

`--iterations` 또는 `-i` 플래그로 iteration 횟수를 설정할 수 있습니다:

```shell
k6 run test.js --iterations 100
k6 run test.js -i 100
```

---

### virtual user 플래그

테스트 실행 시 `-u` 또는 `--vus` 플래그로 virtual user 수를 조정할 수 있습니다:

```shell
k6 run test.js --vus 10 --duration 1m
k6 run test.js -u 10 --iterations 100
```

---

### 환경 변수

환경 변수를 사용하려면 스크립트에서 변수를 정의합니다:

```js [3|5-7]
import http from 'k6/http';

const hostname = `http://${__ENV.DOMAIN}`;

export default function () {
    let res = http.get(hostname + '/my_messages.php');
}
```

---

런타임에 정의하는 방법:

```shell
k6 run test.js -e DOMAIN=test.k6.io
```

---

## k6에서 설정 변경하기

k6는 항상 이 순서로 [설정의 우선순위를 정합니다](https://k6.io/docs/using-k6/k6-options/how-to/#order-of-precedence):

![k6에서 설정 및 설정의 우선순위](../../images/k6-order-of-preference-settings.png)
<!-- .element class="stretch" -->

---

## k6 결과 이해하기

- 이동: [06-understanding-k6-results](?p=06-understanding-k6-results)
