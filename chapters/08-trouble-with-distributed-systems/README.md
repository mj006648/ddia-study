# 08. The Trouble with Distributed Systems

> 작성 원칙: 이 문서는 DDIA의 개념을 한국어로 재구성한 학습 노트이며, 원문 문장을 길게 옮기지 않습니다. 핵심 용어는 필요에 따라 English term을 병기합니다.

## 1. 이 장의 핵심 문제

분산 시스템은 단일 머신과 달리 부분 실패(partial failure)가 일상적으로 발생합니다. 네트워크는 지연·손실·중복될 수 있고, clock은 정확하지 않으며, process는 예기치 않게 멈출 수 있습니다. 핵심 문제는 “신뢰할 수 없는 구성요소 위에서 어떻게 안전한 시스템을 만들 것인가?”입니다.

## 2. 주요 개념

### Partial Failure

일부 노드는 정상이고 일부 노드는 실패하는 상태입니다. 전체가 성공/실패로 명확히 나뉘지 않기 때문에 판단이 어렵습니다.

### Unreliable Network

네트워크는 message loss, delay, duplication, reordering을 일으킬 수 있습니다. timeout은 실패를 증명하지 않고 “응답이 제때 오지 않았다”만 알려줍니다.

### Clock

- **Time-of-day clock**: 실제 시각에 가까운 clock. NTP 조정으로 앞뒤로 움직일 수 있음
- **Monotonic clock**: elapsed time 측정에 적합. 뒤로 가지 않음

분산 시스템에서 clock drift와 clock uncertainty는 lease, timeout, ordering에 영향을 줍니다.

### Process Pause

GC pause, OS scheduling, page fault, virtualization pause 등으로 process가 멈출 수 있습니다. 노드는 자신이 멈췄다는 사실을 모를 수 있습니다.

### Fencing Token

lease 기반 시스템에서 오래 멈췄던 process가 뒤늦게 write하는 문제를 막기 위해 단조 증가 token을 사용합니다. storage가 오래된 token의 write를 거부해야 안전합니다.

## 3. 동작 원리

분산 시스템에서는 “응답 없음”을 해석하기 어렵습니다. 노드가 죽었는지, 네트워크가 느린지, GC pause 중인지 알 수 없습니다. 따라서 timeout과 retry는 필수지만, retry는 duplicate request를 만들 수 있어 idempotency가 필요합니다.

Clock도 절대적 기준으로 쓰기 어렵습니다. 특히 “이 시간이 지났으니 lock은 만료되었다” 같은 판단은 clock drift와 pause 때문에 위험할 수 있습니다.

## 4. 장점과 한계

이 장은 분산 시스템의 현실적 위험을 잘 보여줍니다. 다만 해결책보다 문제의 성격을 강조합니다. 이후 consensus와 consistency 장에서 이런 불확실성을 제어하는 방법을 다룹니다.

## 5. 시스템 설계 관점

- timeout은 실패 감지가 아니라 의심(suspicion)으로 봐야 합니다.
- retry는 idempotency key와 함께 설계해야 합니다.
- lease를 사용할 때는 fencing token이 필요할 수 있습니다.
- clock 기반 ordering보다 logical clock 또는 consensus 기반 ordering이 안전할 수 있습니다.

## 6. 실무/연구 연결점

Kubernetes 환경에서는 pod restart, network partition, node pressure, GC pause가 실제로 발생합니다. Metadata service, catalog, cache update 과정에서 partial failure를 전제로 설계해야 합니다.

예를 들어 Redis lock만 믿고 object storage나 catalog에 write하면 pause 후 stale writer 문제가 생길 수 있습니다. version number, compare-and-swap, fencing token 같은 장치가 필요합니다.

## 7. 헷갈리기 쉬운 부분

- Timeout은 상대 노드가 죽었다는 증거가 아닙니다.
- Clock이 동기화되어 있어도 완전히 같은 시각을 보장하지 않습니다.
- Process가 멈췄다가 다시 실행되면 자신은 계속 lock을 갖고 있다고 착각할 수 있습니다.

## 8. 핵심 질문 정리

- 네트워크 지연과 message loss를 어떻게 처리하는가?
- retry가 중복 실행을 만들어도 안전한가?
- lock/lease가 stale owner 문제를 막는가?
- clock에 의존하는 correctness 조건이 있는가?

## 9. 한 문단 요약

8장은 분산 시스템에서 네트워크, clock, process가 모두 신뢰하기 어렵다는 점을 설명합니다. timeout은 실패의 증거가 아니고, clock은 완전한 순서를 제공하지 않으며, process pause는 lease와 lock을 위험하게 만들 수 있습니다. 안전한 설계를 위해 idempotency, fencing token, logical ordering, 보수적인 failure handling이 필요합니다.
