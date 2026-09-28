---
date: 2026-09-24 15:00:00 +0900
title: "What Deserves Memory Paper review"
excerpt: "무엇을 남길지를 중요도 점수가 아니라 '예측 실패'로 정한다. 기존 지식으로 예상한 것과 실제가 어긋난 만큼만 기억으로 증류하는 프레임워크."
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

What Deserves Memory: Adaptive Memory Distillation for LLM Agents

[https://arxiv.org/abs/2508.03341](https://arxiv.org/abs/2508.03341)

NEMORI는 Fudan University, Shanda Group, Beihang University 등에서 만든 에이전트 메모리 프레임워크이다.

무엇을 기억으로 남길지를 중요도 점수가 아니라 "예측 실패"로 정한다. 기존 지식으로 예상한 것과 실제가 어긋난 만큼만 기억으로 남긴다고 한다.

2025년 8월 5일에 나왔고 2026년 4월 16일에 v4가 올라왔다. 저자 4명, 24쪽이다.

[앞 리뷰](https://momozzing.github.io/paper%20review/Experience-Following-Paper-review/)에서는 무엇을 넣고 지우느냐가 에이전트 행동을 바꾼다는 걸 봤는데, 이 논문은 처음에 무엇을 뽑을지를 다룬다. 뒤에서 볼 [Janus](https://momozzing.github.io/paper%20review/Janus-Selective-Memory-Update-Paper-review/)가 넣은 뒤에 검증하는 쪽이라면 이쪽은 넣기 전이다.

좀 더 자세히 알아보자.

## **1. Introduction**

논문 첫 문단에 문제를 이렇게 적는다.

LLM 에이전트 메모리는 어떤 정보를 남길 가치가 있는지 정하는 게 어렵다고 한다. 기존 방법들은 중요도 점수, 감정 태그, 사실 템플릿 같은 미리 정해둔 휴리스틱을 쓰는데, 이건 데이터에서 배운 게 아니라 설계자의 직관을 그대로 넣은 것이라고 한다.

그래서 생기는 문제가 두 가지다.

1. 주관적 편향 : 증류 단계에서 한 번 잘못 뽑으면 되돌릴 수 없는 정보 왜곡이 생길 수 있다
2. 시스템 비대화 : 그 왜곡을 피하려고 이것저것 다 저장하게 되고, 결국 검색 잡음이 커진다

앞에서 본 [Generative Agents](https://momozzing.github.io/paper%20review/Generative-Agents-Paper-review/)의 recency·relevance·importance 3점수 회상이 딱 이런 휴리스틱이다. importance를 LLM한테 1~10점으로 매기게 하는 방식이 여기서 말하는 설계다.

## **2. Methodology**

### **2.1 Overview & Motivations**

Predictive Coding Theory(Rao & Ballard 1999, Friston 2010, Clark 2013)에서 기준을 가져온다.

상호작용 안의 정보는 대부분 중복이고, 예측 부호화 관점에서 보면 예상 밖의 정보가 기억으로 남길 후보라고 한다.

그래서 경험이 나중에 쓸모 있을지를 "예측할 수 있느냐"의 문제로 바꾼다. 기존 지식으로 에이전트가 예측하지 못한 관측만 증류한다.

이미 아는 걸로 맞출 수 있으면 저장할 필요가 없다는 것이다. 틀린 만큼만 새로운 정보다.

프레임워크는 구조, 표현, 증류 세 가지 사전(prior)을 따르고, 두 개의 모듈이 이어진 구조다. 상보 학습 시스템(CLS)에 대응한다고 한다.

![NEMORI 전체 구조 (논문 Figure 1)](https://momozzing.github.io/assets/images/nemori/fig1-nemori-overview.png)

위쪽 Episodic Memory Integration이 원시 대화를 이야기처럼 이어지는 에피소드로 바꾸고, 아래쪽 Semantic Knowledge Distillation이 예측 오차로 지식을 뽑는다.

오른쪽은 뽑은 지식을 받는 쪽이다. 자체 관리 모듈이나 MemoryOS, A-MEM 같은 외부 시스템에 증류 층으로 붙일 수 있다고 한다.

### **2.2 Episodic Memory Integration**

원시 대화를 이야기처럼 이어지는 에피소드로 바꾼다. 세 단계다.

1. Local Message Partitioning : 메시지 버퍼 `B`에 쌓다가 관측 창 길이 `w`에 도달하면 나눈다. 창 안에서 어디까지가 한 덩어리인지 LLM이 보고 경계를 정한다
2. Narrative Episode Generation : 나눈 묶음을 이야기 형태의 에피소드로 만든다
3. Associative Memory Integration : 기존 에피소드와 이어지는 게 있으면 합치고, 없으면 따로 넣는다

앞에서 본 [LongMemEval 리뷰](https://momozzing.github.io/paper%20review/LongMemEval-Paper-review/)의 CP 1(Value 입자)과 같은 문제인데, 거기서는 "round가 최적"이라는 고정 답이었고 여기서는 LLM이 경계를 찾는다.

### **2.3 Semantic Knowledge Distillation**

이 논문에서 제일 중요한 부분이다. 메모리를 어떻게 관리하든 상관없게(management-agnostic) 만들었다고 한다.

1. Anticipatory Schema Synthesis (예상 스키마 합성)
2. Prediction Error Distillation (예측 오차 증류)
3. Agnostic Knowledge Consolidation

1번은 들어온 에피소드를 보기 전에, 기존 지식만으로 무슨 일이 있었을지 먼저 예측하는 단계다.

관리 시스템에서 관련 맥락 `S_in`을 불러오고(임계값으로 거른 유사도 검색), 에피소드 단서와 불러온 맥락만 가지고 예상 스키마를 만든다. 프롬프트 `P_ant`가 LLM한테 "주어진 맥락으로 실제 무슨 일이 있었을지 예측하라"고 시킨다. 결과는 기존 지식만으로 한 추측이다.

2번은 예상과 실제의 의미 차이에서 새로 알게 된 것을 뽑는다. 맞힌 부분은 버리고 틀린 부분만 남긴다.

-> 3번은 설명이 따로 없다. 뽑은 지식을 관리 시스템에 넣는 단계 같은데 어떻게 합치는지는 모르겠다??

## **3. Experiments**

### **3.1 Main Results (RQ1)**

LoCoMo 결과다.

| 모델 | NEMORI | 최강 베이스라인 | 개선 |
|---|---:|---|---:|
| gpt-4o-mini | 73.0 | Mem0 61.3 | +19.1% |
| gpt-4.1-mini | 80.8 | LangMem 73.4 | +10.1% |

Full Context도 두 모델 모두에서 조금 넘는다고 한다(80.8 vs 80.6, 73.0 vs 72.3).

뒤에서 볼 [Anatomy 리뷰](https://momozzing.github.io/paper%20review/Anatomy-of-Agentic-Memory-Paper-review/)의 ∆(Context Saturation Gap)로 보면 +0.2 ~ +0.7이다. 양수지만 작다.

LoCoMo는 평균 24K 토큰이라 전부 넣어도 되는 벤치마크라서 그렇다. 논문도 이걸 알고 더 긴 벤치마크로 넘어간다.

Temporal Reasoning이 특히 높다. gpt-4.1-mini에서 77.3(A-MEM 대비 +15.9%), gpt-4o-mini에서 67.6(Zep 대비 +14.8%)이다.

에피소드 중심으로 만들어두면 추론 부담 일부가 답변 생성 때가 아니라 메모리 만들 때로 옮겨가기 때문이라고 한다.

### **3.2 Efficiency Analysis (RQ2)**

비용을 본다.

메모리 구축 (gpt-4o-mini)

| 방법 | LLM 점수 | 호출 수 | 총 토큰(k) |
|---|---:|---:|---:|
| LangMem | 51.3 | 920.6 | 1,010.2 |
| Mem0 | 61.3 | 1,602.2 | 1,693.4 |
| A-MEM | 52.5 | 1,175.5 | 1,149.4 |
| MemoryOS | 54.5 | 1,016.1 | 526.5 |
| NEMORI | 73.0 | 373.2 | 322.9 |
| 개선 | +19.1% | −59.5% | −38.7% |

LLM 호출이 59.5% 줄었다. 프롬프트를 여러 개 쓰는 복잡한 파이프라인인데도 그렇다고, 논문도 좀 의외라고 한다.

응답 생성 (gpt-4o-mini)

| 방법 | LLM 점수 | 토큰 | 검색(ms) | 총 지연(ms) |
|---|---:|---:|---:|---:|
| FullContext | 72.3 | 23,653 | – | 5,806 |
| LangMem | 51.3 | 125 | 19,829 | 22,082 |
| Mem0 | 61.3 | 1,027 | 784 | 3,539 |
| A-MEM | 52.5 | 2,614 | 947 | 2,867 |
| MemoryOS | 54.5 | 1,560 | 9,910 | 15,220 |
| NEMORI | 73.0 | 2,745 | 787 | 3,053 |

Full Context 대비 토큰은 88%, 지연은 47% 줄었고 정확도는 더 높다.

LangMem은 토큰이 125개인데 검색에 19.8초를 쓴다. 앞에서 본 [Mem0 리뷰](https://momozzing.github.io/paper%20review/Mem0-Paper-review/)에서도 같은 패턴이었다. 토큰을 줄인다고 빨라지는 건 아니다.

### **3.3 Ablation Study (RQ3)**

예측 오차로 증류하는 것과, 들어온 에피소드에서 바로 지식을 뽑는 직접 증류를 비교한다.

| 모델 | 직접 증류 (NEMORI-s) | 예측 오차 (w/o e) | 개선 |
|---|---:|---:|---:|
| gpt-4o-mini | 52.0 | 65.0 | +25.0% |
| gpt-4.1-mini | 65.5 | 74.9 | +14.4% |

둘 다 답변할 때 에피소드 DB는 빼고 의미 DB만 쓴다. 다른 건 증류 방식뿐이다. 그 조건에서 예측 오차 쪽이 25% 높다.

native 관리 모듈은 거의 차이가 없다. 64.6 vs 65.0, 74.7 vs 74.9다.

논문은 이걸 management-agnostic 설계의 근거로 쓴다. 증류가 거의 다 하고, 관리 방식은 별로 상관없다는 것이다.

### **3.4 Third-Party Integration (RQ5)**

NEMORI를 다른 메모리 시스템 앞단의 증류 모듈로 붙여본다.

원시 메시지 대신 증류한 의미 지식을 넣어주면, A-MEM과 MemoryOS 둘 다 저장량이 45~64% 줄고 평균 성능은 유지(±4%)되고 일부 점수는 올라갔다(+1.9% ~ +6.1%)고 한다.

기존 시스템을 바꾸지 않고 앞에 끼워 넣을 수 있다는 얘기다.

### **3.5 Scalability Analysis (RQ6)**

LongMemEvalS 결과다.

| 질문 유형 | Full-context (101K tok) | NEMORI (3.7–4.8K tok) |
|---|---:|---:|
| Single-session Preference | 16.7 | 86.7 |
| Single-session Assistant | 98.2 | 92.9 |
| Temporal Reasoning | 60.2 | 72.2 |
| Multi-session | 51.1 | 55.6 |
| Knowledge Update | 76.9 | 79.5 |
| Single-session User | 85.7 | 90.0 |
| 평균 | 65.6 | 74.6 |

(gpt-4.1-mini 기준)

∆가 +9.0이다. LoCoMo에서 +0.2였던 게 여기서 커졌다.

맥락이 길어질수록 증류가 더 도움이 된다고 한다. Full Context는 입력이 길면 어텐션이 흐려지는데, NEMORI는 필요한 것만 검색해서 넣기 때문이라고 한다.

컨텍스트를 95~96% 줄이면서 정확도가 더 높다.

single-session-assistant에서만 진다(92.9 vs 98.2). 앞에서 본 [Zep 리뷰](https://momozzing.github.io/paper%20review/Zep-Paper-review/)에서도 같은 패턴이었다. 어시스턴트 발화는 추출하면서 흐려진다.

## **4. 지금 관점: importance 점수를 대체할 수 있나**

이 시리즈에서 다루는 시스템들이 뭘 기준으로 남길지 정했는지 모아보면 이렇다.

- Generative Agents : recency + relevance + importance (LLM이 1~10점)
- Mem0 : LLM이 "salient"한지 판단
- A-MEM : LLM이 노트로 만들 가치가 있는지 판단
- Experience-Following : 평가기가 실행 품질을 채점
- Janus : 넣은 뒤 옛 메모리와 비교
- NEMORI : 기존 지식으로 예측에 실패한 만큼

NEMORI만 기준이 데이터에서 나온다. 나머지는 "무엇이 중요한가"를 사람이나 LLM이 정한다.

실제로 쓴다면 위의 제3자 통합처럼 지금 쓰는 메모리 앞에 끼우는 게 제일 현실적일 것 같다. 저장이 45~64% 주는데 성능은 유지된다.

-> 다만 예상 스키마 만드는 데 LLM 호출이 하나 더 붙는다. 구축 총량이 줄어든 건(−59.5%) 메시지 단위가 아니라 에피소드 단위로 처리해서인 것 같은데, 턴마다 돌리는 구조라면 어떻게 될지??

-> 관측 창 길이 `w`도 하이퍼파라미터라 대화 특성에 맞게 조정해야 할 것 같다. 논문도 창 길이별 성능을 따로 쟀다.

-> 어시스턴트 발화는 약하다. LongMemEvalS single-session-assistant에서 Full Context보다 5.3점 낮다. 안내나 추천을 많이 하는 챗봇이면 그 부분은 원문을 따로 남겨야 할 것 같다.

## **5. Conclusion**

conclusion 부분을 보면, 인지과학에서 아이디어를 가져와 증류 단계에서 경험이 나중에 쓸모 있을지를 판단하는, 학습이 필요 없는 프레임워크를 만들었다고 한다. 예측 오차가 기억으로 남길 가치가 있다는 것이다.

마지막 문장은 이렇다. agentic하다고 해서 휴리스틱일 필요는 없고, 데이터 성질을 보고 기준을 세울 수 있다고 한다. 오래 일관되게 동작해야 하는 에이전트는 대부분 사람이 직접 관리하지 못하는 곳에서 돌기 때문에 이게 점점 중요해진다고 한다.

한계도 적어뒀다. NEMORI는 증류에만 집중했고 관리와 검색은 단순하게 했다. 여기 벤치마크에서는 그걸로 충분했지만, 메모리를 더 복잡하게 추론해야 하는 과제에서는 병목이 될 수 있다고 한다.

여태까지 나온 방법들이 무엇을 남길지를 점수나 LLM 판단으로 정했다면, 이 방법은 기존 지식으로 예측해보고 틀린 부분만 남긴다.

나중에 볼 [MemMachine 리뷰](https://momozzing.github.io/paper%20review/MemMachine-Paper-review/)는 "저장보다 검색을 손봐라"인데, 여기서는 반대로 증류에 집중하고 검색은 단순하게 간다. 둘 다 각자 벤치마크에서 이긴다.

-> 뭐가 병목인지는 데이터마다 다른 것 같다.

다음은 [Memory in the Age of AI Agents](https://momozzing.github.io/paper%20review/Memory-in-the-Age-of-AI-Agents-Paper-review/)다. 장기기억·단기기억 이분법 대신 Forms·Functions·Dynamics 세 축으로 에이전트 메모리를 다시 정리한 107쪽짜리 서베이다.
