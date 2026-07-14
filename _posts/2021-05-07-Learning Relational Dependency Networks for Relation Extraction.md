---
title:  "Learning Relational Dependency Networks for Relation Extraction paper review"
excerpt: "Learning Relational Dependency Networks for Relation Extraction"

categories:
  - Paper review
tags:
  - Paper review
  - NLP
  - Deep learning
  - Relation Extraction
last_modified_at: 2021-05-07T08:06:00-05:00
mathjax: true
---  


[논문 링크](https://arxiv.org/pdf/1607.00424.pdf)
## introduction
Knowledge base 를 구축하기 위해 관계추출을 위한 KBP slot filling을 사용   
관계 추출을 위한 패턴을 학습하기 위해 RDN을 사용하는 파이프라인을 제시   
weak supervision, word2vec, joint learning, human advice 를 사용   

![image](https://user-images.githubusercontent.com/60643542/117434558-c82fcc00-af67-11eb-924b-13cdf7baa20e.png)


## Knowledge Base Population (KBP) 란?

불완전한 지식 기반 (예 : Freebase 또는 Wikipedia 정보 상자의 구조화 된 정보)과 큰 텍스트 코퍼스 (예 : Wikipedia)를
가져와 지식 기반의 불완전한 요소를 완성하는 작업이다.    
즉, 컴퓨터는 텍스트를 "읽고"정보를 얻어야합니다. Stanford는 이 작업의 두 가지 측면에 중점을 두었음.

- Slot filling Slot이 주어지면 그에 맞는 Value값에 대량의 텍스트를 읽고 정보를 입력하는 것 
- Entity Linking Entity가 주어지면 관계성이 있는 Entity와 연결 하는 것.

## Pipeline

![image](https://user-images.githubusercontent.com/60643542/117434891-2e1c5380-af68-11eb-8d2d-30e0e8c8829d.png)

## Features
 
기존 텍스트는 Stanford CoreNLP Toolkit을 사용해서 Feature를 얻음 + 
기존 텍스트를 Word2vec을 사용해서 Feature를 얻음 = 변환된 Feature 는 시스템의 input으로 사용

![image](https://user-images.githubusercontent.com/60643542/117435063-62900f80-af68-11eb-8bdb-a69c8663aa2d.png)

## Labeling
Weak supervision 답과 연결된 것은 별로 없다.    
그래서 라벨링 실행 → 외부 데이터를 사용해서 매핑 or markov logic network MLN 을 사용해서 라벨링

![image](https://user-images.githubusercontent.com/60643542/117435158-7e93b100-af68-11eb-94a6-84da57d30bdf.png)
MLN이란?   
Markov chain은 한 상태의 확률은 단지 그 이전상태에만 의존한다는 것이 핵심.   
이를 이용해 각각의 관계성에 대한 확률논리로써 추론을 가능하게 한다.

![image](https://user-images.githubusercontent.com/60643542/117435283-a551e780-af68-11eb-90e1-156a7f5afaf8.png)

## Relational Dependency Networks (RDN)
RDN의 핵심 아이디어는 주변 분포, 즉 아래의 곱으로 랜덤 변수 집합에 대한 결합 분포를 근사하는 것.    
![image](https://user-images.githubusercontent.com/60643542/117435397-c9152d80-af68-11eb-870a-63dc74befc52.png)   
RDN에서 일반적으로 각 분포는 RPT(관계형 확률 트리)로 표시된다.    
이를 순차적인 방식으로 구축된 relational regression trees로 대체 (Blockeel and Raedt 1998)   
즉, 단일 트리를 gradient boosted trees 로 대체

![image](https://user-images.githubusercontent.com/60643542/117435450-d7634980-af68-11eb-93ea-c81ee051f966.png)

아래와 같은 Rules 생성
![image](https://user-images.githubusercontent.com/60643542/117435464-dd592a80-af68-11eb-9df9-9e70085d75b7.png)

## Human Advice
사람이 RDN을 통해 나온 결과에서 추가적으로 조건을 달아서 정답에 대한 확률을 도출

![image](https://user-images.githubusercontent.com/60643542/117435572-f6fa7200-af68-11eb-8997-40fdccde3d3c.png)

## Joint Learning
RDN안에서 관계가 도출이 되었을때 독립적으로 학습 VS 관계적으로 학습    
→ 관계적으로 학습시키니 성능이 향상되었다.

## Experiments - Weak Supervision
![image](https://user-images.githubusercontent.com/60643542/117435738-24dfb680-af69-11eb-85f3-33cc68716015.png)
## Experiments - Joint learning
![image](https://user-images.githubusercontent.com/60643542/117435769-2f9a4b80-af69-11eb-99f8-e5eb045dd5aa.png)
## Experiments - Word2vec
![image](https://user-images.githubusercontent.com/60643542/117435802-3759f000-af69-11eb-9b82-0a47ee75946f.png)
## Experiments - human advice
![image](https://user-images.githubusercontent.com/60643542/117435831-3fb22b00-af69-11eb-8de8-eb30c665d6a3.png)
## Experiments
![image](https://user-images.githubusercontent.com/60643542/117435886-4ccf1a00-af69-11eb-9e3a-961dd18bee72.png)

## Conclusion
- Knowledge base 를 구축하기 위해 관계 추출을 위한 KBP의 slot filling을 사용 
- 관계 추출을 위한 패턴을 학습하기 위해 RDN을 사용하는 파이프라인을 제시하였다. 
- weak supervision, word2vec, joint learning, human advice 을 각각 적용해 실험하였다. 
- Weak supervision, Word2vec은 큰 효과를 얻지 못함. joint learning도 절반 정도의 성능향상    
  그러나 human advice 에는 큰 효과를 보았다.
- 나중에는 더 깊은 네트워크를 쌓으면 성능이 오를 것 같다고 함.