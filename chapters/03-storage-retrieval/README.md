# 03. Storage and Retrieval

> 작성 원칙: 이 문서는 DDIA의 개념을 한국어로 재구성한 학습 노트이며, 원문 문장을 길게 옮기지 않습니다. 핵심 용어는 필요에 따라 English term을 병기합니다.

## 1. 이 장의 핵심 문제

데이터베이스는 데이터를 저장하고 나중에 효율적으로 찾기 위한 시스템입니다. 이 장은 storage engine이 내부적으로 어떻게 write와 read를 처리하는지 설명합니다. 핵심 문제는 “쓰기 성능, 읽기 성능, 저장 공간, 복구 가능성 사이에서 어떤 구조를 선택할 것인가?”입니다.

## 2. 주요 개념

### Index

Index는 특정 key로 데이터를 빠르게 찾기 위한 부가 구조입니다. 인덱스가 많을수록 read는 빨라질 수 있지만 write 비용과 저장 공간이 증가합니다. 따라서 모든 필드에 인덱스를 만드는 것은 좋은 전략이 아닙니다.

### Hash Index

Hash index는 key를 value의 위치에 매핑합니다. 단일 key lookup에는 빠르지만 range query에는 약합니다. 메모리에 hash map을 유지하고 디스크에는 append-only log를 두는 방식이 대표적입니다.

### SSTable과 LSM-tree

SSTable(Sorted String Table)은 key로 정렬된 segment file입니다. Memtable에 쓰고, 일정 크기가 되면 디스크에 정렬된 파일로 flush합니다. 이후 background compaction으로 여러 SSTable을 병합합니다.

LSM-tree(Log-Structured Merge Tree)는 이런 구조를 체계화한 storage engine 방식입니다.

장점:

- sequential write 중심이라 write throughput이 좋음
- compaction으로 오래된 값을 정리 가능
- compression에 유리

단점:

- compaction 비용이 큼
- read 시 여러 level/file을 확인해야 할 수 있음
- write amplification이 발생할 수 있음

### B-tree

B-tree는 page 단위로 디스크에 저장되는 균형 트리 구조입니다. 대부분의 relational DB에서 널리 사용됩니다.

장점:

- point lookup과 range query에 모두 강함
- 업데이트가 제자리(page update) 중심이라 read path가 안정적
- transaction과 함께 성숙한 구현이 많음

단점:

- random write가 많을 수 있음
- page split, fragmentation, WAL 관리가 필요함

### OLTP와 OLAP

- **OLTP(Online Transaction Processing)**: 짧고 빈번한 read/write, 사용자 요청 처리 중심
- **OLAP(Online Analytical Processing)**: 큰 범위의 scan, aggregation, reporting 중심

OLTP는 row-oriented storage가 유리한 경우가 많고, OLAP는 column-oriented storage가 유리합니다.

### Column-oriented Storage

Column store는 같은 column의 값을 연속적으로 저장합니다. 분석 질의가 일부 column만 읽는 경우 I/O를 줄일 수 있고, 같은 type의 값이 모여 있어 compression 효율이 좋습니다.

## 3. 동작 원리

쓰기 중심 storage engine은 보통 append-only log를 활용합니다. 데이터 변경을 순차적으로 기록하면 random write보다 빠르고 crash recovery에도 유리합니다. 그러나 log가 계속 커지기 때문에 segment와 compaction이 필요합니다.

읽기 중심 구조에서는 B-tree처럼 key 순서를 유지하는 index가 중요합니다. range scan을 빠르게 수행할 수 있고, 업데이트된 page를 WAL과 함께 관리해 durability를 보장합니다.

분석 시스템에서는 tuple 단위보다 column 단위 접근이 중요합니다. 예를 들어 전체 row가 100개 column을 갖고 있어도 query가 3개 column만 사용한다면 column store가 훨씬 적은 데이터를 읽습니다.

## 4. 장점과 한계

LSM-tree는 write-heavy workload에 강하고, B-tree는 범용 OLTP workload에 강합니다. Column store는 analytical scan에 강하지만 단건 업데이트에는 적합하지 않을 수 있습니다.

중요한 점은 storage engine 선택이 workload에 따라 달라진다는 것입니다. 데이터 양, read/write 비율, range query 여부, update pattern, latency requirement를 함께 봐야 합니다.

## 5. 시스템 설계 관점

저장소를 고를 때 다음 질문이 필요합니다.

- write-heavy인가 read-heavy인가?
- point lookup이 많은가 range scan이 많은가?
- 데이터가 자주 update되는가 append-only인가?
- 분석 query가 많은가 transaction query가 많은가?
- tail latency보다 throughput이 중요한가?
- compression과 storage cost가 중요한가?

또한 index는 read optimization이지만 write penalty를 만든다는 점을 항상 고려해야 합니다.

## 6. 실무/연구 연결점

Iceberg 같은 table format은 data file, manifest, metadata file을 통해 대규모 analytical table을 관리합니다. 이는 OLAP 관점에서 columnar file format과 metadata pruning이 중요하다는 점과 연결됩니다.

- Parquet/ORC: column-oriented storage
- Manifest/statistics: file pruning과 query planning
- Metadata cache: catalog lookup latency 감소
- Compaction: small file problem 완화

Redis는 in-memory key-value access, Milvus는 vector index, PostgreSQL은 B-tree/transactional catalog, Iceberg는 columnar analytical storage로 볼 수 있습니다. 각 저장소의 내부 구조가 다르기 때문에 역할을 분리하는 것이 중요합니다.

## 7. 헷갈리기 쉬운 부분

- Index는 공짜가 아닙니다. read를 빠르게 만드는 대신 write와 storage cost를 증가시킵니다.
- Log-structured는 단순 로그 파일이 아니라 compaction과 indexing이 함께 있어야 효율적입니다.
- B-tree는 오래된 구조지만 여전히 강력한 범용 storage engine입니다.
- Column store는 row를 저장하지 않는다는 뜻이 아니라 물리 배치를 column 중심으로 한다는 뜻입니다.

## 8. 핵심 질문 정리

- 이 workload의 read/write 비율은 어떻게 되는가?
- 가장 중요한 query는 point lookup인가 scan인가?
- compaction 비용을 감당할 수 있는가?
- index를 추가했을 때 write path가 얼마나 느려지는가?
- 분석 질의에서 column pruning과 compression 효과를 얻을 수 있는가?

## 9. 한 문단 요약

3장은 데이터베이스의 저장 엔진이 read/write trade-off를 어떻게 다루는지 설명합니다. Hash index, LSM-tree, B-tree, column store는 각각 다른 workload에 최적화되어 있으며, 좋은 storage design은 단순히 빠른 DB를 고르는 것이 아니라 query pattern, update pattern, indexing cost, compaction, compression, OLTP/OLAP 요구를 함께 고려하는 것입니다.
