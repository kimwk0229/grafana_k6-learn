[첫 번째 k6 스크립트를 작성하는](01-Getting-started-with-k6-OSS.md) 방법은 이미 배웠습니다. 왜 스크립트를 _녹화_하고 싶을까요?

k6 스크립트를 직접 작성하는 것은 k6의 작동 방식을 배우는 가장 좋은 방법이며, 이전 섹션에서 배운 기술은 녹화된 스크립트도 수정해야 하기 때문에 더 나은 테스터가 되는 데 도움이 됩니다. 다음과 같은 상황에서는 k6 스크립트를 직접 작성하는 것이 더 나은 옵션일 수 있습니다:
- 일련의 API 호출을 수행하는 스크립트를 원하는 경우
- 테스트가 클라이언트 측에서 부하를 생성하는 것보다 특정 구성 요소 또는 구성 요소 그룹을 대상으로 하는 경우
- 테스트하려는 API 엔드포인트가 잘 문서화되어 있거나 이미 관련 지식이 있는 경우
- 시뮬레이션하려는 흐름을 브라우저에서 쉽게 시뮬레이션할 수 없는 경우

그러나 스크립트를 녹화하면 시간이 절약될 수 있는 상황도 있습니다:
- 웹 페이지를 테스트하고 있으며 HTML만이 아닌 페이지에 내장된 리소스(스크립트와 이미지)에 대한 요청을 포함하려는 경우
- 애플리케이션에 많은 [동적 상관관계](../III-k6-Intermediate/02-Dynamic-correlation-in-k6.md)가 있으며, 대부분의 작업을 자동으로 처리하고 싶은 경우
- 기반 HTTP 요청이 무엇인지 잘 모르거나 API가 잘 문서화되어 있지 않은 경우

이 섹션에서는 k6 스크립트를 녹화하는 방법을 배웁니다.

## k6 Cloud 계정 만들기

k6 Cloud의 스크립트 녹화 기능을 무료로 사용할 수 있다는 것을 알고 계셨나요?

