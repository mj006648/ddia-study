# 11. Stream Processing

> 작성 원칙: 이 문서는 DDIA의 개념을 한국어로 재구성한 학습 노트이며, 원문 문장을 길게 옮기지 않습니다. 핵심 용어는 필요에 따라 English term을 병기합니다.

## 1. 이 장의 핵심 문제

Stream processing은 unbounded event stream을 지속적으로 처리하는 방식입니다. 핵심 문제는 “계속 들어오는 이벤트에서 지연을 낮추면서도 정확하고 재현 가능한 결과를 어떻게 만들 것인가?”입니다.

## 2. 주요 개념

### Event Stream

Event는 특정 시점에 발생한 사실을 나타냅니다. Stream은 이런 event의 append-only log로 볼 수 있습니다.

### Message Broker / Log

- 전통적 message broker: consumer에게 메시지를 전달하고 제거
- log-based broker: 메시지를 일정 기간 보관하고 consumer offset으로 읽기 위치 관리

Kafka 같은 시스템은 event log를 중심으로 stream processing을 구성합니다.

### Event Time과 Processing Time

- **Event time**: 사건이 실제 발생한 시간
- **Processing time**: 시스템이 event를 처리한 시간

늦게 도착하는 event를 다루려면 event time과 watermark가 중요합니다.

### Windowing

무한 stream을 계산 가능한 단위로 자르기 위해 window를 사용합니다.

- tumbling window
- hopping/sliding window
- session window

### Stream Joins

- stream-stream join
- stream-table join
- table-table join에 가까운 materialized view

### Fault Tolerance

Checkpoint, offset, state snapshot을 사용해 장애 후 복구합니다. Exactly-once semantics는 실제로는 source, processing state, sink가 함께 맞아야 합니다.

## 3. 동작 원리

Stream processor는 event를 읽고 operator graph를 따라 변환하며, 필요한 경우 state를 유지합니다. Aggregation, join, deduplication은 stateful operation입니다. 장애가 발생하면 checkpoint로 돌아가 다시 처리합니다.

Stream은 batch와 달리 입력이 끝나지 않습니다. 따라서 결과는 계속 갱신되며, late event가 도착하면 이미 낸 결과를 수정해야 할 수 있습니다.

## 4. 장점과 한계

Stream processing은 low latency와 near-real-time view에 강합니다. 그러나 state management, ordering, late event, exactly-once 처리 때문에 batch보다 운영과 reasoning이 어렵습니다.

## 5. 시스템 설계 관점

- event time 기준 결과가 필요한가 processing time으로 충분한가?
- late event를 얼마나 기다릴 것인가?
- duplicate event를 어떻게 제거할 것인가?
- sink에 idempotent write가 가능한가?
- stream state size가 얼마나 커지는가?

## 6. 실무/연구 연결점

Lakehouse ingestion에서는 stream processing이 raw event를 Bronze table로 적재하고, metadata를 추출해 catalog/cache/vector index를 갱신하는 데 사용될 수 있습니다.

Redis/Milvus/PostgreSQL/Iceberg를 함께 갱신하는 경우 exactly-once를 단일 시스템처럼 보장하기 어렵습니다. 따라서 event ID, idempotent upsert, checkpoint, outbox pattern, replay 전략이 필요합니다.

## 7. 헷갈리기 쉬운 부분

- Exactly-once는 “어떤 코드도 한 번만 실행된다”가 아니라 결과가 한 번 처리된 것처럼 보이게 하는 end-to-end property입니다.
- Stream은 batch의 반대가 아니라 unbounded data를 다루는 처리 모델입니다.
- Event time을 쓰지 않으면 지연 도착 데이터가 결과를 왜곡할 수 있습니다.

## 8. 핵심 질문 정리

- 이벤트의 고유 ID와 순서는 어떻게 정의되는가?
- late event와 duplicate event 정책은 무엇인가?
- stateful operator의 상태는 어디에 저장되는가?
- 장애 후 replay해도 sink 결과가 중복되지 않는가?

## 9. 한 문단 요약

11장은 stream processing을 지속적으로 들어오는 event log를 처리하는 방식으로 설명합니다. Event time, windowing, joins, checkpoint, exactly-once semantics가 핵심이며, 낮은 지연을 얻는 대신 ordering, late event, state management, idempotent sink 같은 복잡성을 명확히 설계해야 합니다.
