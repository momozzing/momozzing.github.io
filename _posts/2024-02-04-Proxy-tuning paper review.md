---
title:  "Proxy tuning Paper review"
excerpt: "Proxy-tuning: Tuning Language Models by Proxy"

categories:
  - Paper review
tags:
  - Paper review
  - NLP
  - Large Language Model

last_modified_at: 2024-02-04T08:06:00-05:00
mathjax: true
# classes: wide
toc: true
toc_sticky: true
---

Proxy tuning은 Allen AI에서 발표한 논문이며, 계속 커져나가는 Large Language model에서 모델 튜닝을
효율적으로 하는 방법을 새롭게 제시한 논문이다. 

좀 더 자세히 알아보자. 


# **Introduction**

계속 커져 나가는 Large language model을 파인튜닝을 하려면, 많은 리소스가 필요하다. 따라서 본 논문은 Proxy tuning이라는 효율적으로 학습을 시킬 수 있는 방법론을 제안한다

이 방법은 LLAMA2처럼 7B, 13B, 70B처럼 같은 토크나이저를 가지고 같은 계열의 모델에 적용 할 수 있다. 

또한 기존의 PEFT처럼 새로운 파라미터를 삽입하는 것이 아니다.

예로 들면, Target으로 튜닝하고 싶은 모델은 LLAMA2 70B 모델이다. 이 70B 모델을 Proxy tuning 하기 위해 LLAMA2 7B를 사용해 원하는 데이터로 fine-tuning을 진행한다. 그리고 학습된 7B모델과 학습이 안된 7B모델의 output logit값을 뺀다. 빼고 난 logit값을, baseline에 더해, 7B에서 학습된 모델의 logit값을 70B에 적용하는, 아주 간단한 메커니즘을 가진 방법이다.

이렇게 했을 때, 전반적인 task내에서 fine-tuning과 비교해서 88퍼 정도의 성능을 달성하며, TruthfulQA에서 fine-tuning 대비 성능이 좋았다고 한다. 

# **Proxy tuning**
![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/4a16f598-af60-44e0-8df6-3e06ca706986)

target large language model(llama-13B) = $S_(M)$

tuned small language model (chat-llama-7B) = $S_(M^+)$

untuned small model (llama-7B) = $S_(M^-)$

target language model의 logit값= softmax(($S_M$의 logit값) + ($S_{M^+}$의 logit값 - $S_{M^-}$의 logit값))

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/4d177af6-4f50-4b97-bcae-7c4748ed154e)

이렇게 해서 디코딩 하면, target language model이 small language model이 튜닝한 학습 경향을 배울 수 있다. 

# **Results**
![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/7562a708-9840-4648-99cb-b861fa819b3b)
전체적인 경향을 보았을 때, fine-tuning과 비교해서 88퍼 정도의 성능을 달성한다.

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/3cbcea70-b53c-4fd4-8818-3abc71f97c9c)
TruthfulQA에서는 full-fine-tuning대비 좋은 성능이 나온다.

![image](https://github.com/momozzing/KLUE-TOD/assets/60643542/c5a8ba38-4e3b-4307-a089-b864e0b5a095)

table6에서 보면 TruthfulQA에서는 Proxy-tuning을 하면, 최다 빈출 Token에 대해서 4-gram을 뽑게 되면, 확률적으로 추론에 대한 context가 많이 추출 된다. 

따라서 figure 2 를 봤을 때, 
$S_(M^+)$의 logit값 - $S_(M^-)$의 logit값에 대해 $Alpha$라는 하이퍼 파라미터를 달았을 때, (튜닝한 모델과 그 베이스라인의 차이에 대한 비율)

튜닝한 모델과 베이스 라인의 크기가 크면, 진실성이 높아지고 정보성이 떨어진다. 

그 이유는, 알파값이 크면, 추론에 대한 context가 추출 될 때, 이 정보에 대한 답변을 예 아니요 라고만 이야기 하고, 다른 추가적인 정보성이 있는 context에 대해서는 답변을 거부하는 경향이 있다고 한다. 

# **Conclusion**
본 논문은 Large Language model에서 모델 튜닝을
효율적으로 하는 방법을 새롭게 제시한 논문이다. 

여태까지 나온 논문들은 파인튜닝을 할때, 다른 파라미터를 프리징하고, 파라미터를 새롭게 추가해 학습해서 리소스에 부담을 줄이거나, 인퍼런스 시에, 프롬프트 튜닝을 해 성능을 올리는 방법론들이 대부분이었다면,

이 방법은 디코딩 시에, 로짓값의 차이를 변경해 Target language model이 튜닝 한것 처럼 사용하는 방법이다. 

---
새롭게 Large language model을 학습하는 방법이 나온것 같아 정말 신기하고 재밌게 읽었던것 같다.

이 논문의 장점: Full fine-tuning할 때, 사용하는 리소스를 줄일 수 있다.

이 논문의 단점: 로짓값을 계산하려면 70B, 7B, 7B 이렇게 3개의 모델을 인퍼런스 해야 해서 생각보다 많은 양의 리소스를 먹긴 한다. 하지만 full-fine-tuning대비해서 확실히 적은양의 GPU가 소모된다. 