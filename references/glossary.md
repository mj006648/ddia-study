# DDIA Glossary

## Reliability
장애나 결함이 있어도 시스템이 기대 가능한 서비스를 계속 제공하는 성질.

## Scalability
부하가 증가해도 성능을 유지하거나 확장할 수 있는 능력. 절대적으로 빠르다는 뜻이 아니라 부하 변화에 대응하는 구조를 의미함.

## Maintainability
시간이 지나도 시스템을 이해, 변경, 운영할 수 있는 성질.

## Tail Latency
평균이 아니라 상위 백분위 응답시간. p95, p99 등이 사용됨.

## System of Record
데이터의 진실 원천. 파생 데이터가 잘못되면 이 데이터를 기준으로 다시 생성해야 함.

## Derived Data
원천 데이터에서 만들어진 캐시, 인덱스, materialized view, 분석 테이블, 검색 벡터 등.

## LSM Tree
쓰기 성능을 높이기 위해 메모리와 정렬된 디스크 파일을 활용하고 compaction으로 병합하는 저장 구조.

## B-Tree
정렬된 페이지 구조로 point lookup과 range scan에 강한 인덱스/저장 구조.

## Replication Lag
리더에 반영된 쓰기가 팔로워에 도달하기까지의 지연.

## Partitioning
데이터를 여러 노드나 파일에 나누어 저장하는 방식.

## Hot Partition
특정 파티션에 부하가 집중되는 현상.

## Transaction
여러 작업을 하나의 논리적 단위로 묶어 atomicity와 isolation을 제공하는 추상화.

## Snapshot Isolation
트랜잭션이 시작 시점의 일관된 스냅샷을 읽는 격리 수준. write skew는 막지 못할 수 있음.

## Linearizability
분산 시스템이 하나의 최신 복사본처럼 보이는 강한 일관성 성질.

## Consensus
여러 노드가 장애 가능성 속에서도 하나의 결정이나 로그 순서에 합의하는 문제.

## Idempotency
같은 작업을 여러 번 실행해도 결과가 한 번 실행한 것과 같게 되는 성질.

## Batch Processing
유한한 대량 데이터를 읽고 변환해 결과를 만드는 처리 방식.

## Stream Processing
끝이 없는 이벤트 흐름을 지속적으로 처리하는 방식.

## Event Time
이벤트가 실제 발생한 시간.

## Processing Time
시스템이 이벤트를 처리한 시간.

## Watermark
스트림 처리에서 어느 시점까지의 이벤트가 대체로 도착했다고 판단하기 위한 기준.

## Lineage
데이터가 어떤 입력, 코드, 작업, 파라미터를 통해 생성되었는지의 계보.
