# 04. Encoding and Evolution

> 작성 원칙: 이 문서는 DDIA의 개념을 한국어로 재구성한 학습 노트이며, 원문 문장을 길게 옮기지 않습니다. 핵심 용어는 필요에 따라 English term을 병기합니다.

## 1. 이 장의 핵심 문제

시스템은 시간이 지나며 변합니다. 필드가 추가되고, 의미가 바뀌고, 서비스가 분리되고, 여러 버전의 코드가 동시에 실행됩니다. 이 장은 데이터 encoding format과 schema evolution 문제를 다룹니다. 핵심은 “시스템을 멈추지 않고 데이터 구조와 서비스를 어떻게 진화시킬 것인가?”입니다.

## 2. 주요 개념

### Encoding / Decoding

- **Encoding**: 메모리의 데이터 구조를 저장·전송 가능한 byte sequence로 바꾸는 것
- **Decoding**: byte sequence를 다시 애플리케이션 데이터 구조로 바꾸는 것

Serialization, marshalling이라는 용어도 비슷한 맥락에서 사용됩니다.

### JSON, XML, CSV

텍스트 기반 포맷은 사람이 읽기 쉽고 언어 독립적입니다. 하지만 type 표현이 약하거나 모호할 수 있고, binary format보다 공간 효율이 낮을 수 있습니다.

### Protocol Buffers, Thrift, Avro

Binary encoding format은 schema를 바탕으로 compact하고 빠른 encoding을 제공합니다.

- Protocol Buffers/Thrift: field tag 중심
- Avro: writer schema와 reader schema를 함께 고려

### Schema Evolution

시스템 업그레이드는 한 번에 일어나지 않습니다. rolling upgrade 중에는 새 코드와 옛 코드가 동시에 존재합니다. 따라서 compatibility가 중요합니다.

- **Backward compatibility**: 새 코드가 옛 데이터도 읽을 수 있음
- **Forward compatibility**: 옛 코드가 새 데이터도 읽을 수 있음

### Dataflow

데이터는 여러 경로로 이동합니다.

- Database에 저장
- Service 간 API 호출
- Message queue/event stream 전달
- File export/import

각 dataflow마다 compatibility 조건이 조금씩 다릅니다.

## 3. 동작 원리

스키마 변경은 보통 다음 방식으로 안전하게 진행합니다.

1. 새 필드를 optional/default로 추가
2. 읽는 쪽이 새/옛 데이터 모두 처리 가능하게 변경
3. 쓰는 쪽을 새 형식으로 변경
4. 충분한 시간이 지난 뒤 옛 필드 제거

이 순서를 지키지 않으면 rolling deployment 중 일부 서비스가 데이터를 읽지 못하는 문제가 생길 수 있습니다.

서비스 간 통신에서는 REST/JSON, RPC, message broker 등이 사용됩니다. 동기 RPC는 호출자가 응답을 기다리므로 실패와 timeout 처리가 중요하고, 비동기 message는 생산자와 소비자의 version 차이를 견딜 수 있어야 합니다.

## 4. 장점과 한계

Schema를 명확히 두면 데이터 품질과 compatibility 관리가 쉬워집니다. 반면 schema-less처럼 보이는 시스템은 초기 개발은 빠를 수 있지만, 장기적으로는 애플리케이션 코드에 암묵적 schema가 흩어질 수 있습니다.

Binary format은 효율적이지만 디버깅과 운영 편의성은 텍스트 포맷보다 떨어질 수 있습니다. 따라서 내부 고성능 통신과 외부 API에서는 다른 포맷을 선택할 수도 있습니다.

## 5. 시스템 설계 관점

스키마 진화 설계에서는 다음을 고려해야 합니다.

- rolling upgrade가 가능한가?
- 옛 버전 consumer가 새 message를 받아도 깨지지 않는가?
- default value와 optional field 정책은 명확한가?
- schema registry가 필요한가?
- event log에 오래된 message가 남아 있어도 재처리 가능한가?

특히 event-driven architecture에서는 과거 이벤트를 다시 replay할 수 있으므로 오래된 schema를 읽는 능력이 중요합니다.

## 6. 실무/연구 연결점

Lakehouse에서는 schema evolution이 핵심 기능입니다. Iceberg는 table schema, partition spec, snapshot metadata를 관리하면서 데이터 파일이 바뀌어도 일관된 table view를 제공합니다.

- Schema evolution: column add/drop/rename/type promotion
- Metadata evolution: snapshot과 manifest 구조의 변경
- API evolution: catalog service, query engine, portal 간 contract 유지
- Event evolution: ingestion pipeline에서 schema 변경 감지

Trident Lakehouse에서는 Accumulation Pipeline과 Staging/Serving Pipeline 사이에서 metadata schema가 바뀌어도 Redis/Milvus/PostgreSQL/Iceberg 계층이 호환되도록 versioning이 필요합니다.

## 7. 헷갈리기 쉬운 부분

- JSON을 쓴다고 schema 문제가 사라지는 것은 아닙니다.
- Backward compatibility와 forward compatibility는 방향이 다릅니다.
- Database schema migration과 API schema evolution은 연결되어 있지만 같은 문제는 아닙니다.
- Event log는 오래된 데이터가 계속 남기 때문에 compatibility 요구가 더 강할 수 있습니다.

## 8. 핵심 질문 정리

- 새 필드를 추가하면 옛 consumer는 어떻게 동작하는가?
- 스키마 변경이 rolling deployment와 충돌하지 않는가?
- 장기 보관된 데이터를 나중에 읽을 수 있는가?
- schema registry 또는 contract test가 필요한가?
- 데이터 포맷 선택이 운영 디버깅에 어떤 영향을 주는가?

## 9. 한 문단 요약

4장은 데이터 시스템이 시간이 지나며 변한다는 전제에서 encoding format과 schema evolution을 설명합니다. 좋은 시스템은 새 코드와 옛 코드가 동시에 존재하는 상황을 고려해 backward/forward compatibility를 설계하고, 데이터베이스·서비스·메시지 큐·파일을 지나는 dataflow마다 안전한 진화 전략을 갖춰야 합니다.

## 10. 설계 체크리스트

- **필드 추가:** 새 필드는 optional/default를 둔다.
- **필드 삭제:** 즉시 삭제하지 말고 deprecated 기간을 둔다.
- **의미 변경 금지:** 같은 필드명에 다른 의미를 넣는 변경은 가장 위험하다.
- **스키마 레지스트리:** 이벤트나 공유 데이터 포맷은 버전과 호환성 검사를 둔다.

## 11. 미니 사례

처음에는 `size` 필드가 byte 단위였는데, 나중에 사람이 읽기 좋게 `MB` 단위로 바꾸면 과거 소비자는 잘못된 계산을 한다. 이런 경우 기존 필드를 바꾸지 말고 `size_bytes`, `size_human`처럼 의미를 분리해야 한다. 스키마 진화의 핵심은 구조뿐 아니라 필드 의미의 호환성이다.

## 12. 한 줄 결론

스키마는 코드보다 오래 살며, 좋은 스키마 진화 전략은 구버전 데이터·구버전 코드·신버전 코드가 동시에 존재한다는 현실을 인정한다.
