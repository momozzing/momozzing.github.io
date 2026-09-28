---
title: "Toolformer Paper review"
excerpt: "사람 어노테이션 없이 LM이 스스로 도구 사용법을 배운다. '이 도구 호출이 다음 토큰 예측을 쉽게 하는가'라는 필터 하나로 학습 데이터를 만든 논문."
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

Toolformer: Language Models Can Teach Themselves to Use Tools

[https://arxiv.org/abs/2302.04761](https://arxiv.org/abs/2302.04761)

Toolformer는 Meta AI에서 만든, 도구 쓰는 법을 스스로 배우는 언어모델이다. (NeurIPS 2023)

[ReAct](https://momozzing.github.io/paper%20review/ReAct-Paper-review/)와 [Reflexion](https://momozzing.github.io/paper%20review/Reflexion-Paper-review/)은 프롬프팅으로 도구를 쓰게 했는데, Toolformer는 파인튜닝으로 모델이 도구를 쓰게 만든다.

그리고 그 학습 데이터를 사람 어노테이션 없이 모델이 스스로 만든다.

좀 더 자세히 알아보자.

## **1. Introduction**

LLM은 few-shot으로 새로운 태스크를 풀 만큼 잘하는데, 사칙연산이나 최신 정보 조회 같은 기본적인 건 계산기나 검색엔진보다 못하다.

도구를 붙이면 되긴 하는데, 기존 방법들은 사람 어노테이션이 많이 필요하거나 특정 태스크에서만 동작했다고 한다.

그래서 Toolformer는 조건을 두 가지 세운다.

1. 도구 사용을 self-supervised로 배울 것. 사람이 유용하다고 생각하는 것과 모델에게 실제로 유용한 것은 다르다고 한다.
2. 모델의 일반성을 잃지 않을 것. 언제 어떤 도구를 쓸지는 모델이 스스로 정한다.

![Toolformer의 예측 예시 (논문 Figure 1)](https://momozzing.github.io/assets/images/toolformer/fig1-examples.png)

학습이 끝난 모델은 텍스트를 생성하다가 필요한 곳에서 `[QA(...)]`, `[Calculator(400 / 1400)]` 같은 API 호출을 스스로 넣는다.

그리고 결과를 받아서 이어서 생성한다.

## **2. Approach**

API 호출을 텍스트로 표현하는 게 출발점이다.

호출은 `[API(입력) → 결과]` 형태의 특수 토큰 시퀀스로 일반 텍스트 사이에 끼워 넣는다. 이렇게 하면 도구 사용 학습이 그냥 language modeling이 된다.

학습 데이터는 3단계로 만든다.

![데이터 생성 3단계 (논문 Figure 2)](https://momozzing.github.io/assets/images/toolformer/fig2-method.png)

1. Sample : few-shot 프롬프트로 LM이 일반 텍스트(CCNet)에 API 호출 후보를 넣게 한다
2. Execute : 후보 API 호출을 실제로 실행해서 결과를 받는다
3. Filter : 호출이 쓸모 있었는지 보고 쓸모없는 호출은 버린다

이 논문에서 제일 중요한 게 3번 필터 기준이다.

API 호출과 그 결과를 프리픽스로 줬을 때, 뒤에 오는 토큰들의 loss가 충분히 줄어드는지를 잰다.

$$L_i^- - L_i^+ \geq \tau_f$$

- $L_i^+$ : 호출+결과를 줬을 때의 loss
- $L_i^-$ : 호출을 아예 안 했을 때와, 결과 없이 호출만 했을 때의 loss 중 작은 것

"이 도구 호출이 다음 토큰 예측에 도움이 됐는가"를 모델 자신의 loss로 판단하는 거다. 사람이 판단할 자리가 없다.

이 필터를 통과한 호출만 원문에 끼워 넣어 증강 데이터셋을 만들고, 그걸로 모델(GPT-J 6.7B)을 파인튜닝한다.

파인튜닝 데이터가 원래 사전학습에 쓰던 것과 같은 종류의 텍스트라서 일반성을 잃지 않는다고 한다.

![QA 도구용 어노테이션 프롬프트 (논문 Figure 3)](https://momozzing.github.io/assets/images/toolformer/fig3-prompt.png)

도구마다 필요한 건 이런 프롬프트 하나랑 예시 몇 개가 전부다.

## **3. Tools**

도구는 5개를 붙였다.

1. QA 시스템 (Atlas)
2. 계산기
3. Wikipedia 검색 (BM25)
4. 기계번역 (NLLB 600M)
5. 달력

전부 입출력을 텍스트로 표현할 수 있는 것들이다.

## **4. Experiments**

모든 태스크를 zero-shot으로 평가한다.

비교 대상은 GPT-J 계열 베이스라인과, 훨씬 큰 OPT(66B), GPT-3(175B)다.

### **4.2 Downstream Tasks**

#### **4.2.1 LAMA**

사실 조회 태스크다.

SQuAD 기준으로 GPT-J 17.8 → Toolformer 33.8이다. 6.7B 모델이 GPT-3 175B(26.8)를 넘는다.

![LAMA 결과 (논문 Table 3)](https://momozzing.github.io/assets/images/toolformer/table3-lama.png)

Toolformer (disabled) 줄은 같은 모델에서 API 호출만 막은 결과다.

모델은 예제의 98.1%에서 QA 도구를 부르기로 스스로 정했다고 한다.

#### **4.2.2 Math Datasets**

ASDiv, SVAMP, MAWPS 전부에서 OPT랑 GPT-3를 크게 이긴다. 계산기 호출 비율은 97.9%다.

![수학 벤치마크 결과 (논문 Table 4)](https://momozzing.github.io/assets/images/toolformer/table4-math.png)

API를 끈 Toolformer(disabled)도 베이스라인보다 오르는데, API 호출을 허용하면 그 두 배 이상이 된다.

-> API를 꺼도 오르는 건 좀 신기하다. 파인튜닝 데이터 자체가 도움이 된 건가??

#### **4.2.3 Question Answering**

WebQS, NQ, TriviaQA에서는 같은 크기 베이스라인은 이기는데 GPT-3 175B에는 진다.

![QA 벤치마크 결과 (논문 Table 5)](https://momozzing.github.io/assets/images/toolformer/table5-qa.png)

논문은 원인을 두 가지로 든다.

1. 검색엔진이 단순해서 결과 품질이 낮다
2. 결과가 나쁠 때 질의를 고쳐서 다시 검색하는 상호작용이 안 된다

-> 2번은 검색 → 결과 확인 → 재검색 루프가 없다는 건데, ReAct가 프롬프팅으로 풀었던 문제랑 같다.

#### **4.2.4 Multilingual Question Answering**

MLQA에서는 번역 도구를 쓰긴 하는데 GPT-J를 일관되게 이기지는 못한다.

CCNet으로 파인튜닝한 게 사전학습 분포와 어긋나서 그런 것 같다고 한다.

### **4.4 Scaling Laws**

GPT-2 계열 작은 모델들로 같은 실험을 해보면, 775M 미만 모델은 도구를 줘도 잘 쓰지 못한다고 한다.

![모델 크기별 도구 활용 효과 (논문 Figure 4)](https://momozzing.github.io/assets/images/toolformer/fig4-scaling.png)

도구가 도움이 되기 시작하는 건 언어 능력이 어느 정도 있는 모델부터다.

그리고 모델이 커져도 도구 유무에 따른 차이는 줄지 않는다고 한다.

-> 언제 무엇을 물어볼지 판단하는 것 자체가 언어 능력이라서 그런 것 같다.

## **5. Limitations**

1. 도구를 연쇄(chain)할 수 없다. 한 도구의 출력을 다른 도구의 입력으로 넣는 걸 말한다. 예를 들어 "미국 1인당 GDP"를 구하려면 검색으로 GDP와 인구를 얻고 그 두 값을 계산기에 넣어야 한다. 그런데 데이터를 만들 때 도구별로 API 호출을 따로 샘플링해서, 이런 예시가 학습 데이터에 없다.
2. 검색 결과를 보고 다시 질의하는 상호작용이 안 된다.
3. 필터를 통과하는 호출이 적어서 sample-inefficient하다. 특히 계산기 호출은 후보 대부분이 버려진다.
4. 호출 비용을 고려하지 않는다.

## **6. 지금 관점: function calling과 비교**

요즘 모델들이 기본으로 갖고 있는 native function calling은 도구 호출 데이터로 학습한 결과다.

Toolformer는 그 학습 데이터를 만드는 방식을 처음 보여준 논문이다. 후보 생성 → 실행 → 필터 → 파인튜닝 구조는 지금 tool-use 학습 데이터 만드는 방식이랑 뼈대가 같다.

필터 기준도 이어진다. 실행해보고 도움이 된 것만 남기는 건 지금의 rejection sampling 기반 데이터 정제와 같은 발상이다.

정답 라벨 대신 모델의 loss를 기준으로 쓴 게 2023년 초에 이 논문이 한 일이다.

한계였던 연쇄와 상호작용은 반대쪽 계열이 채웠다.

ReAct가 루프를 만들고 Reflexion이 루프에 학습을 넣었다면, Toolformer는 루프 안에서 실행되는 도구 호출 능력 자체를 모델에 학습시켰다.

-> 지금 agent는 이 두 계열을 합친 모양인 것 같다. 파인튜닝으로 배운 function calling을 프롬프팅 루프가 돌리는 구조다.

## **7. Conclusion**

사람 어노테이션 없이, "이 도구 호출이 다음 토큰 예측을 쉽게 하는가"라는 필터 하나로 도구 사용 학습 데이터를 만들 수 있다는 걸 보여준다.

6.7B 모델이 175B를 이기는 태스크도 나왔다.

대신 연쇄가 안 되고, 상호작용이 안 되고, 도구가 단순하면 큰 모델에 진다.

여태까지 도구 사용이 사람 어노테이션이나 프롬프팅에 기대고 있었다면, 이 방법은 모델이 자기 loss로 쓸모 있는 호출을 골라 스스로 배운다.

-> 도구를 쓸 줄 아는 것과 잘 쓰는 건 다른 문제 같다. 잘 쓰려면 루프가 필요하고, 이후 연구들이 그쪽으로 갔다.
