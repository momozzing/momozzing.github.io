---
title:  "TripPy + SaCLog Paper review"
excerpt: "Preview, Attend and Review: Schema-Aware Curriculum Learning for
Multi-Domain Dialog State Tracking"

categories:
  - Paper review
tags:
  - Paper review
  - NLP
  - TOD
  - Dialog system
  - DST
  - Dialog State Tracking
last_modified_at: 2021-08-26T08:06:00-05:00
mathjax: true
---
[논문 링크](https://arxiv.org/pdf/2106.00291.pdf)

[이전 리뷰](https://momozzing.github.io/study/TOD,-DST/) -> TOD, DST 설명 

[이전 리뷰](https://momozzing.github.io/paper%20review/TripPy/) -> TripPy paper review
## **introduction**

현재 기준 Multi-domain DST 2위 모델 

기존의 DST모델은 데이터셋의 구조 정보를 무시하고 Random하게 데이터를 뽑아서 학습시킨다. 
본 논문은 Task-Oriented dialog system을 위해 Curriculum구조와 Schema구조를 활용 하기 위해 **S**chema-**a**ware **C**urriculum **L**earning for Dial**og** State Tracking (SaCLog) 방식을 제안

이 방식은 Transformer, RNN기반의 모델에 적용시켜 DST의 성능을 개선한다. 

### **Curriculum 구조를 활용한다** 

사람도 학습할때 쉬운거 어려운거 랜덤하게 순서대로 학습시키면 학습을 잘하지 못한다. 그러므로 모델도 난이도 순서대로 학습시키면 더 잘 학습하게 된다. 그리고 Curriculum Learning을 위해 모듈을 최적화 시킨다.  Curriculum Learning은 아직 DST분야에서 거의 연구가 되지 않았다고 한다.  -> **Curriculum module**

[Curriculum survey 논문](https://arxiv.org/abs/2010.13166) -> 나중에 한번 읽어보자 

### **Schema 구조를 활용한다** 

Schema란 dialog 데이터셋 안에서 가능한 모든 slot-values 값을 정의해둔것으로 slot 간의 의미관계를 나타낸다. 

schema와 dialog context 간의 관계를 학습시키기 위해 DST의 모델의 encoder 부분(BERT, RNN)에  pre-train한다. -> **Preview module** 

그리고 Curriculum learning의 학습을 강화시키기 위해서 모델의 결과가 slot을 잘못 예측 했을 때 그 데이터를 augment 해서 더 많은 데이터를 학습시키고 유사한 사례에서 더 좋은 성능을 발휘하도록 한다. -> **Review module**

정리하면 SaCLog 방식은 schema 정보로 DST 모델을 pre-train하는 **Preview module**, train data의 난이도를 easy to hard로 정리하고 Curriculum learning을 위한 **Curriculum module**, Curriculum learning의 학습을 강화시키기 위해 예측이 잘못된 데이터를 늘리는 **Review module**, 총 3개의 module 구조를 가지고 있다. 

## **Model Architecture**
![image](https://user-images.githubusercontent.com/60643542/131001488-43ca3b35-9d5d-4752-ac5c-7b1e33f39b60.png)


### **define**

$X_t = \{(R_1,U_1),...,(R_t, U_t)\}$

$R_i$ = system utterance, $U_i$ = user utterance

$X_t$에서 주어진 slot-value pair로 turn-level, dialog-level로 DST를 실행

turn-level: $Y_t = \{(s,v_t), s \in S \}$

$S$: $X_t$에서 추출한 set of slot-value pairs

$s, v$:  S안에 있는 slot, value 각각의 값

dialog-level: $Z_t$ = 대화과정에서 표현된 모든 slot-value pair

DST에 대한 대화 데이터: $d_t = \{X_t, Y_t, Z_t\}$, 훈련 데이터: $D$

### **Curriculum Module**

train data의 난이도를 easy to hard로 정리하고 Curriculum learning을 위해 모듈을 최적화 

difficulty scorer와 training scheduler 두개의 sub-module로 나뉨

### **Difficulty Scorer**
대화의 난이도를 계산할때 model-based scoring 방식과 rule-based scoring 방식이 있는데 이 두가지를 합한 hybrid scoring 방식을 사용할것이다. 

**Model-based scoring**

$r_t^{mod}$ = 각 $d_t$에 대해 $Y_t$가 언급된 모든 슬롯의 평균 정확도를 기반으로 계산된다. 

이 score를 $r_t^{mod} \in [0, 1]$ 로 normalize

동일한 architecture와 다른 initialization을 가진 6개의 모델을 훈련 후 평균 정확도 $\bar{r}_t^{mod}$를 얻음.



**Rule-based scoring**

4개의 규칙을 가지고 있다.

1. 현재 dialog turn 번호 → max = 7
2. $(R_t,U_t)$의 총 토큰수 → max = 50
3. $Z_t$의 호텔이름 같은 언급된 entity의 수 → max = 4 
4. $Y_t$에 새로 추가되거나 변경된 슬롯의 수 → max = 6

이 score를 $r_t^{rul,i} \in [0, 1]$ 로 normalize

**Hybrid scoring**

$r_t^{hyb} = \alpha_0 \bar{r_t}^{mod} + \sum_{i=1}^4 \alpha_i r_t^{rul,i}$

여기서 $r_t^{hyb} \in [0,1]$ , $\sum_{i=0}^4 \alpha_i = 1$

### **Training Scheduler**

[링크](https://aclanthology.org/N10-1116.pdf) 이 paper를 base로 함.

score를 N개의 간격으로 쪼개고 정리된걸 N개의 bucket에 분배, 그 후 가장 쉬운 난이도의 bucket부터 training한다. 

### **Preview Module**

schema와 dialog context 간의 관계를 학습시키기 위해 DST의 모델의 encoder 부분(BERT, RNN)에  pre-train한다.

schema 구조를 학습하기 위한 새로운 방법을 제안 

$e_s$ = slot embedding for each input slot $s$

$X_t$를 input으로 넣은 encoder단의 hidden state: $E_t = [e_t^1,e_t^2,...]$

$$B_t^s = [\phi_1^{sig}(e_s \oplus e_t^1), \phi_1^{sig}(e_s \oplus e_t^2),... ]$$

$$c_t^s = \phi_4^{sft}(e_s \oplus Att(e_s,E_t))$$

$B_t^s$ = $X_t$에 속하는지 안하는지 binary sequence값

$c_t^s$ = $y_t$의 4개값 $\mathsf{added,\ deleted,\ changed,\ not\ mentioned}$ classification값

$\mathcal L_{seq}$  = Binary sequence Loss

$\mathcal L_{cls}$  = Classification Loss

$\mathcal L_{aux}$ 는 두가지가 있다. 

- 사용 모델이 Transformer-based 일 경우 BERT의 MLM Loss를 사용한다. 
- 사용 모델이 RNN-based 일 경우 순방향, 역방향의 LM loss의 합을 사용한다. 

$\mathcal L_{seq}$ , $\mathcal L_{cls}$ 는 원본데이터 + 증강데이터 사용

$\mathcal L_{aux}$ 는 원본데이터에만 사용한다. 

이 loss값을 합쳐서 loss가 최소가 되게 학습을 진행

### **Review Module**
3가지 기술을 사용해서 data augmentation을 진행

- Slot Substitution
  
  $(R_t, U_t)$에 언급된 slot name의 value가 $dontcare$일 때 다른 slot name으로 변경된다. 

  먼저 각 slot name에 대한 word set을 수집한다. ex) {'arrive', 'arriving', 'arrived'}

  그 후 현재 턴에 언급되지 않은 것으로 slot name을 대체한다. 

- Value Replacement

  $U_T$ 안에 value 값이 정확히 있을 때 기존 slot의 value값이 $U_T$ 안의 value 값로 대체

  value값 detection은 기존 Schema 를 활용해 각 slot의 value를 생성하고 [논문 링크](https://aclanthology.org/2020.sigdial-1.4.pdf)의 레이블 맵을 사용해 value의 범위를 얻고 동일한 slot의 다른 value값으로 대체를 한다. 

- Dialog Recombination

  대화 데이터를 결합해서 새로운 데이터 생성을 위해 $Y_t$에서 언급된 동일한 slot을 가진 다른 데이터를 무작위로 검색 후 $X_{t-1}$만큼 자르고 현재 턴의 $(R_t, U_t)$를 잘라 연결하고 $Y_t$를 교환해서 두개의 새로운 대화 데이터를 생성한다. 

## **Experiment**

실험은 Transformer-based인 TripPy 모델과 RNN-based인 TRADE 모델에 SaCLog를 적용하여 실험하였다.  
### **Data**
데이터셋은 MultiWOZ 2.1, WOZ 2.0에 대해서 진행한다. 

MultiWOZ2.1은 10000개 이상의 multi-domain 발화로 구성 되어 있으며 30개의 domain-slot쌍과 5개의 도메인(기차, 레스토랑, 호텔, 택시, 명소)로 구성되어 있다. 
WOZ 2.0은 단일 도메인 데이터셋들이며 MultiWOZ2.1 보다 훨씬 작은데이터다.

### **Evaluation**
joint goal accuracy (JGA)로 평가 진행

### **Result**
TripPy + SaCLog

![image](https://user-images.githubusercontent.com/60643542/131026526-a96b7367-f759-47a7-b9c8-8a63cb34d0bc.png)

TripPy + 각각의 실험 진행 

![image](https://user-images.githubusercontent.com/60643542/131026893-683a5b70-c2d3-48a0-9ed3-98fe9fb23776.png)

TRADE + SaCLog

![image](https://user-images.githubusercontent.com/60643542/131026974-e287a08d-328b-44c1-ac91-329a0d02c957.png)



## **Conclusion**

현재 기준 Multi-domain DST 2위 모델 

기존의 DST모델은 데이터셋의 구조 정보를 무시하고 Random하게 데이터를 뽑아서 학습시킨다. 

본 논문은 Task-Oriented dialog system을 위해 Curriculum구조와 Schema구조를 활용 하기 위해 **S**chema-**a**ware **C**urriculum **L**earning for Dial**og** State Tracking (SaCLog) 방식을 제안

SaCLog 방식은 schema 정보로 DST 모델을 pre-train하는 **Preview module**, train data의 난이도를 easy to hard로 정리하고 Curriculum learning을 위한 **Curriculum module**, Curriculum learning의 학습을 강화시키기 위해 예측이 잘못된 데이터를 늘리는 **Review module**, 총 3개의 module 구조를 가지고 있다. 

이 방식은 Transformer-based, RNN-based 모델에 적용시켜 DST의 성능을 개선했다. 

Transformer-based model 은 TripPy를 사용했고 RNN-based model 은 TRADE를 사용했다. 

SaCLog방식을 두 모델에 적용했을시 기존의 두 모델의 성능보다 높은 성능을 내었다. 