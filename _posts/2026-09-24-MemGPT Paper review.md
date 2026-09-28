---
date: 2026-09-24 12:00:00 +0900
title: "MemGPT Paper review"
excerpt: "컨텍스트 창을 물리 메모리로, 외부 저장소를 디스크로 본다. OS의 가상 메모리 페이징을 LLM에 옮긴 2023년 논문이자 이 계열의 출발점."
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
---

MemGPT: Towards LLMs as Operating Systems

[https://arxiv.org/abs/2310.08560](https://arxiv.org/abs/2310.08560)

MemGPT는 UC Berkeley에서 만든 LLM 메모리 관리 시스템이다.

OS가 메모리와 디스크 사이를 페이징하듯이, LLM이 컨텍스트 창과 외부 저장소 사이에서 정보를 옮기게 한다.

2023년 10월 12일에 나왔고 2024년 2월 12일에 v2가 올라왔다. 저자 7명에 13쪽이고, 학회 발표 없이 arXiv에만 있다.

앞의 네 리뷰에서 계속 베이스라인으로 나온 시스템이다. Zep이 DMR에서 비교한 상대였고, Mem0와 A-MEM 표에도 나왔고, ReFind 표에서는 28.0이었다. 시간순으로는 이 계열의 출발점이라 여기서 읽어봤다.

2023년 논문이라 지금 기준으로는 오래되었다. 그래도 이후 메모리 시스템들이 쓰는 용어가 여기서 많이 나왔다.

좀 더 자세히 알아보자.

## **1. Introduction**

introduction 부분을 보면 문제는 단순하다. 고정 길이 컨텍스트 창 때문에 긴 대화나 긴 문서를 다루기 어렵다.

2023년 기준으로 많이 쓰던 오픈소스 LLM은 수십 번 주고받거나 짧은 문서 하나만 넘어가도 최대 입력 길이를 넘었다고 한다.

컨텍스트를 그냥 늘리면 되는 것도 아니라고 한다. 컨텍스트가 큰 모델은 어텐션이 고르게 가지 않는다. 처음과 끝은 잘 기억하고 중간은 잘 못 한다(lost in the middle).

그래서 가상 컨텍스트 관리(virtual context management)를 제안한다.

운영체제의 계층적 메모리에서 아이디어를 가져왔다고 한다. OS는 물리 메모리와 디스크 사이를 페이징해서 메모리가 더 큰 것처럼 보이게 한다.

LLM이 자기 컨텍스트에 뭘 넣을지 스스로 관리하는 'LLM OS'를 만든다는 것이다.

## **2. 두 계층, 네 저장소**

![MemGPT 계층 메모리 구조 (논문 Figure 3)](https://momozzing.github.io/assets/images/memgpt/fig3-hierarchy.png)

메모리를 둘로 나눈다.

- Main context : LLM 프롬프트 토큰. 물리 메모리/RAM에 해당한다. 여기 있는 건 in-context라서 추론할 때 바로 쓸 수 있다
- External context : 컨텍스트 창 밖의 정보. 디스크에 해당한다. 추론에 쓰려면 main context로 직접 옮겨와야 한다

### **2.1 Main context**

프롬프트 토큰을 연속된 세 구역으로 나눈다.

1. System Instructions : 읽기 전용(정적). MemGPT 제어 흐름 정보
2. Working Context : 읽기·쓰기, 함수로 씀. 에이전트가 직접 관리하는 사실
3. FIFO Queue : 읽기·쓰기, 큐 매니저가 씀. 최근 메시지, 시스템 경고, 함수 입출력

FIFO 큐의 첫 인덱스에는 큐에서 밀려난 메시지들을 재귀적으로 요약한 시스템 메시지가 들어간다. 밀려나도 흔적은 남기는 것이다.

### **2.2 External context**

- Recall Storage : MemGPT 메시지 DB. 큐 매니저가 쓰고 함수로 읽는다
- Archival Storage : 함수로 읽고 함수로 쓴다

### **2.3 Queue Manager**

새 메시지가 오면 큐 매니저가 FIFO 큐에 붙이고, 프롬프트 토큰을 이어 붙여서 LLM 추론을 돌린다.

들어온 메시지랑 생성된 출력은 둘 다 recall storage에 쓴다. 함수 호출로 recall storage에서 메시지를 꺼내면 큐 뒤에 다시 붙여서 컨텍스트 창에 넣는다.

컨텍스트가 넘칠 때 처리하는 것도 큐 매니저가 한다.

### **2.4 함수 연쇄와 하트비트**

LLM 출력을 함수 호출로 해석한다. 함수 실행기가 main context와 external context 사이로 데이터를 옮긴다.

여기서 재밌는 게 하나 있다.

LLM이 출력에 `request_heartbeat=true`라는 인자를 넣으면 바로 다음 추론을 요청할 수 있다고 한다. 이렇게 함수를 연쇄해서 여러 단계 검색을 한다.

이 플래그가 없으면(yield) 다음 외부 이벤트(사용자 메시지나 예약된 인터럽트)가 올 때까지 LLM을 돌리지 않는다.

-> ReAct 루프랑 거의 같은 구조다. 앞에서 본 [ReFind](https://momozzing.github.io/paper%20review/ReFind-Paper-review/)에서는 반복 검색이 M 세트에서 20.4점을 차지했는데, MemGPT도 2023년에 이미 반복 검색을 하고 있었다. 그런데 ReFind 표에서 MemGPT가 28.0에 그친 건 검색 인터페이스가 세션·시간·중복을 몰라서인 것 같다.

### **2.5 메모리 압력 경고**

Figure 1 예시를 보면 동작 방식이 잘 보인다.

컨텍스트 공간이 부족해지면 시스템 경고(System Alert: Memory Pressure)가 LLM에 전달되고, LLM이 `working_context.append("Birthday is February 7")` 같은 함수를 직접 호출해서 영구 메모리에 쓴다.

Figure 4는 갱신하는 예시다. 사용자가 헤어졌다고 하니 `working_context.replace("Boyfriend named James", "Ex-boyfriend named James")`를 호출한다.

replace로 덮어쓴다. 앞에서 본 Mem0의 UPDATE·DELETE는 이쪽이고, Zep은 무효화로 처리해서 여기서 갈린다.

## **3. 평가**

### **3.1 DMR**

이전 대화(세션 1~5)에서 나온 주제에 대해 구체적으로 물어본다.

| 모델 | 정확도 | ROUGE-L |
|---|---:|---:|
| GPT-3.5 Turbo | 38.7% | 0.394 |
| + MemGPT | 66.9% | 0.629 |
| GPT-4 | 32.1% | 0.296 |
| + MemGPT | 92.5% | 0.814 |
| GPT-4 Turbo | 35.3% | 0.359 |
| + MemGPT | 93.4% | 0.827 |

고정 컨텍스트 베이스라인보다 훨씬 높다. GPT-4에서 32.1% → 92.5%다.

-> 그런데 앞에서 본 [Zep](https://momozzing.github.io/paper%20review/Zep-Paper-review/)에서 잰 full-conversation 베이스라인이 94.4%였다. MemGPT의 93.4%는 대화를 통째로 넣은 것보다 낮다. MemGPT가 비교한 건 잘린 컨텍스트였지 전체 컨텍스트가 아니었다.

### **3.2 대화 오프너**

에이전트가 먼저 말을 거는 품질을 본다. 페르소나 라벨과의 유사도(SIM-1/3), 사람이 쓴 오프너와의 유사도(SIM-H)로 잰다.

| 방법 | SIM-1 | SIM-3 | SIM-H |
|---|---:|---:|---:|
| Human | 0.800 | 0.800 | 1.000 |
| GPT-3.5 Turbo | 0.830 | 0.812 | 0.817 |
| GPT-4 | 0.868 | 0.843 | 0.773 |
| GPT-4 Turbo | 0.857 | 0.828 | 0.767 |

사람이 쓴 오프너보다 높게 나온다(SIM-1 0.868 vs 0.800). working context에 정보를 저장해두는 게 좋은 오프너를 만드는 데 중요하다고 한다.

### **3.3 문서 분석과 중첩 KV 검색**

문서 QA에서는 컨텍스트 한계를 훨씬 넘는 문서를 처리한다.

그리고 중첩 key-value 검색 과제를 새로 만들었다. 여러 데이터 출처에 걸친 정보를 찾아야 하는 다중홉 검색을 본다.

Wikipedia 2천만 문서 임베딩 데이터셋도 같이 공개했다.

## **4. 지금 관점: 은유가 남긴 것**

벤치마크 숫자보다는 여기서 나온 용어들이 이후에 계속 쓰인다.

- main / external context : 거의 모든 시스템의 기본 구도
- working context : Mem0의 추출된 사실, A-MEM의 노트
- recall storage : Zep의 episode subgraph
- archival storage : ReFind의 원본 아카이브
- 함수로 자기 메모리 관리 : Mem0의 tool call 4연산
- `request_heartbeat` 연쇄 : ReFind의 ReAct 반복 검색
- 메모리 압력 경고 : 모든 압축 트리거

앞에서 본 [Rate-Distortion](https://momozzing.github.io/paper%20review/Rate-Distortion-Memory-Compaction-Paper-review/)에서 가역 방식 예로 "Quest와 MemGPT-archival 방식"을 들었는데, 여기서 온 것이다. archival storage에 원본을 두고 질의할 때 꺼내니까 P-rev를 만족한다.

그런데 MemGPT 안에는 가역과 비가역이 섞여 있다.

- 가역 : recall storage, archival storage. 원본이 남는다
- 비가역 : FIFO 큐에서 밀려난 메시지의 재귀 요약, working context의 `replace`

재귀 요약은 큐에서 밀린 메시지가 요약되고, 그 요약이 또 요약된다. Rate-Distortion 실험 2에서 본 반복 압축 오차 누적이 이 부분이다.

다만 MemGPT는 recall storage에 원본을 남기니까 요약에서 빠진 걸 검색으로 다시 찾을 수는 있다.

-> 설계는 되어 있는데, 에이전트가 검색을 안 하면 소용없는 거 아닌가??

-> 세 구역 나누는 건 지금 봐도 쓸 만한 것 같다. 시스템 지시(정적) / 에이전트가 관리하는 사실(함수로 씀) / 최근 메시지(자동으로 씀). 쓰는 주체를 나누면 프롬프트 관리가 깔끔해진다.

-> 메모리 압력을 이벤트로 알려주는 것도 괜찮아 보인다. 토큰이 차면 그냥 자르는 게 아니라 LLM한테 알려서 뭘 남길지 판단하게 하는 것.

-> 큐 매니저가 들어온 메시지랑 생성된 출력을 둘 다 recall storage에 쓰는 것도 중요해 보인다. 잘라낸 히스토리를 버리지 않고 검색할 수 있는 곳에 두면 되돌릴 수 있다.

한계도 있다.

1. 전부 LLM 판단에 달려 있다. 언제 저장할지, 뭘 검색할지, 언제 함수를 연쇄할지 다 모델이 정한다. 2023년 GPT-4 기준으로 설계됐고, 앞에서 본 [A-MEM](https://momozzing.github.io/paper%20review/A-MEM-Paper-review/)의 Qwen2.5-3b 결과를 보면 약한 모델에서는 잘 안 돌아간다(순위 2.4).
2. 토큰을 많이 쓴다. A-MEM 표에서 MemGPT는 16,900~17,000토큰을 쓰는데, A-MEM은 1,200~2,500이다. main context에 FIFO 큐를 통째로 들고 있어서다.

## **5. Conclusion**

conclusion 부분을 보면 OS에서 아이디어를 가져와 LLM의 제한된 컨텍스트 창을 관리하는 시스템이라고 정리한다. 메모리 계층과 제어 흐름을 OS처럼 설계해서 컨텍스트가 더 큰 것처럼 보이게 한다.

문서 분석에서는 컨텍스트 한계를 훨씬 넘는 텍스트를 페이징으로 처리했고, 대화 에이전트에서는 장기 기억과 일관성을 유지했다고 한다.

계층적 메모리 관리나 인터럽트 같은 OS 기법을 LLM의 자원 문제에 쓸 수 있다는 걸 보였다고 한다.

여태까지 LLM이 컨텍스트 창 안의 정보만 썼다면, MemGPT는 LLM이 함수로 직접 컨텍스트와 외부 저장소 사이를 오가게 한다.

3년이 지난 지금 벤치마크 수치는 대부분 추월당했지만, 여기서 나온 용어들은 이후 논문들에서 계속 쓰이고 있다.

다음은 [Anatomy of Agentic Memory](https://arxiv.org/abs/2602.19320)다. 여기까지 본 시스템들의 평가 방식 자체에 어떤 한계가 있는지 따지는 논문이라, 지금까지의 수치를 어떻게 읽어야 할지 보자.
