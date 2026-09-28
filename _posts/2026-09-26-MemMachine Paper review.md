---
date: 2026-09-26 15:00:00 +0900
title: "MemMachine Paper review"
excerpt: "원문을 그대로 보관하고 LLM 추출을 최소화한다. 저장이 아니라 검색을 손봐야 한다는 걸 6차원 ablation으로 보인 논문."
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


MemMachine: A Ground-Truth-Preserving Memory System for Personalized AI Agents

[https://arxiv.org/abs/2604.04853](https://arxiv.org/abs/2604.04853)

MemMachine은 MemVerge, Inc.에서 만든 오픈소스 메모리 시스템이다.

2026년 4월 6일에 올라왔고, 저자 7명에 18쪽이다.

제목의 ground-truth-preserving은 원문을 그대로 보관한다는 뜻이다. LLM으로 뭔가 뽑아내는 걸 최소한으로 하고, 대신 검색 쪽을 손보는 게 더 효과가 크다고 한다.

원문 보존은 뒤에서 볼 [Rate-Distortion 리뷰](https://momozzing.github.io/paper%20review/Rate-Distortion-Memory-Compaction-Paper-review/)(가역 압축이 비가역보다 낫다)와 [ReFind 리뷰](https://momozzing.github.io/paper%20review/ReFind-Paper-review/)(원문을 안 건드리고 이긴다)에서도 다시 나온다. 여기서는 그걸 제품으로 만들었다.

좀 더 자세히 알아보자.

## **1. Introduction**

설계 입장부터 보자.

단기 메모리, 장기 일화 메모리, 프로필 메모리 세 개를 원문 보존 구조로 묶는다.

원시 대화 에피소드를 그대로 저장하고, LLM으로 추출하는 건 최소화한다고 한다.

없애는 게 아니라 줄이는 거다. 프로필 메모리는 여전히 LLM으로 추출한다.

## **2. MemMachine Architecture**

### **2.1 Contextualization**

대화 메모리 검색이 어려운 이유를 이렇게 설명한다.

맥락상 중요한 에피소드가 질의랑 임베딩이 꽤 다를 수 있다. 일반 RAG 문서는 청크 하나하나가 어느 정도 혼자 말이 되는데, 대화 턴은 서로 강하게 의존한다고 한다.

예를 들어 "그 식당 추천" 관련 질문에 답하려면, 추천이 들어 있는 턴만이 아니라 무엇을 왜 물었고 어떤 제약이 있었는지 나온 주변 턴도 필요하다.

그래서 네 단계로 검색한다.

1. 임베딩 검색으로 핵(nucleus) 에피소드를 찾는다
2. 바로 옆 에피소드를 앞 1개, 뒤 2개 가져와서 클러스터를 만든다
3. 클러스터를 cross-encoder 등으로 재랭킹한다
4. 상위 k개 클러스터를 LLM에 준다

뒤에서 볼 [ReFind 리뷰](https://momozzing.github.io/paper%20review/ReFind-Paper-review/)의 컨텍스트 창 확장(히트 ±2턴을 블록으로)도 거의 같은 장치다. ReFind ablation에서는 이걸 빼면 S에서 −9.2점이다.

-> 앞 1개, 뒤 2개로 비대칭인 건 대화에서 답이 질문 뒤에 오니까 뒤쪽을 더 보는 것 같다.

### **2.2 Profile Memory (Semantic Memory)**

일화 메모리가 원시 상호작용을 그대로 두는 거라면, 프로필 메모리는 사용자 속성을 요약해서 모아둔다.

- 사용자가 스스로 밝힌 인구통계 정보
- 말한 선호와 관심사
- 행동 패턴

LLM 추출은 여기서만 쓴다. 원문은 일화 메모리에 남아 있으니까 프로필 추출이 틀려도 다시 복구할 수 있다.

앞에서 본 [Mem0](https://momozzing.github.io/paper%20review/Mem0-Paper-review/)에서는 사실만 남기고 원문을 버렸는데, 여기서는 원문을 남긴다.

## **3. Retrieval Agent**

다중홉 질의를 위해서 에이전트를 따로 둔다.

### **3.1 The Late Binding Problem**

논문 예시는 이렇다. Acme의 CEO(Person X)를 찾고, 그 배우자 Person Y를 찾고, Person Y의 고용주 Company Z를 찾아야 한다.

질의 시점에는 원래 질의 문자열밖에 없다. 그 임베딩은 "Acme", "CEO", "company" 같은 표면 단어 근처에 모여 있어서, 중간 엔티티를 모르면 Company Z까지 갈 방법이 없다고 한다.

그리고 이건 임베딩 모델 탓이 아니라 다중홉 사슬 자체의 구조 문제라서, 한 번의 벡터 검색으로는 풀 수 없다고 한다.

기존 방법들도 한계가 있다고 한다.

- 질의 확장(HyDE), BM25 하이브리드, 청크 재랭킹 : 단일홉 재현율은 올라가지만 여전히 질의 하나로 검색하니까 의존 사슬은 못 푼다
- 지식그래프 순회 : 정확히 풀긴 하는데 그래프를 미리 만드는 게 비싸고 정보 손실이 있다

나중에 볼 [ReFind](https://momozzing.github.io/paper%20review/ReFind-Paper-review/)도 이 문제를 반복 검색으로 푼다. ReFind 실험에서는 검색을 1회로 묶으면 M 세트에서 20.4점이 빠진다.

### **3.2 Architecture**

해법은 도구 트리다. `ToolSelectAgent`가 질의를 보고 셋 중 하나로 보낸다.

1. MemMachine Agent : 직접 검색
2. SplitQuery Agent : 질의를 병렬로 쪼갬(fan-out)
3. ChainOfQuery Agent : 증거를 반복해서 모음(다중홉)

### **3.3 Benchmark Results**

다중홉 에이전트 결과다.

- HotpotQA hard: 93.2%
- WikiMultiHop(무작위 노이즈 포함): 92.6%

## **4. Results and Analysis**

### **4.1 LoCoMo Benchmark Results**

| 지표 | 점수 |
|---|---:|
| 전체 (gpt-4.1-mini) | 0.9169 |
| Single-hop | 0.9465 |
| Multi-hop | 0.8759 |
| Temporal | 0.7352 |
| Open-domain | 0.7083 |

### **4.2 Comparative Analysis**

차순위 시스템(Memobase)보다 +9.7점이라고 한다.

Temporal만 진다. Memobase가 0.8505로 더 높다.

논문은 타임스탬프를 고려한 검색으로 개선할 수 있다고 보고 있다. 그리고 agent 모드에서는 0.9159까지 올라가서, 시간 추론은 평가 모델 능력에 많이 좌우된다고 한다.

### **4.3 Efficiency Analysis**

효율도 같이 보고한다.

- 입력 토큰 ~80% 절감 (Mem0 대비)
- 메모리 추가 속도 ~75% 향상
- 검색 속도 최대 75% 향상

### **4.4 LongMemEvalS Ablation Study**

500문항 전체에서 여섯 가지를 하나씩 바꿔가며 쟀다. 최고 점수는 93.0%다.

| 최적화 | 단계 | 기여 |
|---|---|---:|
| 검색 깊이 `k` (20→30) | 검색 | +4.2%p |
| 컨텍스트 포맷팅 | 검색 | +2.0%p |
| 검색 프롬프트 설계 | 검색 | +1.8%p |
| CoT 제거 | 검색 | +1.6%p |
| 사용자 질의 편향 보정 | 검색 | +1.4%p |
| 문장 청킹 | 저장 | +0.8%p |

검색 쪽 개선을 합친 게 저장 쪽(문장 청킹 +0.8%)보다 훨씬 크다.

그래서 논문은 메모리 시스템에서는 어떻게 저장하느냐보다 어떻게 꺼내느냐가 더 중요하다고 한다. 단, 저장할 때 원문을 보존한다는 전제에서다.

그리고 사실 추출이나 지식그래프 구축처럼 저장 단계에 LLM을 많이 쓰는 시스템은 엉뚱한 단계를 최적화하고 있을 수도 있다고 한다.

-> Mem0(추출), Zep(그래프), A-MEM(링크+진화)이 다 저장 단계에 LLM을 많이 쓴다. 그 투자가 효과가 있는지 의심하는 거다.

작은 모델이 이긴다는 결과도 있다.

답변 LLM으로 GPT-5-mini가 GPT-5보다 +2.6% 높았다고 한다. 최적화된 프롬프트랑 같이 썼을 때 그렇고, 비용 대비로도 제일 낫다.

이유는 모델이랑 프롬프트를 같이 맞춰야 해서라고 한다. GPT-4.x용으로 만든 chain-of-thought 프롬프트는 GPT-5에는 안 맞고, 단순한 프롬프트가 더 나을 수 있다고 한다.

ablation에서 CoT 제거가 +1.6%p였던 것과 같은 얘기다.

## **5. Discussion**

### **5.1 Architectural Design Tensions**

설계 공간 비교표다.

논문이 MemMachine, Mem0, Zep, MemOS, Full Context를 속성별로 비교해 두었다.

- 메모리 방식 : MemMachine·Mem0·Zep은 검색, MemOS는 하이브리드, Full Context는 in-context
- 원문 보존 : MemMachine과 Full Context는 ✓, Mem0·Zep·MemOS는 부분
- 프롬프트 캐시 가능 : MemMachine 부분, Full Context ✓, 나머지 ✗
- 컨텍스트 창 너머 확장 : Full Context만 ✗, 나머지 ✓
- 메시지당 LLM 호출 : MemMachine 낮음, Mem0 높음, Zep 보통, MemOS 높음, Full Context 없음
- 전용 DB 필요 : Full Context만 아니오, 나머지 예
- 오픈소스 : MemMachine·MemOS ✓, Mem0·Zep 부분, Full Context N/A

메시지당 LLM 호출이 낮다는 게 다른 시스템이랑 다른 점이다.

앞에서 본 [Anatomy 리뷰](https://momozzing.github.io/paper%20review/Anatomy-of-Agentic-Memory-Paper-review/)에서는 A-Mem 구축에 15시간, Nemori 형식 오류 30%가 나왔는데, 둘 다 쓰기 단계 LLM 호출에서 생긴 문제였다. 호출이 적으면 이런 위험도 줄어든다.

### **5.2 When Memory Helps (and When It Doesn’t)**

논문이 이걸 절을 따로 두고 적었다.

도움이 되는 경우

- 다중 세션 상호작용 (고객 지원, 의료, 교육)
- 개인화가 중요한 애플리케이션 (콘텐츠 추천, 개인 비서)
- 상태를 계속 들고 가야 하는 워크플로 (프로젝트 관리, CRM)
- 상호작용 이력이 필요한 컴플라이언스·감사

필요 없거나 오히려 손해인 경우

- 단일 턴, 무상태 질의 (검색, 번역, 단순 QA)
- 양은 많고 개인화는 적은 트래픽

## **6. 지금 관점: 검색부터 손본다**

이 논문에서 해볼 만한 건 저장 구조를 안 바꾸고도 된다.

여섯 가지 중에 제일 큰 게 `k`를 20→30으로 올린 +4.2%p다. 파라미터 하나 바꾼 거다.

이웃 턴을 같이 꺼내는 것도 ReFind랑 이 논문이 따로따로 효과를 봤다. 대화 데이터는 청크 하나만으로는 말이 안 되니까 그렇다.

모델을 바꿀 때 프롬프트도 다시 봐야 한다. CoT 제거 +1.6%p, GPT-5-mini가 GPT-5보다 +2.6%p였다.

-> Mem0·Zep·A-MEM 같은 걸 들이기 전에 이런 것부터 해보는 게 순서일 것 같다.

논문이 밝힌 한계도 있다.

- 벤치마크 결과는 평가 모델, 프롬프트 템플릿, 제공자 쪽 모델 업데이트에 따라 달라진다고 한다
- 시스템 간 비교는 직접 다시 돌린 결과랑 발표된 수치를 섞은 거라 전처리·프롬프트·인프라가 다를 수 있다고 한다
- 토큰 효율 비교는 워크로드에 따라 달라서, 보고된 설정 밖에서는 방향 정도로만 봐야 한다고 한다
- ablation은 차원을 하나씩 따로 바꾼 거라서, 차원끼리 상호작용(예: `k`가 커지면 청킹 이득이 달라지는지)은 안 봤다고 한다
- 일부 구성은 일부 문항으로 먼저 돌린 다음에 500문항으로 늘렸다고 한다

그래서 이 결과를 일반적인 성능 보장이 아니라, 평가한 설정 안에서의 경험적 증거로 봐달라고 한다.

-> 그런데 여기도 full-context 대비 ∆는 없다. 설계 공간 비교에 Full Context가 들어가 있는데 성능 비교는 안 했다??

## **7. Conclusion**

원문 보존, 비용 효율, 개인화를 우선으로 둔 오픈소스 메모리 시스템이다.

단기·장기 일화 메모리 2계층에 프로필 메모리를 더해서, LLM 추출 방식에서 생기는 비용과 오류 누적 없이 과거 경험을 저장하고 꺼내 쓰게 했다고 한다.

LongMemEvalS ablation에서는 검색 단계 최적화가 저장 단계 변경보다 훨씬 효과가 컸고, 프롬프트를 맞춘 작은 모델(GPT-5-mini)이 큰 모델(GPT-5)보다 나았다고 한다.

여태까지 나온 메모리 시스템들이 저장 구조로 경쟁했다면, 이 논문은 그 전에 검색부터 손보라고 한다.

다음은 [LongMemEval-V2](https://momozzing.github.io/paper%20review/LongMemEval-V2-Paper-review/)다. V1이 사용자 이력을 물었다면 V2는 웹 에이전트가 환경에서 쌓은 경험을 묻는다.
