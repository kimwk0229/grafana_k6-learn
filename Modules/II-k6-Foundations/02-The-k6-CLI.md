# k6 CLI

k6 스크립트가 있으면 k6 명령줄 인터페이스(CLI)를 사용하여 터미널에서 스크립트와 상호작용할 수 있습니다. k6 CLI는 서브커맨드와 플래그를 통해 k6 테스트 스크립트를 실행하고 실행 설정을 구성할 수 있습니다.

가장 일반적인 세 가지 명령은 다음과 같습니다:

| 명령       | 설명                           | 사용법           |
| --------- | ------------------------------ | ---------------- |
| `help`    | 가능한 모든 명령을 표시합니다   | `k6 help`        |
| `run`     | k6 스크립트를 실행합니다        | `k6 run test.js` |
| `version` | 설치된 k6 버전을 표시합니다     | `k6 version`     |

_플래그(Flags)_는 구성의 일부를 변경하기 위해 명령에 추가하는 설정입니다. 다음은 `run` 명령에 대한 일반적인 플래그의 개요입니다:


| 플래그                  | 설명                                                         | 사용법                                   |
| ---------------------- | ------------------------------------------------------------ | ---------------------------------------- |
| `--help`               | 주어진 명령에 대한 가능한 플래그를 표시합니다                 | `k6 run --help`                          |
| `--vus` 또는 `-u`      | 가상 사용자 수를 설정합니다                                  | `k6 run test.js --vus 10 --duration 30s` |
| `--duration`           | 테스트 지속 시간을 설정합니다                                 | `k6 run test.js --duration 10m`          |
| `--iterations` 또는 `-i` | k6가 default 함수를 지정한 횟수만큼 반복하도록 지시합니다  | `k6 run test.js -i 3`                    |
| `-e`                   | 스크립트에 전달할 환경 변수를 설정합니다                     | `k6 run test.js -e DOMAIN=test.k6.io`   |

이 섹션에서는 이러한 일반적인 명령과 플래그를 살펴봅니다.

## 도움말(Help)

`k6 help`를 실행하면 사용 가능한 모든 명령 목록이 표시됩니다.

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

특정 명령에 대한 정보를 얻으려면 `--help` 플래그를 추가하세요:

```shell
k6 run --help
```


## 실행 및 실행 옵션: `run`

또 다른 일반적인 k6 명령은 `k6 run [filename].js`입니다. 이전 섹션에서 `k6 run`을 사용하여 기존 k6 스크립트를 JavaScript로 실행하는 방법을 배웠습니다.

