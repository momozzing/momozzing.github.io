---
title:  "PPTOD paper review"
excerpt: "Multi-Task Pre-Training for Plug-and-Play Task-Oriented Dialogue System"

categories:
  - Paper review
tags:
  - Paper review
  - NLP
  - TOD
  - Dialog system
  - Task-Oriented Dialog System
  - Deep learning

last_modified_at: 2022-01-06T08:06:00-05:00
mathjax: true
# classes: wide
toc: true
toc_sticky: true
---

[이전 리뷰](https://momozzing.github.io/study/TOD,-DST/) -> TOD, DST 설명 

[이전 리뷰](https://momozzing.github.io/paper%20review/A-Simple-Language-Model-for-Task-Oriented-Dialogue-(SimpleTOD)/) -> simpleTOD paper review

[이전 리뷰](https://momozzing.github.io/paper%20review/SOLOIST_Building-Task-Bots-at-Scale-with-Transfer-Learning-and-Machine-Teaching/) -> Soloist paper review 

[이전 리뷰](https://momozzing.github.io/paper%20review/UBAR/) -> UBAR paper review 

[논문 링크](https://arxiv.org/pdf/2109.14739.pdf)

# **introduction**
본 논문은 T5 기반으로 TOD 데이터를 pre-training하고 TOD의 downstream task들을 multi-task learning을 할 수 있게 하는 Pre-train language model을 제안. 

- Downstream task들을 해결 할 수 있음으로 부분적으로 labeling된 데이터에서도 학습을 진행할 수 있다. 

- Downstream task들의 출력이 병렬로 생성됨으로 오류 누적 문제를 완화하고 inference time을 줄인다.  

![image](https://user-images.githubusercontent.com/60643542/148384777-eda183bf-4e2e-4a4b-8483-4c54ae67dc4b.png)

# **T5: Text-to-Text Transfer Transformer**

T5는 Transformer 기본 architecture를 사용해서 모든 언어 문제를 text-to-text task로 변환.

기존 Transformer와 다른 점은 Positional encoding말고 relative position encoding을 사용. 총 750G data로 학습한 11B 모델이다. 

## **relative position encoding**

기존 Transformer Positional encoding은 input으로 들어오는 token 위치에 따라 인코딩을 하고 후에 attention을 진행. 

![image](https://user-images.githubusercontent.com/60643542/148387499-b22c547a-08d5-4429-b083-f062ff79cb57.png)

T5의 Relative position encoding은 token을 넣고 self attention 수행 후 나오는 결과에 따라 relative position encoding값을 주는것. 

![image](https://user-images.githubusercontent.com/60643542/148396766-5932d01c-76f6-4804-a4b7-650e067083f6.png)

# **Method**
1. Pre-training Datasets

    In total, there are over 2.3M utterances across 80 domains

    NLU, DST, POL, NLG 각각의 task에 해당하는 dataset들 pre-training

![image](https://user-images.githubusercontent.com/60643542/148396943-f2b66d46-54ca-486c-ba9c-9432f8116b9e.png)

2. Dialogue Multi-Task Pre-training

![image](https://user-images.githubusercontent.com/60643542/148397371-5b8b6311-a2d4-4dd4-b6b9-51ed9734f5ad.png)

$d = (z_t, x, y)$

$t \in \{NLU, DST, POL, NLG\}$

$t$는 샘플 $d$가 속한 TOD 작업. 

$z_t$ 는 작업별 프롬프트 ex) transfer dialog to {t = system response} 

X는 현재 발화 이전의 input dialog context 

Y 는 out dialog context 


![image](https://user-images.githubusercontent.com/60643542/148397449-766cc1a1-449f-4654-a8d8-f4015928c1ae.png)

디코더의 Auto regressive 형태로 인해 

이전 토큰이 입력으로 들어가게 됨으로 현재 토큰과 이전 토큰이 같을 확률로 Loss를 최소화. 

# **Experiments**
1. End-to-end evaluation

![image](https://user-images.githubusercontent.com/60643542/148398066-f2ad7638-ef51-49bc-8f99-89cbff5e5a88.png)

전에 포스팅한 SimpleTOD, SOLOIST, UBAR보다 성능이 높다. 

2. Few-shot evaluation

![image](https://user-images.githubusercontent.com/60643542/148398186-a83b7661-f3d0-47c3-a2f1-f212716a3d7c.png)

적은 데이터에서도 강인하다는 발표를 한 SOLOIST보다 적은 데이터에서도 강인하다. 

3. DST evaluation


![image](https://user-images.githubusercontent.com/60643542/148398307-f7b606d6-59ca-4e96-be68-200c6f0d4b19.png)

Classification-based model 보다는 성능이 낮은데 Generation-based model 보다는 성능이 높다. 

4. Few-shot DST evaluation

![image](https://user-images.githubusercontent.com/60643542/148398436-0d593f8d-65de-4e04-81db-0c0ca26bc6c9.png)

Few-shot 성능도 전의 모델보다 성능이 높다.

5. Comparison between plug-and-play and cascaded generation

![image](https://user-images.githubusercontent.com/60643542/148398596-b67650be-4aad-42a1-a4a1-85170aaff5b6.png)

기존의 계단식 방법으로 학습 진행 시 본 논문에서 제안한 병렬식 방법으로 진행한 것 보다  성능이 낮다.   
병렬식 방법으로 하니 inference시에 latency도 낮아지며 속도도 빨라진다. 

T5 cascaded는 일부러 계단식으로 만든 다음에 실험한것. 

6. Performance of models pre-trained on data with different annotations

![image](https://user-images.githubusercontent.com/60643542/148398700-1c182df6-7faf-4577-872d-45687cbb9382.png)

pre-train할 때 데이터의 label이 부분적으로 있을 때의 실험이다. 

7. Human evaluation

![image](https://user-images.githubusercontent.com/60643542/148398766-afde2c94-63be-48f2-b258-a288a97383e0.png)

human evaluation에 강인하다는 SOLOIST보다 성능이 높다. 

이해: 시스템이 사용자의 목표를 올바르게 이해하는지 여부.   
진실성: 시스템의 응답이 참조에 의해 실제로 지원되는지 여부.   
일관성: 시스템의 응답이 문맥과 의미상 일관성이 있는지 여부.   
유창성: 시스템의 응답이 문법적으로 유창하고 이해하기 쉬운지 여부.   

# **Conclusion**
본 논문은 T5 기반으로 TOD 데이터를 pre-training하고 TOD의 downstream task들을 multi-task learning을 할 수 있게 하는 Pre-train language model

현재까지 나와있는 다른 모델들에 비해 성능이 향상되고 속도도 빨라졌다.  

적은데이터에 보다 강인하며 human evaluation에서도 강인하다. 

역시 데이터가 많으면 많을수록 좋다. 
