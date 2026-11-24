---
date: 2026-11-25 06:00:00 +0900
title: "ProgAgent Paper review"
excerpt: "도구를 하나씩 직접 부르는 대신 도구를 부르는 코드를 쓰게 해서 Qwen3-8B·14B를 학습시킨 논문. 과제 215개만으로 14B 모델이 MCPMark 29.7%(직접 호출로 같은 학습을 하면 8.0%)를 찍는데, 학습 안 한 모델에 코드 인터페이스만 쥐여 주면 점수가 거의 안 오른다."
categories:
  - Paper review
tags:
  - Large Language Model
  - NLP
  - Agent
  - Tool Use
  - Paper review
toc: true
toc_sticky: true
field: agent
---

ProgAgent: Learning Tool Orchestration through Programmatic Training

[https://arxiv.org/abs/2610.14774](https://arxiv.org/abs/2610.14774)

ProgAgent는 홍콩과기대(HKUST), 베이징우전대(BUPT), 난양공대(NTU) 저자 7명이 만든 도구 사용 학습 스택이다.
2026년 10월 10일 arXiv 에 올라온 논문이다.
모델이 도구를 한 번에 하나씩 부르는 대신, 도구를 호출하는 파이썬 코드를 쓰고 중간 결과는 컨텍스트 말고 런타임 변수에 두게 한다.
이 방식을 추론 때만 쓰지 않고 학습 방식으로 썼을 때 무엇이 달라지는지를 Qwen3-8B와 14B로 잰다.
앞에서 본 [UniToolCall](https://momozzing.github.io/paper%20review/UniToolCall-Paper-review/)에서는 39만 건으로 Qwen3-8B를 파인튜닝했는데, 이 논문은 과제 215개(환경 크기 4단계, 860개 인스턴스)로 학습한다.

![ProgAgent 개요. (a) 과제 합성, (b) PTC 런타임, (c) 2단계 학습 (논문 Figure 1)](https://momozzing.github.io/assets/images/progagent/fig1-overview-progagent-task.png)

## **1. Introduction**

지금 에이전트는 대부분 Direct Tool Calling(DTC) 으로 돈다.
모델이 턴마다 도구를 하나 부르고, 도구가 돌려준 결과는 그대로 컨텍스트에 쌓인다.
긴 과제에서는 날것의 도구 출력이 계속 쌓여서 context rot(입력 토큰이 늘수록 모델 성능이 떨어지는 현상)과 orchestration breakdown(도구 호출 순서·흐름이 무너지는 것)이 생긴다.

Programmatic Tool Calling(PTC)은 모델이 도구를 부르는 코드를 생성하고, 그 코드를 샌드박스에서 실행하는 방식이다.
도구 반환값은 변수에 저장되고, 모델 컨텍스트에는 코드가 명시적으로 print 한 것만 들어간다.
PTC는 지금까지 주로 추론 시점의 최적화로 다뤄졌는데, 저자들은 학습 방식으로도 쓸 만하다고 주장하고 그걸 막는 빈자리 세 개를 든다.
PTC 학습용으로 설계된 검증 가능한 긴 과제가 드물고, 턴 사이 실행 상태 유지와 에피소드 격리가 되는 통제된 런타임이 없고, PTC로 학습하면 오케스트레이션이 나아지는지와 그 능력이 DTC로 옮겨 가는지가 알려져 있지 않다.
ProgAgent는 이 세 자리를 각각 과제 합성 파이프라인, 가벼운 PTC 런타임, cold-start SFT\* 뒤 강화학습이라는 2단계 학습으로 채운다.
abstract를 보면 215개 과제를 네 가지 환경 크기로 만들어 학습했고, 네 벤치마크에서 비슷한 크기의 baseline 보다 통과율이 높다.
LOCA-bench(환경 상태 크기를 조절해 컨텍스트가 크게 불어나는 상황에서 에이전트를 재는 벤치마크)의 64k 토큰 설정에서 ProgAgent-14B는 30.7%이고 비교한 같은 크기 baseline 은 0%다.

SFT: Supervised Fine-Tuning. 정답 궤적(여기서는 teacher 모델이 푼 궤적)을 그대로 따라 쓰도록 모델을 미세조정하는 지도학습. 강화학습 전에 모델이 보상을 받을 수 있는 수준까지 올려 두는 용도로 쓰면 cold-start 라고 부른다

## **2. Reframing Tool Use as Code**

### **2.1 Multi-turn Agentic Interaction as a Partially Observable Markov Decision Process**

여러 턴에 걸친 에이전트-환경 상호작용을 POMDP(에이전트가 전체 상태를 못 보고 관측만 받는 의사결정 과정)로 적는다.
상태는 환경 상태(데이터베이스, 워크스페이스 파일, 실행 가능한 도구)와 상호작용 이력(지시문, 돌아온 관측, 추론 내용) 둘로 나뉘고, 행동은 도구 행동과 자연어 응답 두 종류다.
보상은 에피소드가 끝났을 때 검증 스크립트가 최종 응답과 최종 환경 상태를 보고 0 또는 1로 준다.

### **2.2 From Direct Tool Calling to Programmatic Tool Calling**

같은 틀 위에서 DTC와 PTC를 두 가지 하네스로 구분한다.

![주문 800건에서 미발송·100달러 초과 건을 집계하는 과제를 DTC와 PTC로 푸는 모습 (논문 Figure 2)](https://momozzing.github.io/assets/images/progagent/fig2-direct-tool-calling.png)

DTC에서는 턴마다 도구 행동 하나를 내고, 그 행동과 돌아온 관측이 통째로 이력에 붙는다.
과제의 제어 흐름은 도구 호출의 나열로 표현되고, 컨텍스트는 턴마다 행동 길이와 관측 길이만큼 늘어난다.

PTC에서는 턴마다 프로그램 하나를 내고, 이 프로그램은 샌드박스 안에서 도구를 여러 번 부를 수 있고 제어 흐름, 데이터 변환, 예외 처리를 담을 수 있다.
중간 도구 결과는 런타임 변수에 남아 다음 턴에도 쓸 수 있고, 이력에 붙는 건 프로그램 자체와 그 프로그램이 print 한 출력뿐이다.
Figure 2의 예시에서 DTC는 주문 ID 800개 목록과 주문·고객 조회 결과가 컨텍스트에 쌓이고, PTC는 for 루프 안에서 조회·필터·집계를 하고 합계와 상위 3개 지역만 찍는다.

### **2.3 PTC as an Effective Training Paradigm**

저자들이 PTC를 학습 방식으로 보는 이유는 둘이다.
첫째는 사전학습에서 배운 코딩 능력의 전이다.
모델은 사전학습 때 코드의 제어 흐름, 데이터 변환, 에러 처리를 이미 배웠고, 도구가 호출 가능한 함수로 노출되면 한 턴 안에서 서로 얽힌 도구 호출 여러 개를 이 구조로 엮을 수 있다.
둘째는 컨텍스트 관리로, 중간 결과를 변수에 두고 골라서 print 하면 컨텍스트가 덜 불어난다.
이 절은 가설을 세우는 자리이고, 더 적은 학습 과제로 성능이 나오는지와 모델이 출력을 걸러 찍는 행동을 학습으로 익히는지를 뒤에서 확인하겠다고 적는다.

## **3. ProgAgent: From Task Synthesis to Agentic Training**

ProgAgent는 과제 합성 파이프라인(3.1), PTC 런타임(3.2), 학습 레시피(3.3) 세 부분으로 되어 있다.

### **3.1 Synthesizing Verifiable Long-Horizon Agentic Tasks**

#### **3.1.1 Design Principles**

**환경 상태 크기로 난이도를 조절한다**

기존 환경 합성 파이프라인은 도구 종류를 늘리거나 오케스트레이션을 복잡하게 해서 과제를 어렵게 만드는데, 이러면 환경 상태 크기와 난이도를 직접 조절할 수 없다.
ProgAgent는 과제 지시문, 과제 로직, 도구 스키마를 그대로 두고 환경 상태만 단계마다 대략 2배씩 키운다.
환경이 커지면 에이전트는 더 큰 환경에서 정보를 찾아 처리해야 하고, DTC에서는 그 관측이 그대로 컨텍스트에 붙는다.
DeepSeek V4 Pro 의 pass@8(과제당 8번 돌려 잰 통과율. 정확한 계산식은 원문에서 확인 못 함)은 DTC에서 v0 0.95에서 v3 0.42로 떨어지고, PTC에서는 네 단계 모두 0.90 이상이다.

**프로그램으로 풀기 좋은 과제를 만든다**

환경 데이터는 관계형 데이터베이스의 여러 테이블에 나눠 넣고, 과제 제약은 비정형 파일에 둔다.
과제를 풀려면 파일에서 제약을 읽고 여러 출처의 정보를 합쳐야 해서, 루프와 조건 분기가 도구 호출을 엮는 데 쓰인다.

**검증 가능해야 한다**

과제마다 파이썬 검증 스크립트가 붙는다.
검증기는 최종 응답과 최종 환경 상태를 평가 기준과 대조해서, 기준을 모두 만족할 때만 1을 돌려준다.
appendix를 보면 검증에 LLM-as-a-judge\* 는 쓰지 않는다.
과제가 풀리는지는 oracle 풀이(주어진 도구만 써서 만점을 받는 정답 풀이)로 확인하고, oracle 이 검증을 통과 못 하면 자동 수리 루프가 환경·도구·지시문·검증기를 고친다.

#### **3.1.2 Task Construction Pipeline**

과제 하나는 다섯 단계로 만들어진다.
LLM이 23개 도메인에서 사용자 의도를 생성하면, constructor 에이전트가 스크립트로 관계형 데이터베이스와 파일 워크스페이스를 만든다.
이어서 한 가지 일만 하는 도구들(구현은 과제를 푸는 에이전트에게 숨긴다)과 지시문을 합성하고, 검증기와 oracle 풀이를 만들어 oracle 이 만점을 받을 때까지 수리한다.
마지막으로 scaling 설정을 바꿔 v0~v3 네 변형을 만들고, 지시문·로직·도구 스키마는 고정한다.

#### **3.1.3 Verification-Based Filtering**

oracle 검증 뒤에 두 번 더 거른다.
먼저 독립된 하네스 에이전트가 도구, 환경 상태, 검증기가 지시문과 맞는지 감사하고, 문제가 있으면 수리 후 다시 감사한다.
다음은 난이도 필터로, 과제마다, 크기마다 DeepSeek V4 Pro 로 PTC 하네스에서 궤적 8개를 뽑고, 통과율이 모든 크기에서 0 초과 0.8 미만인 과제만 남긴다.
appendix를 보면 후보 250개 중 35개가 여기서 빠져 215개가 남았다.

-> 난이도 필터가 PTC 하네스의 DeepSeek V4 Pro 통과율로 걸러졌다. 학습 과제가 처음부터 "PTC로 풀리는 과제" 쪽으로 골라진 풀이라는 점은 뒤의 DTC 대 PTC 비교를 읽을 때 같이 봐야 한다.

LLM-as-a-judge: 정답 여부나 품질을 규칙·스크립트 대신 다른 LLM에게 판정시키는 평가 방법. 채점이 유연한 대신 판정 모델의 오류와 편향이 점수에 섞인다

### **3.2 A Lightweight PTC Runtime**

PTC 런타임은 MCP(Model Context Protocol, 모델에 외부 도구를 연결하는 표준 프로토콜) 서버로 모델에 노출된다.
에피소드를 시작할 때 데이터 생성기를 목표 크기로 돌려 환경 상태를 초기화하고 지시문을 준다.
런타임은 샌드박스 안에서 도는 persistent IPython kernel(턴이 바뀌어도 변수와 import 가 유지되는 파이썬 실행 세션)이다.
tool proxy 가 프로그램 안의 호출을 환경 도구로 보내 결과를 파이썬 객체로 돌려주고, 커널은 print 된 출력만 관측으로 돌려준다.
샌드박스 격리에는 Bubblewrap(권한 없이 쓰는 리눅스 샌드박스 도구)을 쓰고, 에이전트가 검증기 구현과 oracle 풀이에 접근하지 못하게 막는다.

### **3.3 Training with ProgAgent**

필터를 통과한 215개 과제를 v0~v3 네 크기로 만들어 학습한다.
학습은 teacher 궤적으로 하는 cold-start SFT, 그다음 GSPO\* 강화학습 두 단계다.
같은 레시피를 세 가지 하네스에 적용한다.

- DTC: 환경 도구를 표준 스키마로 노출해 직접 호출하게 한다.
- PTC: 환경 도구 호출이 전부 파이썬 샌드박스를 거쳐야 한다.
- Hybrid: 둘 다 노출하고, 모델이 직접 호출과 프로그램 실행을 섞어 쓸 수 있다.

하네스 효과만 떼어 보려고 세 설정에 같은 과제 풀과 같은 학습 하이퍼파라미터를 쓴다.

?? teacher 궤적을 어느 모델이 만들었는지 명시한 문장을 못 찾았다. appendix에 "모든 실험에 DeepSeek V4 Pro 0813 버전을 썼다"는 문장이 있어 이 모델로 짐작되지만 확인 못 함.

GSPO: Group Sequence Policy Optimization. 같은 과제에서 뽑은 여러 rollout 을 그룹으로 묶어 보상을 비교하는 강화학습 알고리즘으로, 중요도 비율과 clipping 을 토큰 단위 대신 시퀀스(응답 전체) 단위로 계산한다. 별도의 가치 모델(critic)은 두지 않는다

## **4. Experiments**

### **4.1 Experimental Setup**

과제 의도 생성에는 Gemini 3.0 Flash 를, constructor 와 하네스 감사 에이전트에는 Claude Code 를 통한 Claude Opus 4.6 을 썼다.
학습 모델은 Qwen3-8B와 14B이고, DTC·PTC·Hybrid 각각으로 학습한 뒤 학습 때와 같은 하네스로 평가한다.
SFT는 3 epoch(batch size 64), GSPO는 150 step 이고 step 마다 과제 32개에 rollout 16개씩 뽑는다.
appendix를 보면 두 모델 다 thinking mode 로 썼고, 원래 32k 인 컨텍스트 창을 YaRN\* 으로 128k 까지 늘렸다.

평가 벤치마크는 네 개다.

- MCPMark(실제 MCP 서버에서 과제를 푸는 벤치마크). GitHub 묶음은 계정 정지 문제로 빼고 Filesystem, Notion, Playwright, PostgreSQL 네 묶음의 평균을 쓴다.
- MCP-Atlas(실제 MCP 서버로 도구 사용 능력을 재는 벤치마크)
- Toolathlon-Verified(다양한 앱에 걸친 긴 과제 실행을 재는 Toolathlon 의 검증판)
- LOCA-bench. 환경 상태 크기 8k~64k 네 설정으로 돌린다.

통과율은 독립 실행 4번의 평균이다.
비교 대상은 상용 모델(GPT-5.2, Claude Sonnet-4.5, Gemini-3 Pro), 공개 모델(Qwen3, DeepSeek-V3.2), 그리고 학습된 baseline 여섯 개(TOUCAN\*, Simulator\*, EnvScaler\*, AWM\*, Agent-World\*, EnvFactory\*)다.
Table 1의 학습 데이터 규모를 보면 TOUCAN 은 153만 궤적, EnvScaler 는 11,572개, AWM 은 3,315개, EnvFactory 는 2,575개이고 ProgAgent 는 215 × 4개다.
Table 2의 baseline 은 모두 DTC 하네스로 평가했다.

appendix의 evaluation protocol 을 보면 표의 숫자가 전부 저자들이 직접 돌린 값은 아니다.
MCPMark 에서 상용 모델, 공개 모델, EnvFactory-8B를 뺀 baseline 의 점수는 Agent-World 논문에서, 상용 모델의 LOCA-bench·MCP-Atlas 점수는 각 벤치마크 논문에서 가져왔다.
Toolathlon 열에서 † 가 붙은 네 모델 점수는 Verified 판이 아닌 원래 Toolathlon 점수다.

YaRN: 위치 인코딩(RoPE)의 주파수를 조정해, 학습 때보다 긴 컨텍스트 창을 쓸 수 있게 늘리는 기법

TOUCAN: 실제 MCP 환경에서 도구 사용 궤적을 대량으로 합성해 만든 학습 데이터셋과 그걸로 학습한 모델

Simulator: 추론 모델(LLM)이 환경 역할을 맡아 도구 반환값을 생성하게 해서 에이전트 학습 데이터를 만드는 방법

EnvScaler: 도구와 상호작용하는 실행 가능한 환경을 프로그램으로 합성해 규모를 키우는 환경 합성 방법

AWM: Agent World Model. 에이전트 강화학습용 합성 환경을 대량으로 만들어 내는 방법

Agent-World: 시뮬레이션 환경과 실행 가능한 환경을 함께 쓰고, 학습이 진행되면서 환경도 같이 바뀌는 학습용 arena

EnvFactory: 실행 가능한 환경을 합성하고 그 위에서 강화학습으로 도구 사용 에이전트를 학습시키는 방법

### **4.2 Main Results**

아래는 네 벤치마크의 평균 통과율(%)을 모델별로 잰 주 결과 표다.

![네 벤치마크 주 결과. 상용 모델, 공개 모델, 학습된 baseline, ProgAgent (논문 Table 2)](https://momozzing.github.io/assets/images/progagent/table2-main-results-mean.png)

MCPMark 평균은 ProgAgent-8B 18.3%, 14B 29.7%이고, 같은 크기 baseline 중 가장 높은 건 Agent-World-8B 10.0%, Agent-World-14B 15.6%다.
Toolathlon-Verified 는 8B 9.3%, 14B 17.1%이고, baseline 은 8B에서 최대 2.8%, 14B에서 최대 1.9%다.
MCP-Atlas 는 ProgAgent-14B가 45.6%로, DTC 하네스의 Claude Sonnet-4.5(43.8%)와 비슷한 수준이다.
LOCA-bench 에서는 비교한 같은 크기 baseline 이 32k 와 64k 에서 전부 0%인데 ProgAgent 는 64k 에서 8B 17.3%, 14B 30.7%를 낸다.

-> Sonnet-4.5 와 비슷하다는 건 MCP-Atlas 한 곳의 이야기다. MCPMark 평균은 33.6 대 29.7, Toolathlon 은 38.9(†) 대 17.1 로 차이가 남아 있다. 상용 모델은 DTC로만 쟀고 PTC를 쥐여 준 수치는 없다. MCPMark 의 Playwright 열은 ProgAgent 8B·14B 모두 4.0%로 baseline 과 같다. 평균을 끌어올린 건 Filesystem 과 PostgreSQL 이다.

다음은 같은 레시피를 DTC·Hybrid·PTC로 학습했을 때의 최종 통과율이고, 괄호 안은 학습 전 모델을 같은 하네스에서 잰 값이다.

![학습 방식별 통과율. 괄호는 같은 하네스에서 잰 학습 전 모델 점수 (논문 Table 3)](https://momozzing.github.io/assets/images/progagent/table3-training-paradigms-across.png)

학습 전 모델은 PTC로 바꿔도 얻는 게 거의 없고, 세 하네스 중 PTC 점수가 가장 낮은 경우가 많다.
예를 들어 14B의 MCP-Atlas 는 DTC 12.4, Hybrid 11.4, PTC 3.8이다.
학습 뒤에는 순서가 뒤집혀서, 14B 기준 MCPMark 평균은 DTC 8.0, Hybrid 25.4, PTC 29.7이고 MCP-Atlas 는 29.7, 38.6, 45.6이다.
8B에서도 PTC가 네 벤치마크 모두에서 DTC보다 높다.

LOCA-bench 에서는 SFT와 강화학습 둘 다 점수를 올리고(Figure 3), PTC의 평균 통과율은 8B 24.3%, 14B 38.3%다.
Hybrid 는 학습과 평가 양쪽에서 더 유연한 행동 공간을 쓰는데도 PTC보다 낮다.
저자들은 PTC가 도구와 상호작용하는 인터페이스를 하나로 통일해서 학습 난이도를 낮춘다는 가설을 드는데, 이 가설을 따로 검증하는 실험은 본문에 없다.

## **5. Analysis**

### **5.1 What Does PTC Change during Training?**

Qwen3-14B의 강화학습 과정에서 보상, 컨텍스트에 들어온 도구 출력 토큰, 에피소드당 도구 호출 수를 추적했다.

![14B 강화학습 중 보상(왼쪽), 도구 출력 토큰(가운데), 에피소드당 도구 호출 수(오른쪽) (논문 Figure 4)](https://momozzing.github.io/assets/images/progagent/fig4-rl-training-dynamics.png)

DTC의 보상은 0.08에서 0.15로 오르는 데 그치고, PTC와 Hybrid 는 0.50 근처에서 시작해 각각 0.70, 0.72에 이른다.
DTC에서는 도구 출력이 학습 내내 에피소드당 약 30k 토큰을 차지하고, PTC는 학습 동안 약 20% 줄어 20k 아래로 내려간다.
PTC와 Hybrid 의 도구 호출 수는 DTC의 절반 정도이고, PTC 런타임 에러는 에피소드당 14B에서 6.4에서 3.6으로, 8B에서 7.5에서 5.0으로 줄었다.

appendix의 training hyperparameters 에는 이 비교에 걸리는 문장이 하나 있다.
*"For DTC RL, we only use v0 environments, as the other environments are too difficult for the model to obtain any rewards."*

-> DTC 강화학습은 가장 작은 v0 환경에서만 돌았다. 그래프의 DTC 보상 곡선과 PTC 보상 곡선은 같은 환경 분포에서 잰 값이 아닐 수 있다. 3.3절의 "같은 과제 풀"은 과제 215개가 같다는 뜻으로 읽어야 한다.

### **5.2 Does Environment State Scaling Help?**

환경을 키우면 DTC에서 난이도가 실제로 올라간다.
Table 4를 보면 v0에서 v3으로 가면서 데이터베이스 크기는 99KB에서 711KB로, 워크스페이스 파일 수는 평균 12.7개에서 40.3개로 늘고, DeepSeek V4 Pro 의 DTC pass@8 은 0.95, 0.88, 0.65, 0.42로 단조 감소한다.
학습을 v0 환경으로만 제한하면 LOCA-bench 평균이 8B에서 24.3%에서 16.0%로, 14B에서 38.3%에서 23.7%로 떨어진다(Table 7).

### **5.3 How Does PTC Facilitate Tool Orchestration?**

강화학습 중 생성된 코드 블록이 어떤 프로그래밍 구조를 썼는지 셌다(Table 5).
루프는 8B 82.6%, 14B 82.1%, 조건 실행은 65.6%, 64.0%이고, 에러 처리는 실행의 8.2%, 9.5%에서만 쓰였다.

다음으로 PTC 학습의 이득이 PTC 인터페이스 밖에서도 남는지를 본다.
아래 표는 학습 전 모델, DTC로 학습한 모델, PTC로 학습한 모델을 DTC와 PTC 하네스 양쪽에서 평가한 통과율이다(LOCA-bench 는 네 크기 평균, MCP-Atlas).

![학습 하네스와 평가 하네스를 교차한 통과율 (논문 Table 6)](https://momozzing.github.io/assets/images/progagent/table6-cross-harness-evaluation-loca-bench.png)

DTC로 학습한 모델을 PTC 하네스로 옮기면 LOCA-bench 는 오르지만(14B 9.0에서 20.7) MCP-Atlas 는 안 오른다(14B 29.7에서 29.0, 8B는 19.0에서 11.8로 내려간다).
다른 baseline 을 PTC 하네스에 넣어도 통과율은 0 근처에 머문다(Table 7).
저자들은 이걸 두고 *"access to the PTC interface alone is insufficient for effective orchestration"* 이라고 적는다.

같은 PTC 하네스에서는 PTC로 학습한 모델이 DTC로 학습한 모델보다 두 벤치마크 모두에서 높고, DTC 하네스로 평가해도 그렇다.
MCP-Atlas 에서 14B PTC 학습 모델은 DTC 평가로 41.6%, DTC 학습 모델은 29.7%로 11.9점 차이다.
LOCA-bench 의 DTC 평가에서는 14B 13.9 대 9.0, 8B 6.0 대 4.0이고, v0만으로 학습한 설정에서도 이 우위가 유지된다.

-> DTC로 평가해도 PTC 학습 모델이 높다는 결과는 MCP-Atlas 에서는 차이가 크고 LOCA-bench 8B에서는 2점이다. 4회 평균이라는 것만 나와 있고 분산이 없어서 2점이 잡음보다 큰지는 알 수 없다.

?? 왜 PTC로 배운 게 DTC로 옮겨 가는지에 대한 설명은 없다. conclusion 에서도 추가 연구가 필요한 질문으로 남긴다. 위에서 본 것처럼 DTC 강화학습이 v0에서만 돌았다는 차이가 여기에 얼마나 섞였는지도 본문만으로는 가를 수 없다(v0만으로 학습한 비교가 Table 7에 있긴 하다).

### **5.4 What Challenges Remain for PTC Agents?**

저자들이 든 한계는 두 가지다.

**복잡한 환경에서 DTC처럼 행동한다**

환경이 예측하기 어렵거나 구조가 없는 내용을 돌려주면 PTC 에이전트가 DTC처럼 움직인다.
Playwright 에서는 현재 화면 스냅샷을 해석해야 다음 도구 호출을 정할 수 있다.
ProgAgent-14B의 Playwright PTC 호출 중 40.7%는 제어 흐름, 에러 처리, 선택적 print 없이 도구 하나만 부른다(Table 8).
앞에서 본 Playwright 점수 4.0%가 이 한계와 같은 자리다.

**서버가 에러를 조용히 넘긴다**

ProgAgent 의 PTC 하네스는 MCP의 isError 필드를 보고 도구 예외를 일으킨다.
규격을 안 지키는 서버는 이 필드를 빼먹는 경우가 많고, 그러면 예외 기반 복구가 작동하지 않는다.
서버 수준 에러 감지를 끄면 LOCA-bench 통과율이 8B에서 24.3%에서 19.3%로, 14B에서 38.3%에서 34.5%로 떨어진다.

## **6. Conclusion**

ProgAgent 는 여러 환경 크기의 검증 가능한 과제, PTC 런타임, cold-start SFT 뒤 강화학습이라는 레시피를 묶은 학습 스택이다.
215개 과제를 네 크기로 써서 네 벤치마크에서 비슷한 크기의 baseline 보다 높은 통과율을 냈다.
저자들은 PTC가 추론 인터페이스로도, 학습 방식으로도 쓸모 있고 그 이득이 DTC 평가에서도 남는다고 정리한다.

별도의 Limitations 절은 없고, 원문이 밝힌 한계는 5.4의 두 가지(Playwright 같은 환경에서의 DTC식 행동, 서버의 조용한 실패)다.

## **7. 지금 관점: 코드로 도구를 부르게 하는 걸 어디까지 믿고 가져다 쓸까**

블로그에서 본 [Toolformer](https://momozzing.github.io/paper%20review/Toolformer-Paper-review/)는 도구 호출 한 번을 텍스트 중간에 끼워 넣는 법을 배웠고, [UniToolCall](https://momozzing.github.io/paper%20review/UniToolCall-Paper-review/)은 도구 호출 데이터를 한 형식으로 모아 양으로 밀었다.
둘 다 ReAct\* 식으로 한 번에 도구 하나를 부르는 틀 안에 있고, ProgAgent 는 행동의 단위를 프로그램 하나로 바꾼 채 학습까지 시킨 경우다.
성공한 행동을 코드로 남기던 [Voyager](https://momozzing.github.io/paper%20review/Voyager-Paper-review/)나 컨텍스트에 남는 정보의 밀도를 따지던 [GenericAgent](https://momozzing.github.io/paper%20review/GenericAgent-Paper-review/)와 문제의식이 겹친다.
학습한 하네스와 평가 하네스를 엇갈려 재는 Table 6은 [Multi-Harness RL](https://momozzing.github.io/paper%20review/Multi-Harness-RL-Paper-review/)에서 본 질문과 같은 종류다.

에이전트를 만드는 입장에서 바로 가져갈 수 있는 건 학습 결과보다 그 주변의 관찰들이다.
학습 안 한 Qwen3 에 코드 실행 도구만 쥐여 주면 점수가 안 오르고 오히려 내려가는 경우가 많았다.
작은 공개 모델로 에이전트를 돌리면서 "코드 실행 모드로 바꾸면 컨텍스트가 줄겠지"라고 기대하면 이 표와 어긋난다.
반대로 DeepSeek V4 Pro 는 학습 없이도 PTC에서 pass@8 0.90 이상을 유지했으니, 모델이 충분히 강하면 인터페이스 변경만으로 이득이 나는 구간이 있다는 뜻으로 읽힌다.
어느 크기부터 그런지는 이 논문으로는 알 수 없다.
변수를 턴 사이에 유지하는 커널, 출력 길이 상한, isError 를 예외로 바꿔 주는 처리, 그리고 그 처리가 빠지면 LOCA-bench 에서 3.8~5점이 떨어진다는 수치는 PTC식 도구를 직접 구현할 때 확인할 항목이 된다.

그대로 받아들이기 어려운 부분도 있다.
학습 과제는 PTC 하네스 통과율로 걸러졌고, 데이터베이스와 파일을 뒤져 집계하는 유형이라 루프로 풀기 좋게 설계됐다.
DTC 강화학습은 v0에서만 돌았으니, "PTC 학습이 DTC 학습보다 낫다"는 결론에는 과제 풀이 PTC에 유리하게 짜였다는 조건이 붙는다.
비교 baseline 점수의 상당수는 다른 논문에서 가져온 값이고, baseline 이 어떤 backbone 에서 학습됐는지는 본문에서 확인 못 했다.
과제가 215개라 적어 보이지만 과제 하나를 만드는 데 Claude Opus 4.6 에이전트의 구성·수리·감사와 DeepSeek V4 Pro 궤적 32개(크기 4개 × 8개)가 들어가고, 강화학습은 H200 64장 규모다.
과제 수가 적다고 비용까지 적은 건 아니다.

ReAct: 모델이 생각(Thought), 행동(Action), 관측(Observation)을 번갈아 내며 한 턴에 도구 하나를 부르고 그 결과를 컨텍스트에 붙여 다음 턴을 이어 가는 에이전트 루프
