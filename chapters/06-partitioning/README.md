# 06. Partitioning

> 작성 원칙: 이 문서는 DDIA의 개념을 한국어로 재구성한 학습 노트이며, 원문 문장을 길게 옮기지 않습니다. 핵심 용어는 필요에 따라 English term을 병기합니다.

## 1. 이 장의 핵심 문제

Partitioning은 큰 데이터를 여러 노드에 나누어 저장하는 방식입니다. 목적은 dataset size와 query load를 분산하는 것입니다. 핵심 문제는 “데이터와 요청이 특정 노드에 몰리지 않도록 어떻게 나눌 것인가?”입니다.

## 2. 주요 개념

### Key-range Partitioning

key의 범위를 기준으로 partition을 나눕니다. range query에 유리하지만 특정 범위에 write가 몰리면 hot spot이 생길 수 있습니다.

### Hash Partitioning

key의 hash 값을 기준으로 나눕니다. 부하 분산에 유리하지만 range query에는 불리합니다.

### Skew와 Hot Spot

데이터나 요청이 균등하지 않게 특정 partition에 몰리는 현상입니다. celebrity user, sequential key, 특정 시간대 데이터 등이 원인이 될 수 있습니다.

### Secondary Index

Partitioned database에서 secondary index는 어렵습니다.

- **Local index**: partition별 index. write는 쉽지만 query가 scatter/gather가 될 수 있음
- **Global index**: 전체 데이터 기준 index. query는 편하지만 index 유지가 복잡함

### Rebalancing

노드 추가/제거 또는 부하 변화에 따라 partition을 재배치하는 과정입니다. rebalancing 중에도 서비스가 계속되어야 하므로 운영 복잡성이 큽니다.

## 3. 동작 원리

Partitioning은 key를 partition에 매핑하는 함수가 필요합니다. 단순 modulo 방식은 노드 수가 바뀔 때 대부분의 key 위치가 바뀌므로 비효율적입니다. consistent hashing이나 고정된 virtual partition을 사용하면 이동량을 줄일 수 있습니다.

Query routing도 중요합니다. Client가 partition 위치를 알 수도 있고, coordinator node가 요청을 받아 적절한 partition으로 전달할 수도 있습니다.

## 4. 장점과 한계

Partitioning은 scale-out의 핵심이지만 모든 문제를 해결하지는 않습니다. 잘못된 partition key는 hot spot을 만들고, multi-partition transaction과 join은 비싸집니다. 또한 운영 중 rebalancing과 monitoring이 필요합니다.

## 5. 시스템 설계 관점

- partition key는 query pattern과 write distribution을 함께 고려해야 합니다.
- range query가 중요한지 point lookup이 중요한지 확인해야 합니다.
- secondary index가 필요한 query를 미리 파악해야 합니다.
- rebalancing 중 성능 저하와 consistency 영향을 고려해야 합니다.

## 6. 실무/연구 연결점

Lakehouse에서는 partitioning이 query pruning과 밀접합니다. Iceberg는 hidden partitioning과 partition evolution을 제공해 physical layout 변경을 관리합니다. 그러나 partition field를 잘못 고르면 small file, skew, metadata 폭증이 발생할 수 있습니다.

분산 metadata acceleration에서는 dataset ID, table ID, namespace, embedding vector 등 어떤 key로 Redis/Milvus를 나눌지도 중요한 partitioning 문제입니다.

## 7. 헷갈리기 쉬운 부분

- Partitioning은 replication과 다릅니다. partitioning은 데이터를 나누는 것이고, replication은 복사하는 것입니다.
- Hash partitioning은 균등 분산에는 좋지만 범위 질의에는 불리합니다.
- Partition 수와 node 수를 같게 잡으면 rebalancing이 어려울 수 있습니다.

## 8. 핵심 질문 정리

- 어떤 key가 가장 균등하게 요청을 분산하는가?
- hot key가 생겼을 때 어떻게 완화할 것인가?
- secondary index query는 얼마나 자주 발생하는가?
- partition 이동 중에도 service availability를 유지할 수 있는가?

## 9. 한 문단 요약

6장은 partitioning을 통해 대규모 데이터와 부하를 여러 노드에 분산하는 방법을 설명합니다. Key-range와 hash partitioning은 각각 range query와 load balancing에서 장단점이 있으며, skew, hot spot, secondary index, rebalancing을 고려하지 않으면 scale-out이 오히려 복잡성과 병목을 만들 수 있습니다.

## 10. 설계 체크리스트

- **파티션 키 후보:** tenant, dataset_id, time, region, domain, hash key를 비교한다.
- **질의 패턴:** 가장 흔한 query가 단일 파티션에서 끝나는지 확인한다.
- **Hot key:** 특정 대형 사용자, 인기 데이터셋, 최신 시간대에 부하가 몰리는지 확인한다.
- **Rebalancing:** 노드 추가 시 데이터 이동량과 서비스 영향도를 예측한다.

## 11. 미니 사례

시간 기준 파티셔닝은 “최근 1일 데이터 조회”에는 좋지만 모든 쓰기가 최신 파티션에 몰릴 수 있다. 반대로 dataset_id 해시 파티셔닝은 쓰기를 분산하지만 기간별 분석 스캔이 여러 파티션으로 흩어진다. 분석 시스템에서는 시간+도메인 파티션을 쓰고, 메타데이터 조회 시스템에서는 dataset_id 기반 인덱스를 별도로 두는 식의 분리가 필요하다.

## 12. 한 줄 결론

파티셔닝은 데이터를 나누는 일이 아니라, 부하와 질의를 어떤 비용 구조로 만들 것인지 결정하는 일이다.
