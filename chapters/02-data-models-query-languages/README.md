# 02. Data Models and Query Languages

> 작성 원칙: 이 문서는 DDIA의 개념을 한국어로 재구성한 학습 노트이며, 원문 문장을 길게 옮기지 않습니다. 핵심 용어는 필요에 따라 English term을 병기합니다.

## 1. 이 장의 핵심 문제

데이터 모델(data model)은 애플리케이션이 현실 세계를 어떤 구조로 바라보고 저장할지 결정합니다. 이 장은 relational model, document model, graph model과 query language의 차이를 다룹니다. 핵심 문제는 “데이터를 어떤 형태로 표현해야 애플리케이션의 질의와 변경에 잘 맞는가?”입니다.

데이터 모델은 단순 저장 형식이 아닙니다. 개발자가 문제를 생각하는 방식, 쿼리 작성 방식, 스키마 변경 방식, 확장 방식까지 영향을 줍니다.

## 2. 주요 개념

### Relational Model

Relational model은 데이터를 relation, 즉 table의 집합으로 표현합니다. SQL을 통해 선언형(declarative)으로 질의하며, 데이터 중복을 줄이기 위해 normalization을 사용합니다.

장점은 다음과 같습니다.

- join을 통해 다양한 관계를 유연하게 표현 가능
- 스키마와 제약조건을 명확히 정의 가능
- 질의 최적화(query optimizer)가 실행 계획을 선택 가능

단점은 객체지향 애플리케이션 구조와 테이블 구조 사이에 impedance mismatch가 생길 수 있다는 점입니다.

### Document Model

Document model은 JSON, BSON, XML 같은 계층적 문서로 데이터를 저장합니다. 한 객체와 그 하위 데이터를 한 문서에 모을 수 있어 지역성(locality)이 좋습니다.

적합한 경우는 다음과 같습니다.

- 데이터가 자연스럽게 tree 구조인 경우
- 보통 한 번에 전체 document를 읽고 쓰는 경우
- join이 많지 않고 aggregate 단위가 명확한 경우

하지만 many-to-many 관계가 많아지면 중복과 갱신 문제가 커질 수 있습니다.

### Graph Model

Graph model은 vertex(node)와 edge로 데이터를 표현합니다. 관계 자체가 중요한 데이터에 적합합니다.

예시는 다음과 같습니다.

- social graph
- recommendation
- fraud detection
- knowledge graph
- network topology

Graph model에서는 관계 탐색(traversal)이 핵심이며, relational/document model보다 복잡한 연결 구조를 자연스럽게 표현할 수 있습니다.

### Declarative Query

SQL 같은 declarative query language는 “어떻게 계산할지”보다 “무엇을 얻고 싶은지”를 표현합니다. 실행 방식은 optimizer가 결정할 수 있으므로, 병렬화와 최적화에 유리합니다.

반대로 imperative 방식은 처리 절차를 직접 작성합니다. 세밀한 제어가 가능하지만, 시스템이 자동으로 최적화하기 어렵습니다.

## 3. 동작 원리

애플리케이션은 보통 현실의 entity와 relationship을 내부 모델로 바꿉니다. 이때 어떤 관계를 embedding할지, reference로 둘지, join으로 처리할지 결정해야 합니다.

예를 들어 사용자 프로필과 이력서 정보를 생각하면, 이력서의 경력 목록은 한 사용자 안에 포함된 document로 저장하기 쉽습니다. 그러나 회사, 학교, 지역, 추천 관계처럼 여러 객체가 서로 공유하는 데이터는 reference나 join이 필요합니다.

즉 데이터 모델 선택은 다음의 균형입니다.

- 읽기 지역성 vs 중복 최소화
- 단순한 조회 vs 복잡한 관계 탐색
- 스키마 유연성 vs 데이터 품질 보장
- 애플리케이션 로직 단순화 vs DB 질의 능력 활용

## 4. 장점과 한계

Relational model은 범용성과 질의 유연성이 강합니다. Document model은 aggregate 중심 접근에 강하고, Graph model은 연결 관계 탐색에 강합니다. 어느 하나가 항상 우월하지 않으며 데이터의 shape와 access pattern에 따라 달라집니다.

특히 NoSQL의 등장은 relational model이 사라진다는 의미가 아니라, workload와 개발 요구가 다양해지면서 여러 모델이 공존하게 되었다는 의미로 이해해야 합니다.

## 5. 시스템 설계 관점

데이터 모델을 고를 때는 다음을 먼저 봐야 합니다.

- 주요 query pattern은 무엇인가?
- 데이터 관계는 one-to-many인가, many-to-many인가?
- 한 번에 읽는 단위는 무엇인가?
- 스키마 변경이 자주 발생하는가?
- 데이터 중복을 허용할 수 있는가?
- consistency requirement는 어느 정도인가?

모델 선택은 나중에 storage engine, indexing, caching, API 설계에도 영향을 줍니다. 예를 들어 document DB를 선택하면 API 응답 형태와 저장 형태가 비슷해질 수 있지만, 관계형 분석 질의는 어려워질 수 있습니다.

## 6. 실무/연구 연결점

Lakehouse에서는 여러 데이터 모델이 동시에 존재합니다.

- Iceberg table: relational/columnar model에 가까움
- object storage file: physical layout
- metadata catalog: table, snapshot, manifest 관계
- Milvus embedding index: vector search model
- Redis cache: key-value model
- lineage/governance: graph model과 잘 맞음

따라서 “하나의 데이터 모델”보다 각 계층이 어떤 모델을 제공하고, 모델 간 변환이 어디서 일어나는지가 중요합니다.

## 7. 헷갈리기 쉬운 부분

- Document DB는 schema가 없다는 뜻이 아니라 schema-on-read 또는 application-level schema에 가깝습니다.
- Join이 없다고 관계가 없는 것은 아닙니다. 관계 처리를 애플리케이션이 떠안는 경우가 많습니다.
- Graph DB는 단순히 edge가 있다는 뜻이 아니라 관계 탐색이 주요 workload일 때 의미가 큽니다.
- Declarative query는 사용자가 절차를 덜 지정하기 때문에 optimizer가 개입할 여지가 큽니다.

## 8. 핵심 질문 정리

- 내 데이터의 자연스러운 aggregate boundary는 무엇인가?
- 가장 빈번한 query는 어떤 shape인가?
- 중복 저장이 갱신 비용을 감당할 만큼 이득이 있는가?
- 관계가 늘어날 때 현재 모델이 버틸 수 있는가?
- 질의 언어가 개발자와 시스템 모두에게 충분한 abstraction을 제공하는가?

## 9. 한 문단 요약

2장은 데이터 모델이 시스템 설계의 사고방식을 결정한다는 점을 설명합니다. Relational model은 유연한 질의와 join에 강하고, document model은 aggregate 중심 데이터와 지역성에 강하며, graph model은 복잡한 관계 탐색에 강합니다. 좋은 선택은 유행이 아니라 데이터 구조, 질의 패턴, 변경 가능성, consistency 요구사항의 균형에서 나옵니다.
