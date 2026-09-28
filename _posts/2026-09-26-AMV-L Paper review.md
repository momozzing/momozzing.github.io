---
date: 2026-09-26 09:00:00 +0900
title: "AMV-L Paper review"
excerpt: "TTL은 항목의 수명을 제한하지 계산량을 제한하지 않는다. 검색 후보군 크기를 직접 묶어 2초 초과 요청을 13.8%에서 0.007%로 줄인 논문."
categories:
  - Paper review
tags:
  - Large Language Model
  - NLP
  - Agent
  - Memory
  - Paper review
toc: true
toc_sticky: true
field: agent-memory
---

AMV-L: Lifecycle-Managed Agent Memory for Tail-Latency Control in Long-Running LLM Systems

[https://arxiv.org/abs/2603.04443](https://arxiv.org/abs/2603.04443)

AMV-L은 Georgia Tech에서 만든 에이전트 메모리 관리 방법이다.

메모리가 쌓이면 검색이 느려지는데, 검색 후보군 크기를 직접 묶어서 느린 요청(꼬리 지연)을 줄인다고 한다.

2026년 2월 22일에 나왔고, 저자 1명에 15쪽이다.

앞의 논문들이 주로 무엇을 어떻게 기억할지를 다뤘다면, 이 논문은 그게 서비스 지연에 어떤 영향을 주는지를 본다. 시스템 쪽 논문이라 다른 논문들과 좀 다르다.

좀 더 자세히 알아보자.

## **1. Introduction**

문제 정의부터 한다.

논문은 문제를 한 문장으로 정리한다.

TTL은 항목이 얼마나 오래 남을지는 제한하지만, 요청이 들어왔을 때 메모리가 쓰는 계산량은 제한하지 않는다고 한다.

보관된 항목이 쌓이면 검색 후보군과 벡터 유사도 스캔이 예측할 수 없게 커진다. 그래서 지연이 가끔 크게 튀고(heavy-tailed) 처리량이 불안정해진다고 한다.

앞 리뷰들에서 계속 나왔던 느린 검색 수치가 이거다. [Anatomy 리뷰](https://momozzing.github.io/paper%20review/Anatomy-of-Agentic-Memory-Paper-review/)의 MemoryOS 검색 31.2초, [Mem0 리뷰](https://momozzing.github.io/paper%20review/Mem0-Paper-review/)의 LangMem p95 59.8초가 여기 해당한다. 저장은 잘 되는데 꺼낼 때 후보군이 너무 커진다.

## **2. AMV-L Overview**

에이전트 메모리를 그냥 저장소가 아니라 관리해야 하는 시스템 자원으로 본다고 한다.

각 항목에 계속 갱신되는 효용 점수 `V(m)`을 매기고, 그 값에 따라 올리고(승격) 내리고(강등) 빼는(축출) 식으로 계층을 유지한다.

### **2.1 Tiered lifecycle organization**

계층은 세 개다.


- Hot : 평소 요청에서 검색하고 프롬프트에 넣을 수 있는 항목
- Warm : 중간 효용. 기본 검색 경로에서는 빠짐
- Cold : 효용이 낮음. 적은 비용으로 보관만 하고 검색에서는 빠짐

이렇게 나누면 보관(retention)과 검색 대상 여부(eligibility)가 분리된다고 한다. warm이나 cold에 남아 있어도 매 요청마다 비용이 들지는 않는다.

보관은 하지만 검색 대상은 아닌 것이다.

나중에 볼 [Rate-Distortion 리뷰](https://momozzing.github.io/paper%20review/Rate-Distortion-Memory-Compaction-Paper-review/)는 "원본을 남겨라"(가역성)인데, 여기서도 원본은 남긴다. 검색 대상에서만 뺀다.

### **2.2 Lifecycle transitions**

생애주기 전이는 비동기로 한다.

요청을 처리할 때는 사용 기록과 값 갱신만 하고, 계층 이동이나 정리는 요청 처리 경로 밖에서 따로 한다고 한다.

유지보수 때문에 요청 지연이 늘어나지 않게 하려는 것이다.

### **2.3 Bounded retrieval and prompt construction**

통제를 두 가지로 나눈다.

1. Eligibility control (AMV-L 생애주기) : 어떤 항목이 검색에 참여할 수 있는가
2. Injection control (프롬프트 상한) : 검색된 것 중 몇 개를 프롬프트에 넣는가

대부분의 시스템은 2번만 있다고 한다.

프롬프트 상한은 프롬프트 길이는 묶지만, 1번 없이는 큰 후보군을 검색하는 비용을 못 막는다는 것이다.

## **3. Memory Value Model**

`V(m)`은 세 가지 신호로 갱신한다.

- Access : 이번 요청 검색에서 고려되거나 선택됨
- Contribution : 실제로 최종 프롬프트에 들어감
- Elapsed time : 마지막 갱신 이후 지난 시간

일부러 운영 신호(operational)로 골랐다고 한다. 의미 라벨이나 오프라인 학습, 사람 피드백 없이 서빙 시스템이 바로 잴 수 있다.

앞에서 본 [Experience-Following 리뷰](https://momozzing.github.io/paper%20review/Experience-Following-Paper-review/)의 "미래 태스크 평가가 공짜 품질 라벨이 된다"와 비슷하다. 따로 평가 비용 없이 사용 기록에서 효용이 나온다.

갱신은 lazy하게 한다. 항목이 요청에 닿을 때만 하고 전체를 스캔하지 않는다. 지수 감쇠 `V(m) ← V(m)·e^{−λΔt}`를 먼저 적용하고 이벤트에 따라 더한다.

조건은 세 가지다.

1. 국소성 : 항목별 상태와 그 요청의 이벤트만 본다
2. 증분성 : 전체를 다시 계산하지 않고 온라인으로
3. 저오버헤드 : 닿은 항목당 상수 시간

## **4. Results and Discussion**

TTL, LRU를 베이스라인으로 같은 장기 실행 워크로드에서 비교한다. 프롬프트 주입 상한은 모든 조건에서 똑같이 고정했다.

### **4.1 End to end latency and throughput**

| 지표 | TTL | LRU | AMV-L |
|---|---:|---:|---:|
| 성공률(%) | 100.000 | 99.997 | 100.000 |
| 처리량(req/s) | 9.027 | 38.169 | 36.977 |
| 지연 p50(ms) | 814.730 | 153.810 | 194.080 |
| 지연 p95(ms) | 4503.743 | 921.556 | 950.409 |
| 지연 p99(ms) | 5398.167 | 1452.706 | 1233.430 |
| >1s 비율(%) | 39.632 | 3.960 | 3.653 |
| >2s 비율(%) | 13.813 | 0.343 | 0.007 |

TTL과 비교하면 AMV-L은 처리량이 3.1배, 지연은 중앙값 4.2배, p95 4.7배, p99 4.4배 좋아졌다. 2초 넘는 요청이 13.8%에서 0.007%로 줄었다.

LRU와는 주고받는 관계다. 중앙값과 p95는 LRU가 조금 낫다(154 vs 194ms, 922 vs 950ms). 대신 p99는 AMV-L이 낫고(1233 vs 1453ms), 2초 넘는 요청은 98% 줄였다(0.343% → 0.007%).

값 기반으로 관리하면 recency 기반인 LRU에서도 남아 있는 극단적으로 느린 요청을 줄일 수 있다고 한다.

꼬리 분포는 그림으로 보면 더 분명하다.

![요청 지연 CCDF (논문 Figure 1)](https://momozzing.github.io/assets/images/amv-l/fig1-latency-ccdf.png)

지연이 x 이상인 요청 비율(CCDF)을 로그 축으로 그린 것이다.
TTL은 1초 넘는 구간에 요청이 많이 남아 있고, LRU와 AMV-L은 둘 다 꼬리를 크게 줄인다고 한다.
-> 오른쪽 끝을 보면 된다. LRU는 드물게 매우 느린 요청이 길게 남고, AMV-L은 그보다 앞에서 끊긴다.

### **4.2 Mechanism: retrieval working set and vector search footprint**

검색 발자국을 본다.

| 지표 | TTL | LRU | AMV-L |
|---|---:|---:|---:|
| 검색 집합 p95 (\|R\|) | 4,824 | 261 | 690 |
| 스캔한 벡터 p95 | 4,824 | 261 | 690 |

TTL에서는 p95 후보군이 4,824개까지 커진다. AMV-L은 690으로 85.7% 줄이고, LRU는 261로 94.6% 줄인다.

![검색 후보군 크기 분포 (논문 Figure 3)](https://momozzing.github.io/assets/images/amv-l/fig3-retrieval-working-set.png)

검색 후보군 크기 \|R\|의 누적 분포다.
TTL은 위쪽 꼬리가 길고, AMV-L은 hot 항목과 상한이 있는 warm 샘플만 검색 대상으로 두어서 분포가 좁아진다고 한다.
후보군 크기 \|R\|가 유사도 검색 비용을 곱으로 키우는 요인이라서, 이 분포가 곧 검색 비용을 예측한다고 한다.

근데 LRU가 더 적게 스캔하는데 극단 꼬리는 AMV-L이 낫다. 스캔 수만으로는 설명이 안 된다는 건데, 논문도 이걸 시스템적으로 미묘한 부분이라고 한다.

-> 그럼 LRU의 p99가 왜 더 느린지는 뭐 때문인지?? 자세한 이유는 잘 모르겠다.

### **4.3 Cost and quality tradeoffs**

토큰 오버헤드도 LRU보다 약 6% 낮고, 검색 품질(값 평균)은 0~2% 안에서 비슷하다. 품질은 그대로 두고 꼬리 지연을 줄인 것이다.

### **4.4 Discussion**

병목이 무엇인가를 따진다.

10.4절에서 장기 실행 에이전트 메모리의 병목은 저장 용량이 아니라고 한다. 검색 대상을 통제하지 않아서 요청마다 계산이 커지는 게 문제라는 것이다.

프롬프트 주입 상한을 고정해도 지연이 튀고 처리량이 낮게 나온다. 비싼 건 top-n 항목을 넣는 게 아니라 후보군 전체를 스캔하고 점수 매기는 단계이기 때문이라고 한다.

그래서 프롬프트 길이를 묶는 것보다 검색 대상을 묶는 게 더 효과가 크다고 한다.

## **5. 지금 관점: 놓치고 있던 축**

이 시리즈 논문들이 주로 보는 건 정확도다. 이 논문만 지연이 얼마나 예측 가능한지를 본다.

이 시리즈 리뷰들에 나오는 지연 수치를 이 논문 기준으로 다시 보면 이렇다. LME-V2는 뒤에서 볼 논문이다.

- MemoryOS ([Anatomy](https://momozzing.github.io/paper%20review/Anatomy-of-Agentic-Memory-Paper-review/)) : 검색 31.2s. 후보군 제한이 없음
- LangMem ([Mem0](https://momozzing.github.io/paper%20review/Mem0-Paper-review/)) : p95 59.8s. 토큰은 127개인데 검색이 느림
- Mem0 : p95 0.2s. 사실만 남겨서 후보군이 작음
- AgentRunbook-R ([LME-V2](https://momozzing.github.io/paper%20review/LongMemEval-V2-Paper-review/)) : ~26s. 25M+ 토큰 규모

-> 원인은 내가 추정한 거다.

LangMem이 이 논문에서 말하는 경우에 딱 맞는 것 같다. 메모리 토큰을 127개까지 줄였는데 검색에 60초가 걸린다. 넣는 양은 묶었는데 후보군은 안 묶어서 그런 것 같다.

실제로 쓴다면, 원본은 지우지 않고 검색 대상에서만 빼는 계층을 두면 될 것 같다. 이 시리즈에서 "원본을 남겨라"라는 얘기가 여러 번 나오는데, 남긴 걸 전부 검색 대상으로 두면 이 논문의 문제가 생긴다. 남기되 hot에는 안 두는 것이다.

-> 보통 응답 지연만 모니터링하는데, `|R|`(검색 후보군 크기)을 따로 재면 원인 찾기가 쉬울 것 같다. p95가 수천 개면 TTL 문제다.

-> access·contribution·elapsed 세 개면 된다고 하니 의미 라벨도 학습도 필요 없다. Qdrant 같은 벡터 DB에서도 페이로드 필드로 만들 수 있을 것 같다.

다만 저자 1명이 시스템 하나로 한 실험이다. 합성 워크로드이고 비교 대상도 TTL, LRU 둘뿐이다. 앞 리뷰들처럼 여러 메모리 시스템과 비교한 게 아니다.

그리고 중앙값, p95, 처리량은 LRU가 이긴다. AMV-L이 나은 건 극단 꼬리뿐이다. SLO가 p99나 >2s 기준이면 AMV-L, 평균 응답이 중요하면 LRU가 더 단순하다. 논문도 이걸 "절충 프런티어 위의 다른 지점"이라고 한다.

여기까지 보고 남은 것들을 적어둔다. 뒤에서 볼 논문들에서 이어지는 얘기도 같이 표시해둔다.

1. 원본을 남기는 쪽. 앞에서 본 [Zep](https://momozzing.github.io/paper%20review/Zep-Paper-review/)은 삭제 대신 무효화로 풀었고, 이 논문은 남기되 검색에서만 빼는 방법이다. 뒤에서 볼 [Rate-Distortion](https://momozzing.github.io/paper%20review/Rate-Distortion-Memory-Compaction-Paper-review/)은 같은 예산에서 가역이 비가역을 이긴다고 하고, [ReFind](https://momozzing.github.io/paper%20review/ReFind-Paper-review/)는 원본을 안 건드려서 이기고, [Memory Portability](https://momozzing.github.io/paper%20review/Memory-Portability-Paper-review/)는 원본이 없으면 복구가 48건 전부 실패한다고 한다.
2. 저장보다 검색이 병목인 경우. 이 논문의 검색 대상 통제가 그쪽이다. 다만 앞에서 본 [NEMORI](https://momozzing.github.io/paper%20review/NEMORI-Paper-review/)는 반대로 증류로 이겼다. 뒤에서 볼 [MemMachine](https://momozzing.github.io/paper%20review/MemMachine-Paper-review/)의 ablation(검색 최적화 +4.2%p vs 저장 +0.8%p), [Memory Portability](https://momozzing.github.io/paper%20review/Memory-Portability-Paper-review/)에서 RAG 손실의 81%가 검색이었던 것도 같은 방향이다. 그래서 뭐가 병목인지 먼저 재보라는 게 [MemFail](https://momozzing.github.io/paper%20review/MemFail-Paper-review/)의 얘기다.
3. 모델을 키워도 안 풀린다. 앞에서 본 [Anatomy](https://momozzing.github.io/paper%20review/Anatomy-of-Agentic-Memory-Paper-review/)의 형식 오류율이 그렇고, 뒤에서 볼 [MemFail](https://momozzing.github.io/paper%20review/MemFail-Paper-review/)의 Q2와 [Memory Portability](https://momozzing.github.io/paper%20review/Memory-Portability-Paper-review/)의 NOTES 비대칭도 같은 얘기다. 구조 문제라는 것이다.
4. ∆부터 계산해보자. [Anatomy](https://momozzing.github.io/paper%20review/Anatomy-of-Agentic-Memory-Paper-review/)의 Context Saturation Gap이다. full-context보다 나은 게 없으면 메모리 시스템을 쓰는 이유는 정확도가 아니라 지연과 비용이다.

## **6. Conclusion**

conclusion 부분을 보면, AMV-L은 메모리를 관리해야 하는 시스템 자원으로 보고, 계속 갱신되는 효용 점수로 생애주기를 관리한다고 한다.

p95, p99 지연을 최대 2~3자릿수 줄이면서 밀리초 단위 중앙값은 그대로 유지한다고 한다. 전체 보관량과 상관없이 검색 대상 크기를 묶었기 때문이다.

마지막 문장은, 장기 실행 LLM 에이전트가 예측 가능한 성능을 내려면 시간 기반 만료가 아니라 생애주기 관리가 필요하다는 것이다.

한계도 적어뒀다.

1. LRU 베이스라인이 순수 recency 기반이라 의미 효용 신호가 없고, 상황이 바뀔 때 오래됐지만 중요한 항목을 지킬 방법도 없다. recency와 value를 섞은 하이브리드가 더 나을 수 있다고 한다
2. AMV-L은 hot 계층 크기에 딱 정해진 상한이 없다. 값 변화로 사실상 조절되긴 하지만, 엄격한 예산을 두면 최악의 경우를 더 확실하게 보장할 수 있다고 한다

여태까지 메모리 논문들이 무엇을 어떻게 기억할지를 봤다면, 이 방법은 기억한 것 중 무엇을 검색 대상으로 둘지를 관리한다.

다음은 [Multi-Layered Memory](https://momozzing.github.io/paper%20review/Multi-Layered-Memory-Paper-review/)다. 대화 이력을 working·episodic·semantic 세 계층으로 나누고, 계층을 하나씩 떼어보는 ablation으로 각 계층이 얼마나 기여하는지 잰 논문이다.