수정 없이 `k6 run`은 스크립트를 그대로 실행하도록 k6에 지시합니다(k6가 사용할 구성 설정을 결정하는 방법을 이해하려면 [k6에서 설정 변경](02-The-k6-CLI.md#Changing-settings-in-k6)을 참조하세요). 그러나 `run` 명령과 함께 플래그를 사용하여 명령줄에서 스크립트 내의 설정(예: [k6 부하 테스트 옵션](06-k6-Load-Test-Options.md))을 재정의할 수도 있습니다.

### duration 플래그

duration은 테스트가 실행되는 시간을 지정합니다. `--duration` 플래그로 명령줄에서 이를 설정할 수 있습니다:

```shell
k6 run test.js --duration 30s
```

`s`, `h`, `m`을 사용하여 duration을 정의할 수 있습니다. 다음은 이 플래그에 유효하고 동등한 인수들입니다:
- 1h30m10s
- 5410s
- 90m10s

### iterations 플래그

`--iterations` 또는 `-i` 플래그로 반복 횟수를 설정할 수 있습니다:

```shell
k6 run test.js --iterations 100
k6 run test.js -i 100
```

위의 두 줄 중 어느 것을 사용해도 k6는 스크립트를 100번 반복 실행합니다.

### 가상 사용자 플래그

테스트 실행 시 `-u` 또는 `--vus` 플래그로 가상 사용자 수를 조정할 수 있습니다:

```shell
k6 run test.js --vus 10 --duration 1m
k6 run test.js -u 10 --iterations 100
```

위의 두 줄은 동일하며, 둘 다 k6가 10명의 가상 사용자로 `test.js` 파일을 실행하도록 지시합니다. 각각 테스트 duration과 반복 횟수도 설정합니다.

### 환경 변수

지금까지 명령줄에서 실행 옵션을 설정하는 방법, 즉 가상 사용자, 테스트 duration, 반복 횟수, 테스트 내 단계와 같은 테스트 파라미터를 변경하는 방법을 배웠습니다. 명령줄에서 _다른_ 변수를 설정하려면 어떻게 해야 할까요?

그런 경우 환경 변수를 사용할 수 있습니다. 환경 변수는 k6 스크립트 외부에서 값을 설정할 수 있는 변수입니다.

예를 들어, 명령줄을 사용하여 환경 변수를 설정하여 테스트 스크립트가 사용하는 도메인을 변경할 수 있습니다. 이는 스테이징과 테스트 같은 여러 환경을 정기적으로 테스트할 때 유용합니다.

환경 변수를 사용하려면 스크립트에서 변수를 정의하세요:

```js
import http from 'k6/http';

const hostname = `http://${__ENV.DOMAIN}`;

export default function () {
    let res = http.get(hostname + '/my_messages.php');
}
```

위 스크립트에서 `${__ENV.DOMAIN}`은 환경 변수이지만, 스크립트 어디에도 정의되어 있지 않습니다. 런타임에 정의하는 방법은 다음과 같습니다:

```shell
k6 run test.js -e DOMAIN=test.k6.io
```

테스트가 실행되면 `http://test.k6.io/my_messages.php`로 HTTP GET 요청을 보냅니다. 환경 변수를 사용하면 스크립트 자체를 변경하지 않고도 테스트가 타겟으로 하는 도메인을 변경할 수 있습니다.

이름에도 불구하고, 이 변수들은 환경에 관한 정보뿐만 아니라 많은 유형의 정보를 보유할 수 있습니다. 환경 변수를 사용할 수 있는 다른 예시들:
- think time
- 이 실행을 위한 요청에 첨부할 태그
- 사용할 테스트 데이터 파일
- 제외하거나 포함할 페이지
- 시나리오

## k6에서 설정 변경

이 섹션에서는 명령줄 플래그와 환경 변수를 사용하여 스크립트 실행을 변경하는 방법을 배웠습니다. 또한 이전에 스크립트 내에서 이러한 옵션 중 일부를 설정하는 방법도 배웠습니다. 전체 옵션 목록은 [공식 문서](https://k6.io/docs/using-k6/k6-options/reference/)를 확인하세요.

k6 설정을 변경하는 이 두 가지 방법 사이에 충돌이 있으면 어떻게 될까요?

k6는 항상 다음 순서로 [설정에 우선순위를 부여](https://k6.io/docs/using-k6/k6-options/how-to/#order-of-precedence)합니다:
1. 명령줄 플래그
1. 환경 변수
1. Export된 k6 스크립트 옵션
1. 구성 파일
1. 기본값

명령줄 플래그가 가장 높은 우선순위를 가지며 항상 다른 모든 것을 재정의합니다. 이에 따라 스크립트 실행을 계획하세요.

## 지식 확인

### 문제 1

다음 중 k6 스크립트를 실행하는 명령은?

A: `k6 --vus 1 test.js`

B: `k6 run test.js`

C: `run test.js -vus 1`

### 문제 2

다음 중 실행 시 경고를 발생시키는 명령은?

A: `k6 run test.js --duration 10m`

B: `k6 run test.js -i 3`

C: `k6 run test.js --users 3`

### 문제 3

환경 변수 사용에 대한 다음 진술 중 참인 것은?

A: 환경 변수를 사용하려면 스크립트를 업데이트 *하고* 명령줄에서 환경 변수를 설정해야 합니다.

B: 환경 변수를 사용하려면 변수 값을 정의하기 위해 스크립트만 업데이트하면 됩니다.

C: 환경 변수를 사용하려면 명령줄 플래그만 사용하면 되며, 값은 자동으로 스크립트에 전달됩니다.

### 정답

1. B. 첫 번째 옵션은 `run` 키워드가 없고, 세 번째는 `k6`가 없습니다. B만이 실제로 스크립트를 실행합니다.
2. C. `--users`는 유효한 옵션이 아니며 CLI에서 `invalid argument` 오류를 발생시킵니다.
3. A. 환경 변수의 값은 스크립트가 명령줄에서 전달되는 값을 받아들이도록 업데이트되지 않으면 k6 스크립트에서 사용되지 않습니다.
