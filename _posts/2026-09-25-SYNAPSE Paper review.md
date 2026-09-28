---
date: 2026-09-25 12:00:00 +0900
title: "SYNAPSE Paper review"
excerpt: "벡터 유사도 대신 활성 확산으로 관련성을 만든다. 측면 억제와 시간 감쇠로 적대적 질의를 96.6 F1로 거절하고, 질의당 814토큰만 쓴다."
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

SYNAPSE: Empowering LLM Agents with Episodic-Semantic Memory via Spreading Activation

[https://arxiv.org/abs/2601.02744](https://arxiv.org/abs/2601.02744)

SYNAPSE는 University of Georgia 외 5개 기관에서 만든 에이전트 메모리 구조다.

벡터 유사도 대신 활성 확산(spreading activation)으로 기억 사이의 관련성을 찾고, 모르는 질문에는 모른다고 답하게 만든다.

2026년 1월 6일에 나왔고 2월 16일에 v3가 올라왔다. 저자 11명, 17쪽이다.

이름이 같은 다른 논문이 있다. Synapse: Trajectory-as-Exemplar Prompting([2306.07863](https://arxiv.org/abs/2306.07863))은 다른 논문이다. 검색하면 헷갈리기 쉽다.

이 논문은 episodic이랑 semantic 계층을 어떻게 잇느냐를 다룬다. 뒤에서 볼 [Multi-Layered Memory](https://momozzing.github.io/paper%20review/Multi-Layered-Memory-Paper-review/)의 MLMF는 두 계층을 나란히 두기만 하는데, 여기서는 둘을 잇는다. 그리고 그 연결을 미리 계산해두지 않는다.

좀 더 자세히 알아보자.

## **1. Introduction**

문제를 Contextual Tunneling(또는 Contextual Isolation)이라고 부른다.

기존 검색 증강 방식은 장기 에이전트 메모리가 서로 끊겨 있는 문제를 해결하지 못한다고 한다.

RAG는 이력을 벡터 DB에 넣고 의미 유사도로 꺼낸다. 사실 하나를 찾는 데는 괜찮은데, 서로 떨어진 기억을 엮어야 하는 상황에서는 안 된다는 것이다.

그래서 인지과학 쪽에서 방법을 가져온다. 메모리를 동적 그래프로 두고, 관련성은 미리 계산해둔 링크가 아니라 활성 확산에서 나오게 한다고 한다.

앞에서 본 [A-MEM](https://momozzing.github.io/paper%20review/A-MEM-Paper-review/)은 LLM으로 링크를 미리 걸어뒀는데, SYNAPSE는 질의가 올 때 에너지를 흘려서 그때그때 관련된 부분그래프를 찾는다.

-> 나중에 볼 [ReFind 리뷰](https://momozzing.github.io/paper%20review/ReFind-Paper-review/)의 "결정을 질의 시점까지 미룬다"는 것과 비슷하다. 이번엔 그래프 쪽이다.

## **2. Methodology**

### **2.1 Unified Episodic-Semantic Graph**

메모리를 방향 그래프 `G = (V, E)`로 두고, 노드를 두 종류로 나눈다.

- Episodic `V_E` : 상호작용 턴 하나하나 `(c_i, h_i, τ_i)`. 텍스트, 임베딩, 타임스탬프. 턴마다 만듦
- Semantic `V_S` : 엔티티나 선호 같은 추상 개념. LLM으로 추출하고 N=5턴마다 실행

중복은 임베딩 유사도 `τ_dup = 0.92`로 거른다. 임베딩 모델은 all-MiniLM-L6-v2다.

엣지는 두 종류다.

- Temporal Edges : 순서대로 에피소드를 이음 (`v^e_t → v^e_{t+1}`)
- Abstraction Edges : 같은 통합 창(N=5) 안의 에피소드와 개념을 양방향으로 이음

두 번째가 중요하다. 의미가 직접 비슷하지 않아도, 같이 나왔다는 것(co-occurrence)만으로 개념을 이을 수 있다고 한다. 논문 예시로는 "Mark" ↔ "스키 여행".

-> 벡터 유사도로는 "Mark"랑 "스키 여행"이 안 붙는다. 같은 시간대에 나왔다는 것만으로 잇는 것이다.

전체 구조는 논문 그림 하나에 다 들어 있다.

![SYNAPSE 전체 구조 (논문 Figure 1)](https://momozzing.github.io/assets/images/synapse/fig1-synapse-overview.png)

왼쪽은 질의가 어휘·의미 두 트리거로 그래프에 에너지를 넣는 부분, 가운데는 활성 확산, 오른쪽은 세 신호로 순위를 다시 매기는 부분이다.

질의에 없는 "Mark" 노드가 다리 역할로 활성화돼서 "스키 여행"과 "연애"를 잇는다고 한다.

### **2.2 Cognitive Dynamics: Spreading Activation**

Collins와 Loftus(1975)의 사람 의미기억 모델에서 가져왔다고 한다.

먼저 초기화(Initialization)다. 질의 `q`가 오면 먼저 앵커 노드를 찾는다. 두 가지 경로를 쓴다.

- Lexical Trigger : BM25 희소 검색. "Kendall" 같은 고유명사를 정확히 매칭
- Semantic Trigger : dense 검색. "스키 여행" 같은 개념적으로 비슷한 것을 찾음

둘의 Top-k 합집합이 앵커가 되고, 앵커에만 에너지를 넣는다.

앞에서 본 [Zep](https://momozzing.github.io/paper%20review/Zep-Paper-review/)에서도 sparse+dense 조합을 썼는데, 여기서도 나온다. 뒤에서 볼 [ReFind](https://momozzing.github.io/paper%20review/ReFind-Paper-review/)도 같은 조합을 쓴다.

-> 세 논문이 따로 같은 결론에 온 셈이다. 고유명사는 어휘 검색, 주제는 dense 검색이 필요하다.

다음은 전파(Propagation with Fan Effect)다. ACT-R(Anderson, 1983)을 따라서 주의가 나뉘는 걸 모델링한다.

```
u^(t+1)_i = (1−δ)·a^(t)_i + Σ_{j∈N(i)} S · w_ji · a^(t)_j / fan(j)
```

`S = 0.8`이 확산 계수, `fan(j)`가 나가는 엣지 수다. 연결이 많은 노드일수록 이웃 하나하나에 주는 에너지가 줄어든다. 허브 노드 하나가 전부를 활성화하는 걸 막는 장치다.

엣지 가중치는 종류마다 다르다.

- 시간 엣지 : `w_ji = e^{−ρ|τ_i−τ_j|}` (시간 감쇠 `ρ = 0.01`)
- 의미 엣지 : `w_ji = sim(h_i, h_j)`

마지막은 측면 억제(Lateral Inhibition)다. 주의 선택을 모델링한 부분이다. 강하게 활성화된 개념이 경쟁 개념을 누르고 나서 발화한다.

이게 뒤에서 적대적 질의를 거절하는 데 쓰인다.

### **2.3 Triple-Signal Hybrid Retrieval**

점수는 세 개를 합친다.

```
S(v_i) = λ₁·sim(h_i,h_q) + λ₂·a^(T)_i + λ₃·PageRank(v_i)
```

세 신호가 서로 다른 역할을 한다고 한다.

- `sim` : 의미 유사도
- `a^(T)` (활성) : 국소 맥락 신호. 질의마다 관련성이 퍼짐
- PageRank : 전역 구조 사전확률. 질의랑 상관없이 중요한 허브(주요 인물 등)를 먼저

이렇게 나누는 이유는, 새로 나왔지만 지금 질의랑 관련 있는 세부 정보가 전역 허브에 묻히지 않게 하려는 것이라고 한다.

효율 쪽으로는 점수를 캐시해두고 통합 시점(N=5턴)에만 갱신한다. 그래서 질의 지연이 이력 길이 `T`랑 상관없이 유지된다고 한다.

기본 `k = 30`이다.

### **2.4 Uncertainty-Aware Rejection**

없는 엔티티에 대해 묻는 적대적 질의를 다루는 부분이다. 사람 기억의 "Feeling of Knowing"(FOK)에서 아이디어를 가져왔다고 한다. 두 단계다.

1. 신뢰도 기반 게이팅

검색 신뢰도 `C_ret`를 최상위 노드의 활성 에너지로 둔다. `C_ret < τ_gate`(보정값 0.12)이면 부정 확인 프로토콜로 넘어가서 질의를 바로 거절한다.

기억 흔적이 부족할 때 뇌가 답을 안 만들어내는 걸 흉내 낸 것이라고 한다.

2. 명시적 검증 프롬프팅

게이트를 통과한 애매한 경우에는 "엄격한 증거" 조건을 건다.

*"Is this EXPLICITLY mentioned? If not, output 'Not mentioned'."*

생성 모델이 파라미터 지식으로 지어낸 것과 검색에 근거한 것을 구분하게 만든다.

앞에서 본 [LongMemEval](https://momozzing.github.io/paper%20review/LongMemEval-Paper-review/)에서는 ABS(회피)를 다섯 능력 중 하나로 뒀는데, 여기서는 그걸 시스템에서 직접 구현했다.

## **3. Experiments**

LoCoMo 벤치마크, GPT-4o-mini 기준이다.

### **3.1 Main Results**

전체부터 보자.

| 시스템 | 가중 평균 F1 |
|---|---:|
| A-Mem | 33.3 |
| AriGraph | 33.7 |
| Zep | 39.7 |
| SYNAPSE | 40.5 |

적대적 범주를 뺀 가중 평균이다. A-Mem보다 +7.2점이고 태스크 순위 1.0이라고 한다.

범주별로 나누면 이렇다.

| 범주 | A-Mem | SYNAPSE |
|---|---:|---:|
| Temporal Reasoning | 45.9 | 50.1 |
| Multi-Hop | 27.0 | 35.7 |
| Adversarial | LoCoMo 69.2 | 96.6 |

표를 보면,

1. Multi-Hop에서 +8.7점으로 제일 많이 오른다. 활성 확산이 중간 노드를 거쳐 관련성을 퍼뜨려서, 벡터 검색만으로는 못 잇는 사실들을 잇는다고 한다.
2. Temporal에서 +4.2점. 시간 감쇠 덕분에 의미는 비슷하지만 오래된 기억보다 최근 정보를 먼저 쓴다고 한다.
3. Adversarial은 96.6이다. 베이스라인들은 거절하는 장치가 없어서 그럴듯한 답을 지어낸다.

논문의 정성 비교 표에 실제 예시가 있다.

- A-Mem : 질의 'dog'를 의미가 비슷한 'Rex'에 매칭해서, 맥락을 무시하고 "She has a dog named Rex"라고 지어냄
- SYNAPSE : `C_ret < 0.12`로 게이팅이 걸려서 "No record of such pet found."

또 하나.

- A-Mem : "Caroline moved from Sweden 4 years ago"가 유사도 0.92로 제일 위에 검색돼서 "She lives in Sweden"이라고 답함. 오래된 정보라는 걸 모름
- SYNAPSE : 시간 감쇠로 스웨덴 항목은 0.4로 내려가고 미국 항목은 0.95로 올라감

앞에서 본 [Mem0 리뷰](https://momozzing.github.io/paper%20review/Mem0-Paper-review/)에서 OpenAI 메모리가 temporal에서 15% 아래로 떨어졌던 것과 같은 종류의 실패다.

-> 유사도만 보면 오래된 기억이 이긴다.

### **3.2 Ablation Study**

측면 억제가 불확실성 게이트 앞에서 전처리 역할을 한다고 한다. 게이트를 빼면(`τ_gate = 0`) Adversarial F1이 67.2로 떨어지고, 억제까지 빼면 더 떨어진다.

![장치별 제거 실험 (논문 Table 3)](https://momozzing.github.io/assets/images/synapse/table3-mechanism-ablation.png)

위쪽 묶음은 장치를 하나씩 끈 결과, 아래쪽 묶음은 활성 확산이나 그래프 구조 자체를 뺀 결과다.

-> 게이트만으로는 안 되는 것이다. 억제가 먼저 경쟁 노드를 눌러줘야 "최상위 노드의 활성 에너지"를 신뢰도로 쓸 수 있다.

### **3.3 Efficiency Analysis**

| | SYNAPSE | LoCoMo | MemGPT | MemoryOS | LangMem |
|---|---:|---:|---:|---:|---:|
| 질의당 토큰 | ~814 | 16,910 | 16,977 | — | — |
| 1,000질의 비용 | $0.24 | $2.66 | $2.67 | — | — |
| 비용 효율(F1/$) | 167.3 | 9.6 | 10.5 | 126.8 | 150.7 |
| 평균 지연 | 1.9s | 8.2–8.5s | — | — | — |

토큰을 95% 줄였다. 전체 이력을 넣지 않고 관련된 부분그래프만 꺼내기 때문이다. 전체 컨텍스트보다 11배 싸면서 성능은 거의 2배라고 한다.

LangMem도 비용 효율이 150.7로 비슷한데, 성능이 34.3 F1로 낮다.

그래프를 만드는 비용은 에이전트를 쓰는 기간 전체에 나눠지니까 질의당으로 보면 무시할 만하다고 한다.

## **4. 지금 관점: 세 장치를 따로 볼 것**

장치 중에 따로 떼어 쓸 수 있는 것과 같이 써야 하는 것이 나뉜다.

시간 감쇠는 따로 쓸 수 있다. 엣지 가중치에 `e^{−ρ|Δτ|}`를 곱하는 것뿐이라 그래프가 없어도 검색 점수에 곱하면 된다. "스웨덴에 산다" 같은 오래된 정보 오류를 막는 제일 싼 방법 같다.

-> 앞에서 본 [Zep](https://momozzing.github.io/paper%20review/Zep-Paper-review/)의 양시간 모델링이 더 정확하긴 한데 구현이 크고, 이쪽은 한 줄이다.

이중 트리거(BM25 + dense)는 ReFind, Zep, SYNAPSE가 각자 같은 방식으로 왔다. 고유명사는 어휘 검색이 있어야 한다.

거절 게이팅은 같이 써야 한다. 신뢰도 게이트만 떼어 쓰면 67.2로 떨어진다. 측면 억제가 먼저 있어야 한다.

대신 명시적 검증 프롬프팅("EXPLICITLY mentioned인가?")은 따로 쓸 수 있다. 그래프가 없어도 어떤 검색 파이프라인에든 붙일 수 있다.

PageRank를 전역 사전확률로 쓰는 것도 떼어낼 수 있다. 자주 나오는 엔티티에 가중치를 주되, 국소 신호랑 따로 더하는 구조다.

논문이 밝힌 한계도 있다.

1. Cold Start : 활성 확산은 그래프가 충분히 연결돼 있어야 효과가 있다. 이력이 적은 초기 대화에서는 그래프를 유지하는 비용에 비해 단순한 선형 버퍼보다 얻는 게 적다고 한다
2. Cognitive Tunneling : 측면 억제 때문에, 꼼꼼히 다 찾는 게 나은 단순한 질의에서는 성능이 떨어질 때가 있다고 한다

부록에 실패 사례가 하나 있다.

![Cognitive Tunneling 실패 사례 (논문 Figure 4)](https://momozzing.github.io/assets/images/synapse/fig4-cognitive-tunneling.png)

연결이 많은 "Airport" 허브가 활성을 몰아 받으면서, 연결이 적은 "초록 재킷" 에피소드가 억제로 잘려 나간다.

-> 앞 리뷰에서도 나온 모양이다. 구조를 정교하게 만들면 단순한 질의에서 손해를 본다. Mem0g가 multi-hop에서 Mem0에 졌던 것과 비슷하다. 뒤에서 볼 ReFind 표에서도 구조화 메모리가 BM25-RAG에 진다.

평가가 LoCoMo 텍스트 벤치마크 하나뿐이라는 것도 한계로 적었다. 다음에 볼 [Anatomy 리뷰](https://momozzing.github.io/paper%20review/Anatomy-of-Agentic-Memory-Paper-review/) 기준으로 보면 full-context 비교(∆)가 없어서 포화 여부도 알 수 없다.

## **5. Conclusion**

conclusion 부분을 보면, 생물학적 활성 확산을 흉내 내서 기존 검색 시스템의 맥락 고립 문제를 푸는 구조를 제안했다고 한다. 메모리를 동적 연상 그래프로 두고, 끊겨 있는 사실을 잇고 관련 없는 잡음은 거른다.

그리고 neuro-symbolic 방식이 정적인 벡터 검색과 적응적·구조화된 인지 사이를 메울 수 있다는 걸 보였다고 한다.

여태까지 본 메모리 시스템들이 잘 찾는 걸 겨뤘다면, 이 논문은 못 찾았을 때 모른다고 하는 걸 기능으로 넣었고 거기서 96.6 F1을 냈다.

-> 업무용 챗봇에서는 모르는 걸 지어내는 쪽이 더 큰 문제라서, 이쪽이 더 중요할 수도 있을 것 같다.

다음은 [Anatomy of Agentic Memory](https://momozzing.github.io/paper%20review/Anatomy-of-Agentic-Memory-Paper-review/)다. 지금까지의 메모리 벤치마크 수치를 어떻게 읽어야 하는지, 벤치마크 포화·F1과 의미의 어긋남·백본 의존·agency tax를 실측한 논문이다.
