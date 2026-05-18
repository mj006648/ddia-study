# DDIA Study Questions

## Chapter 1

- 우리 시스템의 부하는 무엇으로 정의할 수 있는가?
- p95/p99 latency가 중요한 요청은 무엇인가?
- 장애가 결함에서 전체 장애로 확대되는 경로는 어디인가?

## Chapter 2

- 특정 데이터는 관계형, 문서형, 그래프형 중 무엇에 가까운가?
- 정규화와 비정규화 중 어떤 선택이 운영 비용을 줄이는가?
- source of truth와 파생 뷰는 명확히 분리되어 있는가?

## Chapter 3

- 쓰기 많은 데이터와 읽기 많은 데이터가 같은 저장소에 있는가?
- 인덱스는 실제 질의 패턴에 맞게 설계되었는가?
- OLTP 질의와 OLAP 질의가 서로 영향을 주고 있지 않은가?

## Chapter 4

- 스키마 변경 시 구버전 코드와 신버전 코드가 동시에 동작할 수 있는가?
- 이벤트 스키마는 필드 의미와 단위를 명확히 정의하는가?
- 과거 데이터는 새 코드로 읽을 수 있는가?

## Chapter 5

- 복제 지연이 사용자에게 어떤 문제로 나타날 수 있는가?
- leader failover 시 최근 쓰기 손실을 허용할 수 있는가?
- read-your-writes가 필요한 사용자 흐름은 무엇인가?

## Chapter 6

- 파티션 키가 부하를 고르게 분산하는가?
- hot partition이 생길 수 있는 키는 무엇인가?
- cross-partition transaction이나 join이 많은가?

## Chapter 7

- 반드시 유지해야 하는 데이터 불변조건은 무엇인가?
- 어떤 격리 수준에서 write skew나 lost update가 발생할 수 있는가?
- 분산 트랜잭션 대신 saga나 outbox로 풀 수 있는가?

## Chapter 8

- 재시도해도 안전한 API인가?
- timeout 값은 실제 지연 분포를 반영하는가?
- lease/lock 사용 시 fencing token이 있는가?

## Chapter 9

- linearizable해야 하는 데이터는 무엇인가?
- consensus 저장소가 critical path에 과도하게 들어가 있지 않은가?
- 강한 일관성과 eventual consistency의 경계가 문서화되어 있는가?

## Chapter 10

- 파생 데이터는 원천 데이터와 코드 버전으로 재생성 가능한가?
- batch job 실패 시 어디서부터 재시작하는가?
- lineage와 input snapshot을 기록하는가?

## Chapter 11

- 이벤트 중복 처리에 안전한가?
- event time과 processing time을 구분하는가?
- 늦게 도착한 이벤트가 결과를 어떻게 수정하는가?

## Chapter 12

- system of record와 derived data가 명확히 구분되어 있는가?
- 검색 인덱스, 캐시, 분석 테이블이 잘못되면 재생성 가능한가?
- 데이터플로우 전체의 schema, lineage, permission, quality를 어디서 관리하는가?
