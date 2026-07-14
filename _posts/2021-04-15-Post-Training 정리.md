---
title:  "Post-Training References 정리"
excerpt: "Post-Training"

categories:
  - Paper review
tags:
  - Study
  - NLP
  - Deep learning
  - summary
last_modified_at: 2021-04-15T08:06:00-05:00
mathjax: true
---  

## Post Training in Deep Learning with Last Kernel
![image](https://user-images.githubusercontent.com/60643542/114836883-112fad00-9e0e-11eb-92a3-bb7398c04769.png)

## Stanford Neural Machine Translation Systems for Spoken Language Domains
![image](https://user-images.githubusercontent.com/60643542/114996332-b1063d00-9ed9-11eb-86fc-4098e39c3e69.png)

## BERT Post-Training for Review Reading Comprehension and Aspect-based Sentiment Analysis
![image](https://user-images.githubusercontent.com/60643542/114836991-2dcbe500-9e0e-11eb-8887-56d83536cd41.png)

## An Effective Domain Adaptive Post-Training Method for BERT in Response Selection
![image](https://user-images.githubusercontent.com/60643542/114837022-37554d00-9e0e-11eb-8b8f-5f30b13b19f6.png)

![image](https://user-images.githubusercontent.com/60643542/114837051-3f14f180-9e0e-11eb-86ab-5b23bbae3954.png)

## Domain specialization: a post-training domain adaptation for Neural Machine Translation
![image](https://user-images.githubusercontent.com/60643542/114837086-4805c300-9e0e-11eb-8361-73c94ec3f584.png)

1) Post-train

Post-train의 개념은 스탠포드 대학에서 IWSLT 2015에 등재한 논문에서 처음 등장 하였다. 
이 논문에서는 한 도메인의 많은 양의 데이터에 대해 훈련된 모델은 다른 도메인의 적은 양의
데이터에 대해 fine-tuning할 수 있다고 설명 하였고 데이터에 적응(adapted)됬다고 표현 하였다.
그 후에 EACL 2017에 등재된 논문에서는 Generic한 데이터에 훈련된 모델에 도메인 내에 있는
데이터를 추가로 Re-training을 진행 하였는데 성능이 좋아졌다는 실험 결과가 있었다.
그리고 우리의 실험과 비슷한 NAACL 2019에 등재된 논문에서는 BERT를 사용해서 SQuAD 1.1 의 데이터 
post-train하고 Amazon laptop reviews data 와 Yelp Dataset Challenge reviews data 를 
fine-tuning하였는데 성능 향상을 보였던 실험 결과가 있었다. 그 후에 INTERSPEECH 2020에 등재된 논문에서는 
BERT를 사용해서 post-train을 검색 기반의 대화시스템에 적용을 시킨 논문이며 multi-turn의 응답선택에 중점을
두었다. domain-specific corpus (Ubuntu Corpus)사용하면 General corpus에 나타나지 않는 상황에 맞는
표현과 단어를 훈련시킬수 있다고 제안 하였고 Ubuntu Dialogue Corpus를 post-train하고 
Advising Corpus(DSTC7 task 1: Noetic end-to-end response selection)을 fine-tuning을 하였는데 
성능이 향상되었다는 실험 결과를 보여주었다.