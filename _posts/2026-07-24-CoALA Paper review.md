---
date: 2026-07-24 12:00:00 +0900
title: "CoALA Paper review"
excerpt: "에이전트를 볼 때 세 가지만 물으면 된다. 무엇을 기억하는가, 무엇을 할 수 있는가, 어떻게 결정하는가. ReAct부터 Voyager까지를 이 세 질문으로 정리한 논문."
categories:
  - Paper review
tags:
  - Large Language Model
  - NLP
  - Agent
  - Paper review
toc: true
toc_sticky: true
field: agent
---


Cognitive Architectures for Language Agents

[https://arxiv.org/abs/2309.02427](https://arxiv.org/abs/2309.02427)

CoALA는 Princeton에서 만든 언어 에이전트 프레임워크 논문이다. 2023년 9월에 나왔고, TMLR(Transactions on Machine Learning Research) 2024에 실렸다.
저자에 또 Shunyu Yao가 있다.

지금까지 리뷰한 건 전부 실험 논문이었는데, 이 논문은 성격이 다르다.
새 에이전트를 만들지 않는다. 흩어져 있는 에이전트 연구들을 하나의 틀로 정리한다.

앞에서 리뷰한 [ReAct](https://momozzing.github.io/paper%20review/ReAct-Paper-review/), Voyager, [Generative Agents](https://momozzing.github.io/paper%20review/Generative-Agents-Paper-review/)가 이 논문의 사례로 나와서, 시리즈 중간 정리로 읽어볼 만하다.

## **1. Introduction**

에이전트 연구는 빠르게 쌓였는데 용어가 제각각이다.
어떤 논문은 메모리를 말하고, 어떤 논문은 스킬을 말하고, 어떤 논문은 반성을 말한다. 서로 뭐가 같고 뭐가 다른지 비교할 공통 언어가 없다.

에이전트를 Planning, Memory, Tool Use 세 가지로 나누는 정리가 많이 쓰이긴 한다. 출처는 Lilian Weng의 2023년 블로그 글 "LLM Powered Autonomous Agents"다. 논문이 아니라 블로그 글이다.

논문도 이 글을 인용하는데, empirical survey(나와 있는 연구를 모아 정리한 글)로 분류하면서 자기랑 구분한다.
CoALA가 하려는 건 부품끼리의 관계까지 정하는 이론이다. 이 3분류와 대응시키면 이렇다.

- Tool Use -> grounding(외부 환경에 대한 행동)
- Planning -> 의사결정 사이클
- Memory -> 기억 4종과 읽기/쓰기 행동

3분류에 없던 것도 두 가지 들어간다.

1. 학습을 행동의 한 종류로 둔다.
2. LLM 가중치랑 에이전트 코드 자체를 기억으로 본다.

이걸 새로 발명하지는 않는다.
인지과학이랑 심볼릭 AI가 수십 년 동안 같은 문제(기억, 행동, 의사결정을 가진 시스템을 어떻게 조직할까)를 다뤄왔으니, 거기서 틀을 가져온다.

![LLM 사용의 세 단계 (논문 Figure 1)](https://momozzing.github.io/assets/images/coala/fig1-llm-to-cognitive.png)

다루는 대상부터 그림으로 나눈다.

- A : 텍스트가 들어가고 나오는 LLM 그 자체
- B : LLM을 환경과의 피드백 루프에 넣은 language agent
- C : LLM을 내부 상태 관리(기억, 회상, 학습, 추론)에도 쓰는 cognitive language agent

이 논문의 대상은 C다. 그림 C에 인용된 세 개가 ReAct, [Reflexion](https://momozzing.github.io/paper%20review/Reflexion-Paper-review/), Voyager다.
앞에서 리뷰한 논문들이 딱 이 단계에 있다.

## **2. Background: From Strings to Symbolic AGI**

출발점은 Post(1943)의 production system이다.
원래는 계산의 한계를 따지려고 만든 형식으로, "조건이 맞으면 이 문자열을 저 문자열로 바꾼다"는 규칙의 집합이다.

AI 쪽에서는 Allen Newell이 사람의 문제 풀이를 표현하는 형식으로 가져다 쓰면서 널리 퍼졌다. Newell과 Simon의 책(1972)에 나오는 온도조절기 예시가 대표적이다.

- 온도 < 32° → 수리 요청
- 온도 < 70° ∧ 보일러 꺼짐 → 보일러 켜기

규칙만으로는 부족해서, 이 규칙들을 언제 어떻게 적용할지 관리하는 구조가 필요해졌다. 그렇게 나온 게 인지 아키텍처다.
대표가 Soar다. production을 장기 기억에 저장하고, working memory랑 매칭해서 행동을 고르고, 학습으로 규칙을 갱신한다.

![Soar 아키텍처 (논문 Figure 2)](https://momozzing.github.io/assets/images/coala/fig2-soar.png)

-> 이 그림을 기억해두면 뒤에 나오는 CoALA 그림이 낯설지 않다. 구조가 거의 같다.

## **3. Connections between Language Models and Production Systems**

### **3.1 Language models as probabilistic production systems**

논문은 LLM을 확률적 production system으로 본다.
production이 문자열을 조건에 따라 다른 문자열로 바꾸는 규칙이라면, LLM은 입력 문자열 뒤에 이어질 내용을 확률적으로 생성하는 production이다.

### **3.2 Prompt engineering as control flow**

이렇게 보면 프롬프팅 기법들은 production을 나열한 것이 된다.
Table 1은 대표적인 프롬프팅 기법을 production 순서로 옮겨 적은 표다.

![프롬프팅 기법을 production 시퀀스로 (논문 Table 1)](https://momozzing.github.io/assets/images/coala/table1-prompting.png)

- Zero-shot : production 하나
- RAG(검색한 문서를 프롬프트에 붙이는 방법) : 검색 production 뒤에 LLM production
- Self-Critique : 생성 → 비평 → 수정, production 세 개

프롬프트 엔지니어링을 그때그때 쓰는 요령 대신 제어 흐름 설계로 보는 관점이다.

### **3.3 Towards cognitive language agents**

1. LLM 호출 하나(A)는 프롬프트 구성 → 호출 → 출력 파싱 → 실행이다.
2. 이걸 미리 정한 순서로 엮으면 프롬프트 체이닝(B)이 된다. 여기까지는 흐름이 고정이다.
3. 환경의 피드백이 다음 호출에 들어오는 루프(C)가 생기면 에이전트가 된다.

![LLM 호출에서 에이전트까지 (논문 Figure 3)](https://momozzing.github.io/assets/images/coala/fig3-lm-to-agents.png)

ReAct가 딱 3번이었다.

## **4. Cognitive Architectures for Language Agents (CoALA): A Conceptual Framework**

CoALA는 언어 에이전트를 기억, 행동 공간, 의사결정 세 가지로 정의한다.
LLM은 이 아키텍처의 중심 부품이지 전부는 아니다.

![CoALA 프레임워크 (논문 Figure 4)](https://momozzing.github.io/assets/images/coala/fig4-coala.png)

행동 공간은 CoALA가 제일 많이 넓힌 곳이다.
보통 도구 사용이라고 부르는 건 외부 행동뿐이다. CoALA는 그 옆에 내부 행동 3종을 같이 둔다.

![행동 공간의 구분 (논문 Figure 5)](https://momozzing.github.io/assets/images/coala/fig5-action-space.png)

- 외부 행동 = grounding : 물리 환경 제어, 사람과의 대화, 디지털 환경(API, 웹, 코드 실행) 조작
- 내부 행동 3종 : retrieval(장기 기억 읽기), reasoning(LLM으로 working memory 갱신), learning(장기 기억에 쓰기)

생각하고, 기억을 읽고, 기억에 쓰는 걸 전부 "행동"으로 본다.
-> ReAct가 thought를 action space에 넣었던 걸 프레임워크 전체로 넓힌 것 같다.

### **4.1 Memory**

LLM은 stateless다. 호출 사이에 아무것도 기억하지 않는다.
에이전트가 여러 스텝을 이어가려면 기억 모듈이 필요하고, CoALA는 이걸 4종으로 나눈다.

Working memory는 지금 의사결정 사이클에서 쓰는 정보를 담는다. 지각 입력, 목표, 중간 추론 결과가 여기 있고, 매 LLM 호출의 프롬프트가 여기서 만들어진다.
나머지 셋은 장기 기억이다.

1. Episodic memory(일화 기억) : 과거 경험의 기록. Generative Agents의 memory stream
2. Semantic memory(의미 기억) : 세계에 대한 지식. RAG가 읽는 지식 베이스, Reflexion의 반성문
3. Procedural memory(절차 기억) : 행동하는 방법 그 자체. Voyager의 스킬 코드

Reflexion의 반성문은 경험 기록처럼 보이는데, 논문은 실패한 에피소드(일화)를 보고 뽑아낸 지식이라서 의미 기억에 넣는다.

절차 기억에는 LLM의 가중치와 에이전트의 소스 코드도 들어간다.
그래서 에이전트가 자기 코드를 고치는 건 절차 기억에 쓰는 행동이 된다.

### **4.2 Grounding actions**

grounding은 외부 행동을 실행하고 환경의 피드백을 텍스트로 바꿔 working memory에 넣는다. 바깥 세계와의 상호작용을 관찰도 행동도 텍스트인 "텍스트 게임"처럼 단순하게 만든다고 한다.

### **4.3 Retrieval actions**

retrieval은 규칙 기반, sparse(키워드), dense(임베딩) 검색으로 나뉜다. Voyager의 스킬 검색과 Generative Agents의 3점수 회상(최근성, 중요도, 관련도)이 예시로 나온다.

### **4.4 Reasoning actions**

reasoning은 working memory를 읽고 working memory에 쓴다. 장기 기억을 읽어오는 retrieval과 다른 점이다.
최근 관찰이나 trajectory, 꺼내온 기억을 요약해 새 정보를 만들고, 그 결과는 learning(장기 기억에 쓰기)이나 다음 의사결정의 컨텍스트로 쓰인다.

### **4.5 Learning actions**

learning은 쓰는 대상에 따라 넷이다.

1. 일화 기억에 경험 쓰기
2. 의미 기억에 지식 쓰기 (Reflexion의 반성이 여기 해당)
3. 절차 기억에 LLM 파인튜닝
4. 절차 기억에 에이전트 코드 수정

4번이 제일 위험하다. 논문도 절차 기억에 쓰는 건 일화·의미 기억에 쓰는 것보다 훨씬 위험하다고 적어둔다.
기억을 지우거나 고치는 학습(unlearning)은 아직 거의 연구가 안 됐다고 한다.

### **4.6 Decision making**

에이전트의 동작은 사이클의 반복이다.

1. 관찰이 들어온다.
2. 계획 단계에서 reasoning과 retrieval로 후보 행동을 제안(propose)한다.
3. 평가(evaluate)하고 선택(select)한다.
4. 선택된 행동(grounding 또는 learning)을 실행한다.
5. 새 관찰과 함께 다음 사이클이 시작된다.

논문은 이걸 프로그램의 main 루프에 비유한다.
단순한 에이전트는 제안 하나를 바로 실행하고(ReAct), 복잡한 에이전트는 여러 후보를 평가해서 고른다. ToT(Tree of Thoughts, 생각을 트리로 펼쳐 탐색하는 방법)가 이 자리에 들어간다.

## **5. Case Studies**

Table 2는 최근 에이전트 다섯 개(SayCan, ReAct, Voyager, Generative Agents, ToT)를 장기 기억, 외부 행동, 내부 행동, 의사결정 네 칸으로 나눠 적은 표다. 결과 수치는 없다.

![CoALA로 분류한 에이전트들 (논문 Table 2)](https://momozzing.github.io/assets/images/coala/table2-casestudies.png)

1. SayCan(구글의 로봇 에이전트) : 내부 행동 없이 고정된 스킬 551개를 LLM+가치 함수로 평가해서 고름
2. ReAct : 장기 기억 없음, 내부 행동은 reasoning뿐, 제안 하나를 바로 실행
3. Voyager : 장기 기억은 절차 기억(스킬 코드), 내부 행동은 reasoning, retrieval, learning을 전부 씀
4. Generative Agents : 일화 기억에 경험을 쌓고, 반성 결과를 의미 기억에 씀
5. ToT : 장기 기억 없이 제안-평가-선택을 전부 구현

논문은 ReAct를 내부 행동과 외부 행동을 같이 쓰는 가장 단순한 언어 에이전트로 본다. 그 뒤의 에이전트들은 여기에 기억이나 복잡한 의사결정을 더한 모양이다.
-> Reflexion은 이 표에 없는데, 4.5절 설명대로라면 의미 기억 + learning이 채워진 행이 될 것 같다.

## **6. Actionable Insights**

6장은 프레임워크에서 뽑은 실천 방향이다. 전부 "~를 넘어 생각하라(thinking beyond)" 형식이다.

### **Modular agents: thinking beyond monoliths**

RL(강화학습)에는 MDP(마르코프 결정 과정)라는 이론과 OpenAI Gym이라는 표준 인터페이스가 있다. 에이전트에도 Memory, Action, Agent 같은 표준 클래스가 필요하다는 주장이다.
산업 쪽에는 팀마다 따로 에이전트를 만드는 대신 회사 공용 에이전트 라이브러리 하나를 유지하라고 권한다.

절차 기억의 두 형태도 구분한다.

- 코드 : 해석할 수 있지만 예상 못 한 상황에 약함
- LLM : 유연하지만 속을 알 수 없음

코드는 LLM의 한계를 보완하는 일반 알고리즘에 아껴 쓰는 게 좋다. 예로 자기회귀 생성이 앞만 보는 걸 보완하는 트리 탐색을 든다.

### **Agent design: thinking beyond simple reasoning**

설계 순서를 준다.

1. 필요한 기억 모듈을 정한다.
2. 기억별 읽기/쓰기 권한으로 내부 행동 공간을 정한다.
3. 의사결정 절차를 정한다.

쇼핑 도우미 예시가 나온다.

- 상품 목록 : 의미 기억 (읽기 전용)
- 고객 이력 : 일화 기억 (읽기+쓰기)
- 자기 코드 : 절차 기억 (읽기 전용, 스스로 고치면 안 되니까)

의사결정 절차가 복잡할수록 특정 도메인에 강해지고(Voyager), 단순할수록 범용이 된다(ReAct).

### **Structured reasoning: thinking beyond prompt engineering**

저수준 문자열 조작 대신 구조화된 출력으로 working memory 변수를 갱신하는 쪽을 권한다.
반대 방향도 있다. 에이전트에서 검증된 추론 형식(self-evaluation, reflection)이 LLM 학습 데이터에 들어가 모델 자체를 바꿀 거라는 예상이다.

### **Long-term memory: thinking beyond retrieval augmentation**

RAG는 사람이 쓴 문서를 읽기만 한다. 에이전트의 기억은 스스로 만든 내용을 읽고 쓴다.
평생 학습 경로는 이렇게 그린다.

1. 교과서(의미 기억)로 시작한다.
2. 경험(일화 기억)을 쌓는다.
3. 반성으로 새 지식을 만든다.
4. 코드 라이브러리(절차 기억)로 모은다.

### **Learning: thinking beyond in-context learning or finetuning**

학습을 "장기 기억에 쓰기" 전체로 넓히면, 자기 코드를 고치는 메타 학습까지 들어온다.

### **Action space: thinking beyond external tools or actions**

행동 공간은 명시적으로 정의해야 한다.
행동 공간이 클수록 할 수 있는 건 많지만 의사결정이 어려워진다. 그래서 과제를 푸는 최소 행동 공간이 낫다.

안전도 행동 공간 문제로 본다.

- learning 행동(자기 코드 수정·삭제) : 내부 위험
- grounding 행동(터미널의 rm, 위험 발언) : 외부 위험

키워드 필터 같은 태스크별 임시방편으로는 부족하고, 행동 공간 명세와 최악 시나리오 분석이 필요하다는 입장이다.

### **Decision making: thinking beyond action generation**

대부분의 에이전트가 아직 제안 하나를 바로 실행하는 수준이다.
제안-평가-선택을 제대로 쓰는 의사결정을 가장 유망한 방향 중 하나로 꼽는다.
-> 2023년에 나온 제안인데 지금 보면 꽤 많이 현실이 됐다. 구조화 출력은 function calling으로 흔해졌고, 에이전트 프레임워크 상당수가 Memory/Agent 같은 추상을 갖췄다.

## **7. Discussion**

7장은 프레임워크를 놓고 보면 드러나는 풀리지 않은 질문들이다. 2023년의 질문인데 대부분 아직도 질문이다.

### **LLMs vs VLMs**

추론은 언어로만 해야 하나?
관찰을 캡셔닝 모델로 텍스트로 바꾸는 모듈형과, 이미지를 VLM(이미지와 텍스트를 같이 받는 모델)의 표현 공간에 바로 투영하는 통합형이 있다.
논문은 둘 다 "비언어 모달리티를 추론 모델의 언어 영역으로 토크나이징하는 방식의 차이"로 본다.

### **Internal vs. external**

에이전트와 환경의 경계는 어디인가?
Wikipedia는 의미 기억인가 외부 환경인가? 논문은 통제할 수 있는지와 얼마나 묶여 있는지로 답한다.
위키는 다른 사용자가 언제든 바꿀 수 있으니 외부 환경이다.

### **Physical vs. digital**

동물은 한 번 살지만 디지털 에이전트는 리셋과 병렬 복제가 된다.
웹페이지 백만 개를 열어보는 탐색도 가능하니, 사람 인지에서 가져온 지금의 의사결정과는 다른 형태가 나올 수 있다.

### **Learning vs. acting**

대부분의 에이전트는 학습하는 시점이 정해져 있다(실패하면 반성, 세션 끝에 저장).
생물은 언제 무엇을 배울지도 고른다.
학습을 의사결정 사이클의 선택지로 두고, 적절한 때로 미루기도 하는 에이전트를 다음 단계로 본다.

### **GPT-4 vs GPT-N**

모델이 강해지면 아키텍처는 필요 없어지나?
GPT-2로는 에이전트가 안 됐고, GPT-4에서야 자기 평가가 어느 정도 믿을 만해졌다고 한다.
GPT-N이 기억, grounding, 의사결정을 통째로 컨텍스트 안에서 시뮬레이션하는 사고실험까지 던진다.
-> 프레임워크 논문이 자기 프레임워크가 덜 중요해지는 조건을 직접 적어둔 게 재밌다.

## **8. Conclusion**

에이전트 연구가 각자 만들어내던 개념들(기억, 스킬, 반성, 계획)이 인지 아키텍처가 수십 년 전에 정리한 구조와 거의 그대로 맞아떨어진다.
기억 4종, 내부/외부 행동, 제안-평가-선택 사이클이라는 말을 쓰면 흩어진 논문들이 한 장의 표로 정리된다.

ReAct에서 시작해 Voyager와 스몰빌까지 온 이 시리즈의 논문들도 그 표의 행이었다. 다음에 새 에이전트를 볼 때도 어떤 기억을 쓰는지, 행동 공간에 뭐가 있는지, 의사결정이 얼마나 복잡한지부터 보면 위치가 잡힐 것 같다.

## **9. 지금 관점: 흔히 쓰는 Planning/Memory/Tool Use 분류와 비교**

이 논문에는 벤치마크 수치가 없다. 기여는 전부 개념 정리다.
처음엔 Planning/Memory/Tool Use 3분류로도 같은 걸 물을 수 있지 않나 싶었다.
그런데 에이전트를 하나씩 놓고 보면 차이가 난다. 3분류로 물으면 "Memory 있음/없음"에서 답이 끝난다.

Voyager와 Generative Agents는 둘 다 "Memory 있음"이다.
CoALA로 보면 Voyager의 기억은 절차 기억(실행할 수 있는 코드)이고 Generative Agents의 기억은 일화·의미 기억(자연어)이다. 하나는 기억을 다시 실행하고 하나는 기억을 참고한다는 차이까지 나온다.

Planning도 마찬가지다. ReAct와 ToT는 둘 다 "Planning 함"인데, ReAct는 제안 하나를 바로 실행하고 ToT는 제안-평가-선택을 전부 돌린다.

이 시리즈에서 논문마다 따로 봤던 memory stream, 스킬 라이브러리, 반성문도 같은 틀의 다른 칸이었다.
다만 어떤 조합이 좋은지에 대한 실험 결과는 없다.
-> 새 에이전트를 어디에 놓을지는 알려주는데, 어떻게 만들면 잘 되는지는 직접 해봐야 한다.

7장의 경계 질문은 오히려 더 어려워진 것 같다.
function calling과 긴 컨텍스트가 모델에 들어가면서 working memory와 외부 기억의 경계, reasoning과 grounding의 경계가 논문이 쓰일 때보다 흐려졌다.

다음은 [LLM Agent Survey](https://momozzing.github.io/paper%20review/LLM-Agent-Survey-Paper-review/)다. 에이전트를 Profile·Memory·Planning·Action 네 모듈로 나눠 정리한 서베이다.
