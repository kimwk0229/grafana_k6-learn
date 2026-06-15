<style>
  .container {
    display: flex;
  }

  .col {
    flex: 1;
  }
</style>

## 부하 테스팅

**성능 테스팅 != 부하 테스팅**

---

## 일반적인 테스트 매개변수

_테스트 매개변수_는 부하의 분포, 형태, 패턴을 포함합니다.

- **Virtual users (VUs)**  
- **Iterations**
- **처리량(Throughput)**
- **사용자 흐름(User flows)**
- **부하 프로필(Load profile)**
- **지속 시간(Duration)**

---

## 부하를 시뮬레이션하는 방법

1. **프로토콜 기반 부하 테스팅**
2. **브라우저 기반 부하 테스팅**
3. **하이브리드 부하 테스팅**

---

## 부하 테스트 유형

### 셰이크아웃 테스트(Shakeout test)

![](../../images/scenarios-shakeout.png) 
<!-- .element class="stretch" -->

---

### 평균 부하 테스트(Average load test)

![](../../images/test-scenario-average.png)
<!-- .element class="stretch" -->

---

### 스트레스 테스트(Stress test)

![](../../images/test-scenario-stress.png)
<!-- .element class="stretch" -->

---

### 소크 또는 내구성 테스트(Soak or endurance test)

![](../../images/test-scenario-soak.png)
<!-- .element class="stretch" -->

---

### 스파이크 테스트(Spike test)

![](../../images/test-scenario-spike-test.png)
<!-- .element class="stretch" -->

---

### 브레이크포인트 테스트(Breakpoint test)

![](../../images/test-scenarios-breakpoint.png)
<!-- .element class="stretch" -->

---

## 부하 테스팅 프로세스

<div class="container">
  <div class="col">

  ### 고수준 개요

  - 부하 테스팅 계획
  - 부하 테스트 스크립팅
  - 부하 테스트 실행
  - 부하 테스팅 결과 분석  

  </div>

  <div class="col">

  ![지속적 테스팅 눈덩이](../../images/continuous-testing-snowball.png)

  </div>
</div>

---

## k6 OSS

- 이동: [04-getting-started-with-k6-oss](?p=04-getting-started-with-k6-oss)
