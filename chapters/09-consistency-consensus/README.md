# 09. Consistency and Consensus

> 작성 원칙: 이 문서는 DDIA의 개념을 한국어로 재구성한 학습 노트이며, 원문 문장을 길게 옮기지 않습니다. 핵심 용어는 필요에 따라 English term을 병기합니다.

## 1. 이 장의 핵심 문제

분산 시스템에서 여러 노드가 같은 상태와 같은 결정에 도달하는 것은 어렵습니다. 이 장은 linearizability, ordering, distributed transaction, consensus를 다룹니다. 핵심 문제는 “불확실한 네트워크 위에서 여러 노드가 하나의 일관된 결정을 하도록 만들 수 있는가?”입니다.

## 2. 주요 개념

### Linearizability

시스템이 하나의 최신 값만 가진 것처럼 보이는 강한 consistency model입니다. write가 성공한 뒤의 read는 그 write를 반영해야 합니다.

장점은 이해하기 쉽고 correctness reasoning이 단순하다는 것입니다. 단점은 성능과 availability 비용이 크다는 것입니다.

### Ordering

분산 시스템에서는 event 순서가 중요합니다.

- **Causal order**: 원인-결과 관계를 보존
- **Total order**: 모든 노드가 같은 event 순서를 봄

Total order broadcast는 consensus와 밀접한 관계가 있습니다.

### Distributed Transactions와 2PC

Two-phase commit(2PC)은 여러 participant가 commit/abort를 함께 결정하게 합니다.

- prepare phase
- commit phase

문제는 coordinator 장애 시 participant가 in-doubt 상태에 머물 수 있다는 점입니다.

### Consensus

Consensus는 여러 노드가 하나의 값에 합의하는 문제입니다. 대표 알고리즘은 Raft, Paxos, Zab 등이 있습니다.

Consensus가 제공하는 기능:

- leader election
- replicated log
- atomic broadcast
- configuration change

### Coordination Service

ZooKeeper, etcd 같은 시스템은 consensus를 기반으로 lock, leader election, configuration storage 등을 제공합니다.

## 3. 동작 원리

Linearizability는 최신 값을 읽는 것처럼 보이게 만들지만, replication과 network partition 상황에서는 비용이 큽니다. Consensus는 다수 quorum을 통해 하나의 log 순서를 정하고, 각 노드가 같은 순서로 state machine을 실행하게 합니다.

2PC는 atomic commit을 제공하지만 consensus 자체는 아닙니다. Coordinator가 단일 장애점이 될 수 있으며, participant가 결정을 기다리며 block될 수 있습니다.

## 4. 장점과 한계

Consensus는 강력하지만 비쌉니다. 모든 데이터를 consensus로 처리하면 latency와 throughput이 제한됩니다. 따라서 coordination이 반드시 필요한 metadata, lock, leader election 등에 집중적으로 쓰는 경우가 많습니다.

Linearizability도 모든 데이터에 필요한 것은 아닙니다. 사용자 feed, analytics, cache에는 eventual consistency가 충분할 수 있지만, unique constraint, lock, money movement 등에는 강한 보장이 필요합니다.

## 5. 시스템 설계 관점

- 어떤 operation에 linearizability가 필요한가?
- 어떤 상태를 consensus log로 관리할 것인가?
- quorum 장애 시 availability를 포기할 수 있는가?
- 2PC가 필요한가, 아니면 saga/compensation이 적합한가?
- coordination service에 과도한 부하를 주고 있지 않은가?

## 6. 실무/연구 연결점

Kubernetes는 etcd를 통해 cluster state를 강하게 관리합니다. Lakehouse catalog도 table metadata commit, snapshot pointer update, namespace 관리에서 strong consistency가 필요할 수 있습니다.

Iceberg commit은 optimistic concurrency와 atomic metadata pointer update가 중요합니다. Nessie 같은 catalog는 branch/tag/commit 개념을 제공하며 metadata versioning과 consistency 문제를 다룹니다.

## 7. 헷갈리기 쉬운 부분

- Linearizability와 serializability는 다릅니다. 하나는 single-object operation의 최신성 모델에 가깝고, 다른 하나는 transaction isolation입니다.
- 2PC는 consensus가 아닙니다.
- Consensus는 모든 문제의 만능 해결책이 아니라 비용이 큰 coordination primitive입니다.

## 8. 핵심 질문 정리

- 반드시 하나의 최신 값처럼 보여야 하는 데이터는 무엇인가?
- consensus 없이 처리해도 되는 데이터는 무엇인가?
- atomic commit이 필요한 범위는 어디까지인가?
- coordination service 장애 시 시스템은 어떻게 동작하는가?

## 9. 한 문단 요약

9장은 분산 시스템에서 일관된 상태와 결정을 만들기 위한 linearizability, ordering, distributed transaction, consensus를 설명합니다. 강한 consistency는 correctness를 단순하게 하지만 latency와 availability 비용이 있으며, consensus는 leader election과 replicated log 같은 핵심 coordination에는 유용하지만 모든 데이터 경로에 적용하기에는 무겁습니다.

## 10. 설계 체크리스트

- **강한 일관성 대상:** 리더 선출, lock, 권한, 현재 스냅샷 포인터처럼 반드시 최신이어야 하는 데이터를 분리한다.
- **합의 저장소 사용:** etcd/ZooKeeper/Consul을 critical path에 과도하게 넣지 않는다.
- **순서 보장:** 이벤트 처리 순서가 결과에 영향을 주는지 확인한다.
- **Failover:** 리더 변경 중 중복 리더나 오래된 리더가 쓰지 못하게 한다.

## 11. 미니 사례

Iceberg 테이블에서 새 snapshot을 커밋할 때, 두 writer가 동시에 현재 snapshot을 기준으로 새 파일 목록을 만들 수 있다. 최종적으로 하나의 snapshot pointer만 현재 상태가 되어야 하므로 compare-and-swap 또는 catalog transaction이 필요하다. 검색 인덱스 갱신은 늦어도 되지만 snapshot pointer 자체는 강한 정합성이 필요하다.

## 12. 한 줄 결론

모든 데이터를 강하게 일관되게 만들 필요는 없지만, 시스템의 현재 상태를 결정하는 작은 메타데이터는 강한 합의와 원자성이 필요하다.
