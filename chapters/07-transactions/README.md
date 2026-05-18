# 07. Transactions

> 작성 원칙: 이 문서는 DDIA의 개념을 한국어로 재구성한 학습 노트이며, 원문 문장을 길게 옮기지 않습니다. 핵심 용어는 필요에 따라 English term을 병기합니다.

## 1. 이 장의 핵심 문제

Transaction은 여러 read/write를 하나의 논리적 작업 단위로 묶어주는 abstraction입니다. 핵심 문제는 “동시 실행과 장애가 있어도 애플리케이션이 데이터 정합성을 쉽게 다룰 수 있게 할 것인가?”입니다.

## 2. 주요 개념

### ACID

- **Atomicity**: 일부만 반영되지 않음. 성공 또는 rollback
- **Consistency**: 애플리케이션이 정의한 invariant를 유지
- **Isolation**: 동시에 실행되는 transaction이 서로 방해하지 않는 것처럼 보임
- **Durability**: commit된 데이터는 장애 후에도 유지

### Isolation Levels

- **Read Committed**: dirty read/write 방지
- **Snapshot Isolation**: transaction 시작 시점의 consistent snapshot을 읽음
- **Serializable**: transaction들이 어떤 순차 실행과 같은 결과를 내도록 보장

### Concurrency Anomalies

- **Dirty read**: commit되지 않은 값을 읽음
- **Dirty write**: commit되지 않은 값을 덮어씀
- **Lost update**: 동시에 갱신해 한쪽 변경이 사라짐
- **Write skew**: 각 transaction은 유효해 보이지만 함께 실행되면 invariant가 깨짐
- **Phantom read**: 조건에 맞는 row 집합이 transaction 중 바뀜

### Serializability 구현

- 실제 순차 실행
- Two-phase locking(2PL)
- Serializable Snapshot Isolation(SSI)

## 3. 동작 원리

Transaction은 애플리케이션에게 복잡한 동시성 문제를 숨겨줍니다. 하지만 isolation level이 낮으면 일부 anomaly는 여전히 발생할 수 있습니다. Snapshot isolation은 읽기 성능과 개발 편의성이 좋지만 write skew 같은 문제는 막지 못할 수 있습니다.

Serializable은 가장 강한 보장이지만 성능 비용, abort 증가, 구현 복잡성이 있습니다.

## 4. 장점과 한계

Transaction은 correctness를 단순화하지만 분산 환경에서는 비용이 커집니다. 모든 작업을 강한 transaction으로 묶으면 성능과 availability가 떨어질 수 있습니다. 반대로 transaction을 너무 약하게 쓰면 애플리케이션이 직접 복잡한 race condition을 처리해야 합니다.

## 5. 시스템 설계 관점

- 어떤 invariant가 반드시 지켜져야 하는가?
- lost update나 write skew가 실제 비즈니스 문제를 만드는가?
- 강한 isolation이 필요한 경로와 eventual consistency가 가능한 경로를 구분했는가?
- retry 가능한 transaction으로 설계했는가?

## 6. 실무/연구 연결점

Iceberg table commit은 snapshot pointer를 원자적으로 갱신해야 하므로 transaction 개념과 연결됩니다. 여러 writer가 동시에 table metadata를 갱신할 때 optimistic concurrency control이 필요합니다.

Trident Lakehouse에서는 catalog metadata, Redis cache, Milvus index, object storage 사이의 update가 하나의 DB transaction처럼 묶이지 않을 수 있습니다. 따라서 idempotency, retry, compensation, metadata versioning이 중요합니다.

## 7. 헷갈리기 쉬운 부분

- Consistency in ACID와 distributed consistency는 같은 단어지만 의미가 다릅니다.
- Snapshot isolation은 serializable과 다릅니다.
- Transaction이 있으면 모든 동시성 문제가 자동으로 사라지는 것은 아닙니다. isolation level을 알아야 합니다.

## 8. 핵심 질문 정리

- 어떤 데이터 invariant가 깨지면 안 되는가?
- 현재 DB isolation level은 무엇인가?
- concurrent update가 발생하면 retry/abort 처리는 어떻게 되는가?
- 분산 컴포넌트 사이의 partial failure를 어떻게 복구하는가?

## 9. 한 문단 요약

7장은 transaction이 동시성과 장애로부터 애플리케이션을 보호하는 abstraction임을 설명합니다. ACID와 isolation level을 이해해야 dirty read, lost update, write skew 같은 문제를 피할 수 있으며, 특히 분산 시스템에서는 강한 transaction과 성능·가용성 사이의 trade-off를 명확히 선택해야 합니다.

## 10. 설계 체크리스트

- **불변조건:** 동시에 실행되어도 깨지면 안 되는 규칙을 먼저 쓴다.
- **격리 수준:** read committed, snapshot isolation, serializable 중 필요한 수준을 선택한다.
- **동시성 이상:** lost update, write skew, phantom read가 가능한 흐름을 찾는다.
- **외부 부작용:** DB transaction과 이메일, 파일 업로드, 메시지 발행 같은 외부 작업의 경계를 정한다.

## 11. 미니 사례

두 사용자가 동시에 같은 데이터셋을 공개 상태로 변경하고 권한 정책을 수정한다고 하자. 스냅샷 격리에서는 각 트랜잭션이 서로 다른 행만 수정하면 전체 보안 정책의 불변조건이 깨질 수 있다. 이런 경우 unique constraint, explicit lock, serializable transaction, 또는 상태 전이 테이블을 이용해 불변조건을 DB 수준에서 보호해야 한다.

## 12. 한 줄 결론

트랜잭션은 편의 기능이 아니라 불변조건을 지키는 도구이며, 어떤 불변조건을 지켜야 하는지 모르면 적절한 격리 수준도 고를 수 없다.
