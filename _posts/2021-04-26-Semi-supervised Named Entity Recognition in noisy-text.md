---
title:  "Semi-supervised Named Entity Recognition in noisy-text Paper review"
excerpt: "Semi-supervised Named Entity Recognition in noisy-text"

categories:
  - Paper review
tags:
  - Study
  - NLP
  - Deep learning
  - NER
last_modified_at: 2021-04-26T08:06:00-05:00
mathjax: true
---  

- [깃헙레포링크](https://github.com/napsternxg/TwitterNER)
- [논문링크](https://www.aclweb.org/anthology/W16-3927.pdf)

NER 할때 주로 쓰는 인코딩 방식은 Begin-Inside-Outside (BIO)방식이다. 

BIEOU = BILOU 두개 같은 표현

Begin-Inside-End-Outside-Unigram (BIEOU)

Begin-Inside-Last-Outside-Unigram (BILOU)

HMM 정리 [https://lovit.github.io/nlp/2018/09/11/hmm_based_tagger/](https://lovit.github.io/nlp/2018/09/11/hmm_based_tagger/)

CRF 정리 [https://lovit.github.io/nlp/2018/09/13/crf_based_tagger/](https://lovit.github.io/nlp/2018/09/13/crf_based_tagger/)   
        [https://ratsgo.github.io/machine learning/2017/11/10/CRF/](https://ratsgo.github.io/machine%20learning/2017/11/10/CRF/)

## introduction

기존의 NER방식은 적절한 구문으로 이루어진 뉴스 코퍼스 데이터를 기반으로 구축됨.    
하지만 이렇게 하면 트위터같이 노이즈가 심한 데이터에 대해서는 정확한 결과를 얻기 힘듦    
이 논문은 CRF를 기반으로 하고 BIEOU인코딩 체계를 사용하였고 각종 Feature Engineering을 제안 하였다.    
WNUT 2016 NER 출전 모델 github 에 공개되어 있다.   
https://github.com/napsternxg/TwitterNER.

ST는 얘네가 전에 냈던 WNUT 2016 NER 대회에 제출했던거

SI는 얘네가 제시했던 semi-supervised 한 NER모델 → 이에대해 개선사항을 제시

rf → random feature

## data 

WNUT 2016 Dataset

개체명을 가진 트위터 text데이터 

![image](https://user-images.githubusercontent.com/60643542/116098877-60b38a00-a6e6-11eb-94f6-356f85926a42.png)

![image](https://user-images.githubusercontent.com/60643542/116099032-86409380-a6e6-11eb-9fbf-6737ce48c0b7.png)

## 3. Feature Engineering

- Regex features [RF]

![image](https://user-images.githubusercontent.com/60643542/116099075-90fb2880-a6e6-11eb-8e3b-4ae8ffb122fd.png)

- Gazetteers [GZ]

![image](https://user-images.githubusercontent.com/60643542/116099127-9eb0ae00-a6e6-11eb-8326-58b3db68dbbf.png)

- Word representations [WR]

![image](https://user-images.githubusercontent.com/60643542/116099171-a8d2ac80-a6e6-11eb-889d-b82376131e2d.png)

- Word clusters [WC]

![image](https://user-images.githubusercontent.com/60643542/116099199-b25c1480-a6e6-11eb-9619-75a2d4366397.png)

- Additional features

![image](https://user-images.githubusercontent.com/60643542/116099278-c43db780-a6e6-11eb-8d0b-bad40653898f.png)

- Random up-sampling with feature dropout [RSFD]

![image](https://user-images.githubusercontent.com/60643542/116099317-cf90e300-a6e6-11eb-92b1-f500da065640.png)

## 4. NER classification algorithm

CRF 모델사용 모델은 L2 norm, SGD사용해서 훈련시킴 

[ST]

lexical tokens [LT], Regex features [RF], Random up-sampling with feature dropout [RSFD] 사용 해서 supervised learning함 


[SI]는 ST의 문제점을 개선한것 

ST는 한가지 단점이 있었는데 이는 overfitting 이다. 

이 overfitting을 막기 위해 semi-supervised를 사용했다. 

unlabeled data → train, dev, test병합해서 만든 레이블링되지 않은 데이터 

unlabeled data를 word embedding 후 클러스터링 해서 기존의 데이터에 추가 학습

→ 트윗에 있는 토큰이 증가 →레이블이 지정되지 않은 새로운 테스트 데이터를 사용하여 단어 표현과 클러스터를 개선

→ unlabeled data는 regularization factor 역할을 하여 훈련 데이터에 과적합되는 것을 방지한다. 

[TDTE]는 train, dev, test의 text를 병합

[TD] 테스트 데이터를 뺀 나머지 train dev text 병합

[SI]에 대한 개선사항 

→gazetteer [GZ] 기능을 추가하면 분류 정확도가 크게 향상

[WCBTP] → brown clusters 

[WRFTC] → fine-tuned word representations based clusters

[WCCC] → Clark clusters 

위의 4개 추가 하니까 F1score 향상 

## result

![image](https://user-images.githubusercontent.com/60643542/116099444-ee8f7500-a6e6-11eb-86e4-e0ebfb1f040d.png)

![image](https://user-images.githubusercontent.com/60643542/116099467-f818dd00-a6e6-11eb-92d3-1d2d459f5b5c.png)