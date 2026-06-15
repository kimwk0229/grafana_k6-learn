
<style>
  .container {
    display: flex;
  }

  .col {
    flex: 1;
  }
</style>

## 프론트엔드 성능 테스팅 

![예시 웹사이트 제목 'crocs r cool'](../../images/frontend-performance.png)
<!-- .element class="stretch" -->

---

<div class="container">
  <div class="col">

  ### 프론트엔드 성능 테스팅만으로는 충분하지 않은 이유?

  - 프론트엔드 성능은 내부를 들여다보지 않습니다
  - 부하 없이는 트래픽 조건에서 테스트하지 않습니다
  - 브라우저 수준의 부하 테스팅은 리소스 집약적이며 비용이 많이 듭니다

  </div>

  <div class="col">

  ![브라우저 부하 테스팅이 비용이 많이 들고 리소스 집약적임을 보여주는 그래픽](../../images/frontend-limitations.png)

  </div>
</div>

---

![프론트엔드와 백엔드 응답 시간을 집계한 차트. 동시성이 증가할수록 백엔드 응답 시간이 훨씬 더 길어집니다.](../../images/frontend-backend.png)

---

## 백엔드 성능 테스팅

![백엔드 시스템의 컴포넌트](../../images/backend-component.png)
<!-- .element class="stretch" -->

---

## 백엔드 성능 테스팅이 검증하는 것

- **확장성(Scalability)**
- **탄력성(Elasticity)**
- **가용성(Availability)** 
- **신뢰성(Reliability)** 
- **복원력(Resiliency)**
- **지연 시간(Latency)**

---

<div class="container">
  <div class="col">

  ### 백엔드 성능 테스팅만으로는 충분하지 않은 이유?

  - 사용자 경험을 무시합니다
  - 스크립트 작성이 길어질 수 있습니다
  - 유지 관리가 더 어렵습니다

  </div>

  <div class="col">

  ![브라우저 부하 테스팅이 비용이 많이 들고 리소스 집약적임을 보여주는 그래픽](../../images/frontend-limitations.png)

  </div>
</div>

---

## 부하 테스팅

- 이동: [03-load-testing](?p=03-load-testing)
