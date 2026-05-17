# 05. Replication

> 작성 원칙: 이 문서는 DDIA의 개념을 한국어로 재구성한 학습 노트이며, 원문 문장을 길게 옮기지 않습니다. 핵심 용어는 필요에 따라 English term을 병기합니다.

## 1. 이 장의 핵심 문제

Replication은 같은 데이터를 여러 노드에 복사해 두는 방식입니다. 목적은 availability 향상, read scalability, latency 감소, fault tolerance입니다. 핵심 문제는 “여러 복제본이 있을 때 어떤 순서와 규칙으로 데이터를 전파하고, 일관성 문제를 어떻게 다룰 것인가?”입니다.

## 2. 주요 개념

### Leader-based Replication

하나의 leader가 write를 받고 follower에게 변경을 전달합니다. read는 leader 또는 follower에서 처리할 수 있습니다.

- 장점: write ordering이 비교적 단순함
- 단점: leader 장애 시 failover 필요, replication lag 발생 가능

### Synchronous / Asynchronous Replication

- **Synchronous**: follower 반영을 기다린 뒤 성공 응답. 데이터 손실 가능성은 낮지만 latency와 availability에 불리함
- **Asynchronous**: leader가 먼저 성공 응답. 빠르지만 leader 장애 시 최신 write가 유실될 수 있음

실제로는 일부 follower만 synchronous로 두는 semi-synchronous 방식도 사용됩니다.

### Replication Lag

Follower가 leader보다 늦게 반영되는 현상입니다. 이로 인해 stale read가 발생할 수 있습니다.

대표적인 보장:

- **Read-your-writes consistency**: 사용자가 방금 쓴 데이터는 자신에게 보여야 함
- **Monotonic reads**: 시간이 거꾸로 가는 것처럼 더 오래된 값을 보지 않아야 함
- **Consistent prefix reads**: 원인-결과 순서가 뒤집혀 보이지 않아야 함

### Multi-leader Replication

여러 leader가 write를 받을 수 있습니다. 데이터센터 간 복제나 offline-capable app에 유용하지만 conflict resolution이 어렵습니다.

### Leaderless Replication

Dynamo-style 시스템처럼 특정 leader 없이 여러 replica에 읽고 씁니다. quorum read/write를 사용해 일정 수준의 최신성을 확보합니다.

- **Quorum**: `w + r > n`이면 최신 값을 읽을 가능성이 커짐
- **Read repair**: 읽는 과정에서 오래된 replica를 고침
- **Anti-entropy**: background sync로 replica 차이를 줄임

## 3. 동작 원리

Replication은 write가 발생한 순서와 내용을 다른 replica에 전달하는 문제입니다. Leader 기반에서는 leader의 log를 follower가 따라갑니다. Multi-leader에서는 서로 다른 leader에서 동시에 변경이 생길 수 있어 conflict가 발생합니다. Leaderless에서는 client가 여러 replica에 직접 요청하고 응답 수를 기준으로 성공 여부를 판단합니다.

## 4. 장점과 한계

Replication은 read 확장과 장애 대응에 좋지만 consistency 문제를 만듭니다. 특히 asynchronous replication은 성능과 availability를 얻는 대신 최신성 보장을 약화합니다. Multi-leader와 leaderless 방식은 지리적 분산에 강하지만 conflict 처리 복잡성이 큽니다.

## 5. 시스템 설계 관점

- write가 반드시 한 곳으로 모여야 하는가?
- stale read를 허용할 수 있는가?
- 장애 시 data loss를 어느 정도 허용하는가?
- failover는 자동인가 수동인가?
- conflict resolution 정책은 application-level인가 database-level인가?

## 6. 실무/연구 연결점

Lakehouse metadata catalog에서는 replication lag가 query correctness에 영향을 줄 수 있습니다. 예를 들어 snapshot pointer가 늦게 보이면 사용자가 최신 table state를 보지 못할 수 있습니다. Redis cache를 replica로 쓸 때도 cache invalidation과 catalog consistency를 분리해서 봐야 합니다.

## 7. 헷갈리기 쉬운 부분

- Replica가 많다고 항상 consistency가 좋아지는 것은 아닙니다.
- Asynchronous replication은 정상 상황에서는 잘 동작하지만 장애 시 최신 write 손실 가능성이 있습니다.
- Quorum은 절대적 최신성을 자동 보장하지 않습니다. clock, concurrent write, sloppy quorum 등 변수가 있습니다.

## 8. 핵심 질문 정리

- replication의 주목적은 availability인가 read scale인가 geographic locality인가?
- leader 장애 시 어떤 데이터 손실을 허용하는가?
- 사용자는 stale data를 볼 수 있는가?
- conflict가 발생하면 누가 해결하는가?

## 9. 한 문단 요약

5장은 replication을 통해 availability와 read scalability를 얻는 대신 consistency와 conflict 문제가 생긴다는 점을 설명합니다. Leader-based, multi-leader, leaderless replication은 각각 단순성, 지리적 분산, 장애 허용성에서 다른 trade-off를 가지며, replication lag와 conflict resolution을 명확히 설계해야 합니다.
