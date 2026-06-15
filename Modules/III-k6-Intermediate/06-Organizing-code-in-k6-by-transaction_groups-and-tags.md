# 트랜잭션 그룹과 태그를 사용하여 k6 코드 구성하기

이전에 부하 테스트 스크립트를 작성해 본 적이 있다면, 트랜잭션(transaction)이라는 용어에 익숙할 것입니다. 부하 테스트에서 트랜잭션은 사용자의 단일 동작에 해당하는 요청이나 요청 그룹입니다.

예를 들어, 사용자가 처음으로 웹사이트 메인 페이지를 방문하는 것으로 구성된 `01_Go_to_homepage`라는 트랜잭션이 있을 수 있습니다. 그 트랜잭션에는 메인 HTML에 대한 GET 요청뿐만 아니라 스크립트, 이미지, 폰트와 같은 포함된 리소스도 포함될 것입니다.

트랜잭션은 성능을 사용자 경험 관점에서 재구성하는 데 유용할 수 있습니다. 위의 예에서 애플리케이션의 최종 사용자는 `https://yourdomain.com/script.js`의 응답 시간이 300ms라는 것을 반드시 신경 쓰거나 알아차리지 못할 수도 있습니다. 웹사이트에 대한 사용자 의견은 보통 페이지 단위로 표현되므로, 스크립트가 유사하게 구조화되어야 하는지 고려할 가치가 있습니다. 이해 관계자와 이야기할 때 개별 요청보다는 페이지의 응답 시간을 보고하는 것이 더 쉬울 수도 있습니다.

트랜잭션은 또한 테스트 코드를 정리하여 더 읽기 쉽고 디버깅하기 쉽게 만듭니다. k6에서 트랜잭션을 만드는 여러 가지 방법이 있습니다.

## 그룹(Groups)

