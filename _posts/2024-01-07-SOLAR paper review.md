---
title:  "SOLAR Paper review"
excerpt: "SOLAR 10.7B: Scaling Large Language Models with Simple yet Effective
Depth Up-Scaling Paper review"

categories:
  - Paper review
tags:
  - Paper review
  - NLP
  - Large Language Model

last_modified_at: 2024-01-07T08:06:00-05:00
mathjax: true
# classes: wide
toc: true
toc_sticky: true
---

SOLAR는 업스테이지에서 만든 오픈소스 LLM이다.

허깅페이스의 Open LLM Leaderboard 영어, 한국어에서 가장 뛰어난 성능을 가지는 모델이다.

이 모델이 어떻게 구성이 되어있는지 알아보자.

# **Introduction**
본 논문은 10.7B의 크기를 가진 Large language model(LLM)인 SOLAR 10.7B를 소개한다.

SOLAR는 depthwise scaling, continued pretraining 방법을 적용한 depth up-scaling(DUS)이라는 방법을 제시한다.

다른 최신 LLM 들은 Mixture-of-Experts(MOE) 방법을 사용하는데 

DUS를 사용하면 간단하면서 효과적이라는 것을 실험적으로 보여준다. 

DUS를 적용한 SOLAR에 추가적으로 instruct tuning을 하니 Mixtral-8x7B-Instruct보다 좋은 성능을 낸다.

# **Depth Up-Scaling**
![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/91f851a5-9bbd-4ad3-8493-b287d68be12e)

## **Base model**
아키텍쳐는 LLAMA2를 따름.
LLAMA2와 호환이 되는 Mistral 7B를 baseline 모델로 선정.

## **Depthwise scaling**

1. Figure 1에 따르면, Mistral 7B모델 두개를 가져온다.(32개의 Layer를 가지고 있음.) 
2. 하나는 아래 24개 Layer, 다른 하나는 위의 24개 Layer를 가져와 머지해 48개의 Layer를 가진 Mistral model(10.7B)을 만든다. 
3. 합친 모델을 continual learning 한다. 

## **Continued pretraining**
맨 처음에 그냥 합친 모델은 성능이 기존 7B보다 떨어진다.

따라서 Continued pretraining을 진행했다. 

-> 그래서 어떤 데이터를 썼다는건지?? mistral학습했던 데이터 그대로 추가학습을 했다는건가?

## **Comparison to other up-scaling methods**
다른 up-scaling method들과 비교했을때, 

gating networks or dynamic expert selection같은 모듈들은 필요가 없고,

기본적인 허깅페이스와 같은 프레임워크에 쉽게 통합해서 만들 수 있다. 

# **Training Details**
## **instruction tuning**
평가 매트릭스 중 하나인 GSM8K(Cobbe et al., 2021) 와 겹치지 않기 위해 Math (Hendrycks et al., 2021)에 대해서 데이터를 수집을 했고 MetaMath (Yu et al., 2023) 이 방식을 사용해 데이터를 만들어서 학습했다.

## **Alignment tuning**
DPO활용

Math (Hendrycks et al., 2021)에 대해서 데이터를 수집한 것이 rejected

MetaMath (Yu et al., 2023) 이 방식을 사용해 만든것이 chosen

이렇게 해서 DPO를 학습했다. 

# **Results**

## **Experimental Details**
### **Training datasets.**
학습데이터는 대부분 오픈소스 데이터.

instruct tuning data 
Alpaca 스타일의 템플릿을 사용해 데이터를 다시 포맷.
FLAN(Longpre et al., 2023)에서 파생된 OpenOrca와 같은 데이터는 벤치마크 데이터셋과 겹치는 부분을 필터링 했다.

alignment datasets는 {prompt, chosen, rejected}로 구성. Zephyr(Tunstall et al., 2023)에 따라 전처리 진행.

### **Evaluation.**
HuggingFace Open LLM Leaderboard

## **Main Results**
![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/b1359781-24ad-488c-8f3d-325ba4a6a590)
Solar가 1등을 찍었다

## **Ablation Studies**

### **Instruction Tuning**
**Ablation on the training datasets**
![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/26997684-03fc-40bb-b527-5c7857a84f57)

SFT v3, SFT v4 = OpenOrca 유무에 관계없이 최고의 성능을 발휘하는 모델이므로 병합을 함 
-> SFT v3 + SFT v4 가장 좋았던 애들 합치니까 성능이 좋았음. 

### **Alignment Tuning**
**Ablation on the training datasets**
![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/8a9a24bd-0619-40bf-aebb-bedb8663ccc0)

아까 만든 수학 데이터를 DPO로 사용하는것이 성능향상에 유의미했음. 

**Ablation on the SFT base models.**
![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/6df130ad-8270-4ca5-a5dd-1fb93d28679f)

DPO v3은 SFT 기본 모델로 SFT v3+v4를 사용

SFT 모델의 성능 격차는 DPO모델에서 똑같이 나오지는 않음. 

**Ablation on different merge methods.**
![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/51126040-9000-4da2-9dec-18c32f5a9108)

merge를 하기 위해 후보들을 선택함(Cand. 1, Cand. 2). 

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/941170a1-2ef4-43d2-8852-d564fa270d00)

두개의 merge method를 사용.

1. model의 weight를 비율을 가지고 평균을 내는 방식
2. SLERP (Shoemake, 1985).

첫번째가 제일 좋아서 첫번째를 사용하고, 그중 Merge v1이 가장 좋아서 SOLAR 10.7B-Instruct model이 됨.

# **Conclusion**
Solar가 짱이다. Depth Up-Scaling 는 효과적이었다. 