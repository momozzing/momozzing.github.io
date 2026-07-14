---
title:  "Language-Independent Discriminative Parsing of Temporal Expressions paper review"
excerpt: "Language-Independent Discriminative Parsing of Temporal Expressions"

categories:
  - Paper review
tags:
  - Paper review
  - NLP
  - Deep learning
  - Semantic Parsing
last_modified_at: 2021-05-01T08:06:00-05:00
mathjax: true
---  

- [논문링크](https://aclanthology.org/P13-1009.pdf)

## Introduction 

이 논문은 시간표현에 대한 구문분석 시스템을 제안   
시간표현에 대한 구문분석을 위해 independent semantic parser를 제안한다.    
모델의 파라미터는 부트스트랩 방식으로 학습된다. 영어, 스페인어, 이탈리아어, 프랑스어, 중국어, 한국어 6개의 언어를 test 


시간을 표현하기 위한 방법은 복잡한 시간, 날짜 또는 기간을 설명하는    
텍스트 구문에서 정규화된 시간 표현으로 매핑하는 작업 

시간 표현 방법 정의 크게 Type, Function, Nil, Number로 정의

![image](https://user-images.githubusercontent.com/60643542/116773502-8814a900-aa90-11eb-8dc5-c1f8b1bbd1d2.png)

![image](https://user-images.githubusercontent.com/60643542/116773512-8fd44d80-aa90-11eb-865d-fe59ca7e2480.png)

Nil → 시간을 표현하는데 의미가 담겨있지 않은 전치사 같은 단어들    
Number → 숫자들은 숫자 태그를 달아줌    

## model
EM-style bootstrapping    
optimizer - Adagrad 
기능 2가지 있다.      
### 1번째 
괄호로 묶으면 그안에 포함되있는 정보를 뽑아서 보여준다.   

![image](https://user-images.githubusercontent.com/60643542/116773530-b09ca300-aa90-11eb-9136-84a2487685f8.png)

### 2번째
두개의 범위 안에 있는 결과값이 출력된다.  this ,  week  결합하면 이번주를 출력해줌

![image](https://user-images.githubusercontent.com/60643542/116773556-d9249d00-aa90-11eb-89c2-cc56571170fb.png)

## dataset 
→ TempEval-2 Datasets   

이 데이터 셋 안에서 시간정보를 사용한다.   
이 데이터셋 안에는 영어, 스페인어, 이탈리아어, 프랑스어, 중국어, 한국어 총 6개의 언어가 라벨링 되어있다. 
이중 영어와 스페인어가 가장 많다. 

![image](https://user-images.githubusercontent.com/60643542/116773568-f194b780-aa90-11eb-937b-34784f33f0eb.png)

## result
![image](https://user-images.githubusercontent.com/60643542/116773578-0b35ff00-aa91-11eb-852d-6910c19bd693.png)

Pragmatics - 모호성   
type error - 위에서 정의한 범위, 시퀀스, 기간 타입 오류    
incorrect number 파싱에서 숫자가 생략되거나 숫자를 잘못파싱해서 오류난것    
relative range 결과값이 비슷하게 나오는데 잘못나오는것    
incorrect parse 잘못 파싱한것    
missing context 문맥을 잃는것    
bad reference time 시간을 잘못 알려주는것

## conclusion

이 논문에서는 시간표현에 대한 구문분석 시스템을 제안    
시간표현에 대한 구문분석을 위해 independent semantic parser를 제안한다.    
영어, 스페인어, 이탈리아어, 프랑스어, 중국어, 한국어 6개의 다국어 접근방법을 제시   