[그룹](https://k6.io/docs/using-k6/tags-and-groups/#groups)을 사용하면 여러 요청이 동일한 트랜잭션에 속한다는 것을 식별할 수 있습니다. k6는 그룹화된 요청의 응답 시간을 개별적으로 및 그룹으로 표시합니다. 그룹은 동일한 동작에 의해 트리거된 요청에 가장 적합합니다. 그룹을 사용하여 스크립트를 구조화하는 방법의 예입니다:

```js
import { group } from 'k6';

export default function () {
  group('01_VisitHomepage', function () {
    // ...
  });
  group('02_ClickOnProduct', function () {
    // ...
  });
  group('03_AddProductToCart', function () {
    // ...
  });
  group('04_ViewCart', function () {
    // ...
  });
  group('05_ProceedToCheckout', function () {
    // ...
  });
}

```

[k6 브라우저 레코더를 사용하여 스크립트를 만들](../II-k6-Foundations/09-Recording-a-k6-script.md) 때, k6가 페이지에서 여러 포함된 리소스를 감지할 때마다 이 그룹이 자동으로 생성되는 것을 알 수 있습니다.

그룹을 사용하면 k6는 여전히 각 요청에 대한 메트릭을 개별적으로 보고하고 저장하지만, 각각은 그룹 이름도 포함합니다. `k6 run test.js -o csv=results.csv`를 사용하여 [출력 결과를 CSV로 저장](../II-k6-Foundations/08-k6-results-output-options.md#CSV)하면, 각 요청의 그룹도 보고됩니다:

```plain
metric_name,timestamp,metric_value,check,error,error_code,expected_response,group,method,name,proto,scenario,service,status,subproto,tls_version,url,extra_tags
vus,1645006049,1.000000,,,,,,,,,,,,,,,
vus_max,1645006049,1.000000,,,,,,,,,,,,,,,
http_reqs,1645006050,1.000000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_duration,1645006050,1530.111000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_blocked,1645006050,134.352000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_connecting,1645006050,132.192000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_tls_handshaking,1645006050,0.000000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_sending,1645006050,0.141000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_waiting,1645006050,1120.729000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_receiving,1645006050,409.241000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
http_req_failed,1645006050,0.000000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,
group_duration,1645006069,20792.450912,,,,,::01_Homepage,,,,default,,,,,,
```

위 CSV 파일의 `::01_Homepage`는 요청의 그룹 이름을 참조하며, `::`은 그것이 요청이 아닌 그룹임을 나타냅니다.

마지막 줄은 k6가 `group_duration`이라는 전체 그룹에 대한 집계된 `http_req_duration` 메트릭을 저장했다는 것도 보여줍니다. 지정된 모든 그룹은 자체 `group_duration`을 갖게 됩니다.

그룹은 메트릭을 대체하는 대신 *추가*하기 때문에, 코드를 트랜잭션으로 구성하는 훌륭한 방법이 될 수 있습니다.

## 태그(Tags)

[태그](https://k6.io/docs/using-k6/tags-and-groups/#tags)는 k6에서 요소에 레이블을 붙이는 방법이며, 그룹과 함께 사용할 수 있습니다. 그룹과 달리, 태그는 k6가 태그별로 메트릭을 보고하게 하지 않습니다.

요청에 태그를 붙이는 방법은 다음과 같습니다:

```js
import { sleep, group } from 'k6'
import http from 'k6/http'

export default function () {    
  let response

  group('01_Homepage', function () {
    response = http.get('http://ecommerce.k6.io/', {
      tags: {
        page: 'Homepage',
        type: 'HTML',
      }
    })

	// ... (other requests in the group)

	// ... (other requests in the group)

  })
}

```

요청에 추가된 `tags` 섹션은 두 가지 다른 태그를 정의합니다:
- 이 요청에 대한 값이 `Homepage`인 `page`라는 태그, 그리고
- 이 요청에 대한 값이 `HTML`인 `type`이라는 태그

태그의 이름과 값 모두 적합하다고 느끼는 대로 변경할 수 있으며, 여러 태그를 사용할 수 있습니다.

기본적으로 k6는 모든 요청에 값이 URL과 같은 `name` 태그를 추가합니다. 요청에 더 읽기 쉬운 이름을 부여하고 싶다면 이 동작을 재정의할 수 있습니다.

k6 CSV 결과에서 태그의 모습은 다음과 같습니다:

```plain
metric_name,timestamp,metric_value,check,error,error_code,expected_response,group,method,name,proto,scenario,service,status,subproto,tls_version,url,extra_tags
vus,1645007331,1.000000,,,,,,,,,,,,,,,
vus_max,1645007331,1.000000,,,,,,,,,,,,,,,
http_reqs,1645007331,1.000000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,page=Homepage&type=HTML
http_req_duration,1645007331,1405.515000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,page=Homepage&type=HTML
http_req_blocked,1645007331,139.488000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,page=Homepage&type=HTML
http_req_connecting,1645007331,137.223000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,page=Homepage&type=HTML
http_req_tls_handshaking,1645007331,0.000000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,type=HTML&page=Homepage
http_req_sending,1645007331,0.137000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,type=HTML&page=Homepage
http_req_waiting,1645007331,1171.681000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,page=Homepage&type=HTML
http_req_receiving,1645007331,233.697000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,page=Homepage&type=HTML
http_req_failed,1645007331,0.000000,,,,true,::01_Homepage,GET,http://ecommerce.k6.io/,HTTP/1.1,default,,200,,,http://ecommerce.k6.io/,page=Homepage&type=HTML
```

이 요청의 모든 메트릭의 마지막 필드에는 이제 설정된 태그에 해당하는 `page=Homepage&type=HTML`이 있습니다.

k6가 그룹화된 요청에 대해 하는 것처럼 태그가 붙은 모든 요청에 대한 집계 메트릭을 보고하지 않기 때문에, 그룹이 트랜잭션으로 사용하기에 더 적합합니다. 그러나 태그는 다음과 같은 상황에서 유용할 수 있습니다:
- 더 쉬운 문제 해결을 위해 요청에 메타데이터를 추가하려는 경우(예: 해당 요청에 사용되는 scenario 유형이나 테스트 데이터 추가).
- 여러 페이지에 걸친 패턴을 탐색하려는 경우, 예를 들어 모든 이미지의 응답 시간이 총 시간에 얼마나 기여하는지 확인하고 싶은 경우.
- 요청보다 더 많은 것에 레이블을 붙이고 싶은 경우.

태그를 사용하면 다음에 레이블을 붙일 수 있습니다:
- 요청
- check
- threshold
- 사용자 지정 메트릭
- 테스트 내의 모든 메트릭

태그에 대한 자세한 정보는 [여기를 클릭하세요](https://k6.io/docs/using-k6/tags-and-groups/#tags).

## URL 그룹핑

앞서 언급했듯이, k6는 기본적으로 모든 요청에 `name` 태그를 추가하고 값을 요청 URL로 설정합니다. 이는 각 URL이 다른 페이지를 나타낼 때 유용합니다. 그러나 때로는 테스트 scenario에 하나로 분석되기에 충분히 유사한 페이지가 포함되어야 한다고 결정할 수 있습니다. 예를 들어, 이커머스 사이트를 테스트하는 경우, 서로 다른 제품으로 나누어 보고하는 것보다 모든 제품 페이지를 하나로 보고하는 것이 더 유용할 수 있습니다.

[URL 그룹핑](https://k6.io/docs/using-k6/http-requests/#url-grouping)은 다음과 같이 동적 URL을 생성할 때 특히 유용한 태그 구현입니다:

```js
import http from 'k6/http'

export default function () {
    let product = ['album', 'beanie', 'beanie-with-logo'];
    let rand = Math.floor(Math.random() * product.length);
    let productSelected = product[rand];
    let response = http.get('http://ecommerce.test.k6.io/product/' + productSelected, {
        tags: { name: 'ProductPage'},
    });
}
```

위 예에서 스크립트는 다음 세 제품 페이지 중 하나를 무작위로 방문합니다:
- `http://ecommerce.test.k6.io/product/album`
- `http://ecommerce.test.k6.io/product/beanie`,
- `http://ecommerce.test.k6.io/product/beanie-with-logo` 

URL 그룹핑 없이는 각각이 다른 이름(URL과 동일)으로 태그가 붙여질 것입니다. URL 그룹핑 _사용_ 시, 이 페이지들은 동일한 이름인 `ProductPage`로 태그가 붙여집니다. 다음은 스크립트를 실행하여 얻은 CSV 결과의 한 줄입니다:

```plain
http_req_duration,1645010817,1203.076000,,,,true,,GET,ProductPage,HTTP/1.1,default,,301,,,http://ecommerce.test.k6.io/product/beanie,
```

## 스크립트 가독성 향상

그룹과 태그(URL 그룹핑 포함)를 사용하는 것은 결과를 정리하는 데 좋지만, 스크립트의 복잡성을 더할 뿐입니다. 스크립트 가독성을 향상시키기 위해 다음과 같은 것들을 할 수 있습니다:
- 주석 사용
- 함수 사용
- 스크립트 모듈화

### 주석

스크립트의 모든 동작에 대해 주석을 남기면 코드를 시각적으로 구분하고 코드가 어떤 동작에 해당하는지 설명하는 이중 이점이 있습니다. 예시는 다음과 같습니다:

```js
import http from 'k6/http'

export default function () {
	/************
       STEP 1: Visit homepage.
    ************/

	// Code for Step 1.

    /************
       STEP 2: Click on product to go to product page.
    ************/

	// Code for Step 2.

    /************
       STEP 3: Click Add To Cart button on product page.
    ************/

	// Code for Step 3.
}
```

### 함수

스크립트를 읽기 쉽게 만드는 또 다른 방법은 모든 트랜잭션에 대해 함수를 만든 다음, 다음과 같이 default 함수에서 각각을 호출하는 것입니다:

```js
import http from 'k6/http'

export default function () {

    Homepage();
    ProductPage();
    CartPage();
    CheckoutPage();

}

export function Homepage() {
    // ...
}

export function ProductPage() {
    // ...
}

export function CartPage() {
    // ...
}

export function CheckoutPage() {
    // ...
}
```

이런 방식으로 함수를 사용하면 동일한 스크립트 내에서 여러 사용자 흐름을 정의할 수 있는 추가 이점이 있습니다. 예를 들어, 홈페이지만 방문하는 사용자 흐름, 홈페이지와 몇 가지 제품 페이지를 방문하는 사용자 흐름, 그리고 모든 페이지를 거쳐 결제하는 세 번째 사용자 흐름을 정의할 수 있습니다. [scenario를 사용한 워크로드 모델링](09-Workload-modeling-with-scenarios.md)에서 scenario에 대해 더 배울 것입니다.

### 스크립트 더 모듈화하기

그룹, 태그(URL 그룹핑 포함), 주석, 함수 모두 사용했는데도 스크립트가 *여전히* 너무 길고 읽기 어렵다면 어떻게 해야 할까요? [모듈식 스크립팅](../XX-Future-Ideas/Modular-scripting.md)이 문제의 해결책이 될 수 있습니다.

모듈식 스크립팅은 k6 테스트 스크립트를 여러 스크립트로 분리한 다음, 각각을 모두 조율하는 단일 "실행" 스크립트로 가져오는 것을 포함합니다. k6 테스트를 실행하기 위한 모듈식 프레임워크 구축에 대한 자세한 내용은 [모듈식 스크립팅](../XX-Future-Ideas/Modular-scripting.md)에서 배울 것입니다.

## 지식 확인

### Question 1

부하 테스트 트랜잭션으로 사용하기에 가장 좋은 기법은 무엇입니까?

A: 그룹(Groups)

B: 태그(Tags)

C: URL 그룹핑(URL grouping)

### Question 2

다음 코드를 살펴보세요:

```js
import { sleep, group } from 'k6'
import http from 'k6/http'

export default function () {    
  let response

  group('01_Homepage', function () {
    response = http.get('http://ecommerce.k6.io/', {
      tags: {
        page: 'Homepage',
        type: 'HTML',
      }
    })

	// ... (other requests in the group)

	// ... (other requests in the group)

  })
}

```

HTTP 요청의 `name` 태그 값은 무엇입니까?

A: `Homepage`

B: `HTML`

C: `http://ecommerce.k6.io/`

### Question 3

다음 중 k6 테스트 스크립트를 구성하기 위해 함수를 사용하는 이점은 무엇입니까?

A: 함수는 함수 내의 모든 메트릭을 집계하고 단일 이름으로 보고합니다.

B: 함수는 애플리케이션의 사용자 여정에 해당하는 테스트 scenario를 만들 자유를 줍니다.

C: 함수는 모든 요청에 함수 이름으로 태그를 붙여 테스트 결과에서 함수별로 필터링할 수 있습니다.

### 정답

1. A. 요청 그룹화는 각 요청 *과* 각 그룹에 대한 메트릭을 저장합니다(태그와 URL 그룹과 달리).
2. C. 특정 `name`이 지정되지 않으면, k6는 기본적으로 URL을 이름으로 사용하여 요청을 더 쉽게 식별할 수 있도록 합니다.
3. B. 함수는 메트릭을 집계하지 않으며(그룹이 그것에 더 적합), 내부의 요청에 태그를 붙이지도 않습니다(그러기 위해서는 태그를 추가해야 합니다). 대신, 스크립트 내에서 사용자 동작이나 사용자 흐름을 구성하는 데 더 적합합니다.
