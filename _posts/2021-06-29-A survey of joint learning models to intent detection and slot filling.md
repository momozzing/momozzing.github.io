---
title:  "Joint learning for intent, slot-filling References 정리"
excerpt: "Joint learning for intent, slot-filling"

categories:
  - Paper review
tags:
  - Paper review
  - NLP
  - TOD
  - Dialog system
  - Slot-filling
  - intent detection
last_modified_at: 2021-06-29T08:06:00-05:00
mathjax: true
---



## **intent detection**
intent detection은 사용자의 Utterance에 대한 intent를 Detection하는 과정
intent detection는 문장의 의도에 대해 의도를 분류하는 classification 문제
intent detection은 classification algorithm으로 해결 한다.

ex) 오늘 7시에 모모찡으로 3명 예약 부탁드립니다

Intent -> 예약  

## **slot filling**
slot filling 은 Utterance에서 task에 관련된 의미 구성 요소를 추출하는 작업이다.
slot filling은 NER을 통한 의미 구성 요소를 slot-value pair로 만드는 것.
Slot filling는 Conditional Random Field (CRF)를 주로 사용하였다. 

ex) 오늘 7시에 모모찡으로 3명 예약 부탁드립니다

Date : 오늘         time : 7시         Person Name : 모모찡       Person number : 3명

## **Statistical based**
-  **Triangular-Chain Conditional Random Fields, Jeong and Lee, IEEE 2008**  
joint learning for Intent detection and slot filling의 첫 모델    
토큰화기능, 슬롯 태깅, 의도 분류 세가지 계층이 있는 3개의 CRF layer을 쌓았다. 

-  **Strategies for Statistical Spoken Language Understanding with Small Amount of Data–an Empirical Study, Wang, Interspeech 2010, Microsoft Corporation**   
의도 분류에 Maximum entropy model(MEM)을 사용하고 슬롯 태깅에 위의 3개의 CRF layer을 사용

-  **A Joint Model for Discovery of Aspects in Utterances, Celikyilmaz and Hakkani-Tur, ACL 2012, Microsoft Mountain View** 
Multi-Layer Context Model 을 제안 여기서 Multi-layer는 HMM

-  **Convolutional neural network based triangular crf for joint intent detection and slot filling, IEEE 2013, Microsoft Corporation**  
  2013년에 최초의 신경망 CNN이 사용되었지만 위의 3개의 CRF 모델에 input으로 Feature extractio만함

## **Recursive neural networks**

- **Joint Semantic Utterance Classification and Slot filling with Recursive neural networks, Guo et al , IEEE 2014**

  joint learning을 다루기 위한 신경망 모델의 가장 초기 시도   
재귀 신경망 recursive neural networks (RecNN)을 사용 (RNN과는 아예 다름)  
RecNN은 트리(단어 벡터)에 해당하는 leaf 를 사용하여 발화의 구성 구문 분석 트리에서 작동한다.    
신경망은 트리의 각 노드에서 반복적으로 루트까지 적용되어 각 노드의 상태를 계산,   
각 노드에서 하위 노드의 상태는 노드의 구문 유형을 나타내는 가중치 벡터와 결합한다.   
개별 슬롯 레이블 분류기는 리프에서 루트까지의 경로를 따라 자신과 인접 단어 벡터 및 상태 벡터의 조합을 사용하여 각 리프에 적용된다. 
루트의 상태는 인텐트 분류기로 전달된다.   
슬롯과 인텐트 (및 도메인)에 대한 결합 된 loss은 backpropagation 됨.   
후처리(옵션)와 , Viterbi 디코딩 된 Markov 레이어가 슬롯에 적용된다.   

## **Recurrent neural networks**

- **A Hierarchical LSTM Model for Joint Tasks , Zhou et al, 2016**

  총 2계층의 NSTM layer을 사용 1층의 NSTM layer의 hidden state로 intent detection, 2층의 NSTM layer의 hidden state로 slot filling을 진행


- **Multi-Domain Joint Semantic Frame Parsing using Bi-directional RNN-LSTM, Hakkani-Tür et al, Interspeech 2016, Microsoft Redmond**

  intent detection에 사용하기 위해 발언을 캡슐화하기 위해 특별한 토큰을 추가 → EOS   
slot filling은 BiLSTM에 각각 연결된 hidden state를 사용하여 출력단에 soft max을 적용해 분류한다.










