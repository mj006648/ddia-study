# DDIA Study Notes

> Martin Kleppmann, _Designing Data-Intensive Applications_ 학습 정리 저장소

이 저장소는 DDIA를 장별로 공부하면서 **개념 설명 → 핵심 질문 → 시스템 설계 관점 → 실무/연구 연결점** 순서로 정리하기 위한 개인 학습 노트입니다. 책의 문장을 그대로 옮기기보다는, 각 장의 아이디어를 직접 이해한 내용으로 재구성합니다.

## 정리 원칙

- **상세하지만 복붙이 아닌 정리**: 책의 원문을 길게 인용하지 않고, 개념·구조·예시를 내 언어로 재작성함
- **챕터별 독립 문서화**: 각 장은 `chapters/XX-*` 디렉터리 아래에서 별도 README로 관리함
- **시스템 설계 중심**: 단순 요약보다 “왜 필요한가”, “어떤 trade-off가 있는가”, “언제 선택해야 하는가”를 강조함
- **연구/구현 연결**: Lakehouse, 분산 메타데이터, 스트리밍, 트랜잭션, 일관성 모델 등 연구 주제와 연결 가능한 지점을 별도 정리함

## 전체 구성

DDIA는 크게 세 부분으로 구성됩니다.

| Part | 범위 | 핵심 주제 |
|---|---:|---|
| Part I. Foundations of Data Systems | 1–4장 | 데이터 시스템의 기본 품질, 데이터 모델, 저장소, 인코딩/진화 |
| Part II. Distributed Data | 5–9장 | 복제, 파티셔닝, 트랜잭션, 분산 시스템의 한계, 일관성과 합의 |
| Part III. Derived Data | 10–12장 | 배치 처리, 스트림 처리, 파생 데이터 시스템의 미래 |

## 챕터별 학습 계획

### Part I. Foundations of Data Systems

#### [01. Reliable, Scalable, and Maintainable Applications](chapters/01-reliable-scalable-maintainable/README.md)

데이터 중심 애플리케이션이 갖춰야 할 기본 품질을 다룹니다.

- **Reliability**: 장애가 있어도 시스템이 올바르게 동작하도록 만드는 관점
- **Scalability**: 부하 증가에 대응하기 위한 성능 지표와 확장 방식
- **Maintainability**: 시간이 지나도 운영·수정·확장이 가능한 구조
- **핵심 질문**: 시스템이 “잘 동작한다”는 것을 어떤 기준으로 판단할 것인가?

#### [02. Data Models and Query Languages](chapters/02-data-models-query-languages/README.md)

데이터를 어떤 모델로 표현하고 질의할 것인지 다룹니다.

- **Relational Model**: 테이블, 조인, 정규화, 선언형 질의
- **Document Model**: JSON/XML 기반 문서 구조와 지역성
- **Graph Model**: 복잡한 관계 표현과 탐색
- **Query Language**: SQL, MapReduce 스타일, 그래프 질의 언어
- **핵심 질문**: 데이터 구조와 질의 패턴이 시스템 설계를 어떻게 결정하는가?

#### [03. Storage and Retrieval](chapters/03-storage-retrieval/README.md)

데이터베이스가 데이터를 저장하고 검색하는 내부 구조를 다룹니다.

- **Log-structured Storage**: append-only log, SSTable, LSM-tree
- **Page-oriented Storage**: B-tree 계열 인덱스
- **OLTP vs OLAP**: 트랜잭션 처리와 분석 처리의 저장 방식 차이
- **Column-oriented Storage**: 컬럼 저장, 압축, 벡터화 처리
- **핵심 질문**: 읽기/쓰기 패턴에 따라 저장 엔진 선택이 어떻게 달라지는가?

#### [04. Encoding and Evolution](chapters/04-encoding-evolution/README.md)

데이터 형식과 스키마가 시간이 지나며 바뀌는 문제를 다룹니다.

- **Encoding Format**: JSON, XML, Protocol Buffers, Avro 등
- **Schema Evolution**: forward/backward compatibility
- **Dataflow**: 데이터베이스, 서비스, 메시지 큐를 통한 데이터 흐름
- **핵심 질문**: 시스템을 중단하지 않고 데이터 구조를 어떻게 진화시킬 것인가?

### Part II. Distributed Data

#### [05. Replication](chapters/05-replication/README.md)

같은 데이터를 여러 노드에 복제하는 방식을 다룹니다.

- **Leader-based Replication**: 단일 리더 기반 복제
- **Multi-leader Replication**: 다중 리더와 충돌 처리
- **Leaderless Replication**: quorum, read repair, anti-entropy
- **Replication Lag**: eventual consistency, read-your-writes 등
- **핵심 질문**: 가용성, 지연, 일관성 사이에서 어떤 균형을 선택할 것인가?

#### [06. Partitioning](chapters/06-partitioning/README.md)

큰 데이터를 여러 노드에 나누어 저장하는 방식을 다룹니다.

