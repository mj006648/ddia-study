# 10. Batch Processing

> 작성 원칙: 이 문서는 DDIA의 개념을 한국어로 재구성한 학습 노트이며, 원문 문장을 길게 옮기지 않습니다. 핵심 용어는 필요에 따라 English term을 병기합니다.

## 1. 이 장의 핵심 문제

Batch processing은 bounded data set을 입력으로 받아 결과를 계산하는 방식입니다. 핵심 문제는 “대규모 데이터를 안정적으로 읽고, 변환하고, 다시 계산할 수 있는 구조를 어떻게 만들 것인가?”입니다.

## 2. 주요 개념

### System of Record와 Derived Data

원본 데이터(system of record)에서 index, cache, materialized view, report 같은 파생 데이터(derived data)를 만듭니다. Batch job은 이런 파생 데이터를 재계산하는 데 적합합니다.

### Unix Philosophy

작은 도구가 text stream을 통해 연결되는 방식은 batch processing의 기본 정신과 닮았습니다.

- 입력을 읽음
- 변환함
- 출력을 씀
- 중간 상태를 최소화함

### MapReduce

MapReduce는 대규모 데이터를 분산 처리하기 위한 모델입니다.

- **Map**: 입력 record를 key-value로 변환
- **Shuffle**: 같은 key를 같은 reducer로 모음
- **Reduce**: key별 집계/처리

### Join Strategies

Batch에서 join은 큰 비용을 만듭니다.

- sort-merge join
- broadcast hash join
- partitioned hash join

### Workflow

Batch job은 보통 여러 단계로 구성됩니다. Scheduler가 dependency, retry, backfill, monitoring을 관리합니다.

## 3. 동작 원리

Batch job은 입력 데이터가 고정되어 있다는 점에서 재현성이 높습니다. 실패하면 같은 입력으로 다시 실행할 수 있고, 로직을 고친 뒤 전체 데이터를 backfill할 수 있습니다.

MapReduce 계열 시스템은 데이터를 partition으로 나누고 각 worker가 병렬 처리합니다. Shuffle은 분산 처리에서 가장 비싼 단계 중 하나입니다.

## 4. 장점과 한계

Batch processing은 throughput과 재처리에 강합니다. 그러나 결과가 나오기까지 시간이 걸리므로 low-latency 요구에는 맞지 않습니다. 또한 workflow dependency가 복잡해지면 운영 관리가 중요해집니다.

## 5. 시스템 설계 관점

- 입력 데이터는 immutable하게 보존되는가?
- job은 idempotent하게 재실행 가능한가?
- 실패 시 partial output을 어떻게 처리하는가?
- backfill 비용은 감당 가능한가?
- shuffle과 skew를 어떻게 줄일 것인가?

## 6. 실무/연구 연결점

Lakehouse는 batch processing과 잘 맞습니다. Object storage에 쌓인 Parquet/Iceberg 데이터를 Spark, Trino, Flink batch mode 등으로 처리해 Silver/Gold 데이터를 만들 수 있습니다.

Trident Lakehouse의 Accumulation Pipeline은 raw dataset을 Bronze/Silver로 정리하고, Staging/Serving Pipeline은 분석 목적에 맞게 derived data를 구성하는 batch/interactive workflow로 볼 수 있습니다.

## 7. 헷갈리기 쉬운 부분

- Batch는 낡은 방식이 아니라 재현성과 backfill에 강한 처리 모델입니다.
- MapReduce는 특정 구현이면서 동시에 사고방식입니다.
- Derived data는 원본이 아니라 다시 만들 수 있는 결과물로 관리하는 것이 중요합니다.

## 8. 핵심 질문 정리

- 이 결과는 원본에서 다시 계산 가능한가?
- job 실패 후 안전하게 retry할 수 있는가?
- full recomputation과 incremental update 중 무엇이 적합한가?
- workflow dependency가 명확히 관리되는가?

## 9. 한 문단 요약

10장은 batch processing을 bounded data set에 대한 안정적이고 재현 가능한 처리 방식으로 설명합니다. MapReduce, shuffle, join, workflow scheduler는 대규모 derived data를 만드는 핵심 구성요소이며, batch는 latency보다 correctness, throughput, replay, backfill이 중요한 작업에 적합합니다.
