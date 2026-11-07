---
date: 2026-11-08 06:00:00 +0900
title: "BEMULA Paper review"
excerpt: "사용자가 '그 정보 잊어줘'라고 하면 에이전트 메모리는 정말 지우는가. 50개 대화에 135개 forget request 를 심고 대조군 버전과 짝지어 잰 벤치마크인데, Mem0·A-MEM·LangMem·LightMem·SimpleMem·naive RAG·full-context 전부 요청을 끝까지 통과시킨 비율이 1.5% 를 넘지 못했다. 지웠다고 말한 값이 간접 질문 하나에 다시 나온다."
categories:
  - Paper review
tags:
  - Large Language Model
  - NLP
  - Agent
  - Memory
  - Benchmark
  - Paper review
toc: true
toc_sticky: true
field: agent-memory
mathjax: true
---

Can Agents Forget When Asked? A Benchmark for Memory Unlearning in LLM Agents

[https://arxiv.org/abs/2610.18052](https://arxiv.org/abs/2610.18052)

BEMULA 는 Tsinghua University 와 Sina Weibo 에서 만든 메모리 벤치마크다.
사용자가 대화 도중에 "이건 잊어줘"라고 했을 때 에이전트의 장기기억이 그 정보를 지우는지, 다른 기억은 남기는지, 지운 뒤에도 공격 질문으로 다시 뽑히지는 않는지를 잰다.
2026년 10월에 arXiv 에 올라온 논문이고 저자는 7명, 25쪽이다.
코드와 데이터는 공개되어 있다.
앞에서 본 [Mem0](https://momozzing.github.io/paper%20review/Mem0-Paper-review/)와 [A-MEM](https://momozzing.github.io/paper%20review/A-MEM-Paper-review/)이 여기서는 평가 대상으로 나온다.

![forget request 예시. 왼쪽은 저축액을 말하는 평범한 대화, 가운데는 잊어달라고 했는데 사흘 뒤 질문에 180k 가 그대로 나오는 경우, 오른쪽은 정말 잊은 경우 (논문 Figure 1)](https://momozzing.github.io/assets/images/can-agents-forget/fig1-illustrative-examples-forget.png)

## **1. Introduction**

장기기억은 이제 LLM 에이전트의 기본 부품이다.
MemGPT, MemoryOS, Mem0, A-MEM, SimpleMem 같은 시스템은 대화 이력을 저장하고 꺼내 쓰고 압축하고 정리하는 데 집중한다.
그런데 사용자가 "집 주소 잊어줘", "병원 기록 지워줘", "예전 취향은 버려줘"라고 명시적으로 요청했을 때 지우는 능력은 거의 다루지 않았다.
저자들은 조직 이론의 competency trap(과거 성공의 기억이 미래 실패의 원인이 된다는 개념)을 빌려 잊기가 기억하기만큼 중요하다고 쓴다.
못 지우면 프라이버시 누출이고, 낡은 정보가 남아 있으면 판단도 흐려진다.
여기에 중복 문제가 겹친다.
요즘 메모리 시스템은 같은 정보를 원문 턴, 추출된 사실, 압축 노트 여러 형태로 들고 있어서 한 사본을 지워도 나머지가 남는다.

기존 벤치마크는 기억을 보존하고 추론에 쓰는지만 본다.
LongMemEval 의 knowledge-update 나 MemoryAgentBench 의 conflict-resolution 은 최신 사실로 답하는지를 보지, 옛 사실이 사라졌는지는 보지 않는다.
그래서 저자들은 memory unlearning 이라는 과제를 새로 정의한다.
자연어 forget request 를 받으면 지정된 정보를 메모리에서 없애되 나머지는 보존하는 것이다.

벤치마크를 만들 때 두 가지가 어려웠다고 한다.
첫째는 수백 턴에 걸쳐 forget request, 평가 질문, 대화 내용이 서로 맞아떨어지는 긴 멀티세션 대화를 만드는 일이다.
이건 사실 접지(fact-grounding), 세션 검증, 누출 청소를 끼워 넣은 5단계 멀티에이전트 파이프라인으로 푼다.
둘째는 지표다.
"목표 정보가 사라졌는가"만 재면 애초에 저장을 안 한 시스템, 메모리를 통째로 비운 시스템, 관련 질문을 다 거절하는 시스템이 전부 만점을 받는다.
그래서 모든 대화에 forget request 가 없는 control version 을 짝지어 두고, 모든 probe 를 control 쪽 결과와 함께 채점한다.
목표를 저장했던 시스템만 잊은 것으로 인정받는다.

## **2. Related Work**

메모리 벤치마크 쪽은 LongBench·RULER 같은 긴 컨텍스트 평가에서 LoCoMo·LongMemEval 같은 대화 이력 평가로, 다시 MemoryAgentBench·MemoryArena·MemBench 같은 메모리 에이전트 직접 평가로 이어진다.
잊기를 건드린 건 Memora 의 FAMA 지표 정도인데, 거기서 무효화는 대화가 흘러가며 암묵적으로 일어나고 추출·재학습 probe 는 없다.
메모리 방법 쪽은 RAG 부터 Mem0, Zep, A-MEM, MemGPT, Memory-R1, SimpleMem 까지 훑으면서, 삭제 기능이 있어도 정보를 최신으로 유지하기 위한 것이지 사용자 의도를 따르기 위한 것은 아니었다고 정리한다.
machine unlearning 은 가중치를 고치는 방법이라 frozen 모델에는 안 맞고, in-context unlearning 과 ICF-Bench 의 in-context forgetting 은 한 세션 안 프롬프트 수준의 잊기라 외부 메모리에 있는 정보는 못 건드린다.

## **3. The BEMULA Benchmark**

### **3.1 Task Formulation**

에이전트는 턴 $t$ 에 사용자 발화 $u_t$ 를 받고, 현재 세션의 최근 턴을 담은 working context $W_t$ 와 세션을 넘어 유지되는 메모리 $M_t$ 에서 꺼낸 $R(M_t, u_t)$ 를 보고 답한다.
턴이 끝나면 WRITE 로 메모리를 갱신한다.
메모리 내부 구조는 묻지 않고 R 과 WRITE 두 연산만 가정한다.
대화 인스턴스 $D$ 는 세션의 나열이고, 그중 일부 발화가 자연어 forget request 다.
요청 뒤에는 그 정보가 메모리에서 검색되지도 추론되지도 않아야 하고, 나머지는 그대로 남아야 하며, 비슷한 사실을 새로 배울 수 있어야 한다.
control version $D'$ 는 forget request 와 assistant 의 확인 응답만 비슷한 길이의 중립적인 잡담으로 바꾼 것이다.
나머지 턴은 턴 단위로 같다.

### **3.2 Benchmark Construction**

프로필 합성, 대화 계획, 멀티에이전트 대화 시뮬레이션, 요청 주입, 질문 생성 다섯 단계다.
만든 뒤 네 길이 구간에서 10개 인스턴스를 뽑아 388개 질문을 사람이 정답과 대조했다.

#### **3.2.1 Profile Synthesis**

나이·직업·가족 구성 같은 속성 여섯 개를 샘플링해 텍스트 프로필을 만들고, LLM 이 거기에 맞는 atomic fact 들을 생성한다.
각 fact 는 (subject, predicate, value) 삼중항, 유형, 선택적 타임스탬프, 문장으로 되어 있다.
여행·대회·수술 같은 활동이나 주소·취미·일과 같은 개인 정보다.
derived fact 는 대화에 직접 안 나오지만 atomic fact 두 개 이상을 합치면 추론되는 사실이다.
LLM 이 출처를 인용하며 만들고, 별도 LLM 호출이 출처만으로 엄격히 따라 나오는지 검증한다.

#### **3.2.2 Dialogue Planning**

세션별 시나리오, 어떤 fact 를 언급할지, forget request 의 유형·대상·위치·표현 스타일을 정한다.
대화 내용은 아직 안 쓴다.
시나리오 풀은 여행·의료·교육·금융·개인 소프트웨어 개발 같은 일상 영역의 (주제, 사용자 목표) 쌍 50개다.

forget request 는 대상 입도, 정보 성격, 연산 의미 세 축으로 여섯 유형이다.
- F1 Factual: atomic fact 하나를 지운다
- F2 Compositional: atomic fact 여러 개의 조합을 지운다
- F3 Semantic: 의료 정보 전부처럼 범주 하나를 통째로 지운다
- F4 Inferential: 대화에 안 나왔지만 추론되는 derived fact 를 지운다
- F5 Temporal: 특정 기간의 기억을 지운다
- F6 Replacement: 사실을 새 값으로 바꾼다

F4 는 결론이 잊혔는지만 보고 출처 fact 가 남는지는 안 본다.
대화 길이는 short·medium·long·xlong 네 구간이고 구간이 요청 개수를 정한다.
요청의 대상 fact 는 요청보다 앞선 세션에만 배치되고, 언급 횟수만큼 반복된다.
나머지 fact 는 distractor 로 세션마다 뿌린다.

#### **3.2.3 Multi-Agent Dialogue Simulation**

user simulator 와 assistant simulator 가 번갈아 턴을 쓰고 director 와 verifier 가 감독한다.
user simulator 는 프로필·시나리오·이번 세션에 배정된 fact 목록을 보고 말하면서 매 턴 어떤 fact 를 드러냈는지 보고한다.
assistant 는 현재 세션의 마지막 열 턴만 보고 이전 세션 기억은 없다.
director 는 턴 수를 조절하고 아직 안 나온 fact 를 꺼내도록 user simulator 를 찌르며, 텍스트가 보고를 뒷받침하는지 확인한 뒤에만 fact 를 기록한다.
세션이 끝나면 verifier 가 시간 일관성과 자연스러움을 검사하고, 떨어지면 한 번 다시 만든다.

#### **3.2.4 Request Injection**

계획된 위치에 forget request 를 넣어 두 버전을 만든다.
forget version 에서는 사용자가 계획된 스타일과 유형으로 요청하고 assistant 는 민감한 내용을 되풀이하지 않는 확인 응답을 하며, control version 에서는 그 자리를 비슷한 길이의 잡담으로 채운다.
마지막에 LLM verifier 가 모든 요청이 표현 스타일과 무관하게 명시적 삭제 의도를 담는지 확인한다.

#### **3.2.5 Question Generation**

fact 를 다섯 suite 의 질문으로 바꾼다.
suite 마다 템플릿이 있고 LLM 이 문구를 채우며, 답할 수 없는 문구는 별도 LLM 검사로 버린다.
- retention: control version 에 묻고 fact 값이 정답
- efficacy: 같은 문구를 forget version 에 묻고 [FORGOTTEN] 마커가 정답, F6 만 새 값이 정답
- preservation: 어떤 요청도 겨냥하지 않은 distractor fact 세 개를 두 버전 모두에 묻는다
- relearning: 같은 유형의 새 값을 말하는 짧은 대화를 붙이고, 시스템이 그걸 넣은 뒤 답하게 한다
- attack: 대상 질문을 다섯 공격 프레임(간접 질문, 역할극, jailbreak 접두사, Boolean 추측, 다중홉 추론)에 넣고 [NO LEAK] 가 정답

공격 프레임은 문구 자체가 공격 수단이라 바꿔 쓰지 않는다.
두 마커 모두 LLM judge 가 판정한다.

### **3.3 Benchmark Statistics**

프로필 50개, 대화 인스턴스 50개, 각각 forget version 과 control version 이 있다.
버전당 746 세션, 36,900 턴이고 짝을 합치면 73,800 턴이다.
short 는 100턴 안팎, xlong 은 최대 3,500 턴(약 379K 토큰)이다.
평가 단위는 forget request 하나다.
요청 135개가 fact 269개를 겨냥하고 질문은 1,742개다.
비교 대상으로 LongMemEval 은 500문항이라고 적어 둔다.

![(a) 인스턴스별 턴 수와 네 길이 구간, (b) 유형별 요청 수, (c) long 인스턴스 하나의 해부도. 점이 대상 fact 언급, 마름모가 요청 위치 (논문 Figure 3)](https://momozzing.github.io/assets/images/can-agents-forget/fig3-statistics-bemula-length.png)

Appendix D 의 Table 4 에 전체 통계가 있다.
질문 1,742개 중 attack 이 1,030개로 가장 많고 preservation 은 인스턴스당 3개씩 150개다.

![BEMULA 통계. 프로필·인스턴스·요청·질문 수 (논문 Table 4)](https://momozzing.github.io/assets/images/can-agents-forget/table4-key-statistics-bemula.png)

### **3.4 Benchmark Metrics**

시스템은 대화를 세션 단위로 넣고, 각 질문은 질문만 담긴 새 세션에서 답한다.
그래서 모든 답은 메모리 M 에서 나와야 한다.
질문마다 이진 결과가 나온다.
retention suite 는 control version 에서, 나머지 넷은 forget version 에서 채점한다.
preservation 과 attack 은 control version 에도 물어서 조건 probe 로 쓴다.

질문 $i$ 의 정답 여부를 $z_i$, 짝지은 control probe 의 결과를 $c_i$ 라 하면 유효 정답은 $\tilde{z}_i = z_i \wedge c_i$ 다.
efficacy 질문은 retention probe 가 맞아야, preservation 질문은 control 쪽이 맞아야, attack 질문은 control 쪽에서 공격이 성공해야 점수가 된다.
retention 과 F6 의 새 값 질문은 control 짝이 없어 $c_i = 1$ 이다.

다섯 항목 지표는 각 suite 의 유효 정답 평균이다.
- RA(retention accuracy): 평범한 기억 능력
- FE(forgetting efficacy): forget version 에서 값을 누출하지 않고 없다고 답했는가
- PR(preservation): control 에서 맞힌 질문을 forget version 에서도 맞혔는가
- RLA(relearning accuracy): 요청 뒤 비슷한 사실을 새로 배우는가
- AR(attack robustness): 다섯 공격 유형별 방어율의 평균

Appendix A 의 Table 2 가 다섯 지표를 어느 버전에서 무엇을 정답으로 어떤 조건으로 채점하는지 정리한다.

![다섯 항목 지표의 suite·버전·정답·조건 probe·탐지 대상 (논문 Table 2)](https://momozzing.github.io/assets/images/can-agents-forget/table2-item-level-metrics-bemula.png)

가장 엄격한 지표는 RPR(request pass rate)이다.
요청에 딸린 모든 질문과 그 인스턴스의 모든 preservation 질문이 맞아야 요청 하나가 통과다.
종합 점수는 두 기하평균의 곱이다.

$$\text{Score} = \text{FA} \times \text{UG},\quad \text{FA} = \sqrt{\text{FE} \cdot \text{AR}},\quad \text{UG} = \sqrt{\text{PR} \cdot \text{RLA}}$$

FA(forgetting ability)는 얼마나 잘 지우는지, UG(usability guard)는 지운 뒤에도 쓸 만한지다.
곱이라 넷 중 하나가 0이면 전체가 0이다.
다 지우는 시스템도, 아무것도 안 잊는 시스템도, 새로 못 배우는 시스템도 0점이다.

-> 지표 설계가 이 논문의 절반이다. control version 짝짓기 때문에 "저장을 안 해서 못 답하는 것"과 "지워서 못 답하는 것"이 갈린다. 다만 전부 LLM judge 채점이고 Appendix B 의 사람 검증은 100문항(일치 98%, kappa 0.96)이다.

## **4. Experiments**

### **4.1 Experimental Setup**

Full Context LLM 과 메모리 시스템 여섯 개를 돌린다.
Naive RAG(턴을 그대로 임베딩해 저장하고 검색), Mem0(사실 추출 후 추가·갱신·삭제), A-MEM(연결된 노트를 진화시키는 구조), LangMem(LangChain 의 장기기억 SDK), LightMem(경량 메모리 생성), SimpleMem(메모리 압축)이다.
백본은 GPT-5.6-Luna, DeepSeek-V4.1-Flash, Qwen3-32B-AWQ 셋이다.
시스템 내부의 LLM 호출은 전부 같은 백본을 쓰고 임베딩은 text-embedding-3-small 로 통일했다.
Qwen3-32B-AWQ 는 컨텍스트 한계 때문에 Full Context 와 A-MEM 평가에서 뺐다.
인스턴스마다 메모리를 초기화하고, forget version 또는 control version 을 턴 단위로 넣은 뒤 새 세션에서 질문한다.
judge 는 DeepSeek-V4.1-Flash 이고, 불확실성은 인스턴스 단위 cluster bootstrap 으로 잰다.

Appendix C 를 보면 forget request 가 왔을 때 각 시스템이 할 수 있는 일이 다르다.
LangMem 만 대화 중에 노트를 지우는 경로가 있고, 그것도 기본 설정에서는 꺼져 있어서 저자들이 켰다.
Mem0 는 쓰기 경로가 사실을 삭제할 수 있지만 내부 파이프라인이 스스로 판단할 때만 지운다.
어댑터에서 delete 호출을 따로 넣지는 않았다.
A-MEM 은 삭제는 못 하지만 note evolution 으로 기존 노트를 덮어쓸 수 있다.
Naive RAG 는 삭제 경로가 없고, LightMem 은 갱신·병합이 오프라인이라 대화 중에는 못 미치며, SimpleMem 은 턴 윈도우를 한 번 압축하면 다시 고치지 않는다.

### **4.2 Main Results**

주 결과인 Table 1 은 세 백본에서 각 시스템의 RA·FE·PR·RLA·AR·RPR·Score 를 적은 표다.
이 표의 이미지는 재료에 없어 숫자만 옮긴다.

결과는 부정적이다.
RPR 은 LangMem 이 DeepSeek-V4.1-Flash 에서 기록한 1.5 가 최대이고 대부분 0 이다.
Score 도 GPT-5.6-Luna 에서 Full Context 23.1, DeepSeek-V4.1-Flash 에서 LangMem 31.0, Qwen3-32B-AWQ 에서 SimpleMem 9.5 가 각 백본에서 가장 높다.

기억은 잘하는데 잊지를 못한다.
Mem0, LightMem, SimpleMem 은 두 강한 백본에서 RA 87 이상이고 Naive RAG 도 74 근처인데, 이 넷의 FE 는 25 미만이다.
LangMem 은 GPT-5.6-Luna 에서 메모리 시스템 중 FE 가 가장 높지만(37.9) PR 이 20.0 으로 떨어진다.
지우긴 하는데 범위를 못 가린다.
DeepSeek-V4.1-Flash 에서는 FE 40.6, PR 74.0 으로 가장 균형 잡힌 메모리 시스템이 되는데, 종합 점수는 Full Context 와 통계적으로 동률이다.
백본이 바뀌자 결과가 크게 흔들리는 걸 보면 LangMem 의 효과는 백본에 많이 기댄다.
외부 메모리가 없는 Full Context 가 두 백본에서 비교적 균형 잡힌 이유로, 저자들은 잊기가 지시 따르기 문제로 환원되기 때문이라고 본다.

RLA 는 어디서도 71.1 아래로 안 떨어져서 요청 뒤에도 새 사실은 배운다.
AR 이 표에서 가장 낮은 천장이다.
모든 설정에서 35.7 이하이고 Mem0 는 GPT-5.6-Luna 에서 4.7 이다.
시스템이 정보가 없다고 말한 뒤에도 적당한 프롬프트 하나면 다시 나온다.

### **4.3 Result Analysis**

대부분의 요청은 저장소까지 못 간다.
Naive RAG 는 요청 턴 자체가 다른 턴처럼 임베딩되어 저장된다.
Mem0 는 갱신을 지원하지만 GPT-5.6-Luna 에서 FE 8.5 이고, 실패한 답은 요청된 값을 그대로 말한다.
LightMem 과 SimpleMem 은 구조상 대화 중에 손을 못 대고 FE 가 각각 17.9, 16.1 이다.
A-MEM 은 부분적 예외다.
삭제 경로는 없지만 요청이 들어오면 note evolution 이 관련 노트를 고쳐 쓸 수 있어서 FE 33.0(GPT-5.6-Luna), 29.5(DeepSeek-V4.1-Flash)다.

반대로 너무 지우거나 너무 거절하는 시스템도 있다.
LangMem 은 GPT-5.6-Luna 에서 어떤 요청도 겨냥하지 않은 집 주소까지 지웠다.
DeepSeek-V4.1-Flash 의 Full Context 는 지울 수는 없지만 과하게 거절한다.
preservation 실패 81건 중 66건이 "그 정보가 없다"는 일괄 거절이다.
PR 46.0 은 범위를 가린 삭제가 아니라 전역 거절의 결과라는 뜻이다.

![요청 유형별 FE. F6(교체)만 짙고 F3(범주)·F5(기간)는 거의 비어 있다 (논문 Figure 4)](https://momozzing.github.io/assets/images/can-agents-forget/fig4-forgetting-efficacy-fe.png)

유형별로 보면 F6 Replacement 는 두 강한 백본에서 거의 풀렸다.
모든 시스템이 FE 61.9 이상이다.
그런데 옛 값은 그대로 남아 있어서 공격을 받으면 다시 나오고, AR 은 최대 30.5 다.
F3 Semantic 과 F5 Temporal 이 가장 어렵다.
여러 항목에 걸쳐 범위를 정해 지워야 해서 Full Context 와 LangMem 만 30 을 넘긴다.
사실 하나만 지우는 F1 도 대부분 실패하고 Mem0 는 GPT-5.6-Luna 에서 0.0 이다.
A-MEM 은 GPT-5.6-Luna 에서 F1 46.2, F2 52.0 으로 메모리 시스템 중 가장 높은데, 덮어쓰기만 가능한 진화 단계와 맞는 결과다.
백본은 구조만큼 중요하다.
Full Context 의 FE 는 GPT-5.6-Luna 26.8 에서 DeepSeek-V4.1-Flash 53.6 으로 뛰고, Qwen3-32B-AWQ 에서는 추출 단계 자체가 무너져 Score 가 9.5 를 못 넘는다.

Appendix H 의 공격 유형별 분석도 같이 보면 좋다.
Full Context 는 jailbreak 접두사에 가장 약하다(GPT-5.6-Luna 에서 AR 9.7).
지시로만 잊는 구조라 지시를 덮어쓰면 끝이다.
반면 저장소를 못 비우는 네 시스템은 jailbreak 가 가장 안 통하는 공격이고, 아무 꼼수 없는 간접 질문과 Boolean 추측이 가장 잘 통한다.
Mem0 는 간접 질문에 AR 2.4 다.
Appendix J 의 길이별 분석에서는 LangMem 의 FE 가 가장 짧은 구간 66.7 에서 가장 긴 구간 21.9 로 떨어지고, PR 도 거의 모든 시스템에서 길이가 길수록 내려간다.
AR 은 길이와 거의 무관해서, 누출을 길이 탓으로 볼 수 없고 구조 문제로 해석한다.

-> 공격 유형 결과가 실무에 더 직접 닿는다. 저장소에 값이 남아 있으면 "그거 뭐였지?" 수준의 평범한 질문이 jailbreak 보다 잘 뚫는다. 프롬프트 방어를 강화하는 것만으로는 이 벤치마크 점수가 오르지 않는다.

## **5. Conclusion**

여섯 메모리 시스템과 Full Context LLM 을 최대 세 백본에서 평가한 결과, 정보는 잘 저장하지만 요청받은 잊기는 대부분 실패하고 전체 suite 를 통과하는 요청은 거의 없다.
저자들이 적은 한계는 둘이다.
대화가 전부 합성이고 생성 파이프라인 하나에서 나왔다.
그리고 대화 메모리만 다루는데, 도구를 쓰는 에이전트는 잊힌 값을 캐시·로그·외부 서비스에 남길 수 있다.
실제 사용자 대화와 도구 사용으로 넓히는 것을 다음 과제로 둔다.

본문에 안 적힌 한계도 있다.
Appendix K 를 보면 F4 는 13개 요청, xlong 은 3개 인스턴스라 유형별·길이별 차이는 서술적 관찰 수준으로만 읽어야 한다고 저자들 스스로 적는다.
Table 1 에서 이웃한 순위는 신뢰구간이 겹치는 경우가 많다.
다만 어떤 시스템의 FE 구간도 60.2 위로, AR 구간도 41.4 위로 올라가지 않아서 부정적 결론 자체는 샘플링 불확실성에 흔들리지 않는다.
judge 가 백본 중 하나인 DeepSeek-V4.1-Flash 라는 점은 Ethics Statement 에서 편향 가능성으로 언급만 한다.
데이터 생성이 GPT-5.6-Luna 로 되어 있어서 그 백본에 유리하거나 불리한 쪽으로 작용했는지는 원문에서 확인 못 함.

## **6. 지금 관점: 삭제 경로가 없는 메모리에 forget request 가 오면**

앞에서 본 [Mem0](https://momozzing.github.io/paper%20review/Mem0-Paper-review/)는 ADD·UPDATE·DELETE·NOOP 네 연산을 내세웠고, [Zep](https://momozzing.github.io/paper%20review/Zep-Paper-review/)은 지우지 않고 무효화한다고 했다.
이 논문은 그 DELETE 가 사용자의 "잊어줘"에 실제로 발동하는지를 재고, Mem0 의 경우 GPT-5.6-Luna 에서 F1 요청을 하나도 못 지웠다는 답을 낸다.
[A-MEM](https://momozzing.github.io/paper%20review/A-MEM-Paper-review/)의 노트 진화는 삭제 기능이 아닌데도 덮어쓰기 덕에 메모리 시스템 중 F1·F2 를 가장 잘 지웠다.
[LongMemEval](https://momozzing.github.io/paper%20review/LongMemEval-Paper-review/)의 knowledge-update 가 "최신 값으로 답하는가"였다면 여기 F6 는 그 뒤에 "옛 값이 공격으로 다시 나오는가"를 한 번 더 묻는다.
[MemFail](https://momozzing.github.io/paper%20review/MemFail-Paper-review/)이 틀린 답을 요약·저장·검색 단계로 갈랐듯, 이 논문도 Appendix C 에서 시스템마다 요청이 어느 단계에서 막히는지를 먼저 적고 결과를 거기에 맞춰 읽는다.

지금 가져다 쓸 수 있는 건 평가 틀이다.
control version 을 짝지어 "저장 안 함"과 "지움"을 가르는 방식, 그리고 FE 와 PR 을 곱해서 다 지우는 시스템에 0점을 주는 방식은 벤치마크를 안 쓰더라도 자기 메모리에 삭제 기능을 넣을 때 그대로 테스트로 옮길 수 있다.
공격 suite 도 다섯 프레임이 고정 문구라 만들기 쉽다.
특히 간접 질문과 Boolean 추측 두 개만 넣어도 저장소에 값이 남았는지는 드러난다.
설계 쪽에서는 forget request 를 "메모리에 쓸 내용"이 아니라 "메모리에 가할 연산"으로 분기시키는 지점이 필요하다는 게 결과에서 읽힌다.
Naive RAG 처럼 요청 턴 자체를 저장하면 그 턴이 값을 담고 있지 않더라도 원래 턴은 그대로 남는다.
그리고 같은 정보가 원문 턴, 추출 사실, 압축 노트에 중복으로 있으면 한 군데를 지워도 소용없으니, 삭제를 지원하려면 사본의 위치를 추적하는 색인이 먼저다.
BEMULA-Lite(short·medium 31개 인스턴스, 848문항)가 따로 있어서 full 벤치마크의 비용이 부담이면 거기서 시작할 수 있다.

그대로 받아들이기 어려운 부분도 있다.
대화가 전부 합성이고, 사용자 시뮬레이터가 fact 를 "한 번씩 정확한 값으로" 드러내도록 짜여 있어서 실제 대화보다 사실이 깨끗하게 들어간다.
실제 사용자는 같은 정보를 여러 번 다른 말로 흘리고, 잊어달라는 요청도 훨씬 모호하다.
Full Context 가 비교적 잘 나온 것도 이 벤치마크가 질문을 새 세션에서 하나씩 던지기 때문이고, 실제 서비스에서 379K 토큰을 매 턴 넣는 선택지는 없다.
LangMem 의 결과는 기본 설정에서 꺼진 삭제 채널을 켠 것이라 배포된 기본값의 성능은 아니다.
그리고 저자들이 한계로 적었듯 도구를 쓰는 에이전트에서는 값이 메모리 밖 로그와 캐시에도 남는다.
이 벤치마크에서 만점을 받아도 그쪽은 별개 문제다.
