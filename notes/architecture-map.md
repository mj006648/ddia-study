# DDIA Architecture Map

DDIA 전체를 하나의 데이터 플랫폼 아키텍처로 보면 다음 흐름으로 연결된다.

## 1. 애플리케이션 품질 속성

Chapter 1은 전체 기준을 제공한다.

- 신뢰성: 장애가 있어도 데이터와 서비스가 기대 가능한 상태를 유지함
- 확장성: 부하 증가에 대응할 수 있는 구조를 가짐
- 유지보수성: 시간이 지나도 변경·운영·이해가 가능함

이 세 기준은 나머지 모든 장의 판단 기준이다.

## 2. 데이터 표현 계층

Chapter 2와 4는 데이터가 어떻게 표현되고 진화하는지 다룬다.

- 데이터 모델: 관계형, 문서형, 그래프형
- 질의 언어: 명령형보다 선언형이 최적화와 유지보수에 유리함
- 인코딩: JSON/Avro/Protobuf 등 저장·전송 형식
- 스키마 진화: 오래된 데이터와 새로운 코드의 공존

설계 질문:

- 데이터의 자연스러운 경계는 row, document, graph 중 무엇인가?
- 어떤 스키마 변경을 무중단으로 허용해야 하는가?

## 3. 저장·검색 계층

Chapter 3은 물리 저장과 인덱스를 다룬다.

- B-Tree: 범위 질의와 일반 OLTP에 강함
- LSM Tree: 쓰기 많은 워크로드에 강함
- Column Store: 분석 스캔과 압축에 강함
- Index: 읽기를 빠르게 하지만 쓰기 비용을 늘림

설계 질문:

- 주된 접근 패턴은 point lookup, range scan, full scan 중 무엇인가?
- 데이터 파일과 메타데이터는 같은 저장 엔진을 써야 하는가?

## 4. 분산 데이터 계층

Chapter 5~9는 데이터가 여러 노드에 있을 때의 문제를 다룬다.

- 복제: 가용성과 읽기 확장을 얻지만 일관성 지연이 생김
- 파티셔닝: 처리량을 높이지만 cross-partition 작업이 어려워짐
- 트랜잭션: 동시성과 장애를 숨기는 추상화
- 분산 시스템 문제: 네트워크, 시계, 부분 장애, 재시도
- 합의: 여러 노드가 하나의 결정을 내리는 방법

설계 질문:

- 어떤 데이터는 강한 일관성이 필요하고, 어떤 데이터는 지연을 허용하는가?
- 장애 시 재시도해도 안전한 작업인가?
- 파티션 키가 hot key를 만들지 않는가?

## 5. 파생 데이터 계층

Chapter 10~12는 원천 데이터에서 파생 데이터를 만드는 흐름을 다룬다.

- Batch: 유한 데이터 재처리와 대량 변환
- Stream: 이벤트의 지속 처리와 낮은 지연시간
- Derived Data: 검색 인덱스, 캐시, materialized view, Gold table
- Dataflow: 데이터 이동과 변환을 명시적으로 관리

설계 질문:

- 무엇이 system of record이고 무엇이 derived data인가?
- 파생 데이터는 언제든 재생성 가능한가?
- batch와 stream 결과의 차이를 어떻게 맞출 것인가?

## 6. Lakehouse/Workflow와의 연결

DDIA 관점에서 lakehouse는 단순 저장소가 아니라 다음 구성의 조합이다.

- 원천 데이터 보존: append-only, immutable snapshot, Bronze layer
- 데이터 정제와 파생: Silver/Gold layer, batch/stream processing
- 메타데이터 관리: schema, snapshot, location, permission, lineage
- 검색/가속: cache, vector index, metadata index
- 분산 운영: replication, partitioning, transaction, consensus
- 워크플로우 실행: dataflow, retry, idempotency, reproducibility

따라서 좋은 lakehouse 설계는 “어떤 엔진을 쓰는가”보다 “원천과 파생, 메타데이터와 데이터 파일, 강한 일관성과 지연 허용 영역을 어떻게 나누는가”로 설명해야 한다.
