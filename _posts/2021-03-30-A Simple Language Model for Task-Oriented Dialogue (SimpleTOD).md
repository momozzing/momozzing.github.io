---
title:  "SimpleTOD Paper review"
excerpt: "SimpleTOD: A Simple Language Model for Task-Oriented Dialogue"

categories:
  - Paper review
tags:
  - Paper review
  - NLP
  - TOD
  - Dialog system
  - Deep learning

last_modified_at: 2021-03-30T08:06:00-05:00
mathjax: true
---

- [논문 링크](https://arxiv.org/abs/2005.00796)
- TOD, DST의 Background를 보고 오시면 도움이 됩니다.
  [링크](https://momozzing.github.io/momo.github.io/TOD,-DST/)

## **introduction**

- SimpleTOD는 DST를 위한 생성 모델 - SOTA

- SimpleTOD는 dialogue state tracking, action decisions, response generation을 함께 수행하는
   최초의 End to End방식 모델 - SOTA

- SimpleTOD는 nosiy-labeled annotation 에서도 강인하다

- END토큰의 중요성을 보여준다. 

- MultiWOZ dataset를 사용

- Base 모델은 GPT-2이다.  -> hugging face 에서 불러와서 사용 



![image](https://user-images.githubusercontent.com/60643542/112971984-bb5ed200-918a-11eb-8d32-70ced7eb3496.png)

1. User의 발화가 들어감
2. 비어있는 slot의 value값을 User의 발화를 보고 생성
3. 완성된 Belief값을 DB Quary에 입력한다.
4. DB는 입력된 Quary를 가지고 Result값을 출력 후 입력으로 사용
5. Result값으로 Action값 생성
6. 이 Action값으로 System 발화 생성

## **Model Architecture**

![image](https://user-images.githubusercontent.com/60643542/112972870-b1899e80-918b-11eb-8d24-839db68ffe26.png)

1. User, system의 대화를 가지고 SimpleTOD(GPT-2 Architecture)안에 넣어서 Belief state생성
2. Belief state 값을 DB에 넣고 결과값 받은후 다시 입력으로 사용
3. 대화쌍, Belief값, DB값을 SimpleTOD에 넣고 Action값 생성
4. 대화쌍, Belief값, DB값, Action값을 SimpleTOD에 넣고 Response 생성

### **Training Objective**

![image](https://user-images.githubusercontent.com/60643542/112973471-53a98680-918c-11eb-9b49-cd17be67378e.png)

- GPT-2는 Autoregressive 한 모델 즉 본인이 생성한 값을 다시 입력받고 다음 값 생성
- (생성한 값)과 (데이터 입력된 값 -1step) 과 CE를 통한 LOSS 최소화하면서 학습

## **Experiment**

### **Data**

데이터셋은 MultiWOZ 2.1, MultiWOZ 2.0에 대해서 진행한다. 

이들은 10000개 이상의 multi-domain 발화로 구성 되어 있으며 35개의 domain-slot쌍과 5개의 도메인(기차, 레스토랑, 호텔, 택시, 명소)로 구성되어 있다. 공개적으로 사용 가능한 가장 큰 3개의 multi-domain TOD dataset이다. 

### **Result**


MultiWOZ2.0

![image](https://user-images.githubusercontent.com/60643542/131679702-92c2618e-c8d2-4524-94b3-fb169bf8c898.png)

MultiWOZ2.1

![image](https://user-images.githubusercontent.com/60643542/112974136-142f6a00-918d-11eb-8465-3e47bc7f20b1.png)

- (O) MultiWOZ 2.1 그대로 넣음 nosiy-labeled annotation 에도 강인한 걸 보여줌   
- (*) MultiWOZ 2.1 라벨 청소   
- (+) MultiWOZ 2.1 라벨 정규화 

![image](https://user-images.githubusercontent.com/60643542/112974373-58bb0580-918d-11eb-9b87-92d4ec24edb8.png)

- 여기서 User/System은 대화임 
  - 대화가 없으니 점수가 현저히 낮은걸 볼 수 있음
- 각각의 Context, Belief State, DB Search, Action, Response를 BOS, EOS를 각각 설정해줌
  - BOS, EOS를 잘 설정해 주니 점수가 높은걸 볼 수 있음
  
이 두개를 합쳐서 사용하니 매우 높은 점수가 나왔다고 한다. 

## Conclusion
- SimpleTOD는 dialogue state tracking, action decisions, response generation을    
  함께 수행하는 최초의 End to End방식 모델

- DST분야에서는 SOTA를 찍은 모델이다. 

- 다른 sub-task를 분류하는 end토큰을 사용하니 성능이 높아졌다. 

- nosiy-labeled annotation에서도 강인했다
