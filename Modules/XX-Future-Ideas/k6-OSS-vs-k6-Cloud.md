첫 번째 k6 부하 테스팅 스크립트를 만들 준비가 되셨나요! 그 전에 선택지에 대해 이야기해 봅시다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/ncxCIuo5tUU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
_k6가 무엇인지와 k6 OSS와 k6 Cloud의 차이점에 대한 비디오 개요_

## k6 오픈 소스 소프트웨어 (OSS) vs k6 Cloud

처음부터 테스트 스크립트를 작성하는 방법은 두 가지입니다: k6 OSS와 k6 Cloud. k6 OSS는 부하를 생성하기 위한 무료 오픈 소스 도구입니다. k6 Cloud는 동일한 도구인 k6 OSS를 사용하지만 온디맨드 클라우드 인프라, 실시간 대시보드 및 통합과 같은 기능도 추가하는 SaaS(Software as a Service) 플랫폼입니다. k6 OSS와 k6 Cloud의 더 자세한 비교는 [이 페이지](https://k6.io/oss-vs-cloud/)에서 확인할 수 있습니다.

k6 OSS가 적합한 상황:
- k6를 시도해보고 사용 방법을 배우고 싶은 경우.
- 로컬 또는 사설 네트워크 내에서 부하를 생성하고 싶은 경우.
- 분산 실행이 필요하지 않거나, 분산 테스트 실행을 위해 [k6 operator](https://github.com/grafana/k6-operator)를 설정할 의사가 있는 경우.
- JavaScript로 스크립팅하는 것이 괜찮은 경우.

k6 Cloud가 적합한 상황:
- 부하 테스트 실행에 있어 가장 원활한 경험을 원하는 경우 (코딩 불필요).
- 사용자 수나 다양한 지리적 지역에서 부하를 생성해야 하는 요구 사항으로 인해 분산 실행이 필요한 경우.
- 테스트가 실행되는 동안 부하 테스팅 결과를 보여주는 실시간 대시보드를 원하는 경우.
- 팀과 k6 결과를 공유하고 싶은 경우.

이 워크샵에서는 k6 OSS와 k6 Cloud 모두 사용하는 방법을 배웁니다. k6 OSS에서 첫 번째 k6 스크립트를 시작하지만, 나중에 k6 Cloud를 통합하는 방법도 배웁니다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/zcYeboT5FYE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
