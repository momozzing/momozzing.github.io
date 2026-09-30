---
date: 2026-09-25 09:00:00 +0900
title: "Memory in the Age of AI Agents Paper review"
excerpt: "장기기억·단기기억이라는 이분법으로는 지금의 에이전트 메모리를 담을 수 없다. Forms·Functions·Dynamics 세 축으로 다시 짠 107쪽짜리 서베이."
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

Memory in the Age of AI Agents: A Survey — Forms, Functions and Dynamics

[https://arxiv.org/abs/2512.13564](https://arxiv.org/abs/2512.13564)

Memory in the Age of AI Agents는 에이전트 메모리 연구를 전부 모아서 정리한 서베이 논문이다.

2025년 12월에 arXiv에 올라온 107쪽짜리 서베이로, 저자가 47명이다. 정리한 논문 목록은 [Agent-Memory-Paper-List](https://github.com/Shichun-Liu/Agent-Memory-Paper-List)에 따로 공개돼 있다.

예전에 리뷰한 [CoALA](https://momozzing.github.io/paper%20review/CoALA-Paper-review/)가 에이전트 전체 구조를 정리했다면, 이 논문은 그중 메모리만 떼어서 다시 정리했다. 이 논문은 장기/단기 같은 기존 분류로는 요즘 시스템들을 다 담을 수 없고, episodic·semantic 같은 용어가 늘어나면서 개념이 더 흐려졌다고 한다. CoALA의 working·episodic·semantic·procedural 같은 분류 용어도 그 연장선에 있다고 보고 읽었다.

앞에서 메모리 논문을 아홉 편 봤는데, 여기서 전체를 한 번 정리하고 가려고 이 논문을 읽었다. 좀 더 자세히 알아보자.

![분류 체계 전체 조감도 (논문 Figure 1)](https://momozzing.github.io/assets/images/agent-memory-survey/fig1-taxonomy-overview.png)

Figure 1은 논문 전체를 한 장으로 요약한 그림이다. 바닥 격자의 가로축이 저장 형태(Forms), 대각선 축이 기능(Functions)이고, 그 위에 실제 시스템들을 배치했다.

요즘 많이 보이는 Mem0랑 Zep이 Token-level 장기기억 칸에 같이 있고, SnapKV, H2O 같은 KV 캐시 압축 기법들이 Working memory로 들어가 있다.

-> KV 캐시 압축을 메모리로 분류하는 건 좀 의외였다. 뒤에 2.3.1에서 이유가 나온다.

## **1. Introduction**

요즘 에이전트는 LLM 위에 추론, 계획, 지각, 메모리, 도구 사용을 얹은 구조라는 게 어느 정도 합의가 됐다.

그런데 이 중 추론이랑 도구 사용은 강화학습으로 모델 안에 상당 부분 들어갔는데, 메모리는 아직 바깥에 따로 붙여서 쓰고 있다고 한다.

LLM은 파라미터를 바로바로 바꿀 수 없으니까, 환경이랑 상호작용하면서 계속 적응하려면 메모리가 필요하다. 개인화 챗봇, 추천, 시뮬레이션처럼 이력을 다뤄야 하는 곳은 다 여기 해당한다.

새로 분류해야 하는 이유는 두 가지라고 한다.

1. 기존 분류가 오래됐다. 2025년에 방법론이 많이 나왔는데 그 전에 만든 분류라서, 과거 경험에서 도구를 뽑아내는 방식이나 memory-augmented test-time scaling 같은 게 들어갈 자리가 없다.
2. 개념이 너무 흩어져 있다. 다들 "agent memory"라고 하는데 구현도 목표도 전제도 다르다. declarative, episodic, semantic, parametric 같은 용어만 늘어났다. 특히 장기/단기로 나누는 방식으로는 요즘 시스템들을 구분할 수가 없다고 한다.

-> KV 캐시 압축이랑 사용자 선호 기억을 둘 다 "단기/장기"로만 나누면 같은 기준으로 비교가 안 되긴 한다.

논문은 질문 다섯 개를 세우고 장마다 하나씩 답한다. 정의, Forms, Functions, Dynamics, 앞으로의 방향 순이다. 가운데 세 개(Forms–Functions–Dynamics)가 이 논문의 분류 틀이다.

## **2. Preliminaries: Formalizing Agents and Memory**

2장은 두 가지를 한다. 에이전트와 메모리를 수식으로 정의하고, 비슷한 개념들과 경계를 나눈다.

### **2.1 LLM-based Agent Systems**

에이전트는 관측을 받아서 행동을 한다. 행동 종류가 자연어 생성, 도구 호출, 계획 출력, 환경 조작, 에이전트 간 통신으로 다양하고, 행동을 정할 때 메모리에서 꺼낸 정보도 같이 쓴다.

### **2.2 Agent Memory Systems**

메모리 쪽 정의에서 볼 만한 게 두 가지 있다.

첫째, 메모리 내부 구조를 정해두지 않는다. 텍스트든 key-value든 벡터 DB든 그래프든 상관없다고 한다.

둘째, 장기랑 단기를 구조로 나누지 않는다.

*"Both roles are supported within a single memory container, with temporal distinctions emerging from usage patterns rather than architectural separation."*

저장소는 하나고, 어떻게 쓰느냐에 따라 장기·단기가 갈린다. 1장에서 장기/단기 이분법이 부족하다고 한 게 여기서 정의로 연결된다.

메모리가 움직이는 과정은 세 단계로 본다. 새 정보를 기억으로 만드는 형성, 기존 저장소에 합치는 진화, 필요할 때 꺼내는 검색. 이게 5장 Dynamics의 뼈대가 된다.

이 세 단계를 매번 다 할 필요는 없다. 태스크 시작할 때 한 번만 검색하는 시스템도 있고 계속 꺼내는 시스템도 있다.

### **2.3 Comparing Agent Memory with Other Key Concepts**

agent memory를 기준으로 헷갈리기 쉬운 개념 세 개랑 비교한다.

![Agent Memory와 인접 개념 비교 (논문 Figure 2)](https://momozzing.github.io/assets/images/agent-memory-survey/fig2-concept-comparison.png)

#### **2.3.1 Agent Memory vs. LLM Memory**

거의 다 agent memory에 포함된다고 본다.

MemoryBank나 MemGPT는 스스로를 "LLM memory"라고 불렀는데, 실제로 푼 문제가 사용자 선호 추적, 대화 상태 유지 같은 에이전트 문제였다. 논문 설명으로는 2023~2024년에는 에이전트 정의가 제대로 없어서, LLM이 계산기만 불러도 에이전트라고 부르던 경우가 있었다. 그래서 그때는 LLM memory라고 불렀다.

다만 KV 캐시 관리, long-context 처리, 아키텍처 변경(RWKV, Mamba, diffusion LM)은 LLM memory로 남긴다. 모델 내부를 직접 건드리느냐가 기준이다.

겹치는 부분(Overlap)도 적어둔다. KV 압축이나 컨텍스트 창 관리도 한 과제 안에서 중요한 정보를 유지하려고 쓰면 에이전트 관점에서 단기 메모리로 기능한다고 본다.

#### **2.3.2 Agent Memory vs. RAG**

여기는 경계가 흐려지는 중이다.

원래는 RAG가 정적인 문서에서 한 번 찾아오는 거고, agent memory는 환경이랑 계속 상호작용하면서 쌓아가는 거였다. 그런데 HippoRAG 같은 건 RAG 쪽이랑 메모리 쪽 양쪽에서 다 인용된다.

그래서 논문은 차라리 어떤 태스크로 평가하느냐로 나누자고 한다. 아래는 논문이 양쪽 대표로 든 벤치마크다.

| | 대표 벤치마크 |
|---|---|
| RAG | HotpotQA, 2WikiMQA, MuSiQue |
| Agent memory | LoCoMo, LongMemEval, GAIA, XBench, BrowseComp, SWE-bench Verified, StreamBench |

근데 이것도 agent memory라고 하면서 HotpotQA로 평가하는 논문이 많아서 완벽하진 않다.

Graph RAG랑 그래프 기반 메모리의 차이도 정리한다. 그래프를 계속 변하는 경험으로 다루면 메모리, 고정된 지식으로 쓰면 RAG다.

#### **2.3.3 Agent Memory vs. Context Engineering**

둘이 겹치는 부분이 있다고 본다.

Context engineering은 컨텍스트 창을 제한된 자원으로 보고 뭘 넣을지 최적화하는 쪽이고, agent memory는 에이전트가 무엇을 알고 무엇을 겪었는지를 다루는 쪽이다.

겹치는 건 working memory 부분이다. 대화를 요약해서 롤링하는 건 컨텍스트 관리이기도 하고 짧은 기억이기도 해서, 논문은 이 지점에서 둘의 경계가 사실상 없어진다고 본다.

## **3. Form: What Carries Memory?**

기억이 어디에 어떤 형태로 저장되느냐로 나눈다. 세 가지다.

- Token-level : 텍스트처럼 눈에 보이는 단위. 하나씩 꺼내고 고칠 수 있음
- Parametric : 모델 파라미터 안에 저장
- Latent : 모델 내부 표현(KV 캐시, hidden state). 사람이 읽을 수 없음

### **3.1 Token-level Memory**

단위들 사이에 어떤 구조를 두느냐로 1D, 2D, 3D로 나눈다.

![Token-level memory의 차원별 분류 (논문 Figure 3)](https://momozzing.github.io/assets/images/agent-memory-survey/fig3-token-level.png)

#### **3.1.1 Flat Memory (1D)**

그냥 하나씩 쌓아두는 방식. 항목끼리 관계는 저장하지 않는다.

뭘 저장하느냐로 Dialogue(대화), Preference(취향), Profile(프로필·캐릭터 설정), Experience(행동 궤적·피드백), Multimodal(이미지·영상·오디오) 다섯 가지로 나눈다.

예를 들면 MemGPT와 Mem0는 대화 내용과 요약을, Reflexion은 행동 궤적과 피드백을 쌓는다. 다섯 가지 모두 저장하는 내용만 다르고, 항목을 서로 잇지 않고 하나씩 쌓는 방식은 같다.

Dialogue 쪽은 처음엔 대화를 그냥 저장하거나 요약하다가, 저장 단위를 어떻게 자를지 고민하는 쪽으로 가고, 요즘은 생각이나 반성을 저장하는 쪽(Think-in-Memory, RMM)까지 갔다고 한다.

#### **3.1.2 Planar Memory (2D)**

항목끼리 연결을 만드는데 한 층 안에서만 만든다.

- Tree : HAT(긴 대화를 나눠서 단계적으로 묶음), MemTree
- Graph : 2D에서 제일 많이 쓰는 방식. Ret-LLM, KGT, A-MEM, HuaTuo(의료 KG)

#### **3.1.3 Hierarchical Memory (3D)**

층 사이에도 연결을 만든다. 옆으로만이 아니라 위아래 층으로도 찾아갈 수 있다.

Pyramid와 Multi-layer 두 가지로 나눈다. Pyramid는 위로 갈수록 더 요약된 층을 쌓고 위에서 아래로 좁혀 찾는다. Zep이 여기 들어간다(엔티티를 커뮤니티로 묶어 요약). Multi-layer는 정보 종류나 기능별로 층을 나눈다. HippoRAG가 예인데, 지식 그래프 층과 원문 passage 층을 따로 둔다.

두 방식 모두 층마다 다른 단위의 정보를 두고, 층 사이 연결로 오간다.

실제 서비스는 아직 대부분 1D에 있고, 2D·3D는 Zep이나 A-MEM 같은 최근 연구들이다. 올라갈수록 만들고 유지하는 비용이 든다.

### **3.2 Parametric Memory**

모델 파라미터에 기억을 넣는 쪽이다. 원본 모델을 직접 바꾸느냐, 옆에 모듈을 붙이느냐로 나뉜다.

#### **3.2.1 Internal Parametric Memory**

원본 가중치에 직접 넣는다. 언제 넣느냐로 또 나눈다.

1. Pre-train : LMLM, HierMemLM은 검색용 정보만 모델에 넣고 지식은 외부에 둔다
2. Mid-train : 추가 사전학습 때 경험을 넣는다 (Agent-Founder, Early Experience)
3. Post-train : 파인튜닝이나 지식 편집으로 넣는다 (Character-LM, SELF-PARAM, KnowledgeEditor, MEND, AlphaEdit 등)

추론할 때 비용이 안 드는 대신, 새로 넣으려면 재학습을 해야 하고 옛날 걸 잊기 쉽다. 그래서 도메인 지식처럼 큰 덩어리용이지, 사용자 개인 정보처럼 자주 바뀌는 데는 안 맞는다고 한다.

#### **3.2.2 External Parametric Memory**

원본은 그대로 두고 adapter 같은 추가 파라미터에 넣는다. K-Adapter, WISE, MLP-Memory, T-Patcher, MemLoRA 등.

모듈이라 붙였다 뗐다 할 수 있고 롤백도 된다. 대신 모델 내부 계산을 거쳐서 영향을 주는 거라 효과가 간접적이다.

### **3.3 Latent Memory**

KV 캐시나 hidden state처럼 모델 내부 표현으로 기억한다. 잠재 상태를 어디서 가져오느냐로 나눈다.

![Latent Memory 통합 구조 (논문 Figure 4)](https://momozzing.github.io/assets/images/agent-memory-survey/fig4-latent.png)

#### **3.3.1 Generate**

별도 모델이나 인코더가 새로 잠재 표현을 만들어서 준다. 예로 MemGen, SoftCoT이 있다. 이 계열은 긴 입력이나 태스크 궤적을 짧은 연속 벡터로 만들어 두고, 나중 추론에 넣어 쓴다.

별도 모델을 학습하니까 Parametric이랑 헷갈릴 수 있는데, 만들어진 표현이 따로 저장돼서 재사용되면 Latent로 본다고 한다.

#### **3.3.2 Reuse**

이미 계산된 KV 캐시를 고치지 않고 그대로 다시 쓴다. Memorizing Transformers가 예인데, 지난 KV 쌍을 저장해 두고 추론 때 kNN 검색(가장 가까운 k개를 찾는 검색)으로 꺼낸다. 이 계열은 모두 원래 KV를 기억 항목으로 쓰고, 어떤 KV를 남기고 어떻게 찾을지가 문제다.

#### **3.3.3 Transform**

KV 캐시를 골라내거나 압축해서 쓴다.

- Scissorhands : 캐시가 넘치면 어텐션 점수 낮은 토큰을 버림
- SnapKV : 중요한 prefix KV만 모음
- PyramidKV : 층마다 KV 예산을 다르게 줌
- RazorAttention : head마다 필요한 범위만 남김
- H2O : 최근 토큰이랑 중요 토큰만 남김

압축할수록 가벼워지지만 정보가 빠지고 확인하기 어려워진다.

### **3.4 Adaptation**

그래서 어떤 형태를 언제 쓰냐를 정리한 부분이다.

![세 가지 메모리 형태 비교 (논문 Figure 5)](https://momozzing.github.io/assets/images/agent-memory-survey/fig5-three-paradigms.png)

- Token-level : 투명하고 수정이 빠름. 챗봇, 개인화, 추천, 법률·금융·의료처럼 근거가 필요한 곳
- Parametric : 일반화가 잘 되지만 수정이 느리고 잊어버리기 쉬움. 역할극, 추론 위주 태스크
- Latent : 사람이 못 읽지만 토큰 효율이 좋음. 멀티모달, 온디바이스

Token-level은 모델을 안 건드리고 붙이는 방식이라 최신 모델로 바꿔도 그대로 쓸 수 있다.

-> 실무에서 Token-level부터 쓰게 되는 이유가 이거인 것 같다. 고칠 수 있고, 볼 수 있고, 모델 바꿔도 남는다.

## **4. Functions: Why Agents Need Memory?**

3장이 어떻게 저장하느냐였다면, 4장은 왜 기억하느냐로 나눈다.

LLM은 원래 stateless라서 에이전트로 쓰려면 기억이 필요하고, 컨텍스트 창을 늘린다고 해결되지도 않는다고 본다.

![기능 기준 분류 (논문 Figure 6)](https://momozzing.github.io/assets/images/agent-memory-survey/fig6-functions.png)

세 가지로 나눈다.

- Factual : 에이전트가 무엇을 아는가
- Experiential : 에이전트가 어떻게 나아지는가
- Working : 지금 무엇을 생각하고 있는가

기존 episodic/semantic 구분은 정보의 종류로 나눈 거고, 이 분류는 그 기억으로 뭘 하려는지로 나눈 것이다. 사용자 선호 기억이랑 실패에서 배운 교훈은 둘 다 "장기기억"이지만 쓰임새가 다르다. 전자는 사용자가 말을 바꾸면 고쳐야 하고, 후자는 잘 안 먹히면 고쳐야 한다.

### **4.1 Factual Memory**

사용자에 대한 것과 환경에 대한 것으로 나뉜다.

#### **4.1.1 User factual memory**

사용자 이름, 선호, 약속 같은 사실을 세션이 바뀌어도 유지한다.

기억이 없으면 생기는 문제를 세 가지로 이름 붙였다.

1. coreference drift : 대화가 길어지면 "그거", "아까 그 사람"이 뭘 가리키는지 흐려짐
2. repeated elicitation : 이미 말한 걸 또 물어봄
3. contradictory responses : 앞에서 한 말이랑 다른 답을 함

-> 챗봇 만들면서 셋 다 겪어본 문제다. 이걸 "메모리 문제"로 묶어서 보니까 정리가 된다.

#### **4.1.2 Environment factual memory**

사용자 말고 바깥 정보. 긴 문서, 코드베이스, 도구 같은 것. HippoRAG, MemTree, LMLM 등이 여기 들어간다.

### **4.2 Experiential Memory**

경험을 얼마나 가공해서 남기느냐로 네 가지로 나눈다.

![Experiential memory 분류 (논문 Figure 7)](https://momozzing.github.io/assets/images/agent-memory-survey/fig7-experiential.png)

#### **4.2.1 Case-based Memory**

있었던 일을 거의 그대로 남긴다. 그대로 예시로 쓸 수 있다. Memento, JARVIS-1.

#### **4.2.2 Strategy-based Memory**

있었던 일에서 다음에 어떻게 할지를 뽑아서 남긴다. 추론 패턴, 워크플로 같은 것. Reflexion이 여기 들어간다.

#### **4.2.3 Skill-based Memory**

실행할 수 있는 함수나 API로 만든다. 호출할 수 있고, 결과를 확인할 수 있고, 다른 스킬이랑 조합할 수 있어야 한다. Voyager, SkillWeaver, Memp.

#### **4.2.4 Hybrid memory**

위의 것들을 섞어서 쓴다. ExpeL(성공·실패 궤적을 그대로 두면서 거기서 규칙도 뽑아 쌓는 방법)이 예다.

-> 로그만 쌓고 있으면 case, 회고해서 규칙을 뽑으면 strategy, 그걸 도구로 만들면 skill로 보면 될 것 같다. 위로 갈수록 재사용은 잘 되는데 만드는 비용이 든다.

### **4.3 Working Memory**

지금 작업 중인 내용을 잠깐 들고 있는 공간이다.

논문은 LLM의 컨텍스트 창이 그냥 읽기만 하는 버퍼라서 진짜 working memory라고 보기 어렵다고 본다. 뭘 남기고 뭘 버릴지 스스로 정하는 기능이 없어서다.

#### **4.3.1 Single-turn Working Memory**

한 번에 들어오는 긴 입력을 처리한다. 토큰 수를 줄이거나(LLMLingua 같은 프롬프트 압축), 구조화된 표현으로 바꾼다.

#### **4.3.2 Multi-turn Working Memory**

대화가 길어지면서 쌓이는 이력을 관리한다. 이력이 쌓이면 어텐션이 흐려지고 느려지고 목표를 잃는다(goal drift). 그래서 고정 크기 상태로 압축하거나(MemAgent, MemSearcher, ReSum), 이력을 접어두는(HiAgent, Context-Folding, AgentFold) 방식을 쓴다.

Figure 1에서 KV 캐시 압축이 Working memory 칸에 있던 근거는 2.3.1의 Overlap이다. 한 과제 안에서 중요한 정보를 유지하는 데 쓰면 단기 메모리로 기능한다고 본다.

## **5. Dynamics: How Memory Operates and Evolves?**

기억이 만들어지고, 바뀌고, 꺼내지는 과정을 다룬다.

![메모리 생애주기 (논문 Figure 8)](https://momozzing.github.io/assets/images/agent-memory-survey/fig8-dynamics.png)

### **5.1 Memory Formation**

원본 데이터에서 기억을 만드는 단계. 다섯 가지 방식이 있다.

1. Semantic Summarization : 전체 요약
2. Knowledge Distillation : 쓸 만한 사실이나 규칙만 뽑기
3. Structured Construction : 그래프나 트리 같은 구조로 만들기
4. Latent Representation : 처음부터 임베딩으로 저장
5. Parametric Internalization : 모델 파라미터에 넣기

-> 1번이랑 2번이 헷갈리는데, 요약은 "대충 무슨 대화였나"이고 증류는 "여기서 건질 사실이 뭔가"라고 보면 될 것 같다.

### **5.2 Memory Evolution**

새 기억을 기존 저장소에 합치는 단계.

그냥 뒤에 계속 붙이면 서로 모순되는 게 생기고, 오래된 정보가 계속 남는다.

![메모리 진화 메커니즘 (논문 Figure 9)](https://momozzing.github.io/assets/images/agent-memory-survey/fig9-evolution.png)

#### **5.2.1 Consolidation**

비슷한 기억끼리 묶어서 더 큰 단위로 만든다. 어느 크기로 묶을지가 어렵다.

#### **5.2.2 Updating**

새 정보랑 충돌하면 기존 기억을 고친다. 외부 저장소를 고치는 방식이랑 모델을 편집하는 방식이 있다.

#### **5.2.3 Forgetting**

오래되거나 쓸모없는 걸 지운다. 너무 많이 쌓이면 검색이 느려지고 잡음이 는다. 대신 너무 많이 지우면 가끔 필요한 걸 잃는다.

지우는 기준은 세 가지다.

- Time-based : 오래된 순. MemGPT는 컨텍스트가 넘치면 제일 오래된 메시지부터 뺀다
- Frequency-based : 안 꺼내 쓴 것
- Importance-driven : 중요도 평가

CoALA(2023년 9월)에서는 "기억을 지우거나 고치는 학습은 아직 거의 연구가 안 됐다"고 했는데, 2년 남짓 사이에 망각이 따로 한 항목이 됐다.

### **5.3 Memory Retrieval**

꺼내 쓰는 단계. 네 단계로 나눈다.

![메모리 검색 분류 (논문 Figure 10)](https://momozzing.github.io/assets/images/agent-memory-survey/fig10-retrieval.png)

#### **5.3.1 Retrieval Timing and Intent**

언제 꺼낼지. MIRIX는 매번 여섯 개 DB를 다 뒤지고, MemGPT는 LLM이 필요할 때 직접 검색 함수를 부른다.

-> 매 턴 무조건 메모리를 넣는 구현이 많은데, 그것도 선택지 중 하나일 뿐이다.

#### **5.3.2 Query Construction**

질문을 그대로 검색에 넣으면 잘 안 맞아서, 쪼개거나 다시 써서 검색한다.

#### **5.3.3 Retrieval Strategies**

키워드 / 임베딩 / 그래프 / 하이브리드. Zep이랑 HippoRAG가 그래프 쪽, MemGPT랑 Voyager가 임베딩 쪽이다.

#### **5.3.4 Post-Retrieval Processing**

꺼낸 결과를 다시 정렬하고 중복을 뺀다. 그냥 다 넣으면 길어지고 서로 충돌한다.

## **6. Resources and Frameworks**

실제로 쓸 수 있는 벤치마크와 프레임워크를 정리한 장이다.

### **6.1 Benchmarks and Datasets**

메모리용 벤치마크를 세 가지로 나눈다.

1. Memory : 얼마나 잘 기억하고 꺼내는가 (예: LoCoMo)
2. Lifelong learning : 새 정보가 계속 들어오고 옛날 정보가 틀려지는 상황 (예: LongMemEval)
3. Self-evolving : 에이전트가 자기 기억이나 전략을 스스로 고치는가 (예: MemoryAgentBench)

셋 다 여러 턴이나 여러 태스크에 걸쳐 앞에서 얻은 정보를 뒤에서 쓰게 만든 벤치마크고, 뒤로 갈수록 기억을 고치고 바꾸는 쪽을 더 본다.

그 외에 메모리 전용은 아니지만 메모리가 필요한 벤치마크로 ALFWorld, WebArena, ToolBench, SWE-bench Verified, GAIA 같은 것도 정리해뒀다.

### **6.2 Open-Source Frameworks**

논문 Table 9는 오픈소스 메모리 프레임워크 25종이 어떤 기억을 지원하고 어떤 벤치마크로 평가했는지 정리한 표다. 그중 일부만 옮겼다. `Fac.`는 factual, `Exp.`는 experiential, `MM.`은 multimodal 지원 여부고, 마지막 열은 공개된 평가 벤치마크다.

| Framework | Fac. | Exp. | MM. | 내부 구조 | 보고된 평가 |
|---|:---:|:---:|:---:|---|---|
| MemGPT | ✔ | ✔ | ✘ | hierarchical (S/LTM) | LoCoMo |
| Mem0 | ✔ | ✔ | ✘ | graph + vector | LoCoMo |
| Memobase | ✔ | ✔ | ✘ | structured profiles | LoCoMo |
| MIRIX | ✔ | ✔ | ✔ | structured memory | LoCoMo, MemoryAgentBench |
| MemoryOS | ✔ | ✔ | ✘ | hierarchical (S/M/LTM) | LoCoMo, MemoryBank |
| MemOS | ✔ | ✔ | ✘ | tree memory + memcube | LoCoMo, PrefEval, LongMemEval, PersonaMem |
| Zep | ✔ | ✔ | ✘ | temporal knowledge graph | LongMemEval |
| LangMem | ✔ | ✔ | ✘ | core API + manager | — |
| Cognee | ✔ | ✔ | ✔ | knowledge graph | — |
| Memary | ✔ | ✔ | ✘ | stream + entity store | — |
| MemEngine | ✔ | ✔ | ✔ | modular space | — |
| ReMe (AgentScope) | ✔ | ✔ | ✘ | agentscope | BFCL, AppWorld |
| Pinecone / Chroma / Weaviate | ✔ | ✘ | 일부 ✔ | vector(+graph) DB | — |

표를 보면,

- 벡터 DB(Pinecone, Chroma, Weaviate)는 factual만 지원한다
- 25종 중 벤치마크 점수를 낸 건 8종뿐이고, 나머지 17종은 점수가 없다
- 점수를 낸 8종 중 6종이 LoCoMo로 평가했고, 그중 3종(MemGPT, Mem0, Memobase)은 LoCoMo 하나뿐이다. LongMemEval을 본 건 MemOS랑 Zep 둘

-> 점수 없이 API만 있는 게 대부분이라, 도입하려면 직접 돌려봐야 한다.

## **7. Positions and Frontiers**

앞으로의 방향을 여덟 가지로 정리한다. 각 절마다 지금까지 어땠고 앞으로 어떻게 될지를 짝으로 적었다.

-> 앞의 세 개(7.1~7.3)는 같은 얘기 같다. 사람이 정하던 걸 에이전트가 스스로 하게 만들자는 것.

### **7.1 Memory Retrieval vs. Memory Generation**

검색에서 생성으로 가는 흐름이다.

#### **7.1.1 Look Back: From Memory Retrieval to Memory Generation**

지금까지는 저장해둔 걸 잘 찾아오는 게 목표였다. 인덱싱, 유사도, 리랭킹을 개선하는 연구가 대부분이다.

여기엔 저장소가 이미 잘 만들어져 있다는 전제가 깔려 있다.

그래서 최근엔 필요할 때 기억을 새로 만들어내는 쪽(memory generation)으로 간다고 한다. 두 가지 방식이 있다.

1. retrieve-then-generate : 검색한 걸 재료로 다시 정리해서 만든다 (ComoRAG, G-Memory, CoMEM)
2. direct generation : 검색 없이 바로 만든다 (MemGen, VisMem)

#### **7.1.2 Future Perspective**

생성형 메모리가 갖춰야 할 것 세 가지.

1. 지금 태스크에 맞게 만들어야 한다
2. 텍스트, 코드, 도구 결과 같은 여러 정보를 하나로 합쳐야 한다. 논문은 여기에 Latent 메모리(3.3)가 맞을 것 같다고 한다
3. 언제 어떻게 만들지를 사람이 정하지 말고 학습해야 한다

### **7.2 Automated Memory Management**

수작업으로 짜던 메모리를 자동으로 구축하는 쪽이다.

#### **7.2.1 Look-Back: From Hand-crafted to Automatically Constructed Memory Systems**

지금 메모리 시스템은 대부분 사람이 규칙을 정한다. 뭘 저장할지, 언제 쓸지, 어떻게 고칠지. Mem0는 상세한 프롬프트로, MemoryOS는 임계값으로 정한다.

빠르게 만들 수 있고 동작을 예측할 수 있지만, 상황이 바뀌면 잘 안 맞는다.

최근엔 CAM이나 Memory-R1처럼 에이전트가 스스로 관리하게 하는 시도가 있다.

#### **7.2.2 Future Perspective**

메모리 조작(add/update/delete/retrieval)을 에이전트의 도구 호출로 만들어서, 에이전트가 자기가 뭘 저장하고 뭘 지우는지 알게 하자고 한다.

-> CoALA에서 retrieval이랑 learning을 행동으로 넣자고 했던 게 2년 남짓 뒤에 이렇게 돌아왔다.

### **7.3 Reinforcement Learning Meets Agent Memory**

휴리스틱에서 RL로 넘어가는 얘기다.

#### **7.3.1 Look-Back: RL is Internalizing Memory Management Abilities for Agents**

메모리도 강화학습 쪽으로 가고 있다고 한다. Figure 11은 이걸 세 단계로 그렸다. 7.3.1에서는 앞의 두 단계를 다루고, 세 번째는 7.3.2에서 다룬다.

![RL 기반 메모리 시스템의 진화 (논문 Figure 11)](https://momozzing.github.io/assets/images/agent-memory-survey/fig11-rl-evolution.png)

1. RL-free : 지금 대부분. 고정 임계값, 고정 검색 파이프라인. LLM이 관여하는 것처럼 보여도 프롬프트로만 돌아간다 (Mem0, MemOS, ExpeL, G-Memory 등)
2. RL-assisted : 일부만 RL. RMM은 검색 결과 재정렬에, Mem-α와 Memory-R1은 메모리 만드는 과정에 RL을 쓴다
3. Fully RL-driven : 메모리 구조와 관리 방식까지 전부 RL로 end-to-end 학습. Figure 11에도 예시 시스템이 없고, 논문이 앞으로 올 단계로 본다

#### **7.3.2 Future Perspective**

세 번째 단계인 fully RL-driven이 다음 단계라고 본다. 조건이 두 개다.

1. 사람이 만든 분류(episodic/semantic 같은)에 기대지 말 것. 그게 AI한테 맞는 구조라는 보장이 없다고 한다
2. 형성·진화·검색을 전부 에이전트가 관리할 것

-> 1번은 좀 과감한 주장 같다. 사람이 이해할 수 없는 구조가 나오면 디버깅은 어떻게 하지??

### **7.4 Multimodal Memory**

이미지·영상 쪽이 제일 앞서 있다.

### **7.5 Shared Memory in Multi-Agent Systems**

멀티에이전트에서 각자 기억하다가 공유 저장소로 가는 중이다. 대신 서로 덮어쓰는 문제가 생긴다.

### **7.6 Memory for World Model**

월드 모델은 다음 상태를 예측하려면 이전 상태를 기억해야 해서 메모리가 중요하다.

### **7.7 Trustworthy Memory**

메모리에 개인정보가 쌓이니까 유출 위험이 있다. 나중엔 OS처럼 버전 관리되고 감사할 수 있는 메모리가 필요하다고 한다.

### **7.8 Human-Cognitive Connections**

컨텍스트 창과 외부 저장소를 나누는 지금 구조를 Atkinson-Shiffrin 다중저장 모델에, 상호작용 로그·세계 지식·스킬로 나누는 걸 Tulving의 일화·의미·절차 기억 구분에 대응시킨다.

지금 에이전트 메모리는 사람 기억 구조를 많이 닮았는데, 사람은 기억을 매번 새로 재구성하고 에이전트는 저장된 걸 그대로 꺼낸다. 그래서 사람이 잘 때 기억을 정리하는 것처럼 오프라인으로 정리하는 시간을 두자고 제안한다.

## **지금 관점: CoALA 분류와 비교**

CoALA 리뷰를 읽고 나서 제일 궁금했던 게, CoALA의 네 가지 기억이랑 이 논문의 세 가지가 어떻게 대응되냐였다.

맞춰보면 working은 거의 그대로 Working으로 가고, 여기서는 KV 캐시 압축까지 들어간다. semantic 중 세계 지식은 Factual에 대응되는데 사용자/환경으로 한 번 더 나뉜다. 같은 semantic이라도 Reflexion의 반성문처럼 경험에서 뽑아낸 교훈은 Experiential의 strategy-based로 간다. episodic은 Experiential과 반만 맞는다. CoALA의 episodic은 있었던 일을 기록하는 거고, Experiential은 거기서 뽑아낸 교훈과 스킬까지 포함한다.

procedural은 대응되는 게 없다. CoALA는 기억을 담긴 내용의 종류로 나눴고, 이 논문은 왜 기억하느냐(Functions)와 어디에 담느냐(Forms)를 따로 나눴다. 그래서 절차 기억은 Functions 쪽으로는 Experiential의 skill-based로, Forms 쪽으로는 스킬 코드면 Token-level, 모델 가중치면 Parametric으로 흩어졌다.

실제로 쓴다면 Functions로 필요한 기억이 factual인지 experiential인지 먼저 정하고, Dynamics로 어떻게 만들고 고치고 꺼낼지 정하고, Forms는 마지막에 구현 방식으로 고르면 될 것 같다. 벡터 DB부터 정하고 기능을 맞추는 경우가 많은데, 6장 표를 보면 벡터 DB는 factual만 지원한다.

## **8. Conclusion**

이 논문은 새 시스템을 만들지 않고, 흩어진 개념을 정리한 서베이다.

abstract를 보면 Functions 쪽에서 시간 기준의 거친 분류(장기/단기)를 넘어 factual, experiential, working으로 더 잘게 나누자고 한다. 이게 이 논문의 제일 큰 주장이다.

그리고 메모리가 그냥 보조 저장소가 아니라 에이전트가 오래 일관되게 동작하게 하는 기반이라고 정리한다.

*"memory is not merely an auxiliary storage mechanism, but an essential substrate through which agents achieve temporal coherence, continual adaptation, and long-horizon competence"*

개인적으로는 분류보다 2장에서 RAG, 컨텍스트 엔지니어링이랑 경계를 나눈 부분이 더 쓸모 있었다. 내가 하려는 게 어느 쪽인지 정해야 벤치마크도 맞게 고를 수 있다.

다음은 [SYNAPSE](https://momozzing.github.io/paper%20review/SYNAPSE-Paper-review/)다. 벡터 유사도 대신 활성 확산으로 기억 사이의 관련성을 찾고, 모르는 질문에는 모른다고 답하게 만드는 메모리 구조다.