k6 스크립트를 녹화하려면 k6 Cloud 계정이 필요하지만, 유료 구독이나 신용 카드 정보를 입력할 필요가 없습니다. 시작하려면 [계정에 가입하세요](https://app.k6.io/). 진행하기 전에 로그인되어 있는지 확인하세요.

다음으로, 브라우저용 k6 브라우저 녹화 확장 프로그램을 다운로드하고 설치하세요-- [Chrome](https://chrome.google.com/webstore/detail/k6-browser-recorder/phjdhndljphphehjpgbmpocddnnmdbda?hl=en)과 [Firefox](https://addons.mozilla.org/en-US/firefox/addon/k6-browser-recorder/)가 지원됩니다. 확장 프로그램 바에서 k6 아이콘을 볼 때 녹화를 시작할 준비가 된 것입니다.

![](../../images/browser-extension-icon.png)

그런 다음 새 탭을 열고 k6 브라우저 확장 프로그램 아이콘을 클릭하세요. 몇 가지 초기 옵션이 있는 대화 상자가 표시됩니다:

![k6-browser-recorder-01](../../images/k6-browser-recorder-01.png)
해당 옵션들의 의미는 다음과 같습니다:
- **HAR 파일 다운로드**: HAR은 HTTP ARchive의 약자로, HTTP 요청 및 타이밍과 같은 네트워크 정보를 저장하는 JSON 형식의 파일입니다. **k6 이외의 다른 곳에 녹화를 저장하려면 이 옵션을 활성화하세요**.
- **캐시 지우기 (최근 7일):** 이 옵션을 활성화하면 지난 주의 브라우저 캐시가 삭제됩니다. 이는 애플리케이션의 신규 사용자를 시뮬레이션하려는 경우 유용합니다. **기존 사용자를 시뮬레이션하려면 이 옵션을 비활성화하세요.**
- [캐싱 옵션](../XX-Future-Ideas/Caching-options.md)에 대해 더 자세히 읽어보세요.
- **요청/응답 데이터 상관관계:** k6 레코더는 동적 값이 애플리케이션 서버에 전달되는 시점을 자동으로 감지하고 이를 상관시키려고 시도할 수 있습니다. 이것이 복잡한 애플리케이션에서는 항상 작동하는 것은 아니지만, 스크립팅의 좋은 출발점이 될 수 있습니다. **원시 녹화를 선호하지 않는 한 이 옵션을 활성화하세요**.

다음으로 _녹화 시작_ 버튼을 클릭하세요. 지난 주의 브라우저 캐시가 삭제되는 데 몇 초가 걸릴 수 있습니다. 다음 화면이 표시될 때까지 기다리세요:


![k6-browser-recorder-02](../../images/k6-browser-recorder-02.png)

이제 녹화를 시작할 준비가 되었습니다! 웹 애플리케이션으로 이동하여 사용자 중 한 명이 하는 방식으로 상호작용하세요.

### 녹화 팁

녹화를 최대한 활용하려면 다음 팁을 염두에 두세요:
- [sleep](05-Adding-think-time-using-sleep.md) 시간이 얼마나 되어야 하는지 파악하려면, 신규 사용자가 할 것처럼 양식을 작성하고 텍스트를 읽어보세요. Sleep 타이밍은 확장 프로그램에 의해 기록됩니다.
- 다음 페이지나 작업으로 진행하기 전에 페이지의 모든 리소스가 로드될 때까지 기다리세요. 확실히 하려면 브라우저의 DevTools를 열고 네트워크 탭에 활동이 없을 때까지 기다리세요.
- 수행하는 작업에 대해 별도로 메모를 작성하여 나중에 생성된 스크립트를 이해하는 데 도움이 되도록 하세요.
- 한 번에 하나의 비즈니스 플로우를 녹화하세요. 여러 개의 길거나 복잡한 플로우를 녹화하면 스크립트가 너무 크고 다루기 어려워질 수 있습니다. 여러 스크립트를 결합하는 것이 나중에 분리하는 것보다 쉽습니다.

## 녹화된 스크립트를 k6 Cloud에 저장하기

완료되면 브라우저 레코더에서 _중지_를 클릭하세요. 자동으로 k6 Cloud 계정으로 리디렉션됩니다:

![](../../images/k6-browser-recorder-3.png)

![](../../images/k6-browser-recorder-4.png)

옵션들을 살펴봅시다!

- **프로젝트:** 이 드롭다운 메뉴에서 이 녹화를 저장할 k6 Cloud 프로젝트를 선택할 수 있습니다. 아직 프로젝트를 만들지 않았다면 _기본(Default)_ 프로젝트가 선택된 것을 볼 수 있습니다.
- **제목:** 프로젝트 선택 드롭다운 옆에서 나중에 이 녹화가 무엇인지 기억하는 데 도움이 되는 설명적인 이름을 지정하세요.
- **테스트 빌더 또는 스크립트 편집기:** 녹화된 플로우를 테스트 빌더(GUI)나 스크립트 편집기(일반 JavaScript) 중 어디서 보고 싶은지 결정하세요. 이 옵션들에 대해 나중에 더 자세히 배울 것입니다. 지금은 인터페이스 상호작용이 더 편안하다면 테스트 빌더를 선택하고, 코드를 보고 싶다면 스크립트 편집기를 선택하세요. 확실하지 않다면 테스트 빌더를 선택하면 마음이 바뀌면 나중에 스크립트 편집기로 전환할 수 있습니다.
- **요청 및 응답 데이터 상관관계:** k6가 동적 데이터를 자동으로 감지하고 상관시키도록 하려면 이 옵션을 선택하세요.
- **정적 자산 포함:** 테스트 스크립트에 스크립트, 폰트, 이미지와 같은 내장 리소스에 대한 요청을 포함하려면 이 옵션을 선택하세요. 이 옵션을 선택하면 더 많은 요청이 발생하지만 스크립트가 더 [현실적](../III-k6-Intermediate/03-Workload-modeling.md)이 됩니다.
- **sleep 생성:** 녹화 중 실제 지연을 기반으로 k6가 [think time을 추가](05-Adding-think-time-using-sleep.md)하도록 하려면 이 옵션을 선택하세요.
- **서드파티 도메인 필터링:** 기본적으로 k6는 처음에 탐색한 도메인 외의 도메인으로 전송된 요청을 무시합니다. 이렇게 하면 본인 소유가 아닌 서버를 부하 테스트하는 것을 실수로 방지할 수 있습니다. 나열된 도메인을 살펴보고, 테스트에 포함하려는 도메인을 선택하세요. _참고: 본인 소유가 아닌 서버를 테스트하면 법적 또는 재정적 결과를 초래할 수 있으므로, 본인 소유의 서버만 테스트하는 것을 권장합니다._

그런 다음 저장을 클릭하세요!

테스트 빌더에서 스크립트 편집기로 전환한 경우, 스크립트는 _읽기 전용_ 모드로 표시됩니다:

![](../../images/k6-cloud-script-editor-from-recording.png)

스크린샷에서 주황색으로 강조 표시된 _스크립트 복사_를 클릭하여 전체 스크립트를 선택하고 복사하세요. 그런 다음 스크립트를 IDE에 붙여넣어 [로컬에서 스크립트를 실행](01-Getting-started-with-k6-OSS.md#Hello-World-running-your-k6-script)할 수 있습니다.

대신 이전 녹화 대화 상자에서 _스크립트 편집기_ 옵션을 선택한 경우, k6 Cloud에서 편집 가능한 모드로 테스트 스크립트가 표시됩니다.

![](../../images/k6-cloud-script-editor.png)

여기서 스크립트를 수정하거나 _스크립트 복사_ 버튼을 클릭하여 스크립트의 모든 텍스트를 선택할 수 있습니다. 그런 다음 스크립트를 선택한 IDE에 붙여넣어 계속 작업할 수 있습니다.

<details>
<summary> 녹화된 스크립트 예시 </summary>

```js
// Scenario: Scenario_1 (executor: ramping-vus)

import { sleep, group } from 'k6'
import http from 'k6/http'

export const options = {
  ext: {
    loadimpact: {
      distribution: { 'amazon:us:ashburn': { loadZone: 'amazon:us:ashburn', percent: 100 } },
      apm: [],
    },
  },
  thresholds: {},
  scenarios: {
    Scenario_1: {
      executor: 'ramping-vus',
      gracefulStop: '30s',
      stages: [
        { target: 20, duration: '1m' },
        { target: 20, duration: '3m30s' },
        { target: 0, duration: '1m' },
      ],
      gracefulRampDown: '30s',
      exec: 'scenario_1',
    },
  },
}

export function scenario_1() {
  let response

  group('page_1 - http://ecommerce.k6.io/', function () {
    response = http.get('http://ecommerce.k6.io/', {
      headers: {
        'upgrade-insecure-requests': '1',
        accept:
          'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
        'accept-encoding': 'gzip, deflate',
        'accept-language':
          'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
      },
    })
    sleep(1.2)

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-includes/css/dist/block-library/style.min.css?ver=5.9',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/packages/woocommerce-blocks/build/vendors-style.css?ver=4.0.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/packages/woocommerce-blocks/build/style.css?ver=4.0.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/css/base/gutenberg-blocks.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/style.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/css/base/icons.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/css/woocommerce/woocommerce.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-includes/js/jquery/jquery.min.js?ver=3.6.0',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-includes/js/jquery/jquery-migrate.min.js?ver=3.3.2',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )
    sleep(0.7)

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/assets/js/jquery-blockui/jquery.blockUI.min.js?ver=2.70',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/assets/js/frontend/add-to-cart.min.js?ver=5.0.1',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/assets/js/js-cookie/js.cookie.min.js?ver=2.1.4',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/assets/js/frontend/woocommerce.min.js?ver=5.0.1',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/assets/js/frontend/cart-fragments.min.js?ver=5.0.1',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/js/navigation.min.js?ver=3.5.0',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/js/skip-link-focus-fix.min.js?ver=20130115',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/js/vendor/pep.min.js?ver=0.4.3',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/js/woocommerce/header-cart.min.js?ver=3.5.0',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/js/footer.min.js?ver=3.5.0',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-includes/js/wp-emoji-release.min.js?ver=5.9',
      {
        headers: {
          accept: '*/*',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/K6-logo.png',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/album-1.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/beanie-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/beanie-with-logo-1.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/belt-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/cap-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/hoodie-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/hoodie-with-logo-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/hoodie-with-zipper-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/logo-1.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/long-sleeve-tee-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/polo-2.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/single-1.jpg',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.post(
      'http://ecommerce.k6.io/?wc-ajax=get_refreshed_fragments',
      {
        time: '1645005395795',
      },
      {
        headers: {
          accept: '*/*',
          'x-requested-with': 'XMLHttpRequest',
          'content-type': 'application/x-www-form-urlencoded; charset=UTF-8',
          origin: 'http://ecommerce.k6.io',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )
    sleep(0.6)

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/K6-logo.png',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/uploads/2021/02/K6-logo.png',
      {
        headers: {
          accept: 'image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )
    sleep(6.7)

    response = http.get('http://ecommerce.k6.io/', {
      headers: {
        accept: '*/*',
        'accept-encoding': 'gzip, deflate',
        'accept-language':
          'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
      },
    })

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-includes/css/dist/block-library/style.min.css?ver=5.9',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/packages/woocommerce-blocks/build/vendors-style.css?ver=4.0.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/plugins/woocommerce/packages/woocommerce-blocks/build/style.css?ver=4.0.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/css/base/gutenberg-blocks.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/style.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/css/base/icons.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )

    response = http.get(
      'http://ecommerce.test.k6.io/wordpress/wp-content/themes/storefront/assets/css/woocommerce/woocommerce.css?ver=3.5.0',
      {
        headers: {
          accept: 'text/css,*/*;q=0.1',
          'accept-encoding': 'gzip, deflate',
          'accept-language':
            'en-GB,en-US;q=0.9,en;q=0.8,es;q=0.7,nl;q=0.6,de;q=0.5,eo;q=0.4,fr;q=0.3,fil;q=0.2',
        },
      }
    )
  })
}
```
</details>

### 테스트 빌더

이전 단계에서 테스트 빌더 옵션을 선택했다면, 다음과 같은 화면이 표시됩니다:

![](../../images/k6-browser-recorder-5.png)

여기서 계속 탐색하고, 스크립트를 구축하며, 쉐이크아웃 또는 부하 테스트로 실행하도록 확장할 수 있습니다. [테스트 빌더 사용에 대한 자세한 정보](../XX-Future-Ideas/Creating-a-script-using-the-Test-Builder.md)를 확인할 수 있습니다. 그러나 k6 Cloud에서 테스트를 실행하면 무료 테스트 실행 중 하나가 사용된다는 점을 염두에 두세요. 로컬에서 테스트를 실행하려면 _빌더_에서 _스크립트_로 전환하는 토글을 클릭하세요.

## 스크립트 녹화의 한계

이 섹션에서는 k6 Cloud의 무료 버전을 사용하여 브라우저에서 비즈니스 플로우를 녹화하고 그 녹화에서 k6 스크립트를 생성하는 방법을 배웠습니다. 이 방식으로 스크립트를 생성하면 스크립팅 프로세스를 시작할 때 상당히 적은 준비 시간이 필요할 수 있습니다.

그러나 스크립트 녹화에는 다음과 같은 한계가 있습니다:
- 웹 브라우저에서 수행할 수 있는 작업이 포함된 사용자 플로우만 녹화할 수 있습니다.
- 자동 요청 및 응답 상관관계는 일반적으로 사용되는 동적 값에 대해서만 작동합니다.
- 레코더는 브라우저가 보내고 받는 모든 정보를 저장하므로 스크립트에 노이즈가 증가할 수 있습니다.
- Sleep 시간은 녹화된 것을 기반으로 상수 값으로 하드코딩됩니다.

이러한 이유로, 스크립트 녹화가 k6 스크립트를 시작하는 데 많은 초기 작업을 수행해 줄 것으로 합리적으로 기대할 수 있지만, 원하는 방식으로 작동하도록 스크립트를 수정하고 [디버깅](../III-k6-Intermediate/01-How-to-debug-k6-load-testing-scripts.md)하고 [더 현실적으로 만드는](../XX-Future-Ideas/Best-practices-for-designing-realistic-k6-scripts.md) 것은 여전히 모범 사례입니다.

스크립트를 녹화한 후 고려해야 할 사항들:
- 녹화된 [그룹](https://k6.io/docs/using-k6/tags-and-groups/#groups) 이름 변경 또는 수정
- [스크립트에 check 추가](04-Adding-checks-to-your-script.md)
- [think time을 동적으로 만들기](05-Adding-think-time-using-sleep.md#Dynamic-think-time)
- [테스트 옵션](06-k6-Load-Test-Options.md) 추가 또는 수정
- 코드를 [정리](https://k6.io/docs/using-k6/tags-and-groups/)하고 동료들이 스크립트가 무엇을 하는지 이해할 수 있도록 도움 추가
- 스크립트를 몇 번 실행하고 [디버깅](../III-k6-Intermediate/01-How-to-debug-k6-load-testing-scripts.md)하여 쉐이크아웃하기

## 지식 확인

### 문제 1

다음 상황 중 스크립트를 처음부터 작성하는 것보다 녹화하는 것이 더 적절한 경우는?

A: 여러 API 엔드포인트를 테스트하고 싶은데 프론트엔드가 아직 구축되지 않은 경우.

B: 테스트 중인 페이지에 많은 이미지가 있으며 테스트 스크립트에 해당 이미지를 검색하는 데 걸리는 시간을 포함하려는 경우.

C: HTTP 요청을 모킹 서비스로 수정하려는 경우.

### 문제 2

k6 브라우저 레코더 사용 비용은 얼마인가요?

A: 무료입니다.

B: 첫 달은 무료입니다.

C: k6 Cloud 플랜에 가입해야 합니다.

### 문제 3

브라우저 녹화에 대한 다음 진술 중 참인 것은?

A: 스크립트를 녹화하고 수정 없이 바로 재생할 수 있어야 합니다.

B: 녹화 후 첫 번째 테스트로 100명의 사용자로 확장하고 실행하는 것이 좋습니다.

C: 브라우저 레코더를 사용한다고 해서 스크립트가 작동하도록 테스트 코드를 수정할 필요가 없다는 것을 의미하지 않습니다.

### 정답

1. B. 스크립트 녹화는 단일 작업(링크 클릭 등)이 이미지나 스크립트와 같은 많은 다른 내장 리소스를 다운로드해야 하는 웹 애플리케이션에 특히 유용합니다.
2. A. k6 브라우저 레코더는 무료이며 k6 Cloud 구독이 필요하지 않습니다.
3. C. 스크립트를 녹화한 후, 부하 테스트 내에서 일관되고 반복적으로 실행할 수 있기 전에 동적 데이터를 상관시키거나, 테스트 데이터를 추가하거나, 다른 방식으로 수정해야 할 가능성이 높습니다.