- **Key-range Partitioning**: 범위 기반 파티셔닝
- **Hash Partitioning**: 해시 기반 분산과 hot spot 완화
- **Secondary Index**: 파티션 환경에서 보조 인덱스 관리
- **Rebalancing**: 노드 추가/제거 시 데이터 재배치
- **핵심 질문**: 데이터와 요청 부하를 어떻게 균등하게 분산할 것인가?

#### [07. Transactions](chapters/07-transactions/README.md)

동시성과 장애 상황에서 데이터 정합성을 보장하는 추상화를 다룹니다.

- **ACID**: 원자성, 일관성, 격리성, 지속성
- **Isolation Levels**: Read Committed, Snapshot Isolation, Serializable 등
- **Concurrency Bugs**: lost update, write skew, phantom read
- **Serializability**: 가장 강한 격리 수준과 구현 방식
- **핵심 질문**: 애플리케이션이 직접 처리하기 어려운 동시성 문제를 DB가 어디까지 맡아야 하는가?

#### [08. The Trouble with Distributed Systems](chapters/08-trouble-with-distributed-systems/README.md)

분산 시스템에서 신뢰하기 어려운 요소들을 다룹니다.

- **Partial Failure**: 일부 노드/네트워크만 실패하는 상황
- **Unreliable Network**: 지연, 손실, 중복, 순서 뒤바뀜
- **Clock and Timing**: clock drift, timeout, lease 문제
- **Process Pause**: GC pause, scheduling delay 등
- **핵심 질문**: 불확실한 네트워크와 시간 위에서 어떻게 안전한 시스템을 만들 것인가?

#### [09. Consistency and Consensus](chapters/09-consistency-consensus/README.md)

분산 시스템에서 일관성과 합의를 달성하는 방식을 다룹니다.

- **Linearizability**: 단일 최신 값처럼 보이는 일관성
- **Ordering Guarantees**: causal order, total order
- **Distributed Transactions**: 2PC와 그 한계
- **Consensus**: leader election, atomic broadcast, coordination service
- **핵심 질문**: 여러 노드가 하나의 결정에 안전하게 도달하려면 무엇이 필요한가?

### Part III. Derived Data

#### [10. Batch Processing](chapters/10-batch-processing/README.md)

대량 데이터를 오프라인으로 처리하는 방식을 다룹니다.

- **Unix Philosophy**: 단순한 도구 조합과 데이터 파이프라인
- **MapReduce**: 분산 배치 처리 모델
- **Joins and Grouping**: 대규모 데이터 조인/집계 전략
- **Workflow**: 배치 작업의 의존성과 재실행
- **핵심 질문**: 대규모 데이터를 안정적으로 다시 계산할 수 있는 구조는 무엇인가?

#### [11. Stream Processing](chapters/11-stream-processing/README.md)

지속적으로 들어오는 이벤트를 처리하는 방식을 다룹니다.

- **Event Stream**: 메시지 브로커, 로그, pub/sub
- **Stream Joins**: stream-stream, stream-table join
- **Time and Window**: event time, processing time, windowing
- **Fault Tolerance**: exactly-once semantics, checkpointing
- **핵심 질문**: 끊임없이 들어오는 데이터에서 정확하고 재현 가능한 결과를 어떻게 만들 것인가?

#### [12. The Future of Data Systems](chapters/12-future-of-data-systems/README.md)

앞선 개념들을 종합해 데이터 시스템의 방향을 다룹니다.

- **Derived Data Integration**: 배치/스트림/인덱스/캐시의 통합적 관점
- **Unbundling Databases**: DB 기능을 독립 컴포넌트로 분리하는 접근
- **Correctness and Constraints**: end-to-end correctness, uniqueness, integrity
- **Ethics and Data**: 데이터 사용의 책임과 사회적 영향
- **핵심 질문**: 데이터 시스템은 앞으로 어떤 방식으로 조합·진화해야 하는가?

## 각 챕터 문서 템플릿

각 장은 다음 구조로 정리합니다.

```markdown
# XX. Chapter Title

## 1. 이 장의 핵심 문제
## 2. 주요 개념
## 3. 동작 원리
## 4. 장점과 한계
## 5. 시스템 설계 관점
## 6. 실무/연구 연결점
## 7. 헷갈리기 쉬운 부분
## 8. 핵심 질문 정리
## 9. 한 문단 요약
```

## 진행 상태

| Chapter | Status |
|---|---|
| 01. Reliable, Scalable, and Maintainable Applications | 초안 작성 |
| 02. Data Models and Query Languages | 초안 작성 |
| 03. Storage and Retrieval | 초안 작성 |
| 04. Encoding and Evolution | 초안 작성 |
| 05. Replication | 초안 작성 |
| 06. Partitioning | 초안 작성 |
| 07. Transactions | 초안 작성 |
| 08. The Trouble with Distributed Systems | 초안 작성 |
| 09. Consistency and Consensus | 초안 작성 |
| 10. Batch Processing | 초안 작성 |
| 11. Stream Processing | 초안 작성 |
| 12. The Future of Data Systems | 초안 작성 |

## 참고

- Book: Martin Kleppmann, _Designing Data-Intensive Applications_
- 이 저장소는 개인 학습 목적의 요약·해설 노트이며, 책의 원문을 대체하지 않습니다.
