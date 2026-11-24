---
date: 2026-11-24 06:00:00 +0900
title: "LVRL Paper review"
excerpt: "에이전트 강화학습에서 critic을 따로 학습시키지 않고, 얼려 둔 LLM에게 끝난 궤적을 보여 주고 스텝마다 성공 확률을 매기게 한다. Claude Opus 4.6을 critic으로 쓰면 Qwen3-8B가 AppWorld 26.3%, SWE-bench Verified 20.2%로 GRPO의 19.8%, 12.0%를 넘는데, critic을 Qwen3-235B로 바꾸면 이득이 거의 사라진다."
categories:
  - Paper review
tags:
  - Large Language Model
  - NLP
  - Agent
  - Planning
  - Tool Use
  - Paper review
toc: true
toc_sticky: true
mathjax: true
field: agent
---

LLM Value Functions for Credit Assignment in Agentic RL

[https://arxiv.org/abs/2610.15840](https://arxiv.org/abs/2610.15840)

LVRL(LLM Value Function RL)은 Apple에서 만든 에이전트 강화학습 방법이다.
저자는 6명이고, arXiv 에 올라온 논문이다.
에피소드 끝에 성공/실패 한 비트만 주어지는 환경에서, 그 보상을 스텝별로 나눠 주는 일을 학습된 critic 대신 얼려 둔(frozen) LLM에게 맡긴다.
같은 신호를 증류에 쓰는 LVOPD도 함께 제안한다.
앞에서 본 [Multi-Harness RL](https://momozzing.github.io/paper%20review/Multi-Harness-RL-Paper-review/)에서는 보상 정규화를 어떤 묶음으로 하느냐를 비교했는데, 이 논문은 그 보상을 궤적 안에서 어떻게 쪼갤지를 다룬다.

![그룹 롤아웃, LLM 가치 추정, 스텝 advantage 계산, 정책 최적화로 이어지는 전체 흐름 (논문 Figure 1)](https://momozzing.github.io/assets/images/llm-value-functions/fig1-overview-proposed-framework.png)

## **1. Introduction**

에이전트가 수십 스텝을 움직인 뒤 받는 보상은 보통 성공이면 1, 실패면 0 하나다.
이 한 비트를 어느 행동의 공으로 돌릴지가 temporal credit assignment(시간에 걸친 공로 배분) 문제다.

지금 많이 쓰는 두 방법은 각각 약점이 있다.
GRPO\*는 궤적 하나의 모든 행동에 같은 advantage를 준다.
그래서 성공한 궤적 안의 실수는 양의 신호를 받고, 실패한 궤적 안의 쓸모 있는 행동은 음의 신호를 받는다.
PPO\*는 critic 학습에 계산이 더 들고, 긴 이력과 드문 보상에서는 critic이 잘 안 맞는다.

LVRL은 끝난 궤적을 강한 LLM에게 통째로 보여 주고 각 중간 상태의 값을 한 번에 매기게 한다.
끝까지 본 뒤에 매기므로, 무시된 도구 오류나 뒤늦은 복구 같은 것을 앞 스텝의 평가에 반영할 수 있다.
가치는 정책에 따라 달라지는 값이라서, 같은 과제의 다른 롤아웃 요약과 그룹 성공률을 프롬프트에 넣어 현재 정책 수준에 맞춘다(파라미터 갱신은 없다).

GRPO: 별도의 가치 모델(critic) 없이, 같은 과제를 여러 번 돌린 궤적 그룹 안에서 보상을 서로 비교해 advantage를 정하는 강화학습 알고리즘. 한 궤적의 모든 행동이 같은 advantage를 받는다

PPO: 정책이 한 번에 너무 크게 바뀌지 않도록 갱신 폭을 잘라(clip) 안정적으로 학습하는 강화학습 알고리즘. 보통 가치 모델(critic)을 함께 학습해 스텝별 advantage를 계산한다

## **2. Related Work**

스텝 단위 credit assignment 쪽에서는 학습된 critic(ArCHer\*, GenAC\*), 중간 상태에서 롤아웃을 더 뽑는 방식(VinePPO\*), 공유 상태의 보상을 모으는 방식(GiGPO\*, SALT\*), process reward model\*을 든다.
저자들은 이들이 사람 주석, 추가 롤아웃 예산, 대리 지표 중 하나에 기댄다고 정리한다.
가장 가까운 선행 연구로는 GVL\*을 꼽는다.
GVL은 얼린 vision-language 모델로 로봇 영상의 진행도를 매기는데, 정책과 무관한 진행 점수를 낸다는 점이 다르다고 한다.
on-policy distillation\*(OPD) 쪽에서는 SAGE-OPD\*, FutureBridge-OPD\* 같은 선택적 증류 방법을 언급한다.

ArCHer: 가치 함수를 학습하는 actor-critic 방식을 다중 턴 언어 에이전트로 넓힌 방법. 턴 단위로 critic을 학습하고 그 값으로 토큰 단위 정책을 갱신하는 계층 구조다

GenAC: ArCHer와 함께 원문이 "가치 함수를 학습하는 actor-critic을 다중 턴 언어 에이전트로 넓힌 방법"으로 묶어 소개하는 방법(자세한 구조는 원문에서 확인 못 함)

VinePPO: 궤적의 중간 상태에서 이어지는 롤아웃을 추가로 돌려, 그 성공 비율로 상태 가치를 추정해 PPO의 critic을 대신하는 방법. 추가 롤아웃 예산이 든다

GiGPO: 여러 궤적에서 같은 상태가 다시 나오면 그 상태들을 한 그룹으로 묶고, 거기서 이어진 보상을 비교해 스텝 단위 advantage를 얻는 방법. 롤아웃을 더 돌리지 않고 이미 뽑은 궤적들을 재활용한다

SALT: GiGPO처럼 여러 궤적의 공유 상태에서 보상을 모아 스텝 단위 신호를 얻는 방법(원문 소개 기준, 자세한 구조는 원문에서 확인 못 함)

process reward model(PRM): 최종 답만이 아니라 중간 단계마다 점수를 매기도록, 중간 단계에 대한 정답 표시로 따로 학습한 보상 모델

GVL: 얼려 둔 vision-language 모델에게 로봇 영상을 보여 주고, in-context learning으로 과제가 얼마나 진행됐는지 점수를 매기게 하는 방법. 점수는 어떤 정책이 움직였는지와 무관하다

on-policy distillation(OPD): 학생 모델이 직접 생성한 문맥(prefix) 위에서 교사 모델의 토큰 분포를 따라가도록 학습하는 증류. 학생이 실제로 마주칠 상황에서 배우게 해 학습·추론 때의 분포 차이를 줄인다. 교사의 토큰 확률을 볼 수 있어야 한다

SAGE-OPD: 다중 턴 OPD에서 학생이 고른 행동이 유효한 대안이면 그 스텝은 교사 감독을 건너뛰게 하는 방법

FutureBridge-OPD: 교사가 끼어들 후보 지점마다 학생의 이어 쓰기를 돌려 보고, 교사 기준 선호도로 점수를 매겨 감독할지를 정하는 방법

## **3. Method**

### **3.1 Preliminaries**

과제는 유한 길이 MDP로 본다.
상태 $s_t$ 는 목표와 지금까지의 관측·행동 이력이고, 행동 $a_t$ 는 토큰 열이다.
보상은 끝에 한 번 0 또는 1로 주어지고, 할인율은 1이라서 가치 $V^\pi(s_t)$ 는 그 상태에서 과제를 끝내 성공할 확률이 된다.
여기서 $t$ 는 환경 스텝 번호다(토큰 번호와 다르다).

GRPO는 같은 과제에서 뽑은 $G$ 개 궤적의 보상 평균 $\mu_G$ 와 표준편차 $\sigma_G$ 를 구하고, 궤적의 보상에서 평균을 빼 표준편차로 나눈 값을 advantage로 쓴다.
이 값 하나가 그 궤적의 모든 행동에 똑같이 붙는다.

### **3.2 LLM Value Function RL**

절차는 Figure 1의 네 단계 그대로다.

**그룹 롤아웃과 가치 추정**

과제 하나에 대해 현재 정책으로 $G$ 개 궤적을 뽑고 평균 보상 $\mu_G$ 를 구한다.
그다음 대상 궤적 $\tau_i$ 하나마다 그룹 안의 다른 궤적 $k$ 개를 무작위로 골라 문맥으로 넣는다.
문맥 궤적은 스텝당 한 줄로 요약해서 넣고, 대상 궤적은 요약하지 않고 그대로 넣는다.
질의 한 번으로 그 궤적의 모든 중간 상태 값을 받는다.
부록의 프롬프트를 보면 "이 지점부터 과제를 성공할 확률을 0.0~1.0으로 추정하라"고 묻고, 스텝마다 값과 짧은 이유를 JSON으로 받는다.
프롬프트에는 대상 궤적의 최종 결과(성공/실패)도 들어간다.

**양 끝 고정**

첫 상태의 값은 그룹 성공률 $\mu_G$ 로, 마지막 상태의 값은 실제 받은 보상 $R(\tau_i)$ 로 덮어쓴다.

**스텝 advantage**

연속한 두 상태의 값 차이를 그 스텝의 advantage로 쓴다.

$$\bar{A}_{i,t} = V_\phi(s_{i,t+1}) - V_\phi(s_{i,t})$$

그룹 전체 스텝에서 구한 표준편차 $\sigma_{step}$ 으로 나눠 정규화한다.
이 값을 PPO의 clipped surrogate에 학습된 advantage 대신 넣는다.
한 행동 안의 토큰은 모두 같은 advantage를 받고, likelihood ratio와 clipping은 토큰마다 계산한다.

양 끝을 고정했기 때문에 차이를 다 더하면 중간 항이 지워진다.

$$\sum_{t=0}^{T_i-1} \bar{A}_{i,t} = R(\tau_i) - \mu_G$$

오른쪽은 GRPO advantage의 분자와 같다.
즉 critic이 얼마나 틀리든 궤적 하나가 받는 신호의 총합(정규화 전)은 환경이 준 결과로 정해진다.
process reward model은 학습된 보상을 목적함수에 더하므로 정책이 추정 오차를 파고들어 실패한 궤적에서도 양의 총보상을 쌓을 수 있는데, LVRL에서는 그럴 수 없다는 게 저자들의 설명이다.
원문 표현은 이렇다.

*"the frozen critic cannot inject new reward; it only redistributes it."*

저자들은 $V_\phi$ 가 끝난 궤적과 그 결과를 보고 매긴 hindsight 추정치이지 진짜 상태 가치 함수는 아니라고 적어 둔다.

-> 총합이 고정될 뿐, 궤적 안에서 어느 스텝이 양이고 음인지는 전부 critic 판단에 달려 있다. 그룹 전원이 실패하면 양 끝이 모두 0이라 총합도 0이다. 이때 스텝별 신호는 환경 보상과 무관하게 critic의 판단만으로 만들어진다. BabyAI 결과가 이 경우에 해당한다.

### **3.3 LLM Value Function OPD**

토큰 분포를 볼 수 있는 더 강한 교사 모델이 있을 때, 같은 advantage를 어디에 교사 감독을 걸지 정하는 데 쓴다.

일반 OPD는 학생이 만든 모든 토큰에서 교사와의 KL을 줄인다.
그런데 KL이 커도 저장소를 다른 순서로 살펴보는 것처럼, 교사와 다르지만 유효한 방식일 수 있다.

LVOPD는 advantage가 음수인 스텝에서만 증류를 켠다.
게이트는 다음과 같다.

$$g(A_{i,t}) = \max(-A_{i,t},\ 0)$$

값이 떨어진 스텝일수록 증류 가중치가 커지고, 값이 그대로거나 오른 스텝은 증류 손실이 0이다.
게이트는 스텝 단위로 정해져 그 행동의 모든 토큰이 공유하고, reverse KL은 토큰 위치마다 계산한다.
전체 손실은 LVRL 손실에 게이트가 걸린 증류 손실을 $\beta$ 배 해서 더한 것이다.
KL은 계산량 때문에 top-k 근사를 쓴다(교사 top-32, 학생 top-16).

## **4. Experiments**

### **4.1 Setup**

환경은 세 가지다.
- AppWorld: 9개 앱 API를 파이썬 코드로 호출해 일상 과제를 푼다. 학습 90개 과제, 평가는 Test Challenge split(417개)
- BabyAI: 텍스트로 관측을 받는 격자 세계. 2×3으로 배치한 여섯 방에서 문을 열고 목표 물건을 집는 Pickup 변형. 학습 285개 배치
- R2E-Gym: 저장소 단위 소프트웨어 과제 3,200개로 학습하고 SWE-bench Verified(500개)로 평가

BabyAI 평가는 학습과 같은 문 거리(1~3)의 ID 300개와, 학습에 없던 문 거리(4, 5)의 OOD 200개로 나뉜다.
문 거리는 시작 위치에서 목표까지 최단 경로에 있는 문의 수다.

정책 모델은 Qwen3-4B, 8B(AppWorld, BabyAI)와 Qwen3-8B, 32B(R2E-Gym)다.
전부 non-thinking 모드로 돌리고 모든 파라미터를 파인튜닝하며, 그룹 크기는 8이다.
시드는 AppWorld와 BabyAI가 3개, R2E-Gym은 비용 때문에 1개다.

LVRL의 critic은 Claude Opus 4.6이고 문맥 롤아웃은 3개를 넣는다.
비교 대상은 GRPO와 PPO이고, PPO의 critic은 학생 LM의 hidden state에 붙인 4층 MLP다.

LVOPD는 AppWorld 위주로 본다.
교사와 critic 둘 다 Qwen3-235B(정식 이름 Qwen3-235B-A22B-Thinking-2507-FP8)가 맡고, $\beta = 1$ 이다.
비교 대상은 모든 토큰에 감독을 거는 Pure OPD와, GRPO advantage로 궤적 전체에 가중치 하나를 거는 Trajectory-gated OPD다.

### **4.2 Does LVRL Improve Long-Horizon Agent Learning?**

주 결과는 논문 Table 1이다.
세 환경에서 베이스 모델과 GRPO, PPO, LVRL로 학습한 모델의 평가 성공률(%)이고, BabyAI는 ID / OOD로 나눠 적혀 있다.

![AppWorld, BabyAI, SWE-bench Verified에서 Base, GRPO, PPO, LVRL의 평가 성공률 (논문 Table 1)](https://momozzing.github.io/assets/images/llm-value-functions/table1-lvrl-evaluation-performance.png)

AppWorld와 SWE-bench Verified의 네 설정 모두에서 LVRL이 가장 높다.
BabyAI에서는 GRPO와 PPO가 3% 아래에 머무는 동안 LVRL은 ID 69.0~75.0, OOD 52.2~65.5다.

abstract의 "GRPO 대비 최대 13.6점, 8.6점"은 각각 AppWorld Qwen3-4B와 SWE-bench Verified Qwen3-32B에서 나온 차이다.

BabyAI에서 차이가 가장 크다.
베이스 모델 성공률이 0%에 가까워서 초기 롤아웃 보상이 거의 다 0이다.
보상에 분산이 없으면 GRPO advantage는 0이 되고, PPO critic은 어디서나 0을 예측하게 된다.
LVRL은 전원 실패한 그룹에서도 중간 값이 오르내리므로, 문을 열어 목표가 보이게 된 스텝 같은 데에 양의 신호를 줄 수 있다.

아래는 세 환경의 학습 보상 곡선이다.

![AppWorld, BabyAI, R2E-Gym 학습 보상 곡선. LVRL, PPO, GRPO 비교 (논문 Figure 9)](https://momozzing.github.io/assets/images/llm-value-functions/fig9-rl-training-reward.png)

LVRL이 세 환경 모두에서 먼저 올라가고, BabyAI에서는 GRPO와 PPO 곡선이 바닥에 붙어 있다.

-> AppWorld 곡선을 보면 PPO가 500스텝 근처에서 LVRL과 학습 보상이 거의 같아진다. 그런데 평가 성공률은 14.8% 대 24.3%다. 학습 과제 90개에서의 보상과 처음 보는 과제에서의 성공률이 따로 움직인 것인데, 왜 그런지는 본문에 설명이 없다 ??
-> SWE-bench Verified 수치는 학습 시드 1개에서 나왔고, 표의 표준편차는 같은 체크포인트를 세 번 평가한 값이다. 학습을 다시 돌렸을 때의 흔들림은 이 표로는 알 수 없다.

비용도 본문에 적혀 있다.
대상 궤적 하나에 입력 9k~20k 토큰, 출력 0.6k~1.9k 토큰이 든다.
부록의 추정으로는 Opus 4.6 표준 요금 기준 질의당 AppWorld 0.061~0.062달러, BabyAI 0.107달러, R2E-Gym 0.140달러다.
토큰 수는 API 사용량 기록이 남지 않아 cl100k_base 토크나이저로 다시 센 값이다.
롤아웃 요약 호출은 여기서 빠져 있고, 요약 비용은 가치 추정과 비슷한 수준이라고 한다.

-> 부록 표의 질의 수(AppWorld Qwen3-4B 92,718건)에 단가를 곱하면 약 5,650달러다. 요약 호출까지 치면 그 두 배 근처일 것이다. 이건 내가 곱한 값이고 논문에 총액은 없다.

### **4.3 Does LVOPD Improve On-Policy Distillation?**

AppWorld Test Challenge에서 세 가지 증류 방식을 비교한 표다.

![AppWorld에서 OPD, Trajectory-gated OPD, LVOPD의 성공률 (논문 Table 2)](https://momozzing.github.io/assets/images/llm-value-functions/table2-opd-results-appworld.png)

OPD, Trajectory-gated, LVOPD 순으로 Qwen3-4B는 17.0, 19.2, 20.2이고 Qwen3-8B는 22.2, 21.2, 23.3이다.
저자들은 스텝 단위 게이팅이 더 믿을 만한 증류 신호를 준다고 해석한다.

-> LVOPD와 차순위의 차이가 4B에서 1.0점, 8B에서 1.1점인데 LVOPD의 표준편차가 1.8, 1.6이다. 시드 3개로는 이 차이를 확정하기 어려워 보인다.

학습 곡선은 아래와 같다.

![AppWorld에서 OPD 계열 세 방법의 학습 보상 곡선 (논문 Figure 10)](https://momozzing.github.io/assets/images/llm-value-functions/fig10-opd-training-reward.png)

Pure OPD가 초반에 빨리 오르고, LVOPD가 100스텝쯤에서 넘어선다.
모든 토큰에 감독을 걸면 초반 안내는 촘촘하지만 나중에는 학생이 갈 수 있는 범위를 좁힌다는 것이 저자들의 해석이다.

BabyAI에서는 OPD 계열 세 방법 모두 0%대에 머문다.
Pure OPD부터 실패했으니 Qwen3-235B의 토큰 감독만으로는 부족하고, critic도 같은 Qwen3-235B라서 가치 신호로도 메우지 못했다고 설명한다.
critic만 Opus 4.6으로 바꾸면 나아질 것이라는 가설은 실험하지 않고 future work로 남겼다.

### **4.4 Validating Step-Level Value Signals**

critic이 매긴 값의 변화가 실제 성공 확률의 변화와 맞는지를 AppWorld에서 확인한다.

정답 역할은 Monte-Carlo\* 롤아웃이 한다.
학습 25%, 50%, 75% 시점 체크포인트 세 개에서 과제 30개씩을 뽑고, 과제마다 성공 궤적 하나와 실패 궤적 하나를 골라 궤적 180개, 상태 3,899개를 만든다.
각 상태에서 환경을 처음부터 다시 실행해 같은 지점까지 되돌린 뒤, 같은 체크포인트로 이어서 32번 돌린다.
성공한 비율이 그 상태의 MC 가치다.

두 상태의 MC 가치 차이가 -3σ 이하면 pitfall, +3σ 이상이면 recovery, 절댓값이 σ 미만이면 MC-neutral로 부른다.
σ는 0.125로, 32번 표본 두 개의 차이가 가질 수 있는 최대 표준편차다.
이렇게 pitfall 76개, recovery 38개, neutral 1,533개가 나왔다.

결과는 이렇다.
- pitfall의 72%에서 예측 값이 내려감
- recovery의 66%에서 예측 값이 올라감
- neutral에서는 8%만 내려가고 22%가 올라감
- 환경 출력에 Traceback 같은 오류 문구가 있는지만 보는 규칙 기반 방법은 pitfall의 13%만 잡음

오류 문구로는 못 잡는 경우의 예가 부록에 있다.
Venmo 요청을 세 건 만들었고 실행도 성공했지만, 설명 문구 끝의 마침표가 빠져 exact-match 검증에 걸리는 스텝이다.
MC 가치는 0.94에서 0.00으로 떨어졌고 critic 예측은 0.80에서 0.10으로 떨어졌다.
critic은 이유로 성공한 다른 롤아웃들이 전부 마침표가 있는 문구를 썼다는 점을 들었다.

위치별로 나누면 정확도가 달라진다.
부호 일치율은 궤적의 앞 1/3에서 44%, 가운데 78%, 뒤 1/3에서 88%다.
앞쪽 이벤트의 29%에서는 critic이 값을 아예 바꾸지 않았고, 뒤쪽에서는 4%다.

앞쪽 불일치 19건 중 12건은 API 문서 확인, 로그인, 데이터 조회 같은 준비 행동이었다.
critic은 이를 진전으로 보고 값을 유지하거나 올렸는데 MC 가치는 내려갔다.
저자들은 critic의 추정이 엄밀하게 on-policy는 아니라고 해석한다.

*"it overestimates the current student’s ability to successfully complete the task."*

-> neutral 스텝의 22%에서 값이 올라간다는 수치와 같은 방향이다. critic은 뭔가 이뤄진 것처럼 보이는 스텝에 값을 올려 주는 쪽으로 기울어 있다.
-> 이 분석에 쓴 학습 실행이 어느 학생 모델, 어느 critic의 것인지는 원문에서 확인 못 함. 검증 환경도 AppWorld 하나다.

Monte-Carlo 추정: 같은 지점에서 끝까지 여러 번 실행해 본 결과의 평균으로 값을 추정하는 방법. 표본이 많을수록 정확하지만 그만큼 실행 비용이 든다

### **4.5 Ablation Studies**

두 가지를 뗀다.

**In-context calibration**

문맥 롤아웃 요약과 양 끝 고정을 한꺼번에 빼고 대상 궤적만 critic에게 준다.
AppWorld, Qwen3-8B 결과다.

![in-context calibration을 뺐을 때의 AppWorld 성공률 (논문 Table 3)](https://momozzing.github.io/assets/images/llm-value-functions/table3-in-context-calibration-ablation.png)

26.3%에서 22.1%로 내려가고, GRPO(19.8%)보다는 높다.
문맥 롤아웃과 양 끝 고정 각각의 효과, 문맥 개수에 따른 변화는 나눠 보지 않았다.

-> 양 끝 고정을 빼면 3.2절의 "총합이 그룹 상대 보상과 같다"는 성질이 깨진다. 그 상태에서 advantage를 어떻게 계산했는지는 본문에 없다 ??

**Choice of the frozen LLM critic**

Qwen3-4B를 AppWorld에서 LVRL로 학습시키되 critic만 바꾼 결과다.

![critic 모델별 AppWorld 성공률 (논문 Table 4)](https://momozzing.github.io/assets/images/llm-value-functions/table4-choice-frozen-llm.png)

Opus 4.6은 24.3%, GLM-5.2는 22.3%, Qwen3-235B는 9.8%다.
세 critic 모두 베이스(4.2%)보다는 높고, 저자들은 스텝 단위 credit assignment의 효과가 critic 능력에 막혀 있다고 정리한다.

-> Table 1에서 같은 설정의 GRPO가 10.7%였다. Qwen3-235B를 critic으로 쓴 9.8%는 critic 없이 돌린 GRPO와 비슷하거나 조금 낮다. critic이 약하면 스텝별로 쪼개는 것이 이득을 못 준다는 뜻으로 읽힌다.

## **5. Conclusion and Discussion**

LVRL은 얼린 LLM에게 끝난 궤적의 중간 상태 값을 매기게 하고, 양 끝을 그룹 성공률과 실제 보상에 고정해서 스텝별 advantage의 합이 그룹 상대 보상과 같아지게 만든다.
LVOPD는 그 advantage가 음수인 스텝에만 교사 증류를 건다.
세 환경에서 LVRL은 GRPO와 PPO를 넘었고, LVOPD는 AppWorld에서 두 증류 베이스라인보다 평균이 높았다.

저자들이 적은 한계는 세 가지다.
- critic 능력에 크게 의존한다. 약한 critic은 이득이 훨씬 작다
- 학습 중 모든 궤적마다 큰 LLM을 불러야 해서 추론 비용이 든다. 값이 불확실할 때만 부르거나 작은 critic으로 증류하는 것을 future work로 든다
- 텍스트 스텝으로 자연스럽게 나뉘는 다중 턴 에이전트 환경을 전제로 설계했다. 수학 추론 같은 비에이전트 영역은 다루지 않았다

## **6. 지금 관점: 학습 없이 스텝별 평가만 가져다 쓸 수 있을까**

[Reflexion](https://momozzing.github.io/paper%20review/Reflexion-Paper-review/)\*은 실패한 시도를 LLM이 말로 되짚어 다음 시도의 프롬프트에 넣었다.
LVRL은 같은 종류의 사후 판단을 숫자로 받아서 가중치 갱신에 쓴다.
[Multi-Harness RL](https://momozzing.github.io/paper%20review/Multi-Harness-RL-Paper-review/)이 보상을 어떤 묶음으로 정규화할지를 바꿔 봤다면, 이 논문은 정규화된 보상을 궤적 안에서 쪼개는 쪽을 바꿨다.
SWE-bench Verified 수치를 읽을 때는 [What Does a Harness Buy](https://momozzing.github.io/paper%20review/What-Does-a-Harness-Buy-Paper-review/)에서 같은 설정을 한 번 더 돌려 잡음 크기를 쟀던 것을 같이 떠올리게 된다.
여기서는 그 환경만 학습 시드가 1개다.

파인튜닝을 하지 않는 사람에게도 쓸 만한 부분은 4.4절과 부록의 프롬프트다.
끝난 궤적 하나, 같은 과제의 다른 실행 요약 몇 개, 그 과제의 성공률을 같이 주고 스텝마다 성공 확률과 이유를 받는 구성은 에이전트 로그를 사후 분석하는 데 그대로 옮길 수 있다.
오류 문구만 찾는 규칙이 pitfall의 13%를 잡을 때 이 방식이 72%를 잡았다는 비교는, 실행은 성공했는데 의미가 틀린 스텝을 찾는 용도로 볼 만하다는 근거가 된다.
다만 같은 과제를 여러 번 돌린 기록이 있어야 하고, 그중에 성공 사례가 섞여 있어야 비교 근거가 생긴다.
한 번씩만 실행되는 실제 서비스 로그에는 이 전제가 잘 안 맞는다.
궤적 앞 1/3에서는 부호가 맞는 비율이 44%였으니, 초반 스텝에 대한 평가는 덜 믿는 편이 맞다.

학습 방법으로 받아들일 때 걸리는 것은 critic이다.
Opus 4.6이나 GLM-5.2를 쓰면 이득이 나고 Qwen3-235B를 쓰면 GRPO 수준으로 돌아간다.
어떤 모델이 critic으로 충분한지 미리 알 방법은 논문에 없고, 써 보기 전에는 판단할 기준이 없다.
4B~32B 학생을 가르치는 데 그보다 훨씬 큰 API 모델을 궤적마다 부르는 구조라서, 학생이 critic 수준에 가까워질 때도 같은 이득이 나는지는 이 실험 범위 밖이다.
비용은 질의당 0.06~0.14달러에 요약 호출이 더해지는데, 롤아웃 자체의 GPU 비용과 견줘 어느 정도인지는 논문에 비교가 없다.

Reflexion: 에이전트가 실패한 시도를 스스로 말로 되짚어 기록하고, 그 반성문을 다음 시도의 프롬프트에 넣어 가중치 갱신 없이 성능을 올리는 방법
