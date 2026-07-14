---
title:  "SOLOIST paper review"
excerpt: "SOLOIST: Building Task Bots at Scale with Transfer Learning and Machine Teaching"

categories:
  - Paper review
tags:
  - Paper review
  - NLP
  - soloist
  - TOD
  - Dialog system
last_modified_at: 2021-05-10T08:06:00-05:00
mathjax: true
---

[논문 링크](https://arxiv.org/pdf/2005.05298.pdf)

## introduction
SOLOIST 는 Transfer learning을 사용한 Task-oriented dialog system    
SOLOIST 는 Transformer 기반의 Auto-regressive language model -> GPT2 사용    
데이터가 적은 Task-oriented dialog system 안에서 해결방법을 제시  

![image](https://user-images.githubusercontent.com/60643542/117806543-6fc53b00-b295-11eb-867f-e0adfdacf01a.png)

Natural Language Understanding (NLU)은 사용자 의도를 식별하고    
사용자 입력에서 슬롯 및 해당 값과 같은 관련 정보를 추출한다.    
Dialog State Tracking (DST)은 대화 기록에서 Belief State(대화 정보)를 추론한다.    
Belief State는 DB State를 얻기 위해 Data Base에 쿼리를 보내는데 사용    
그후 Dialog History, Belief State, DB State 정보로 Dialog Policy를 정하고    
Natural Language Generation(NLG)은 답변을 생성한다.   

## Architecture
![image](https://user-images.githubusercontent.com/60643542/117672087-7647aa00-b1e4-11eb-9044-1ddd783320a7.png)

![image](https://user-images.githubusercontent.com/60643542/117672142-85c6f300-b1e4-11eb-99fb-c0e206b2ba27.png)

![image](https://user-images.githubusercontent.com/60643542/117672179-90818800-b1e4-11eb-9eb4-609b303f162e.png)

여기서 DB는 Belief State를 쿼리로 해서 받아옴으로 1로 설정 그래서 날릴수 있다.

![image](https://user-images.githubusercontent.com/60643542/117672395-c6bf0780-b1e4-11eb-930e-c8573cd60aa6.png)

task 1,2 -> Auto Regressive model 이니 순차적으로 토큰별레벨에 의해 생성됨    

Task3 -> Cross Entropy 사용 시퀀스의 항목이 일치하는지 안하는지 예측    

Full Pre-Training Objective ->Log-likelihood를 최대화 하는 방향으로 학습시킴   

![image](https://user-images.githubusercontent.com/60643542/117806650-908d9080-b295-11eb-9bf5-e03bacea91d2.png)

## DATA
![image](https://user-images.githubusercontent.com/60643542/117672582-f4a44c00-b1e4-11eb-87e8-3a74098aba62.png)

## Experiments
![image](https://user-images.githubusercontent.com/60643542/117672632-fcfc8700-b1e4-11eb-9506-f2ae16fa7562.png)

![image](https://user-images.githubusercontent.com/60643542/117672665-02f26800-b1e5-11eb-9765-e3f92cffef54.png)

![image](https://user-images.githubusercontent.com/60643542/117672691-08e84900-b1e5-11eb-9604-8fda004863c4.png)

## Conclusion
![image](https://user-images.githubusercontent.com/60643542/117672722-0e459380-b1e5-11eb-84b1-881b5fc10ada.png)