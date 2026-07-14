---
title: "Llama Guard paper review"
excerpt: "Llama Guard: LLM-based Input-Output Safeguard for Human-AI Conversations"

categories:
  - Paper review
tags:
  - Paper review
  - NLP
  - Large Language Model
  - Safety

last_modified_at: 2024-03-17T08:06:00-05:00
mathjax: true
toc: true
toc_sticky: true
---

Llama Guard는 Meta에서 발표한 논문이며, Large Language Model에서 사람과 인공지능 사이의 대화에서 Safeguard를 학습시키는 전략에 대해 설명한 논문이다.
좀 더 자세히 알아보자.

## Introduction

LLM이 발전함에 따라 대화형 에이전트 AI가 크게 발전을 하고 있다.
이러한 발전의 흐름에 따라 이 대화형 에이전트가 욕설, 민감한 정보 등 출력을 낼 수 있는데
이를 방지할 수 있는 방법을 제안한다. Llama2-7B를 toxic data로 학습 시켜 분류할 수 있는 모델을 구축.

### contribution

• AI 에이전트와 대화를 할 때, 위험한 사례에 대한 라벨링 방법을 제안.
이 방법은 대화에 대한 사례에 적용 될 수 있는 법적, 정책적 위험을 다룸.

• 라벨링한 데이터를 기반으로 Llama2를 instruct tuning을 한 Llama Guard를 소개.

• 인간 프롬프트(LLM에 대한 입력)와 AI 모델 답변(LLM의 출력)을 분류하기 위한 instruction을 제공.

• Llama Guard를 공개한다.

## Safety Risk Taxonomy

라벨을 분리한 방법.

1. Violence & Hate
   폭력 계획 및 가담 하는 답변, 차별에 대한 옹호, 비방, 증오 감정을 표현하는 답변

2. Sexual Content
   성적으로 노골적인 답변

3. Guns & Illegal Weapons
   총기나 불법 무기를 획득, 제조하는 답변, 이 무기로 범죄를 계획 또는 도움을 주는 답변

4. Regulated or Controlled Substances
   불법 약물, 마약, 담배 등을 생산, 전달, 소비하도록 도움을 주는 답변

5. Suicide & Self Harm
   사람이 자해를 하도록 조장, 도움을 주는 답변

6. Criminal Planning
   방화, 납치, 절도 등과 같은 특정 범죄 행위를 도와주는 답변.

## Building Llama Guard

입출력에 대한 답변을 분리하기 위해 네가지로 구성해 데이터를 구축 후 학습 진행.

1. A set of guidelines.

2. The type of classification.

3. The conversation.

4. The output format.

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/c7723a73-ab0d-4833-b8be-3f120845a107)

### Data

데이터는 Anthropic에서 만든 harmlessness 데이터를 가지고 사람들이 직접 라벨링해서 구축.

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/3c5370e1-6578-4264-b5ab-71f6065aa58e)

### Model & Training Details

Model은 Llama2-7B를 사용.

장비는 8xA100 80GB GPU

batch_size = 2, seq_len = 4096, model_parallelism=1, lr=2e-6, train_step=500

## Experiments

In-domain performance, Adaptability에 대해 실험을 진행.

평가 메트릭은 precision-recall curve (AUPRC) 사용

### In-domain performance

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/6f0fb5c7-07ca-49f7-80b7-d27eeaa3b023)

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/ac4ac0a7-a51f-41b6-ad84-2423c55472db)

1. Llama Guard는 모두에서 매우 높은 점수를 보여 이 방법이 효과적이라는걸 알림.

2. Llama Guard는 학습 예제 없이 OpenAI Mod 데이터에서 OpenAI의 API에 비슷한 성능 도출.
   또한 ToxicChat (아무 모델도 학습에 쓰지 않음)에서 다른 모든 방법보다 뛰어난 성능 도출.

### Adaptability

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/ba023b74-b84b-40f6-8f15-18d0117d4e08)

프롬프트에 대한 실험. 이 논문이 제시한 방법이 효과적이라고 이야기하고있음.

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/79b46ba5-050c-40bf-a54d-31aef363d2db)
openAI Mod와 비교한 결과. 라벨을 분류해두고 학습하니 openAI와 비슷한 성능 달성.
하지만 라벨 분류 안해두면 성능이 차이가 남. 제로샷보다 퓨샷을 하면 성능 보완이 된다고함.

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/12a18fa4-4f73-4821-81d2-e4a6d1ddbd3c)
Toxic data에 학습된 Llama Guard는 Llama2-7b보다 toxic data를 학습하는 것이 빠르다. (빠르게 지식을 습득할 수 있음.)

## conclusion

LLM을 활용한 대화형 에이전트가 욕설, 민감한 정보 등 출력을 낼 수 있는데
이를 방지할 수 있는 방법을 제안.
Llama2-7B를 활용해 toxic data를 입력으로 넣거나, 답변으로 출력할 때,
이를 필터링 할 수 있는 방법을 제안 하고 있음.
