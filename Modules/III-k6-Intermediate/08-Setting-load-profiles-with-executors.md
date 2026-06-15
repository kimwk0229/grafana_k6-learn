# Executor를 사용하여 부하 프로필 설정하기

테스트의 더 복잡한 측면 중 하나는 원하는 부하 프로필을 모델링하는 것입니다. 이 프로필들은 테스트 중인 시스템으로 들어오는 요청의 양과 속도를 정의합니다. k6는 테스트의 실행 패턴을 형성하기 위한 [executor](https://k6.io/docs/using-k6/scenarios/executors/) 세트를 제공합니다.

> :point_up: 각 executor는 VU 수 및/또는 테스트 사이클당 iteration 수나 심지어 속도를 제어한다는 점을 기억하는 것이 중요합니다.

사용 가능한 각 executor로 작업할 때 세부 사항을 살펴볼 수 있는 예제 연습을 제공했습니다.

### Shared Iterations
_Shared Iterations_는 executor 중 가장 기본적인 것입니다. 이름에서 유추할 수 있듯이, 주요 초점은 테스트의 _iteration_ 수, 즉 테스트 함수가 실행되는 횟수입니다.

| 옵션          | 설명                                                          | 기본값  |
|---------------|---------------------------------------------------------------|---------|
| `iterations`  | 테스트가 몇 번 실행되어야 합니까?                              | `1`     |
| `maxDuration` | 이 시간 내에 완료되지 않으면 테스트를 강제로 중지합니다       | `"10m"` |
| `vus`         | 동시에 실행할 가상 사용자 수                                  | `1`     |

앞서 언급했듯이, 주요 목표는 `maxDuration`을 초과하지 않는 시간 동안 `iterations` 횟수만큼 테스트를 수행하는 것입니다. 테스트 scenario 동안 어느 시점에서도 원하는 총 iteration 수에 도달하지 않는 한 `vus` 횟수의 iteration이 진행되어야 합니다. 이 scenario에서는 일부 VU가 다른 VU보다 더 많은 작업을 수행할 수 있습니다.

[_Shared Iterations_를 직접 실험해 보세요!](./08-Setting-load-profiles-with-executors/Shared-Iterations-Exercises.md)

### Per VU Iterations
_Per VU Iterations_는 _Shared Iterations_ executor의 약간 발전된 형태입니다. 이 executor를 사용하면 여전히 _iteration_ 수에 초점을 맞추지만, 이번에는 **각 _가상 사용자_**가 **동일한 수의** iteration을 실행하도록 합니다.

| 옵션          | 설명                                                          | 기본값  |
|---------------|---------------------------------------------------------------|---------|
| `iterations`  | 각 VU에 대해 테스트가 몇 번 실행되어야 합니까?               | `1`     |
| `maxDuration` | 이 시간 내에 완료되지 않으면 테스트를 강제로 중지합니다       | `"10m"` |
| `vus`         | 동시에 실행할 가상 사용자 수                                  | `1`     |

이 경우, 각 `vus` VU가 `maxDuration`을 초과하지 않는 시간 동안 테스트를 `iterations` 횟수만큼 수행하도록 합니다. 이는 테스트 iteration이 _공정하게_ 분배됨을 의미합니다. 어떤 단일 VU도 다른 VU보다 더 많이 수행하지 않습니다.

[_Per VU Iterations_를 직접 실험해 보세요!](./08-Setting-load-profiles-with-executors/Per-VU-Iterations-Exercises.md)

### Constant VUs
_Constant VUs_는 지정된 시간 동안 테스트를 지속적으로 수행하는 데 초점을 맞춥니다. 이를 통해 각 가상 사용자는 허용된 시간 내에 가능한 한 많은 요청을 수행할 수 있습니다.

| 옵션            | 설명                                      | 기본값        |
|-----------------|-------------------------------------------|--------------|
| **`duration`** | 전체 scenario 기간                         | - (필수)     |
| `vus`          | 동시에 실행할 가상 사용자 수               | `1`          |

위에서 언급했듯이, 주요 목표는 각 `vus` VU가 필요한 `duration` 기간 동안 가능한 한 많은 테스트 iteration을 수행하는 것입니다(예: `"30s"`, `"1h"` 등).

[_Constant VUs_를 직접 실험해 보세요!](./08-Setting-load-profiles-with-executors/Constant-VUs-Exercises.md)

### Ramping VUs
_Ramping VUs_는 **_stages_**를 도입하는 _Constant VUs_ executor의 발전된 형태입니다. 이를 통해 k6는 원하는 VU 수를 한 stage에서 다른 stage로 전환할 수 있습니다. 각 stage는 모든 VU가 테스트를 지속적으로 수행할 자체 시간대를 정의합니다.

| 옵션               | 설명                                                                             | 기본값        |
|--------------------|----------------------------------------------------------------------------------|--------------|
| **`stages`**       | 원하는 VU 수에 대한 시간 `duration`과 `target`으로 구성됩니다                    | - (필수)     |
| `gracefulRampDown` | 램프 다운 시 VU를 종료하기 전에 테스트 iteration이 완료되기를 기다리는 유예 기간  | `"30s"`      |
| `startVUs`         | 테스트 시작 시 가상 사용자 수                                                   | `1`          |

시간 기반 scenario로서, 총 기간은 각 `stage`의 `duration` 시간대의 합과 같습니다. 첫 번째 stage는 `startVUs` VU로 시작하고 구성된 `duration` 동안 stage 내에 지정된 `target` VU 수까지 선형적으로 램프 업(또는 다운)합니다. 다음 stage가 구성되어 있으면, 지정된 `duration` 시간대 동안 해당 시점에서 원하는 `target` VU까지 램프 업하거나 다운합니다. 이 패턴은 나머지 각 stage에 대해 계속됩니다. _Constant VUs_와 마찬가지로, 실행 중인 각 VU는 scenario가 종료될 때까지 테스트 iteration을 지속적으로 수행합니다.

[_Ramping VUs_를 직접 실험해 보세요!](./08-Setting-load-profiles-with-executors/Ramping-VUs-Exercises.md)

### Constant Arrival Rate
_Constant Arrival Rate_를 사용하면 이제 특정 기간 동안 테스트 iteration이 수행되는 속도에 초점을 맞추기 시작합니다. k6는 원하는 속도를 달성하기 위해 VU 수를 동적으로 조정합니다.

| 옵션                  | 설명                                                              | 기본값            |
|-----------------------|-------------------------------------------------------------------|----------------|
| **`duration`**        | 전체 scenario 기간                                                | - (필수)        |
| **`preAllocatedVUs`** | 테스트 시작 시 가상 사용자 수                                      | - (필수)        |
| **`rate`**            | 달성하고 유지할 `timeUnit`당 원하는 iteration                      | - (필수)        |
| `maxVUs`              | 확장할 수 있는 최대 가상 사용자 수                                  | - (확장 없음)   |
| `timeUnit`            | 원하는 `rate`가 적용되는 기간                                      | `"1s"`         |

주요 초점은 원하는 `duration` 동안 `timeUnit`당 `rate`의 _iteration 속도_를 달성하고 유지하는 것입니다. 원하는 속도를 달성하기 위한 VU 수는 k6에 의해 관리되며, `preAllocatedVUs`에서 `maxVUs`까지 어디든지 될 수 있습니다. VU 생성과 관련된 시간 및 리소스 오버헤드가 있으므로, 합리적인 `preAllocatedVUs`를 정의하면 원하는 속도에서 더 많은 테스트 시간을 허용할 것입니다.

[_Constant Arrival Rate_를 직접 실험해 보세요!](./08-Setting-load-profiles-with-executors/Constant-Arrival-Rate-Exercises.md)

### Ramping Arrival Rate
_Ramping Arrival Rate_는 **_stages_**를 도입하는 _Constant Arrival Rate_ executor의 발전된 형태입니다. 이것은 실제 세계 테스트 scenario를 모델링하는 데 아마도 가장 좋은 후보로, k6가 원하는 _iteration 속도_를 한 stage에서 다른 stage로 전환할 수 있습니다. 각 stage는 원하는 속도를 달성하기 위한 자체 시간대를 정의합니다.

| 옵션                  | 설명                                                                           | 기본값            |
|-----------------------|--------------------------------------------------------------------------------|----------------|
| **`preAllocatedVUs`** | 테스트 시작 시 가상 사용자 수                                                   | - (필수)        |
| **`stages`**          | `timeUnit`당 원하는 iteration에 대한 시간 `duration`과 `target`으로 구성됩니다  | - (필수)        |
| `maxVUs`              | 확장할 수 있는 최대 가상 사용자 수                                               | - (확장 없음)   |
| `startRate`           | 달성하고 유지할 `timeUnit`당 원하는 iteration                                   | `0`            |
| `timeUnit`            | 원하는 `rate`가 적용되는 기간                                                   | `"1s"`         |

_Constant Arrival Rate_와 유사하게, 주요 초점은 대상 _iteration 속도_를 달성하는 것입니다. 주요 차이점은 원하는 속도가 정의된 각 `stage` 내에서 달성된다는 것입니다. 전체 기간은 각 `stage`의 `duration` 시간대의 합과 같습니다. 첫 번째 stage는 `preAllocatedVUs` 가상 사용자가 수행하는 `timeUnit`당 `startRate`의 _iteration 속도_로 시작합니다. _iteration 속도_는 stage에 지정된 `target` 속도까지 구성된 `duration` 동안 선형적으로 램프 업(또는 다운)합니다. 다음 stage가 구성되어 있으면, 지정된 `duration` 시간대 동안 해당 시점에서 원하는 `target` 속도까지 램프 업하거나 다운합니다. 이 패턴은 나머지 각 stage에 대해 계속됩니다. 이 stage들로 _스파이크_, _밸리_, _플래토_를 시뮬레이션할 수 있습니다.

[_Ramping Arrival Rate_를 직접 실험해 보세요!](./08-Setting-load-profiles-with-executors/Ramping-Arrival-Rate-Exercises.md)

### Externally Controlled
_Externally Controlled_는 테스트를 시작하고 VU 및 기간에 대한 제한을 설정하는 것 외에는 가상 사용자 수를 변경하지 않는 완전히 다른 executor입니다. 이는 k6 REST API 또는 k6 CLI를 사용하는 외부 프로세스에서 제공될 것으로 예상됩니다.

| 옵션           | 설명                                               | 기본값        |
|----------------|---------------------------------------------------|--------------|
| **`duration`** | 전체 scenario 기간                                  | - (필수)     |
| `maxVUs`       | 활용할 수 있는 최대 가상 사용자 수                   | `0`          |
| `vus`          | 동시에 실행할 가상 사용자 수                         | `0`          |

이 executor의 주요 목표는 테스트의 `duration` 시간대를 정의하는 것입니다. 다른 것이 제공되지 않으면 k6는 본질적으로 명령을 기다리는 상태가 됩니다. 명령이 없으면, 기간에 도달하면 k6는 단순히 종료합니다. `vus`가 0이 아닌 값으로 지정되면, executor는 외부 액터에 의해 작동될 때까지 _Constant VUs_와 유사하게 실행됩니다. `maxVUs` 설정은 스케일 업 요청을 받을 때 적용될 제한을 설정합니다.

[_Externally Controlled_를 직접 실험해 보세요!](./08-Setting-load-profiles-with-executors/Externally-Controlled-Exercises.md)

## 지식 확인

다음 퀴즈로 executor에 대한 이해도를 확인해 보세요. 페이지 하단의 [정답 키](#Answers)로 답을 확인하세요.

### Question 1

시스템은 약 80명의 동시 사용자로 실행되며 50명의 추가 사용자가 가끔 스파이크를 일으킵니다. 이것을 가장 잘 모델링하는 방법은 무엇입니까?

A: `ramping-arrival-rate`를 여러 stage와 함께 사용합니다. 80 VU로 시작하여 짧은 기간 동안 유지하는 stage, 그다음 짧은 기간 내에 130명의 사용자를 목표로 하는 stage, 그다음 80명의 사용자로 돌아오는 stage를 추가합니다.

B: `constant-vus`를 사용하여 130명의 가상 사용자를 설정합니다. 이를 5분 기간으로 실행합니다.

C: `ramping-vus`를 여러 stage와 함께 사용합니다. 80 VU로 시작하여 짧은 기간 동안 유지하는 stage, 그다음 짧은 기간 내에 130명의 사용자를 목표로 하는 stage, 그다음 80명의 사용자로 돌아오는 stage를 추가합니다.

### Question 2

기존 서비스에 대한 SLA를 정의하기 위한 기준 메트릭을 설정하려고 합니다. 이를 달성하는 가장 빠른 방법은 무엇입니까?

A: `shared-iterations` executor를 사용하여 1,000개의 요청을 처리합니다.

B: `constant-arrival-rate`를 사용하여 초당 50개 요청(RPS)을 일정하게 유지하는 데 필요한 가상 사용자 수를 확인합니다.

C: `externally-controlled` executor를 사용하여 k6를 서버 모드로 시작하여 _멋진_ Bash 스크립트로 가상 사용자를 램프 업합니다.

### Question 3

SRE 팀이 인스턴스가 30 RPS를 초과하기 시작하면 가비지 수집 일시 중지가 발생하는 서비스 문제를 발견했습니다. 아직 Kubernetes가 없어 스케일링이 쉬운 옵션이 아닙니다. 개발자들은 json 마샬링 코드를 테스트하기 위해 로컬에서 부하를 어떻게 시뮬레이션할 수 있습니까?

A: `constant-vus`를 사용하여 30명의 가상 사용자가 가능한 한 빠르게 요청을 수행하도록 시뮬레이션합니다.

B: `constant-arrival-rate`를 사용하여 초당 30개 요청(RPS)을 일정하게 유지합니다.

C: `per-vu-iterations` executor를 사용하여 30명의 가상 사용자가 각각 1,000개의 요청을 실행하도록 합니다.

> :rocket: **더 원하신다면** *k6의 Executor*에 대해 심층적으로 이야기한 [k6 Office Hours](https://www.youtube.com/playlist?list=PLJdv3RhAQXNE1TFXn2pp9h_Ul1q_kJrEZ) 세션을 확인해 보세요!

[![k6 Office Hours](../../images/office-hours-executors-k6.png)](https://www.youtube.com/playlist?list=PLJdv3RhAQXNE1TFXn2pp9h_Ul1q_kJrEZ)

#### 정답
1. C. `ramping-vus`는 동시 _가상 사용자_의 스파이크를 모델링할 수 있습니다.
2. A. `shared-iterations`를 사용하면 동시성 유무에 관계없이 고정된 수의 테스트 iteration을 쉽게 실행할 수 있습니다. 기준을 설정하기 위한 간단한 서비스 수준을 원하므로 지나치게 복잡한 것은 필요하지 않을 것입니다.
3. B. `constant-arrival-rate` executor를 사용하면 스크립트가 목표 요청 속도를 달성하고 유지하여 메모리 힙을 모니터링할 수 있습니다.
