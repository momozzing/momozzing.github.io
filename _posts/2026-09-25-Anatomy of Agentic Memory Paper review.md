---
date: 2026-09-25 15:00:00 +0900
title: "Anatomy of Agentic Memory Paper review"
excerpt: "지금까지의 메모리 벤치마크 수치를 어떻게 읽어야 하는가. 벤치마크 포화, F1과 의미의 어긋남, 백본 의존, 그리고 agency tax를 실측한 논문."
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

Anatomy of Agentic Memory: Taxonomy and Empirical Analysis of Evaluation and System Limitations

[https://arxiv.org/abs/2602.19320](https://arxiv.org/abs/2602.19320)

Anatomy of Agentic Memory는 UT Dallas · UC Davis · Texas A&M에서 쓴 에이전트 메모리 평가 논문이다.

2026년 2월 22일에 나왔고 5월 20일에 v2가 올라왔다. 저자 11명에 19쪽이다.

앞 리뷰들에서 수치를 많이 가져왔다. Mem0의 66.88, Zep의 71.2, A-MEM의 순위 1.0. 뒤에서 볼 ReFind의 58.2도 그런 수치다.

이 논문은 그 수치들을 어떻게 읽어야 하는지를 따진다. 결론부터 말하면 생각보다 약한 근거인 게 많다.

좀 더 자세히 알아보자.

## **1. Introduction**

introduction 부분을 보면 문제 제기가 직설적이다.

아키텍처는 빠르게 발전했는데 실험적 근거는 약하다고 한다. 벤치마크는 대부분 작고, 평가 지표는 의미랑 어긋나고, 성능이 백본 모델에 따라 크게 달라지고, 시스템 비용은 잘 안 본다는 것이다.

질문 네 개로 정리한다.

1. 벤치마크 타당성 : 우리가 재는 게 메모리인가 컨텍스트 길이인가?
2. 지표 신뢰성 : 어휘 지표가 의미적 일관성을 잡아낼 수 있는가?
3. 시스템 효율 : 지연과 비용의 "agency tax"
4. 백본 민감도 : 오픈웨이트 모델에서 메모리 연산의 "silent failure"

## **2. Taxonomy of Agentic Memory**

Memory-Augmented Generation(MAG) 시스템을 메모리 구조에 따라 넷으로 나눈다.

- Lightweight Semantic Memory : 가벼운 의미 저장소
- Entity-Centric and Personalized Memory : 엔티티 중심 개인화
- Episodic and Reflective Memory : 일화·반성
- Structured and Hierarchical Memory : 구조화·계층

앞에서 본 것들을 넣어보면 Mem0가 둘째, A-MEM이 셋째, Zep이 넷째다.

분류는 앞에서 본 [서베이](https://momozzing.github.io/paper%20review/Memory-in-the-Age-of-AI-Agents-Paper-review/)랑 크게 다르지 않다. 실제 분석은 뒤에서부터 나온다.

## **3. Evaluation and Pain Points**

### **3.1 Benchmark Scalability: The Context Saturation Risk**

벤치마크 포화 문제다.

에이전트 메모리를 쓰는 이유는 유한한 컨텍스트 창을 넘어서 추론하려는 것이다.

그런데 컨텍스트 창이 128k에서 1M으로 늘어나면서 많은 벤치마크가 포화될 위험이 생겼다. 필요한 정보가 전부 프롬프트 하나에 들어가면 외부 메모리가 필요 없어 보인다.

#### **3.1.1 Dimensions of Limitation**

포화 위험을 세 가지 축으로 본다.

- Volume (총 토큰 양) : HotpotQA(~1k), MemBench(~100k)는 128k 창에 다 들어간다. 포화 위험이 높다
- Interaction depth (상호작용 깊이)
- Entity diversity (엔티티 다양성)

겉보기 난이도가 아니라, 이런 구조적 특성이 long-context LLM이 감당할 수 있는 범위를 넘느냐로 포화 위험이 정해진다고 한다.

#### **3.1.2 Context Saturation Gap as an Empirical Diagnostic**

그래서 진단 지표를 하나 제안한다.

```
∆ = Score_MAG − Score_FullContext
```

같은 백본, 같은 평가 방식에서 메모리 에이전트 점수랑 대화를 통째로 넣은 full-context 점수의 차이다.

∆가 크게 양수면 외부 메모리가 증거를 전부 프롬프트에 넣는 것보다 낫다는 뜻이다. 특히 창 밖(out-of-window)이나 lost-in-the-middle 상황에서 그렇다고 한다.

다만 ∆는 합격/불합격 기준이 아니라 진단 신호로 봐야 한다고 한다. full-context가 잘 나와도 효율, 갱신 가능성, 강건성, 증거 충실성 쪽에서 메모리를 볼 수 있기 때문이다.

이 시리즈 리뷰들 수치로 ∆를 계산해보면 이렇게 된다. ReFind는 뒤에서 볼 논문이다.

| 논문 | MAG | Full-context | ∆ |
|---|---:|---:|---:|
| MemGPT (DMR, GPT-4 Turbo) | 93.4 | 94.4 (Zep 측정) | −1.0 |
| Mem0 (LOCOMO, 전체 J) | 66.88 | 72.90 | −6.02 |
| Zep (LongMemEval, GPT-4o) | 71.2 | 60.2 | +11.0 |
| ReFind (LongMemEval-S, GPT-5-mini) | 93.2 | 82.0 | +11.2 |

MemGPT와 Mem0는 ∆가 음수다. 대화를 통째로 넣는 게 더 정확하다.

-> 그러면 이 두 시스템은 정확도보다는 지연이랑 토큰 비용 때문에 쓰는 거라고 봐야 할 것 같다.

Zep과 ReFind는 ∆가 +11 정도다. LongMemEval이 115k 토큰이라 포화되지 않는 벤치마크여서 그렇다.

그래서 작고 얕은 데이터셋에서는 full-context 베이스라인이랑 같이 평가해야 메모리 덕분에 좋아졌다고 말할 수 있다고 한다.

### **3.2 LLM-as-a-Judge Evaluation**

F1과 의미가 어긋난다는 부분이다.

F1, BLEU 같은 어휘 지표는 토큰이 얼마나 겹치는지를 본다. 에이전트 메모리처럼 정확히 찾아서 일관되게 종합해야 하는 과제에는 부족하다고 한다.

LoCoMo에서 여섯 아키텍처를 F1 순위와 LLM 판정 순위로 각각 매겨 비교한다. 결과가 어긋난다.

| 시스템 | F1 | F1 순위 | 의미 순위 |
|---|---:|---:|---:|
| A-Mem | 0.116 | 5 | 4 |
| SimpleMem | 0.268 | 높음 | 의미 점수 < 0.30 |

A-Mem은 의미 기준으로는 괜찮은데(순위 4) F1에서는 낮게 나온다(순위 5, 0.116). 원문 단어를 그대로 쓰지 않기 때문이다.

반대로 SimpleMem은 복잡한 답을 잘 종합하지 못하는데도(의미 점수 0.30 미만) F1은 0.268로 상대적으로 높다.

논문은 F1만 보고 최적화하면 추론이나 메모리 통합보다 표면적인 암기를 좋아하게 된다고 한다.

앞에서 본 [Mem0](https://momozzing.github.io/paper%20review/Mem0-Paper-review/)에서 A-Mem의 F1이 낮았는데, 여기서 보면 A-MEM이 나빴다기보다 지표가 추상적인 메모리 시스템에 불리했던 것이다.

LLM-as-a-judge가 프롬프트에 과적합되는 게 아니냐는 걱정도 있는데, 서로 다른 프롬프트 세 개로 확인해보니 상대 순서는 유지됐다고 한다. 그래도 프롬프트 설계는 조심해야 한다고 덧붙인다.

### **3.3 Backbone Sensitivity and Format Stability**

에이전트 메모리에서는 백본 모델이 질문에 답하는 것과 메모리 연산(갱신·통합)을 둘 다 해야 한다. 그래서 오래 쓰려면 출력 형식을 정확히 지켜야 한다.

API 모델(gpt-4o-mini)과 오픈웨이트(Qwen-2.5-3B)를 비교한다.

| 백본 | 방법 | 답변 점수 | 형식 오류율 |
|---|---|---:|---:|
| gpt-4o-mini | SimpleMem | 0.289 | 1.20% |
| gpt-4o-mini | Nemori | 0.781 | 17.91% |
| Qwen-2.5-3B | SimpleMem | 0.102 | 4.82% |
| Qwen-2.5-3B | Nemori | 0.447 | 30.38% |

논문은 이걸 silent failure라고 부른다.

에이전트가 당장은 대화를 잘하는데, 쓰기 연산이 실패해서 장기 메모리가 망가진다는 것이다. 대화는 멀쩡해 보이는데 메모리가 조용히 망가진다.

아키텍처 복잡도에 따라 영향이 다르다고 한다.

- Append-only 시스템 : 구조화 생성이 적어서 비교적 버틴다
- 그래프 기반·일화 기반 : 엔티티 추출, 관계 구성, 중복 제거 때문에 형식 오류가 많이 늘어난다. 약한 백본에서는 구조가 불안정해지거나 메모리 유지가 무너지는 경우가 많다

앞에서 본 [A-MEM](https://momozzing.github.io/paper%20review/A-MEM-Paper-review/)에서는 "조직의 품질이 기반 모델 능력에 좌우될 수 있다"고 걱정만 했는데, 여기서는 30.38%라는 숫자가 나왔다.

gpt-4o-mini에서도 Nemori 형식 오류가 17.91%다. API 모델이라고 안전한 것도 아니다.

### **3.4 System Performance Evaluation**

논문이 agency tax라고 부르는 부분이다.

정확도 말고 지연과 비용도 봐야 한다. 읽기만 하는 RAG랑 다르게 에이전트 메모리는 추출·갱신·통합 같은 유지보수 연산이 더 붙는다.

| 방법 | 검색 `T_read` | 생성 `T_gen` | 합계(초) | 구축 시간(h) | 구축 토큰(k) |
|---|---:|---:|---:|---:|---:|
| Full Context | N/A | 1.726 | 1.726 | N/A | N/A |
| LOCOMO | 0.415 | 0.368 | 0.783 | 0.86 | 1,623 |
| A-Mem | 0.062 | 1.119 | 1.181 | 15.00 | 1,486 |
| MemoryOS | 31.247 | 1.125 | 32.372 | 7.83 | 4,043 |
| Nemori | 0.254 | 0.875 | 1.129 | 3.25 | 7,044 |
| MAGMA | 0.497 | 0.965 | 1.462 | 7.28 | 2,725 |
| SimpleMem | 0.009 | 1.048 | 1.057 | 3.45 | 1,308 |

두 가지가 보인다.

1. MemoryOS는 검색이 31.2초다. 턴마다 32초면 대화형으로는 못 쓴다. 정확도 표에서는 안 보이던 부분이다.
2. A-Mem은 구축이 15시간이다. 검색은 0.062초로 빠른 편인데 오프라인 구축 비용이 제일 크다. 앞에서 본 [A-MEM](https://momozzing.github.io/paper%20review/A-MEM-Paper-review/)에서는 쓰기 비용이 표에 없었는데 여기서 나온다. 노트 구성 + 링크 판단 + 진화가 전부 LLM 호출이라 그런 것 같다.

"intelligence tax"라는 것도 있다. 어떤 시스템은 정확도는 높은데 토큰을 3M 쓴다. MAGMA는 2.7M으로 균형이 더 낫다고 한다.

유지보수 비용은 비동기로 처리되는 경우가 많아서 표에서 뺐다고 한다. 그러니까 이 표도 전체 비용은 아니다.

## **4. 지금 관점: 앞 리뷰들을 다시 읽기**

이 논문을 읽고 나니 앞 리뷰들 수치가 좀 다르게 보인다.

-> 메모리 시스템을 볼 때는 같은 백본의 full-context 점수를 옆에 놓고 봐야겠다. 그게 없으면 컨텍스트 창으로 풀리는 문제를 메모리로 푼 걸 수도 있다. Mem0는 ∆가 −6.02니까, Mem0를 쓰는 이유는 정확도보다는 p95 0.2초 쪽이다.

-> F1 수치도 그대로 믿기는 어렵다. 추상적인 메모리 시스템은 F1에서 불리하다. LLM 판정도 같이 보되 프롬프트를 바꿔도 순서가 유지되는지 봐야 할 것 같다.

-> 형식 오류율은 직접 재봐야겠다. 메모리 시스템을 붙이면 JSON 파싱 실패율 정도는 로그로 남겨야 할 것 같다. 17~30%가 조용히 실패하고 있을 수도 있다.

-> 백본을 바꾸면 구조화 메모리는 다시 확인해야 한다. append-only는 버티는데 그래프·일화 계열은 무너질 수 있다. 모델을 자주 바꾸는 환경이면 단순한 구조가 나을 수도 있다.

-> 검색 지연도 따로 재야 한다. MemoryOS의 31초는 정확도 표에 안 나온다.

## **5. Conclusion and Future Directions**

conclusion 부분을 보면 에이전트 메모리가 아키텍처뿐 아니라 평가 타당성, 확장성, 강건성에서도 제약을 받는다고 정리한다. 제안은 두 가지다.

1. 벤치마크와 평가를 다시 설계할 것. 앞으로 벤치마크는 포화를 고려해야(saturation-aware) 한다고 한다. 컨텍스트 창이 커지면 full-context로 풀리는 과제가 많아져서 외부 메모리 효과를 따로 보기 어렵다. 태스크 양, 시간적 깊이, 엔티티 다양성, 장거리 의존성을 늘리고 Context Saturation Gap(∆)을 진단 신호로 쓰자고 한다.
2. 확장 가능하고 강건한 시스템을 설계할 것. 구조화 메모리는 추론은 좋아지지만 유지보수 비용이 들고, 가벼운 방식은 효율적이지만 추상화가 부족할 수 있다. 쓰기 지연, 유지보수 처리량, 사용자 쪽 비용을 따로 보고, 백본에 맞춘 메모리 연산을 써야 한다고 한다.

여태까지 앞의 논문들이 각자 좋아 보이는 수치를 내놨다면, 이 논문은 그 수치가 뭘 뜻하는지 다시 재보라고 한다. ∆ 계산이랑 형식 오류율은 메모리 시스템을 볼 때 같이 봐야 할 것 같다.

다음은 [AMV-L](https://momozzing.github.io/paper%20review/AMV-L-Paper-review/)이다. 메모리가 쌓이면 검색이 느려지는데, 검색 후보군 크기를 직접 묶어서 느린 요청(꼬리 지연)을 줄이는 논문이다.
