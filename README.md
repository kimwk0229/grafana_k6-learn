# k6-Learn에 오신 것을 환영합니다

이 저장소는 다음을 위한 자료를 제공합니다:
- 슬라이드 프레젠테이션 제작
- 워크샵 진행
- k6 발표
- k6 학습

## 다른 시작 자료

아래 자료들도 k6를 배우거나 가르치는 데 활용할 수 있습니다. 이 프로젝트들은 자신의 목적과 청중에 맞게 복사하고 수정하여 사용할 수 있습니다. 시작점으로 활용하되, 청중에게 맞지 않는 내용은 제거하거나 자신만의 스타일을 보여주는 새로운 내용을 추가해 보세요.

- [k6 스타터 슬라이드 덱](https://docs.google.com/presentation/d/1gviRg7RTzT0Y2_5WPBADyn5xpa96PIqWivGAThNW6pM/edit?usp=sharing)
- [k6 OSS 워크샵](https://github.com/grafana/k6-oss-workshop)

## 더 실습 위주의 내용을 원하신다면?

k6 워크샵을 직접 운영해 보세요. 아래에 워크샵 구성 예시와 각 주제에 활용할 수 있는 모듈들이 정리되어 있습니다. 자신에게 가장 적합한 부분을 자유롭게 선택해서 사용하세요!

슬라이드도 기본 제공되며, 자유롭게 편집할 수 있습니다. 슬라이드를 커스터마이징하려면 이 저장소를 fork한 후 원하는 워크샵 구성에 맞게 편집하세요.

슬라이드는 [reveal.js](https://revealjs.com/)로 제작되었으며 [slides](./slides/) 폴더에서 찾을 수 있습니다.

### 슬라이드 실행 방법

터미널에서 다음 명령어를 실행하세요:

```
npm install
npm run slides
```

### I: 성능 테스트 원칙

- [성능 테스트 소개](Modules/I-Performance-testing-principles/01-Introduction-to-Performance-Testing.md)
- [프론트엔드 vs. 백엔드 성능 테스트](Modules/I-Performance-testing-principles/02-Frontend-vs-backend-performance-testing.md)
- [부하 테스트](Modules/I-Performance-testing-principles/03-Load-Testing.md)
- [부하 테스트 프로세스 개요](Modules/I-Performance-testing-principles/04-High-level-overview-of-the-load-testing-process.md)

### II: k6 기초

- [k6 OSS 시작하기](Modules/II-k6-Foundations/01-Getting-started-with-k6-OSS.md)
- [k6 CLI](Modules/II-k6-Foundations/02-The-k6-CLI.md)
- [k6 결과 이해하기](Modules/II-k6-Foundations/03-Understanding-k6-results.md)
- [스크립트에 체크 추가하기](Modules/II-k6-Foundations/04-Adding-checks-to-your-script.md)
- [sleep을 이용한 think time 추가](Modules/II-k6-Foundations/05-Adding-think-time-using-sleep.md)
- [k6 부하 테스트 옵션](Modules/II-k6-Foundations/06-k6-Load-Test-Options.md)
- [임계값으로 테스트 기준 설정하기](Modules/II-k6-Foundations/07-Setting-test-criteria-with-thresholds.md)
- [k6 결과 출력 옵션](Modules/II-k6-Foundations/08-k6-results-output-options.md)
- [k6 스크립트 녹화하기](Modules/II-k6-Foundations/09-Recording-a-k6-script.md)

### III: k6 중급

- [k6 부하 테스트 스크립트 디버깅 방법](Modules/III-k6-Intermediate/01-How-to-debug-k6-load-testing-scripts.md)
- [k6의 동적 상관관계](Modules/III-k6-Intermediate/02-Dynamic-correlation-in-k6.md)
- [워크로드 모델링](Modules/III-k6-Intermediate/03-Workload-modeling.md)
- [테스트 데이터 추가하기](Modules/III-k6-Intermediate/04-Adding-test-data.md)
- [k6에서 병렬 요청 처리](Modules/III-k6-Intermediate/05-Parallel-requests-in-k6.md)
- [트랜잭션별 코드 구성 - 그룹과 태그](Modules/III-k6-Intermediate/06-Organizing-code-in-k6-by-transaction_groups-and-tags.md)
- [Setup과 Teardown 함수](Modules/III-k6-Intermediate/07-Setup-and-Teardown-functions.md)
- [Executor로 부하 프로파일 설정하기](Modules/III-k6-Intermediate/08-Setting-load-profiles-with-executors.md)
- [시나리오로 워크로드 모델링하기](Modules/III-k6-Intermediate/09-Workload-modeling-with-scenarios.md)
- [실행 컨텍스트 변수 활용하기](Modules/III-k6-Intermediate/10-Using-execution-context-variables.md)
- [커스텀 메트릭 생성 및 활용](Modules/III-k6-Intermediate/11-Creating-and-using-custom-metrics.md)

## 기여자

k6-learn은 이 멋진 기여자들 덕분에 만들어질 수 있었습니다! 🌟

- [Imma Valls](https://github.com/immavalls)
- [Krzysztof Widera](https://github.com/kwidera)
- [Leandro Melendez](https://github.com/srperf)
- [Marie Cruz](https://github.com/mdcruz)
- [Matt Dodson](https://github.com/MattDodsonEnglish)
- [Nicole van der Hoeven](https://github.com/nicolevanderhoeven)
- [Paul Balogh](https://github.com/javaducky)

## 기여 방법

1. 이슈가 없다면 https://github.com/grafana/k6-learn/issues 에서 먼저 이슈를 생성하세요.
2. 이 [저장소](https://github.com/grafana/k6-learn)를 [Fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks) 하세요.
3. Fork한 저장소에서 변경 사항을 작성하세요. 새 브랜치, 변경, 푸시는 모두 본인의 저장소에만 영향을 줍니다.
4. 준비가 되면 [Fork에서 Pull Request를 생성](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request-from-a-fork)하고, 해결하는 이슈(1단계)를 언급하세요. 필요하다면 [동기화](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/syncing-a-fork)가 필요할 수 있습니다.
5. Pull Request를 생성하면 변경 사항이 검토되어 승인되거나 수정 요청을 받게 됩니다.

Fork에 대한 자세한 내용은 https://docs.github.com/en/get-started/quickstart/fork-a-repo 를 참고하세요.
