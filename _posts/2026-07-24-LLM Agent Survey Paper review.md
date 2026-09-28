---
title: "LLM Agent Survey Paper review"
excerpt: "Profile·Memory·Planning·Action, 에이전트 4모듈이라는 통념의 출처. CoALA가 이론이라면 이 서베이는 2023년까지의 에이전트 연구 전체를 정리한 카탈로그다."
categories:
  - Paper review
tags:
  - Large Language Model
  - NLP
  - Agent
  - Paper review
mathjax: true
toc: true
toc_sticky: true
field: agent
---


A Survey on Large Language Model based Autonomous Agents

[https://arxiv.org/abs/2308.11432](https://arxiv.org/abs/2308.11432)

이 논문은 LLM 기반 자율 에이전트 연구를 모아서 정리한 서베이 논문이다.

[CoALA 리뷰](https://momozzing.github.io/paper%20review/CoALA-Paper-review/)에서 "통념의 3분류", "empirical survey"라고 불렀던 정리의 원류 중 하나가 이 논문이다.

CoALA가 인지 아키텍처에서 가져온 이론이라면, 이 서베이는 2021~2023년에 쏟아진 에이전트 연구를 다 읽고 거기서 공통점을 묶은 카탈로그다.

같은 대상을 반대 방향에서 정리한 두 논문이라 같이 읽어보면 좋다.

좀 더 자세히 알아보자.

## **1. Introduction**

예전 자율 에이전트 연구(강화학습 계열)는 고립된 환경에서 제한된 지식으로 정책을 학습했다.

LLM 기반 에이전트는 출발점이 다르다고 한다. 웹 지식을 통째로 가진 모델이 중심 제어기가 되니까, 도메인 데이터로 학습하지 않아도 행동할 수 있고 자연어로 상호작용할 수 있다.

![분야의 성장 추세 (논문 Figure 1)](https://momozzing.github.io/assets/images/wang-survey/fig1-growth.png)

2021년 WebGPT에서 시작해서 2023년에 논문이 확 늘어나는 그래프다.

앞에서 리뷰한 Toolformer(2023-2), Generative Agent(2023-4), Voyager(2023-5), ToT(2023-5)가 전부 이 타임라인 위에 있다.

이렇게 늘어난 연구들을 정리하는 게 이 논문의 목표라고 한다.

## **2. LLM-based Autonomous Agent Construction**

### **2.1 Agent Architecture Design**

에이전트를 4개 모듈로 나눈다.

1. Profile : 누구인가
2. Memory : 무엇을 기억하는가
3. Planning : 어떻게 계획하는가
4. Action : 무엇을 하는가

![에이전트 구조의 통합 프레임워크 (논문 Figure 2)](https://momozzing.github.io/assets/images/wang-survey/fig2-framework.png)

Profile이 Memory와 Planning에 영향을 주고, 셋이 모여서 Action을 정한다.

CoALA랑 바로 비교되는 게 있다. CoALA에는 Profile에 해당하는 모듈이 없다.

-> 과제를 푸는 에이전트 입장에서는 역할 설정이 부차적이다. 그런데 사람 행동을 시뮬레이션하는 쪽(Generative Agents 등)에서는 에이전트가 누구인지가 출발점이다. 서베이가 시뮬레이션 쪽까지 담으려다 보니 모듈이 하나 더 필요했던 것 같다.

#### **2.1.1 Profiling Module**

에이전트의 역할을 정하는 모듈이다.

나이, 직업 같은 인구통계 정보, 성격, 다른 에이전트와의 관계가 들어가고, 보통 프롬프트에 써서 LLM의 행동을 바꾼다.

만드는 방법은 셋이다.

1. 수작업 (스몰빌의 자연어 한 문단)
2. LLM 생성 (시드 몇 개로 대량 생성)
3. 데이터셋 정렬 (실제 인구 분포에 맞춤)

#### **2.1.2 Memory Module**

세 가지로 나눠서 본다.

- 구조 : 단기 기억만 쓰는 unified, 단기+장기를 나누는 hybrid
- 형식 : 자연어, 임베딩, DB, 구조화 리스트
- 연산 : 읽기, 쓰기, 반성

읽기는 기존 연구들을 수식 하나로 묶는다.

$$m^* = \arg\max_{m \in M} \left( \alpha \cdot s^{rec}(q,m) + \beta \cdot s^{rel}(q,m) + \gamma \cdot s^{imp}(m) \right)$$

recency, relevance, importance의 가중합이다.

-> 스몰빌 리뷰에서 본 3점수 회상을 일반형으로 쓴 것이다.

쓰기에서는 중복 기억 처리와 저장 한도 초과 문제를 다룬다.

반성은 Generative Agents와 Reflexion에서 본 그 반성이다.

#### **2.1.3 Planning Module**

계획을 피드백이 있는지 없는지로 나눈다.

피드백 없는 계획은 이렇다.

- 단일 경로 추론 : CoT, 한 줄로 쭉
- 다중 경로 추론 : CoT-SC, ToT, 트리로 탐색
- 외부 플래너 : LLM이 계획을 형식 언어로 바꾸고 고전 플래너가 푼다

![단일 경로와 다중 경로 추론 (논문 Figure 3)](https://momozzing.github.io/assets/images/wang-survey/fig3-planning.png)

피드백 있는 계획은 이렇다.

- 환경 피드백 : ReAct, Voyager
- 사람 피드백
- 모델 피드백 : 자기 비평

-> CoALA에서는 ToT가 왜 에이전트 논문들이랑 같이 놓이는지를 의사결정 축으로 설명했는데, 여기서는 "피드백 없는 다중 경로 계획"이라는 칸으로 설명된다. 같은 논문이 두 정리에서 다른 칸에 들어가 있다.

#### **2.1.4 Action Module**

행동 모듈은 질문 4개로 정리한다.

1. 무엇을 위해 행동하는가 (과제 완수, 탐색, 소통)
2. 행동을 어떻게 만드는가 (기억 회상, 계획 따르기)
3. 무엇을 할 수 있는가 (도구, 모델 자체 지식)
4. 행동의 결과는 무엇인가 (환경 변화, 새 행동 유발, 내부 상태 변화)

### **2.2 Agent Capability Acquisition**

프레임워크 말고 이 그림도 남는다.

![능력 획득 전략의 전환 (논문 Figure 4)](https://momozzing.github.io/assets/images/wang-survey/fig4-eras.png)

모델 능력을 얻는 전략이 시대마다 쌓여왔다고 한다.

1. 머신러닝 시대 : 파라미터 학습이 전부
2. LLM 시대 : 파라미터 학습 + 프롬프트 엔지니어링
3. 에이전트 시대 : 여기에 mechanism engineering이 더해진다

mechanism engineering은 파라미터도 프롬프트도 아니고, 모듈과 동작 규칙을 설계해서 능력을 만드는 것이다.

시행착오 루프(Voyager의 반복 개선), 반성 축적(Reflexion), 경험으로 스스로 배우기가 전부 여기 들어간다.

-> 이 시리즈에서 리뷰한 논문들이 다 mechanism engineering이었던 셈이다. 가중치를 안 바꾸고 에이전트를 강하게 만드는 방법들에 이 서베이가 이름을 붙여줬다.

## **3. LLM-based Autonomous Agent Application**

응용은 세 분야로 나눈다.

- 사회과학 : 사회 시뮬레이션, 심리, 법
- 자연과학 : 문서 관리, 실험 보조, 교육
- 공학 : SW 개발, 로봇, 산업 자동화

![응용 분야와 평가 전략 (논문 Figure 5)](https://momozzing.github.io/assets/images/wang-survey/fig5-app-eval.png)

그림 오른쪽은 다음 장에서 다루는 평가 전략이다.

## **4. LLM-based Autonomous Agent Evaluation**

평가는 두 가지로 나눈다.

- 주관 평가 : 사람 채점, 튜링 테스트류
- 객관 평가 : 메트릭, 프로토콜, 벤치마크

스몰빌의 believability 인터뷰가 주관 평가, Voyager의 테크 트리가 객관 평가다.

## **5. Challenges**

서베이가 꼽은 풀리지 않은 과제 6개다. 지금 봐도 대부분 그대로다.

1. Role-playing : LLM이 학습 데이터에 드문 역할(특정 전문가, 소수 성향)은 잘 연기하지 못한다.
2. Generalized human alignment : 시뮬레이션이 목적이면 나쁜 사람까지 정확히 시뮬레이션해야 하는데, 정렬된 모델은 그걸 거부한다. 정렬이 시뮬레이션을 막는 셈이다.
3. Prompt robustness : 모듈이 늘수록 프롬프트가 서로 얽혀서 조금만 고쳐도 전체가 흔들린다.
4. Hallucination : 에이전트는 환각이 행동으로 이어지니까 피해가 커진다.
5. Knowledge boundary : 시뮬레이션에서는 모델이 너무 많이 아는 게 문제다. 일반인을 연기해야 하는데 웹 전체를 아는 사람처럼 군다.
6. Efficiency : 행동 하나에 LLM 호출이 여러 번이라 느리고 비싸다.

-> 2번이랑 5번은 "모델이 좋아질수록 시뮬레이션은 어려워진다"는 쪽의 문제다. 성능이 올라간다고 다 풀리는 건 아닌 것 같다.

## **6. 지금 관점: 두 정리의 역할 분담**

이 서베이(2023-8)와 CoALA(2023-9)는 한 달 차이로 나온 같은 분야의 정리다.

이 서베이는 무엇이 있었는지를 빠짐없이 모은 귀납적 카탈로그고, CoALA는 어떻게 조직해야 하는지를 제시한 연역적 이론이다.

같은 항목끼리 붙여보면 이렇다.

1. 어떻게 만든 정리인가 : 서베이는 논문 수백 편을 읽고 공통점을 묶었다. CoALA는 인지과학의 오래된 이론을 가져와 적용했다.
2. 에이전트를 몇 조각으로 나누나 : 서베이는 4개(역할, 기억, 계획, 행동), CoALA는 3개(기억, 행동, 결정)다.
3. 역할(페르소나) : 서베이는 첫 번째 모듈로 크게 다룬다. CoALA에는 없다.
4. 학습 : 서베이는 따로 다루지 않고 여기저기 흩어져 있다. CoALA는 행동의 한 종류로 정식으로 다룬다.
5. LLM은 무엇인가 : 서베이에서는 에이전트의 두뇌, CoALA에서는 여러 부품 중 하나다.
6. 언제 꺼내 쓰나 : 서베이는 구현 옵션을 고를 때의 선택지 목록, CoALA는 구조를 뜯어볼 때의 질문지다.

3번이랑 4번이 반대로 되어 있다. 서베이는 역할을 크게 두고 학습을 흩어놨고, CoALA는 역할을 빼고 학습을 정식으로 올렸다.

-> 사람을 시뮬레이션하는 관점이랑 과제를 수행하는 관점의 차이가 그대로 나온 것 같다.

서베이의 4모듈 용어(Profile, Memory, Planning, Action)는 이후 강의, 문서, 프레임워크 설명이 따라가는 사실상 표준이 됐다. CoALA 리뷰에서 "통념"이라고 부른 것도 상당 부분 여기서 왔다.

쓰는 때도 다르다.

특정 모듈의 구현 옵션을 고를 때(기억을 어떤 형식으로 저장할까, 계획에 피드백을 넣을까)는 이 서베이의 분류가 선택지 목록이 된다. 에이전트 전체 구조를 따질 때는 CoALA의 질문이 낫다.

-> 카탈로그랑 이론은 서로를 대신하지 않는다.

## **7. Conclusion**

2021년부터 2023년까지의 LLM 에이전트 연구를 Profile, Memory, Planning, Action 4모듈로 정리하고, 응용과 평가까지 한 장의 지도로 만들었다.

새 기법을 만든 논문이 아니고 용어를 정리한 논문인데, 그 용어가 실제로 표준이 됐다.

능력 획득의 세 시대 구분도 남는다. 파라미터 학습, 프롬프트 엔지니어링, 그리고 mechanism engineering이다.

여태까지 에이전트 논문들이 각자 기법을 하나씩 내놨다면, 이 서베이는 그것들을 모아서 이름을 붙였다.

이 시리즈에서 리뷰한 논문들은 세 번째 시대의 초기 기록이다.
