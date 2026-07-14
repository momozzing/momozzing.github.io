---
title:  "ISO standard for dialog act annotation"
excerpt: "ISO standard for dialog act annotation"

categories:
  - Study
tags:
  - Study
  - NLP
  - TOD
  - Dialog system
  - Task-Oriented Dialog System
  - Deep learning

last_modified_at: 2022-02-24T08:06:00-05:00
mathjax: true
# classes: wide
toc: true
toc_sticky: true
---

ISO standard for dialogue act annotation


![image](https://user-images.githubusercontent.com/60643542/155507026-8eb0a66e-c4a3-4797-a69e-a37731c368b3.png)


## **Social Convention**
사회적 행동에 대한 규칙적인 행동에 대한 Dialog act

1. hi 
- 첫인사를 하는 것.
- ex) hello, how are you

2. bye
- 끝인사를 하는 것.
- ex) goodbye

3. thank_you
- 감사를 표현하는 것. 
- ex) thank you 

4. repeat 
- 사용자가 지난 턴에 한 말을 다시 반복하도록 요청하는 것

5. welcome 
- 시스템이 제공할 수 있는 정보를 방송하는 공식 텍스트의 단락을 나타내는 것. 
- ex) welcome to Cambridge restaurant
- ex) we can help you to order food, you can find restaurants by talking about [food], [area], [price range]

6. dont_understand
- 시스템이 사용자가 말하는 것을 이해할 수 없다는 것
- ex)사용자가 시스템이 처리할 수 있는 의미 범위를 넘어서는 무언가에 대해 이야기할 때

## **Directive**
제안 또는 명령을 제공하는 Dialog act

1. propose
- 시스템이 사용자의 이익에 부합한다고 믿는 특정 행동의 수행을 사용자가 고려하도록 하기 위해 제안/추천하는 것
- ex) How about we find a good place to have fun

2. direct
- 명령을 나타내는 명령형 응답을 출력하는 것. 
- ex) you need to open the light before going to bed

## **Information Seeking**
요청에 대한 작업을 수행하는 Dialog act

1. request
- 특정한 value에 대해 사용자에게 묻는 것
- ex) what area do you like?

2. select
- 후보 중에서 선호하는 것을 선택하도록 요청하는 것

3. reqalts
- 사용자에게 추가 정보를 요청하는 것
- ex) what else information do you want. 

## **Information Providing**
사용자에게 특정 답변을 제공하는 Dialog act

1. affirm
- 긍정적인 답변을 요청하는 것.
- ex) Yes, it is

2. not_sure
- 시스템이 사용자의 확인에 대해 확실하지 않음을 의미

3. negate 
- 부정적인 답변을 요청하는 것. 
- ex) No, it is not

4. inform 
- 사용자가 요구하는 정보를 제공하는 것. 
- ex) The hotel is in the east area

5. offer
- 시스템이 사용자가 필요한 DB의 검색 결과를 제공하는 것
- ex) There are 10 restaurants I’ve found for you

6. notify_success
- 사용자의 목표가 성공적으로 완료되었음을 시스템이 사용자에게 알리는 것
- ex) Sure, the XXX is a good one
- ex) I’ve booked it for you

7. notify_failure
- 사용자의 목표가 성공적으로 완료되지 않았음을 시스템이 사용자에게 알리는 것
- ex) ‘Sorry, I can not book it for you now, because it is full

## **Information Checking**
시스템이 사용자에게 사실인지 여부를 확인하기 위해 사용자에게 질문하는 Dialog act

1. expl-confirm
- 사용자에게 명시적으로 무언가를 확인하도록 요청하는 것
- ex) Do you need to cheap restaurant?

2. impl-confirm 
- 사용자가 말한 것을 반복하는 문장에서 종종 암시적으로 무언가를 확인하는 것을 의미
- ex) You want a cheap restaurant, Okay 