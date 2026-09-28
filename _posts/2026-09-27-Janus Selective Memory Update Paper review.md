---
date: 2026-09-27 15:00:00 +0900
title: "The Past Is Prologue Paper review"
excerpt: "메모리 갱신을 '배포 결정'으로 다룬다. 후보 갱신을 무조건 받아들이는 대신 옛 메모리와 붙여 보고 나은 쪽을 남기는 플러그인 컨트롤러."
categories:
  - Paper review
tags:
  - Large Language Model
  - NLP
  - Agent
  - Memory
  - Paper review
toc: true
toc_sticky: true
field: agent-memory
---

The Past Is Prologue: A Plug-in Controller for Selective Updates in Sequentially Evolving LLM Memory

[https://arxiv.org/abs/2606.31121](https://arxiv.org/abs/2606.31121)

Janus는 University of Virginia, Princeton, UCF에서 만든 메모리 갱신 컨트롤러다.

기존 메모리 갱신기가 새 메모리를 제안하면, 그걸 바로 쓰지 않고 옛 메모리랑 비교해서 나은 쪽을 남긴다.

2026년 6월 30일에 올라왔고, 저자 7명에 15쪽이다.

앞에서 본 [Experience-Following 리뷰](https://momozzing.github.io/paper%20review/Experience-Following-Paper-review/)에서는 전부 넣으면 고정 메모리보다 못하다는 걸 봤고, 거기서는 평가기로 걸러서 넣으라고 했다. 여기서는 넣기 전에 옛 메모리랑 비교해 본다.

좀 더 자세히 알아보자.

## **1. Introduction**

기존 시스템은 메모리 갱신이 앞으로의 행동을 좋게 만드는지 확인하지 않고 그냥 반영한다고 한다.

그래서 세 가지 문제가 생긴다.

1. 지금 태스크에는 도움이 되는 갱신이 쓸모 있던 지식을 덮어쓴다
2. 너무 구체적인 규칙이 들어온다
3. 최종 메모리가 최근 예시 쪽으로 치우친다

Figure 1 그래프를 보면 태스크가 진행되면서 테스트 정확도가 오르다가 정체하거나 떨어진다.

![순차 메모리 갱신과 GPQA 중간 스냅샷 정확도 (논문 Figure 1)](https://momozzing.github.io/assets/images/janus/fig1-nonmonotonic-updates.png)

위쪽은 태스크마다 메모리를 M1, M2, ..., MT로 갱신하는 흐름이다.

아래쪽은 Qwen3-8B GPQA에서 DC-RS, ExpeL의 중간 메모리 스냅샷으로 잰 테스트 정확도라고 한다.

갱신을 많이 한다고 앞으로의 행동이 항상 나아지는 건 아니라고 한다.

여기서 다루는 메모리는 앞의 리뷰들처럼 대화를 회상하는 게 아니다. 과거 기록이 아니라, 다음 태스크에서 LLM이 어떻게 행동할지를 바꾸는 test-time 적응 수단으로 본다고 한다.

앞에서 본 [서베이](https://momozzing.github.io/paper%20review/Memory-in-the-Age-of-AI-Agents-Paper-review/) 분류로 보면 Functions는 experiential memory이고, Dynamics는 evolution에 해당한다.

## **2. Method**

### **2.1 Janus: Plug-in Memory Control**

ExpeL, DC-RS 같은 기존 갱신기를 감싸는 플러그인이다. 갱신 규칙 자체는 안 바꾼다.

태스크 `t`에서 기존 갱신기가 후보 메모리 `M̂_t`를 내놓으면, Janus가 이걸 쓸지 이전 메모리 `M_{t-1}`을 유지할지 정한다.

![Janus 전체 구조 (논문 Figure 2)](https://momozzing.github.io/assets/images/janus/fig2-janus-overview.png)

후보 갱신이 Momentum Trigger에 걸리지 않으면 그대로 배포된다.

걸리면 Hybrid Evaluation Set으로 `M_{t-1}`과 `M̂_t`를 비교해서 더 나은 쪽을 배포한다고 한다.

정해야 할 게 두 가지다.

1. 언제 비교할 것인가 (when to compare)
2. 무엇으로 비교할 것인가 (what to compare)

#### **2.1.1 Memory Momentum Trigger (MMT)**

언제 비교할지를 정하는 부분이다.

매번 비교하면 비싸다.

그래서 후보 갱신이 최근 갱신 흐름에서 벗어나는지를 먼저 본다. 벗어나면 비교하고, 아니면 후보를 바로 받아서 비용을 아낀다고 한다.

그래서 이름이 momentum이다. 갱신들이 대체로 같은 방향으로 가다가 갑자기 다른 방향으로 튀면 그때 확인하자는 것이다.

#### **2.1.2 Hybrid Trigger-Time Evaluation Set**

무엇으로 비교할지를 정하는 부분이다.

이력 전체를 다시 돌리는 건 비싸서, 작게 만든 평가 집합을 쓴다. 세 가지를 섞는다.

- Coverage : 지금까지 본 태스크 분포를 넓게 대표하는 것들
- Boundary : 메모리 상태에 따라 예측이 바뀔 만한 경계 사례들
- Fresh : 새로운 태스크

이 집합으로 옛 메모리랑 새 메모리를 평가해서 더 나은 쪽을 쓴다.

## **3. Experiment**

### **3.1 Experimental Settings**

데이터셋 여섯 개(MATH500, GPQA Diamond, MMLU-Pro Eng./Phy., APIBench-HF, HumanEval), 백본 두 개, 갱신기 두 개로 실험했다.

### **3.2 Main Results**

Qwen3-8B 결과다.

| 방법 | MATH500 | GPQA | MMLU-Eng | MMLU-Phy | APIBench | HumanEval | 평균 |
|---|---:|---:|---:|---:|---:|---:|---:|
| Memory-free | 78.2 | 57.4 | 61.2 | 84.0 | 65.2 | 90.3 | 72.7 |
| ExpRAG | 82.0 | 69.2 | 66.2 | 87.2 | 72.0 | 91.2 | 78.0 |
| DC-RS | 81.4 | 73.4 | 64.4 | 84.0 | 80.8 | 93.2 | 79.5 |
| DC-RS + Janus | 83.6 | 81.5 | 68.0 | 89.2 | 82.8 | 93.9 | 83.2 |
| ExpeL | 80.0 | 72.9 | 68.8 | 90.4 | 66.4 | 91.2 | 78.3 |
| ExpeL + Janus | 81.6 | 78.5 | 71.6 | 92.8 | 70.4 | 93.9 | 81.5 |

DeepSeek-V4-Flash 결과다.

| 방법 | 평균 |
|---|---:|
| Memory-free | 74.3 |
| DC-RS | 76.7 |
| DC-RS + Janus | 81.3 |
| ExpeL | 79.6 |
| ExpeL + Janus | 82.3 |

갱신기 두 개, 백본 두 개에서 모두 +2.7 ~ +4.6점 올랐다. 갱신기는 그대로 두고 감싸기만 해서 얻은 거다.

제일 많이 오른 건 GPQA에서 DC-RS 73.4 → 81.5로 8.1점이다.

### **3.3 MMT Trigger Ablation**

트리거 시점이 중요하다는 실험이다.

MMT를 네 가지 방식이랑 비교했다.

- Base : 비교 안 하고 후보를 바로 받음
- Always : 태스크마다 비교
- Random : Janus와 같은 트리거 확률로 무작위 비교
- Periodic : N단계마다 비교. N은 Janus의 트리거 횟수에 맞춤

![MMT 트리거 ablation (논문 Table 2)](https://momozzing.github.io/assets/images/janus/table2-mmt-ablation.png)

Qwen3-8B로 GPQA와 HumanEval에서 쟀다고 한다.

Trig. Rate는 옛 메모리와 새 메모리를 비교한 비율이다. Random과 Periodic은 Janus와 비슷한 트리거 예산으로 맞췄다.

무작위나 주기적으로 비교하는 것보다 MMT가 낫다고 한다. 비교 횟수가 같아도 언제 비교하느냐에 따라 달라진다는 것이다.

Always랑은 비교 횟수가 훨씬 적은데도 비슷하게 나온다. GPQA의 DC-RS에서는 Janus가 Always보다 오히려 높았다고 한다.

평가 집합이 작고 잡음이 있을 수 있어서, 많이 비교한다고 꼭 좋은 건 아니라는 것이다.

### **3.4 Support Set Composition Ablation**

평가 집합의 세 구성요소가 다 기여하는지 본다.

평가 집합에서 하나씩 빼 봤다.

- coverage 제거 : 일관되게 성능 하락
- boundary 제거 : 일관되게 성능 하락
- fresh 제거 : 가장 크게 하락 (GPQA·HumanEval 둘 다)

![평가 집합 구성요소 ablation (논문 Figure 3)](https://momozzing.github.io/assets/images/janus/fig3-evalset-ablation.png)

Qwen3-8B와 DC-RS 갱신기로 GPQA, HumanEval의 최종 테스트 정확도를 쟀다고 한다.

세 막대 중 w/o Fresh가 제일 짧다.

본 태스크로만 평가하면 메모리 선택이 치우친다고 한다.

-> 본 것만으로 평가하면 최근 예시에 맞춘 메모리가 유리해질 테니까, 새 태스크를 섞어야 일반화가 보이는 것 같다.

coverage는 본 분포를 넓게 대표하는 역할을, boundary는 메모리 상태에 민감한 사례를 맡는다고 한다.

### **3.5 Memory Deployment Ablation**

갱신을 다 받으면 중간에 정체한다는 실험이다.

스트림의 20%, 40%, 60%, 80%, 100% 시점마다 중간 메모리로 정확도를 쟀다.

![옛 메모리 대 새 메모리 배포 결정 ablation (논문 Figure 4)](https://momozzing.github.io/assets/images/janus/fig4-deployment-ablation.png)

Qwen3-8B로 GPQA와 MMLU-Pro (Eng.)에서 쟀다고 한다.

빨간 선이 갱신을 다 받는 기존 갱신기, 파란 선이 Janus를 붙인 경우다.

DC-RS와 ExpeL 둘 다 갱신을 전부 받으면 초중반에는 좋아지다가, 그 뒤로는 정체하거나 떨어진다고 한다.

앞에서 본 [Experience-Following 리뷰](https://momozzing.github.io/paper%20review/Experience-Following-Paper-review/)의 Add all 결과랑 같은 모양이다. 거기서는 최종 성능만 봤는데, 여기서는 곡선이 꺾이는 지점까지 보여준다.

## **4. 지금 관점: 갱신을 배포로 본다**

앞 논문들에서 본 갱신 방식을 모아보면 이렇다.

- Mem0 : LLM이 ADD/UPDATE/DELETE/NOOP 중에 고름. 검증 없음
- Zep : LLM이 모순을 판단해서 무효화. 검증 없음
- A-MEM : LLM이 이웃 메모리를 진화시킴. 검증 없음
- Experience-Following : 평가기로 넣을지 거름. 넣기 전에 필터
- Janus : 넣은 결과를 옛 것과 비교. 넣은 다음에 검증

앞의 셋은 LLM 판단을 그대로 믿는다. Janus는 그 결과를 재보고 나쁘면 되돌린다.

-> 되돌리려면 이전 메모리가 남아 있어야 한다. 메모리를 그 자리에서 덮어쓰면 안 된다. 바로 다음에 볼 [Rate-Distortion 리뷰](https://momozzing.github.io/paper%20review/Rate-Distortion-Memory-Compaction-Paper-review/)의 가역성이 여기서도 필요하다. 버전을 남기든지 적어도 직전 상태는 들고 있어야 할 것 같다.

-> 세 평가 집합 중에 boundary는 직접 만들기 어려울 것 같다. 메모리가 바뀌면 답이 달라질 만한 질의를 어떻게 골라야 하는지?? fresh는 안 본 질의를 조금 떼어두면 되니까 쉽고, 기여도 제일 컸다.

-> 매번 검증할 필요도 없다. 비교 자체가 비용이고 평가 집합이 작으면 잡음이 끼니까, 갱신 흐름이 튈 때만 보면 된다.

논문 마지막 문단에서는 이렇게 말한다.

순차적으로 바뀌는 메모리는 경험을 더 많이 저장하거나 더 자주 갱신하는 것과는 다른 스케일링 문제라고 한다. 이력이 길어질수록 추가 추론을 어디에 쓸지가 중요해지고, 새 메모리를 다 믿어서도 안 되고 모든 갱신을 비싸게 검증할 필요도 없다고 한다.

## **5. Conclusion**

메모리 갱신을 배포 결정으로 다룬 논문이다.

국소적으로 만든 메모리를 무조건 받아들이던 걸, 어떤 메모리가 앞으로의 추론에 쓰일지 통제하는 쪽으로 바꿨다.

순차적으로 바뀌는 메모리를 튼튼하게 만들려면, 메모리를 잘 쓰는 것뿐 아니라 메모리를 고치는 게 비용만큼 가치가 있는지 판단하는 장치도 필요하다고 한다.

한계도 적었다.

- 범위 : 프롬프트 기반 순차 메모리만 다룬다. LLM은 고정이고 메모리는 검색·요약·반성·치트시트 같은 외부 텍스트로 갱신되는 설정이다. RL로 정책이나 스킬을 갱신하는 학습 기반 진화로 넓히는 건 후속 과제라고 한다
- 평가 범위 : 여러 태스크와 백본 두 개를 봤지만 모든 순차 에이전트 환경을 다 본 건 아니다. 장기 상호작용 환경, 멀티에이전트, 비정상 분포에서는 다른 문제가 있을 수 있다고 한다

-> 두 번째는 [Experience-Following 리뷰](https://momozzing.github.io/paper%20review/Experience-Following-Paper-review/)랑 겹친다. 거기서는 태스크 분포가 바뀌면 history-based 삭제가 오히려 불리했다. Janus의 coverage도 본 분포를 전제로 하니까 분포가 바뀌면 같은 문제가 생길 것 같다. 논문도 이걸 한계로 적었다.

여태까지 메모리 갱신은 LLM이 제안하면 그대로 반영했다면, 이 방법은 옛 메모리와 비교해서 나을 때만 반영한다.

다음은 [What to Keep, What to Forget](https://momozzing.github.io/paper%20review/Rate-Distortion-Memory-Compaction-Paper-review/)이다. KV 캐시 축출, 프롬프트 압축, 에이전트 메모리 요약을 하나의 rate-distortion 문제로 묶은 서베이다.
