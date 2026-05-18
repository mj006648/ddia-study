# 01. Reliable, Scalable, and Maintainable Applications

> 작성 원칙: 이 문서는 DDIA의 개념을 한국어로 재구성한 학습 노트이며, 원문 문장을 길게 옮기지 않습니다. 핵심 용어는 필요에 따라 English term을 병기합니다.

## 1. 이 장의 핵심 문제

데이터 중심 애플리케이션(data-intensive application)은 단순히 코드를 실행하는 프로그램이 아니라, 데이터 저장소, 캐시, 검색 인덱스, 스트림 처리기, 배치 처리기 등 여러 컴포넌트가 결합된 시스템입니다. 이 장은 이런 시스템을 평가하는 가장 기본적인 세 기준인 **Reliability**, **Scalability**, **Maintainability**를 설명합니다.

핵심은 “기능이 동작한다”만으로는 충분하지 않다는 점입니다. 실제 시스템은 장애, 부하 증가, 요구사항 변화 속에서도 계속 운영되어야 합니다. 따라서 좋은 데이터 시스템은 다음 질문에 답할 수 있어야 합니다.

- 장애가 발생해도 사용자가 기대하는 기능을 제공할 수 있는가?
- 데이터와 요청량이 증가해도 성능을 예측 가능하게 유지할 수 있는가?
- 시간이 지나도 사람이 이해하고 수정할 수 있는 구조인가?

## 2. 주요 개념

### Reliability

Reliability는 하드웨어, 소프트웨어, 사람의 실수에도 시스템이 올바른 기능을 계속 제공하는 성질입니다. 여기서 “올바르다”는 것은 시스템이 명시한 보장과 사용자의 기대를 만족한다는 의미입니다.

장애(fault)와 실패(failure)는 구분해야 합니다.

- **Fault**: 시스템 일부가 잘못된 상태가 되는 원인 또는 결함
- **Failure**: 사용자 관점에서 시스템 전체가 기대한 서비스를 제공하지 못하는 결과

좋은 시스템은 fault가 있어도 failure로 이어지지 않도록 설계합니다. 이를 fault-tolerant 또는 resilient하다고 표현합니다.

### Scalability

Scalability는 부하(load)가 증가할 때 시스템이 이를 처리할 수 있는 능력입니다. 단순히 “빠른 시스템”이라는 뜻이 아니라, 부하가 달라질 때 성능 특성이 어떻게 변하는지 설명하는 개념입니다.

부하는 시스템마다 다르게 정의됩니다.

- 웹 서비스: requests per second
- 데이터베이스: reads/writes per second, active users, dataset size
- 메시징 시스템: events per second, queue length
- 분석 시스템: data volume, job size, query complexity

성능(performance)을 볼 때는 평균보다 percentile이 중요합니다. 특히 p95, p99 latency는 일부 느린 요청이 사용자 경험과 운영 안정성에 큰 영향을 준다는 점을 보여줍니다.

### Maintainability

Maintainability는 시스템을 오래 운영하면서 수정·확장·디버깅할 수 있는 성질입니다. 초기 개발보다 장기 운영에서 더 중요해지는 품질입니다.

세부적으로는 다음 요소가 중요합니다.

- **Operability**: 운영자가 시스템 상태를 이해하고 문제를 해결하기 쉬운가?
- **Simplicity**: 불필요한 복잡성을 줄이고 핵심 모델을 이해하기 쉬운가?
- **Evolvability**: 요구사항 변화에 맞춰 구조를 바꾸기 쉬운가?

## 3. 동작 원리

데이터 시스템은 여러 도구를 조합해 만들어집니다. 예를 들어 하나의 서비스가 PostgreSQL에 원본 데이터를 저장하고, Redis를 캐시로 사용하며, Elasticsearch로 검색 인덱스를 만들고, Kafka로 이벤트를 전달할 수 있습니다. 사용자는 하나의 애플리케이션을 본다고 생각하지만, 내부적으로는 여러 저장·처리 시스템이 연결되어 있습니다.

이때 reliability, scalability, maintainability는 개별 컴포넌트가 아니라 전체 조합의 속성입니다. 데이터베이스가 안정적이어도 캐시 무효화가 잘못되면 잘못된 데이터를 보여줄 수 있고, 메시지 큐가 빠르더라도 소비자가 밀리면 전체 처리 지연이 커질 수 있습니다.

## 4. 장점과 한계

이 장의 장점은 데이터 시스템을 특정 기술이 아니라 품질 속성 관점으로 바라보게 해준다는 점입니다. 어떤 DB가 좋은가보다, 어떤 요구조건에서 어떤 trade-off가 발생하는지가 중요합니다.

