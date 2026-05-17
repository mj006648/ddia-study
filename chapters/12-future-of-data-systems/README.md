# 12. The Future of Data Systems

> 작성 원칙: 이 문서는 DDIA의 개념을 한국어로 재구성한 학습 노트이며, 원문 문장을 길게 옮기지 않습니다. 핵심 용어는 필요에 따라 English term을 병기합니다.

## 1. 이 장의 핵심 문제

마지막 장은 앞에서 다룬 storage, replication, transaction, batch, stream 개념을 종합해 데이터 시스템의 미래 방향을 논의합니다. 핵심 문제는 “여러 데이터 시스템을 어떻게 조합해 정확하고 유연하며 진화 가능한 전체 시스템을 만들 것인가?”입니다.

## 2. 주요 개념

### Data Integration

현실의 시스템은 하나의 DB만 쓰지 않습니다. Search index, cache, analytics table, materialized view, event log 등 여러 derived data system이 함께 존재합니다.

문제는 이들이 서로 다른 시점과 형식의 데이터를 가질 수 있다는 점입니다.

### Derived Data

Derived data는 원본에서 계산해 만든 데이터입니다. Index, cache, summary table, recommendation model, feature store 등이 여기에 해당합니다. Derived data는 원칙적으로 재생성 가능해야 합니다.

### Unbundling Databases

전통적 DB가 제공하던 기능을 여러 component로 분리해 조합하는 접근입니다.

- storage
- query processing
- indexing
- caching
- stream processing
- transaction/coordination

이 접근은 유연하지만 integration complexity를 증가시킵니다.

### Correctness

End-to-end correctness는 개별 컴포넌트의 보장만으로 달성되지 않습니다. 데이터 흐름 전체에서 idempotency, ordering, constraint, recovery를 고려해야 합니다.

### Data Ethics

데이터 시스템은 기술적 문제만이 아니라 사회적 책임도 갖습니다. 개인정보, 권한, 편향, 감시, 데이터 오용 가능성을 고려해야 합니다.

## 3. 동작 원리

미래의 데이터 시스템은 여러 specialized system을 event log와 schema/metadata를 통해 연결하는 방향으로 발전합니다. 원본 변경을 capture하고, 이를 stream/batch pipeline이 읽어 search index, cache, analytics view를 갱신합니다.

이때 중요한 것은 각 derived data가 어떤 원본과 어떤 version에서 만들어졌는지 추적하는 것입니다. Lineage와 metadata가 없으면 결과의 신뢰성을 판단하기 어렵습니다.

## 4. 장점과 한계

Component를 분리하면 각 workload에 맞는 최적 도구를 사용할 수 있습니다. 그러나 correctness가 application과 pipeline 전체의 책임으로 이동합니다. 운영자는 더 많은 moving parts를 이해해야 하고, 장애 복구 시 어느 derived data가 stale한지 파악해야 합니다.

## 5. 시스템 설계 관점

- system of record는 무엇인가?
- derived data는 어떻게 재생성할 수 있는가?
- event log와 batch storage 중 무엇을 기준으로 복구하는가?
- constraint와 uniqueness는 어디서 보장하는가?
- metadata, lineage, access control은 dataflow 전체에 적용되는가?

## 6. 실무/연구 연결점

Trident Lakehouse 같은 구조는 이 장의 관점과 잘 맞습니다. Iceberg는 system of record와 snapshot history를 제공하고, Redis/Milvus는 derived acceleration layer로 볼 수 있습니다. PostgreSQL governance/catalog, Keycloak identity, portal/workflow는 데이터 사용의 제어와 관찰성을 제공합니다.

핵심은 derived layer를 원본처럼 착각하지 않는 것입니다. Redis cache나 Milvus index는 빠른 접근을 위한 파생 상태이므로, 원본 metadata와 version을 기준으로 재생성·검증할 수 있어야 합니다.

## 7. 헷갈리기 쉬운 부분

- 여러 전문 시스템을 조합하면 자동으로 좋은 아키텍처가 되는 것은 아닙니다.
- Cache와 index는 원본 데이터가 아니라 derived data입니다.
- Correctness는 DB 하나의 문제가 아니라 end-to-end dataflow의 문제입니다.
- 기술적 효율성과 데이터 윤리는 분리된 문제가 아닙니다.

## 8. 핵심 질문 정리

- 각 데이터의 source of truth는 무엇인가?
- 파생 데이터가 stale하거나 깨졌을 때 어떻게 감지하고 복구하는가?
- 전체 dataflow의 lineage를 추적할 수 있는가?
- 사용자 권한과 개인정보 정책이 derived data에도 적용되는가?

## 9. 한 문단 요약

12장은 현대 데이터 시스템을 여러 specialized component와 derived dataflow의 조합으로 바라봅니다. 좋은 시스템은 storage, stream, batch, index, cache, catalog를 목적에 맞게 분리하면서도 source of truth, lineage, correctness, recovery, ethics를 end-to-end로 관리해야 합니다.
