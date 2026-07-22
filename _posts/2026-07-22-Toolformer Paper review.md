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
---

Toolformer: Language Models Can Teach Themselves to Use Tools

[https://arxiv.org/abs/2302.04761](https://arxiv.org/abs/2302.04761)

Toolformer는 Meta AI에서 나온 논문이다. (NeurIPS 2023)

[ReAct](https://momozzing.github.io/paper%20review/ReAct-Paper-review/)와 [Reflexion](https://momozzing.github.io/paper%20review/Reflexion-Paper-review/)이 프롬프팅으로 도구를 쓰게 했다면, Toolformer는 접근이 다르다. **파인튜닝으로 도구 사용을 모델 스스로 한다.** 그리고 그 학습 데이터를 사람 어노테이션 없이 모델이 스스로 만든다.

## **Introduction**

LLM은 few-shot으로 새로운 태스크를 풀 만큼 똑똑한데, 정작 사칙연산이나 최신 정보 조회 같은 기본 기능에서는 계산기나 검색엔진보다 못하다. 도구를 붙이면 해결되지만, 기존 접근은 대량의 사람 어노테이션이 필요하거나 특정 태스크 전용으로만 동작했다.

Toolformer가 세운 조건은 두 가지다.

- 도구 사용을 **self-supervised**로 배울 것. 사람이 유용하다고 생각하는 것과 모델에게 실제로 유용한 것은 다르다
- 모델의 일반성을 잃지 않을 것. 언제 어떤 도구를 쓸지 **모델이 스스로 결정**할 것

![Toolformer의 예측 예시 (논문 Figure 1)](https://momozzing.github.io/assets/images/toolformer/fig1-examples.png)

학습이 끝난 모델은 텍스트를 생성하다가 필요한 지점에서 `[QA(...)]`, `[Calculator(400 / 1400)]` 같은 API 호출을 스스로 삽입하고, 결과를 받아서 이어서 생성한다.

## **Approach**

핵심 아이디어는 API 호출을 텍스트로 표현하는 것이다. 호출은 `[API(입력) → 결과]` 형태의 특수 토큰 시퀀스로 일반 텍스트 사이에 끼워 넣는다. 그러면 도구 사용 학습이 그냥 language modeling이 된다.

학습 데이터는 3단계로 만든다.

![데이터 생성 3단계 (논문 Figure 2)](https://momozzing.github.io/assets/images/toolformer/fig2-method.png)

1. **Sample**: few-shot 프롬프트로 LM이 일반 텍스트(CCNet)에 API 호출 후보를 삽입하게 한다
2. **Execute**: 후보 API 호출을 실제로 실행해서 결과를 받는다
3. **Filter**: 호출이 쓸모 있었는지 검사해서 쓸모없는 호출을 버린다

필터 기준이 이 논문의 핵심이다. **"API 호출과 그 결과를 프리픽스로 줬을 때, 뒤따르는 토큰들의 loss가 충분히 줄어드는가"**를 잰다.

$$L_i^- - L_i^+ \geq \tau_f$$

$L_i^+$는 호출+결과를 줬을 때의 loss, $L_i^-$는 호출을 아예 안 하거나 결과 없이 호출만 했을 때의 loss 중 작은 것이다. 즉 "이 도구 호출이 다음 토큰 예측에 실제로 도움이 됐는가"를 모델 자신의 loss로 판정한다. 사람의 판단이 개입할 자리가 없다.

이 필터를 통과한 호출만 원문에 끼워 넣어 증강 데이터셋을 만들고, 그 데이터로 모델(GPT-J 6.7B)을 파인튜닝한다. 파인튜닝 데이터가 원래 사전학습에 쓰던 것과 같은 종류의 텍스트라서 일반성을 잃지 않는다.

![QA 도구용 어노테이션 프롬프트 (논문 Figure 3)](https://momozzing.github.io/assets/images/toolformer/fig3-prompt.png)

도구별로 필요한 것은 이런 프롬프트 하나와 예시 몇 개가 전부다.

도구는 5개를 붙였다. QA 시스템(Atlas), 계산기, Wikipedia 검색(BM25), 기계번역(NLLB 600M), 달력. 전부 입출력이 텍스트로 표현되는 것들이다.

## **Experiments**

전 태스크 zero-shot으로 평가한다. 비교 대상은 GPT-J 계열 베이스라인과 훨씬 큰 OPT(66B), GPT-3(175B)다.

![LAMA와 수학 벤치마크 결과 (논문 Table 3, 4)](https://momozzing.github.io/assets/images/toolformer/table34-lama-math.png)

**사실 조회(LAMA)**: SQuAD 기준 GPT-J 17.8 → Toolformer 33.8로, GPT-3 175B(26.8)를 6.7B 모델이 넘는다. 모델은 예제의 98.1%에서 QA 도구를 호출하기로 스스로 결정했다.

**수학(ASDiv, SVAMP, MAWPS)**: 전 벤치마크에서 OPT와 GPT-3를 크게 이긴다. 계산기 호출 비율은 97.9%다. 재미있는 것은 API를 끈 Toolformer(disabled)도 베이스라인보다 오르는데, API 호출을 허용하면 그 두 배 이상이 된다는 점이다.

![QA 벤치마크 결과 (논문 Table 5)](https://momozzing.github.io/assets/images/toolformer/table5-qa.png)

**QA(WebQS, NQ, TriviaQA)**: 같은 크기 베이스라인은 이기지만 **GPT-3 175B에는 진다.** 논문이 밝히는 원인은 두 가지다. 검색엔진이 단순해서 결과 품질이 낮고, 결과가 나쁠 때 질의를 고쳐서 다시 검색하는 **상호작용이 불가능**하다. 검색 → 결과 확인 → 재검색 루프가 없다는 것인데, 이건 ReAct가 프롬프팅으로 풀었던 바로 그 문제다.

**다국어 QA(MLQA)**: 번역 도구를 쓰긴 하지만 GPT-J를 일관되게 이기지 못한다. CCNet 파인튜닝이 사전학습 분포와 어긋난 탓으로 추정한다. 이것도 논문이 숨기지 않는 실패다.

## **Scaling**

![모델 크기별 도구 활용 효과 (논문 Figure 4)](https://momozzing.github.io/assets/images/toolformer/fig4-scaling.png)

GPT-2 계열 작은 모델들로 같은 실험을 하면, **775M 미만 모델은 도구를 줘도 활용하지 못한다.** 도구 사용이 도움이 되기 시작하는 것은 충분한 언어 능력이 있는 모델부터다. 언제 무엇을 물어볼지 판단하는 것 자체가 언어 능력이기 때문이다. 그리고 모델이 커져도 도구 유무의 격차는 줄지 않는다.

## **Limitations**

- 도구를 **연쇄(chain)**할 수 없다. 연쇄란 한 도구의 출력을 다른 도구의 입력으로 넣는 것이다. 예를 들어 "미국 1인당 GDP"를 구하려면 검색으로 GDP와 인구를 얻고 그 두 값을 계산기에 넣어야 하는데, 데이터 생성 시 도구별로 API 호출을 독립적으로 샘플링하기 때문에 이런 예시가 학습 데이터에 존재하지 않는다
- 검색 결과를 보고 재질의하는 **상호작용**이 안 된다
- 필터를 통과하는 호출이 적어서 sample-inefficient하다. 특히 계산기 호출은 후보의 대부분이 버려진다
- 호출 **비용**을 고려하지 않는다

## **지금 관점: function calling 학습의 원형**

요즘 모델들이 기본으로 갖춘 native function calling은 도구 호출 데이터로 학습된 결과다. Toolformer는 그 학습 데이터를 만드는 파이프라인의 원형이다. 후보 생성 → 실행 → 필터 → 파인튜닝이라는 구조는 지금의 tool-use 학습 데이터 구축과 같은 뼈대다.

필터 기준도 이어지고 있다. "실행해보고 도움이 된 것만 남긴다"는 self-supervised 필터링은 지금의 rejection sampling 기반 데이터 정제와 같은 발상이다. 정답 라벨 대신 모델의 loss를 기준으로 쓴 것이 2023년 초 시점의 기여다.

한계였던 연쇄와 상호작용은 반대편 계열이 채웠다. ReAct가 루프를 만들고 Reflexion이 루프에 학습을 넣었다면, Toolformer는 루프 안에서 실행되는 도구 호출 능력 자체를 모델에 학습시켰다. 지금의 agent는 이 두 계열의 결합 위에 서 있다. 파인튜닝으로 모델에 학습시킨 function calling을, 프롬프팅 루프가 굴리는 구조다.

## **Conclusion**

사람 어노테이션 없이, "이 도구 호출이 다음 토큰 예측을 쉽게 하는가"라는 필터 하나로 도구 사용 학습 데이터를 만들 수 있다는 것을 보였다. 6.7B 모델이 175B를 이기는 태스크가 나올 만큼 효과는 분명하다.

동시에 한계도 분명하다. 연쇄가 안 되고, 상호작용이 안 되고, 도구가 단순하면 큰 모델의 순수 능력에 진다. 도구를 쓸 줄 아는 것과 도구를 잘 쓰는 것은 다른 문제고, 후자는 루프 구조가 필요하다는 것이 이후 연구들이 간 방향이다.
