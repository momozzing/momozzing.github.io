---
date: 2026-09-26 12:00:00 +0900
title: "Multi-Layered Memory Paper review"
excerpt: "working·episodic·semantic 세 계층을 나누고 하나씩 떼어보는 ablation. 계층 도입 판단에 쓸 만한 수치는 있으나 서지 정보에 검증 못 한 부분이 있다."
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

Multi-Layered Memory Architectures for LLM Agents: An Experimental Evaluation of Long-Term Context Retention

[https://arxiv.org/abs/2603.29194](https://arxiv.org/abs/2603.29194)

Multi-Layered Memory는 Fulloop에서 만든 에이전트 메모리 구조다.

대화 이력을 working·episodic·semantic 세 계층으로 나누고, 계층을 하나씩 떼어보는 ablation으로 각 계층이 얼마나 기여하는지 잰다.

2026년 3월 31일에 arXiv에 올라왔고, 저자 2명에 8쪽이다.

앞 리뷰들에서 계속 미뤄둔 질문이 계층을 몇 개 둘 것인가였다. [CoALA](https://momozzing.github.io/paper%20review/CoALA-Paper-review/)는 네 개(working·episodic·semantic·procedural)를 말했고 [Zep](https://momozzing.github.io/paper%20review/Zep-Paper-review/)은 세 개(episode·entity·community)를 썼다. 그런데 계층을 하나 뺐을 때 얼마나 나빠지는지 잰 실험은 없었다. 이 논문의 ablation이 그걸 보여준다.

다만 이 논문은 앞의 논문들보다 근거가 약하다. 확인이 안 되는 부분을 먼저 적어두고 읽는다.

좀 더 자세히 알아보자.

## **1. 검증 못 한 부분**

앞에서 본 [Anatomy 리뷰](https://momozzing.github.io/paper%20review/Anatomy-of-Agentic-Memory-Paper-review/)에서 평가가 타당한지 따져보라고 했으니, 그 기준을 여기에도 대본다.

1. 같은 벤치마크를 두 번 센 것 같다.

abstract를 보면 *"Experiments on LOCOMO, LOCCO, and LoCoMo"* 라고 쓴다. 표에서도 `LOCOMO`랑 `LoCoMo`를 다른 행으로 둔다. 본문 전체에서 LOCCO가 21회, LOCOMO가 15회, LoCoMo가 14회 나온다.

LoCoMo는 Maharana et al. (2024)의 대화 메모리 벤치마크다. 그런데 이 논문의 `LOCOMO` 행은 참조가 [10] HiAgent(계층적 working memory 논문)로 되어 있다.

-> 그러면 `LOCOMO`는 데이터셋이 아니라 비교 대상 시스템 아닌가?? 데이터셋이 세 개가 아니라 두 개일 수도 있다. 본문에 데이터셋 설명이 없어서 확실하게는 모르겠다.

2. 베이스라인이 참조 번호로만 나온다.

표에 `[10]`, `[13]`, `[14]`, `[18]`, `[20]`으로만 적혀 있다. 따라가 보면 HiAgent, Truth-Maintained Memory Agent, Jia et al., EvolveMem, LaVa다.

-> 서로 다른 과제랑 지표를 쓰는 논문들인데, 한 표에서 바로 비교해도 되는 조건인지 알 수가 없다.

3. 날짜가 안 맞는다.

PDF 머리에는 *"Published online in June 2025"* 라고 찍혀 있는데 arXiv 제출은 2026년 3월 31일이다. `journal_ref`는 비어 있다. 어디에 실렸는지 확인이 안 된다.

4. ∆가 없다.

full-context 베이스라인이랑 비교를 안 해서, Anatomy 논문에서 말한 Context Saturation Gap을 계산할 수 없다.

-> 이 네 가지 때문에 절대 수치는 인용하기 어려울 것 같다. 대신 같은 시스템 안에서 계층을 하나씩 떼는 ablation은 조건이 같으니까 상대 비교로는 볼 만하다. 아래는 그 부분 위주로 본다.

## **2. 문제 설정**

연구 문제는 이렇다고 한다.

여러 세션에 걸친 대화가 있을 때, 컨텍스트가 2차로 늘거나 추론 비용이 커지지 않으면서 세션을 넘어 안정적인 의미 표현을 유지하려면 어떻게 해야 하는가.

기존 방법의 한계를 네 가지로 정리한다.

1. 계층적 working memory : 컨텍스트는 줄이지만 세션 안의 과제만 보고, 세션 간 지속성이 없다
2. 검색 기반 통합 : 관련된 걸 더 잘 고르지만 거짓 기억이 쌓이기 쉽다
3. 파라미터 기반 메모리 : 보존율은 재지만 의미가 안정적인지 구조적으로 통제하지 않는다
4. 컨텍스트 압축 : 토큰은 줄지만 세션 간에 다층 메모리를 유지하지 않는다

정리하면 효율은 올리는데 보존이 안정적이지 않거나, 재현율은 올리는데 드리프트를 못 막는다고 한다.

## **3. MLMF**

대화 이력을 세 계층으로 나눈다.

- Working memory : 제한된 창 안에서 최근 상호작용을 보존
- Episodic memory : 압축한 세션 요약을 쌓음
- Semantic memory : 구조화된 엔티티 단위 추상을 유지

여기에 장치 두 개를 더한다.

- Adaptive retrieval gating : 계층마다 검색 중요도를 조절하는 가중치
- Retention regularization : 세션을 넘어가면서 의미가 드리프트하는 걸 막는 손실항

[서베이](https://momozzing.github.io/paper%20review/Memory-in-the-Age-of-AI-Agents-Paper-review/)의 Forms 분류로 보면 Token-level의 Hierarchical(3D)이고, Functions로는 Working + Factual 조합이다.

Experiential은 없다. CoALA의 procedural에 해당하는 계층도 없다.

## **4. Ablation**

모듈을 하나씩 떼면서 F1, 6기간 보존율, 거짓 기억률(FMR)을 잰다.

| 변형 | F1 | 보존율(6기간) | FMR |
|---|---:|---:|---:|
| −M(s) semantic 계층 제거 | 0.591 | 50.84% | 6.4% |
| −M(e) episodic 통합 제거 | 0.602 | 52.13% | 6.1% |
| −L_ret 보존 손실 제거 | 0.608 | 53.27% | 6.9% |
| adaptive gating 제거 | 0.604 | 52.98% | 6.5% |
| 전체 MLMF | 0.618 | 56.90% | 5.1% |

표를 보면,

1. semantic 계층을 빼면 보존율이 제일 많이 떨어진다. 56.90% → 50.84%로 6.06%p다. F1도 0.618 → 0.591로 제일 많이 준다. episodic을 빼면 4.77%p 떨어져서 semantic보다는 작다.
2. 보존 손실을 빼면 거짓 기억이 제일 많이 는다. 5.1% → 6.9%다. F1은 0.010만 떨어져서 제일 작은데 FMR만 튄다.

-> 1번을 보면 세션 요약(episodic)만으로는 부족하고, 엔티티 단위로 정리해둬야 오래 남는다는 것 같다.

-> 2번은 드리프트를 막는 장치가 정확도보다는 환각을 막는 쪽으로 효과가 있다는 얘기 같다. 정확도만 보면 효과가 거의 없어 보이는데, 없는 걸 지어내는 비율로 보면 제일 크게 기여한다.

앞에서 본 [LongMemEval 리뷰](https://momozzing.github.io/paper%20review/LongMemEval-Paper-review/)에서 ABS(회피)를 따로 능력으로 둔 것과 이어지는 얘기다.

## **5. 지금 관점: 계층을 몇 개 둘 것인가**

ablation만 놓고 보면 순서가 이렇게 나온다.

1. semantic (엔티티 추상) : 빼면 보존율 −6.06%p, F1 −0.027로 제일 큼
2. episodic (세션 요약) : 빼면 −4.77%p
3. adaptive gating : −3.92%p
4. retention 제약 : F1 영향은 작지만 FMR에는 제일 크게 기여

semantic 계층이 1순위라는 건 Zep, Mem0랑 맞는다. 둘 다 엔티티를 중심에 둔다. Zep은 semantic entity subgraph, Mem0g는 엔티티 노드다.

-> 세션 요약만 쌓는 설계(초기 MemoryBank 계열)가 약했던 이유가 이거인 것 같다.

앞에서 본 [Experience-Following 리뷰](https://momozzing.github.io/paper%20review/Experience-Following-Paper-review/)에서는 과거 실행 경험이 행동을 직접 바꿨는데, 여기 ablation에는 experiential 계층이 아예 없다. 사실 기억만 다룬 실험이다.

FMR을 따로 재는 건 가져와 볼 만하다.

-> 정확도가 같아도 지어내는 비율은 다를 수 있다. 업무용 챗봇에서는 지어내는 쪽이 더 큰 문제다.

다만 1장에 적은 문제들 때문에 절대 수치(0.618, 56.90%, 5.1%)는 인용하지 않는 게 나을 것 같다. ablation의 상대 순서만 참고한다.

## **6. Conclusion**

conclusion 부분을 보면, 계층적 메모리 분해에 adaptive retrieval gating이랑 retention regularization을 붙인 프레임워크를 제안했다고 한다. working·episodic·semantic을 나눠서 세션 간 드리프트를 막으면서 컨텍스트가 늘어나는 것도 막는다.

그리고 ablation으로 semantic consolidation(의미 통합)이랑 보존 통제가 장기 성능에 기여한다는 걸 확인했다고 한다.

한계 절은 없다. 8쪽에 Limitations도 Future Work도 없다.

여태까지 본 논문들이 계층을 몇 개 둘지 각자 정해서 썼다면, 이 논문은 계층을 하나씩 빼보면서 어떤 계층이 얼마나 기여하는지를 쟀다.

앞에서 본 [SYNAPSE](https://momozzing.github.io/paper%20review/SYNAPSE-Paper-review/)가 episodic이랑 semantic을 잇는 방식까지 다뤘다면, 이 논문은 두 계층을 나란히 두기만 한다.

-> 검증이 안 되는 부분이 많아서 계층 ablation 표 하나 말고는 가져오기 어렵다. 그래도 그 표는 다른 데서 못 본 수치라서 읽었다.

다음은 [MemMachine](https://momozzing.github.io/paper%20review/MemMachine-Paper-review/)이다. 원문을 그대로 보관하고 LLM 추출을 최소화하는 개인화 메모리 시스템으로, 저장이 아니라 검색을 손봐야 한다는 걸 ablation으로 보인 논문이다.