한계는 아직 구체적 구현 방법보다는 평가 기준을 제시하는 수준이라는 점입니다. 이후 장에서 replication, partitioning, transactions, consensus, batch/stream processing을 다루며 이 기준들이 실제 설계 문제로 확장됩니다.

## 5. 시스템 설계 관점

시스템 설계에서 이 장은 요구사항을 정리하는 기준으로 사용할 수 있습니다.

- Reliability 요구사항: 허용 가능한 downtime, data loss, recovery time은 얼마인가?
- Scalability 요구사항: 현재 부하와 예상 성장률은 어떻게 되는가?
- Maintainability 요구사항: 누가 운영하고, 어떤 방식으로 배포·모니터링·디버깅하는가?

특히 성능 요구사항은 평균 latency가 아니라 tail latency와 부하 조건을 함께 적어야 합니다. 예를 들어 “응답이 빨라야 함”보다 “초당 1,000 요청에서 p95 latency 200ms 이하”가 설계에 유용합니다.

## 6. 실무/연구 연결점

Lakehouse나 분산 메타데이터 시스템에서도 같은 기준을 적용할 수 있습니다.

- Reliability: metadata catalog, object storage, query engine 중 일부 장애가 전체 실패로 이어지는가?
- Scalability: dataset 수, file 수, metadata query 수가 증가할 때 병목은 어디인가?
- Maintainability: catalog, cache, compute engine, governance component를 독립적으로 교체할 수 있는가?

Trident Lakehouse 관점에서는 Redis/Milvus 같은 acceleration layer가 추가될 때 성능은 좋아질 수 있지만, cache consistency와 운영 복잡성이 증가합니다. 따라서 scalability 개선이 maintainability 저하로 이어지지 않도록 설계해야 합니다.

## 7. 헷갈리기 쉬운 부분

- Fault와 failure는 다릅니다. fault가 있어도 failure가 발생하지 않도록 만드는 것이 목표입니다.
- Scalability는 “현재 빠르다”가 아니라 “부하 증가에 대응 가능하다”는 뜻입니다.
- 평균 latency만 보면 tail latency 문제를 놓칠 수 있습니다.
- Maintainability는 코드 가독성뿐 아니라 운영, 관찰 가능성, 배포, 장애 대응까지 포함합니다.

## 8. 핵심 질문 정리

- 이 시스템에서 가장 중요한 reliability guarantee는 무엇인가?
- 부하를 어떤 지표로 표현할 것인가?
- p95/p99 latency는 어느 정도까지 허용 가능한가?
- 복잡성을 줄이기 위해 어떤 abstraction을 둘 것인가?
- 앞으로 요구사항이 바뀔 때 어떤 부분이 가장 바꾸기 어려운가?

## 9. 한 문단 요약

1장은 데이터 중심 애플리케이션을 평가하는 기본 기준으로 reliability, scalability, maintainability를 제시합니다. 장애를 완전히 없애는 것이 아니라 fault가 failure로 이어지지 않게 만들고, 부하 증가에 대한 성능 변화를 측정 가능하게 표현하며, 장기적으로 운영·수정 가능한 단순한 구조를 만드는 것이 좋은 데이터 시스템 설계의 출발점입니다.

## 10. 설계 체크리스트

- **신뢰성 경계:** DB, 캐시, 메시지 큐, 워크플로우 엔진 중 어느 컴포넌트가 실패해도 사용자 요청이 계속 처리되어야 하는지 구분한다.
- **복구 목표:** RPO와 RTO를 정한다. RPO는 잃어도 되는 데이터 양이고, RTO는 복구까지 허용되는 시간이다.
- **부하 모델:** QPS, 동시 사용자, 데이터 증가량, 요청별 fan-out, background job 수를 분리해 본다.
- **관찰 가능성:** 장애를 감지하려면 latency, error rate, saturation, queue length, replication lag, retry count를 봐야 한다.

## 11. 미니 사례

메타데이터 검색 서비스가 느려졌다고 가정한다. 평균 latency는 80ms지만 p99는 3초라면, 대부분 사용자는 괜찮아도 일부 workflow가 timeout으로 실패할 수 있다. 이 경우 “평균 성능 개선”보다 tail latency를 만드는 원인, 예를 들어 특정 인기 데이터셋의 lock contention, slow query, cache miss storm을 찾아야 한다.

## 12. 한 줄 결론

데이터 시스템 설계의 출발점은 제품명이 아니라, 어떤 장애를 견디고 어떤 부하를 감당하며 얼마나 쉽게 바뀔 수 있어야 하는지 정의하는 것이다.
