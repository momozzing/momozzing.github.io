---
date: 2026-09-28 04:00:00 +0900
title: "Agent Memory 논문 21편 회고"
excerpt: "MemGPT부터 Memory Portability까지, 에이전트 메모리 연구 3년을 네 시기로 나눠 다시 읽었다. 구조를 쌓던 흐름이 '정말 구조가 필요한가'로 돌아오기까지."
categories:
  - Study
tags:
  - Large Language Model
  - Agent
  - Memory
toc: true
toc_sticky: true
field: agent-memory
---

에이전트 메모리 논문 21편을 일주일 동안 리뷰했다.

LLM은 호출 사이에 아무것도 기억하지 않는다(stateless). 한 번에 읽을 수 있는 양도 컨텍스트 창(모델이 한 번에 입력으로 받는 토큰 범위)만큼으로 정해져 있다. 그래서 대화가 길어지거나 세션이 바뀌면 앞의 내용을 어딘가에 따로 남겨두고 필요할 때 다시 넣어줘야 한다. 이걸 어떻게 하느냐가 에이전트 메모리 연구다.

한 편씩 읽을 때는 다들 좋은 수치를 내놓는 것처럼 보였는데, 논문이 나온 순서대로 다시 늘어놓으니까 흐름이 보였다.

처음에는 구조를 쌓는 쪽으로 가다가, 최근 1년은 그 구조가 정말 필요했는지 다시 묻는 쪽으로 돌아오고 있다.

개별 논문 내용은 각 리뷰에 있으니 여기서는 메모리를 어떻게 나눠왔는지부터 보고, 큰 흐름을 정리해보자.

처음 보는 거라면 [MemGPT](https://momozzing.github.io/paper%20review/MemGPT-Paper-review/) → [LongMemEval](https://momozzing.github.io/paper%20review/LongMemEval-Paper-review/) → [Mem0](https://momozzing.github.io/paper%20review/Mem0-Paper-review/) → [서베이(Memory in the Age of AI Agents)](https://momozzing.github.io/paper%20review/Memory-in-the-Age-of-AI-Agents-Paper-review/) → [ReFind](https://momozzing.github.io/paper%20review/ReFind-Paper-review/) 순으로 보면 된다.

나머지는 아래 표에서 궁금한 것만 골라 읽어도 된다.

## **1. 메모리는 어떻게 나눠왔나**

논문들을 보기 전에 기본부터 정리해두자. 메모리를 뭐로 나누는지가 3년 사이에 꽤 바뀌었다.

### **1.1 사람 기억: 단기와 장기**

출발점은 사람 기억이다. From Human Memory to AI Memory에서 Atkinson-Shiffrin 다중저장 모델로 정리한 걸 보면 이렇다.

- 단기 기억 : 적은 양을 초~분 단위로 잠깐 들고 있는 것
  - 감각 기억 : 눈, 귀, 촉각으로 들어온 걸 아주 잠깐 저장
  - 작업 기억 : 지금 문제 푸는 데 쓰려고 들고 있는 정보
- 장기 기억 : 분 단위부터 평생까지 남는 것
  - explicit : 일화 기억(겪은 일), 의미 기억(아는 사실)처럼 의식적으로 떠올리는 것
  - implicit : 절차 기억(자전거 타는 법 같은 기술), 조건반사처럼 의식하지 않고 쓰는 것

에이전트 메모리 용어는 거의 다 여기서 가져왔다.

### **1.2 LLM에 옮기면**

앞에서 말했듯 LLM은 stateless다. 그래서 LLM에서는 대충 이렇게 나뉜다.

- 단기 : 컨텍스트 창 안에 있는 것 (지금 대화, 프롬프트, KV 캐시)
- 장기 : 컨텍스트 창 밖에 따로 저장해둔 것, 꺼내서 컨텍스트에 넣어야 쓸 수 있음

KV 캐시는 모델이 이미 읽은 토큰의 중간 계산값(key, value)을 GPU 메모리에 들고 있는 것이다. 다음 토큰을 만들 때 앞을 다시 계산하지 않으려고 쓴다.

MemGPT가 이걸 운영체제에 빗대서 정리했다. 컨텍스트 창(main context)은 RAM, 바깥 저장소(external context)는 디스크고, LLM이 함수 호출로 둘 사이에 정보를 옮긴다.

CoALA는 여기서 한 단계 더 나눴다. 단기는 working memory 하나고, 장기를 사람 기억처럼 셋으로 쪼갰다.

- Working : 지금 의사결정에 쓰는 정보, 매 LLM 호출의 프롬프트가 여기서 만들어짐
- Episodic : 과거 경험 기록 (Generative Agents의 memory stream)
- Semantic : 세계에 대한 지식 (RAG가 읽어오는 지식 베이스, Reflexion의 반성문)
- Procedural : 행동하는 방법 (Voyager의 스킬 코드, LLM 가중치)

RAG(Retrieval-Augmented Generation)는 질문과 관련된 문서를 검색해서 프롬프트에 붙여 넣고 답하게 하는 방식이다.

2023~2024년 논문들은 대부분 이 틀 안에서 얘기한다.

### **1.3 단기/장기만으로는 부족해졌다**

2025년에 나온 두 서베이가 이 틀을 다시 짰다.

From Human Memory to AI Memory(2025-04)는 단기/장기를 축 하나로 남겨두고, 누구의 기억인지(personal/system), 어디에 담는지(parametric/non-parametric)를 축으로 더해서 8칸으로 나눴다. 사용자 정보랑 에이전트 자기 작업 기록을 나눈 게 이 논문의 특징이다.

Memory in the Age of AI Agents(2025-12)는 한 발 더 나가서 장기/단기 이분법으로는 요즘 시스템을 다 담을 수 없다고 한다. 저장소는 하나고, 어떻게 쓰느냐에 따라 장기·단기가 갈린다고 본다.

대신 세 가지 기준으로 나눈다.

- Forms (어디에 담나) : Token-level / Parametric / Latent
- Functions (무엇을 위해) : Factual / Experiential / Working
- Dynamics (어떻게 움직이나) : 만들기 → 합치기·고치기·지우기 → 꺼내기

CoALA의 네 가지랑 맞춰보면 이렇다.

- Working → Working : 거의 그대로, KV 캐시 압축까지 포함
- Semantic → Factual : 사용자에 대한 사실과 환경에 대한 사실로 나눔
- Episodic → Experiential : 겪은 일뿐 아니라 거기서 뽑은 교훈, 스킬까지 포함
- Procedural → 없어짐 : 스킬 코드는 Token-level, 가중치는 Parametric으로 흩어짐

-> 내가 이해한 건 이렇다. 단기/장기는 "지금 컨텍스트에 있냐 없냐" 정도로 보면 되고, 3년 동안 바뀐 건 장기 쪽을 무슨 기준으로 나누느냐였다. 처음엔 사람 기억 이름을 그대로 가져왔다가, 나중엔 무엇을 위한 기억인지로 나누게 됐다.

그리고 실제로 나온 시스템들은 거의 다 장기 쪽이다. Mem0, Zep, A-MEM, MemMachine, NEMORI가 대화에서 나온 사용자 사실(Factual)을, Experience-Following, Janus가 문제를 풀며 쌓은 경험(Experiential)을 다룬다. 단기 쪽은 KV 캐시 압축이나 컨텍스트 관리처럼 모델 쪽 연구에 가깝다.

## **2. 논문이 나온 순서대로 다시 정리**

리뷰도 이 순서대로 올려뒀다. 한 편씩 읽으면 각자 따로 노는 것 같은데, 표로 한 번에 놓고 보면 어떻게 발전했는지가 보인다.

표는 22행이다. 이번 시리즈 21편에, 7월에 따로 리뷰한 CoALA(2023)를 배경으로 맨 위에 같이 넣었다.

| 나온 시기 | 논문 | 무엇을 했나 | 기억할 것 |
|---|---|---|---|
| 2023-09 | [CoALA](https://momozzing.github.io/paper%20review/CoALA-Paper-review/) | 에이전트 기억을 네 종류로 분류 | 인지과학의 기억 분류를 에이전트에 가져옴 |
| 2023-10 | [MemGPT](https://momozzing.github.io/paper%20review/MemGPT-Paper-review/) | LLM이 스스로 기억을 넣고 빼게 함 | 컨텍스트 창은 RAM, 외부 저장소는 디스크 |
| 2024-05 | [HippoRAG](https://momozzing.github.io/paper%20review/HippoRAG-Paper-review/) | 문서에서 개념 그래프를 만들어 검색 | 그래프로 찾고 답은 원문 구절로 |
| 2024-10 | [LongMemEval](https://momozzing.github.io/paper%20review/LongMemEval-Paper-review/) | 챗봇 장기 기억 벤치마크 | 5가지 능력, 모른다고 답하기 포함 |
| 2025-01 | [Zep](https://momozzing.github.io/paper%20review/Zep-Paper-review/) | 사실을 시간 정보가 붙은 그래프로 저장 | 틀린 사실을 지우지 않고 만료 표시 |
| 2025-02 | [A-MEM](https://momozzing.github.io/paper%20review/A-MEM-Paper-review/) | 새 기억이 들어오면 관련 기억과 자동 연결 | 옛 기억의 설명까지 고쳐 씀 |
| 2025-04 | [Mem0](https://momozzing.github.io/paper%20review/Mem0-Paper-review/) | 대화에서 사실만 뽑아 저장 | 추가·수정·삭제를 LLM이 판단, 빠름 |
| 2025-04 | [From Human Memory to AI Memory](https://momozzing.github.io/paper%20review/From-Human-Memory-to-AI-Memory-Paper-review/) | 인간 기억 분류를 AI에 대응 | 사용자 기억과 시스템 기억을 구분 |
| 2025-05 | [Experience-Following](https://momozzing.github.io/paper%20review/Experience-Following-Paper-review/) | 경험을 쌓을수록 좋아지는지 실험 | 전부 넣으면 오히려 나빠짐 |
| 2025-08 | [What Deserves Memory](https://momozzing.github.io/paper%20review/NEMORI-Paper-review/) | 무엇을 기억할지 자동 판단 | 예상과 달랐던 것만 저장 |
| 2025-12 | [Memory in the Age of AI Agents](https://momozzing.github.io/paper%20review/Memory-in-the-Age-of-AI-Agents-Paper-review/) | 메모리 연구 전체 분류 | 장기/단기 대신 형태·목적·동작으로 분류 |
| 2026-01 | [SYNAPSE](https://momozzing.github.io/paper%20review/SYNAPSE-Paper-review/) | 연관된 기억이 연쇄로 떠오르게 검색 | 모르는 건 모른다고 답함 |
| 2026-02 | [Anatomy of Agentic Memory](https://momozzing.github.io/paper%20review/Anatomy-of-Agentic-Memory-Paper-review/) | 기존 평가 방식이 맞는지 검증 | 메모리 점수를 full-context와 비교하자(∆), 작은 모델 형식 오류 |
| 2026-02 | [AMV-L](https://momozzing.github.io/paper%20review/AMV-L-Paper-review/) | 기억이 쌓여도 느려지지 않게 관리 | 자주 쓰는 기억만 검색 대상으로 |
| 2026-03 | [Multi-Layered Memory](https://momozzing.github.io/paper%20review/Multi-Layered-Memory-Paper-review/) | 기억 계층을 하나씩 빼보는 실험 | 개념 기억 층이 가장 중요 |
| 2026-04 | [MemMachine](https://momozzing.github.io/paper%20review/MemMachine-Paper-review/) | 원문을 그대로 저장하는 메모리 시스템 | 저장 방식보다 검색 튜닝 효과가 큼 |
| 2026-05 | [LongMemEval-V2](https://momozzing.github.io/paper%20review/LongMemEval-V2-Paper-review/) | 웹 에이전트용 장기 기억 벤치마크 | 사용자 정보 대신 업무 요령을 기억하나 |
| 2026-05 | [MemFail](https://momozzing.github.io/paper%20review/MemFail-Paper-review/) | 메모리가 어디서 틀리는지 진단 | 요약·저장·검색 중 어디서 깨졌나 |
| 2026-06 | [The Past Is Prologue](https://momozzing.github.io/paper%20review/Janus-Selective-Memory-Update-Paper-review/) | 기억을 고치기 전에 검증 | 새 버전이 나쁘면 옛 버전 유지 |
| 2026-07 | [What to Keep, What to Forget](https://momozzing.github.io/paper%20review/Rate-Distortion-Memory-Compaction-Paper-review/) | 무엇을 버릴지를 이론으로 정리 | 되돌릴 수 있는 방식이 이김 |
| 2026-08 | [ReFind](https://momozzing.github.io/paper%20review/ReFind-Paper-review/) | 구조 없이 원문 검색만 반복 | 그래프·요약 방식을 전부 이김 |
| 2026-09 | [Memory Portability](https://momozzing.github.io/paper%20review/Memory-Portability-Paper-review/) | 모델을 바꿔도 기억이 유지되나 실험 | 요약형 기억은 모델 따라 흔들림 |

이렇게 놓고 보면 대충 네 시기로 나뉜다.

## **3. 네 시기**

### **3.1 다른 분야에서 아이디어를 가져온 시기 (2023 ~ 2024)**

이때 질문은 하나였다. 컨텍스트 창보다 긴 걸 어떻게 다루나.

답은 다른 분야에서 가져왔다.

- MemGPT : 운영체제의 가상 메모리, 페이징
- CoALA : 인지과학의 기억 분류
- HippoRAG : 신경과학의 해마 색인 이론

셋 다 지금까지 남아 있는 게 있다.

MemGPT의 main/external context 구분이랑 함수로 자기 메모리를 관리하는 방식은 이후 거의 모든 시스템의 기본 틀이 됐다. CoALA의 네 분류는 2025년 서베이가 나오기 전까지 다들 쓰던 용어였다. HippoRAG는 약 2년 뒤 ReFind 비교표에서 후속판 HippoRAG 2가 구조화 메모리 중에 유일하게 BM25(단어가 겹치는 정도로 점수를 매기는 고전 키워드 검색)를 넘었다.

HippoRAG가 살아남은 이유는 이때 이미 있었던 것 같다. 그래프를 만들긴 하는데 검색 결과로는 원래 구절을 돌려줬다. 그래프는 색인이고 저장소가 아니었다.

-> 이 차이가 2년 뒤에 이렇게 크게 돌아올 줄은 몰랐다.

### **3.2 구조를 쌓아 올린 시기 (2025 상반기)**

LongMemEval이 평가 기준을 세우고 나서 제품형 시스템이 쏟아졌다. Zep, A-MEM, Mem0가 반년 사이에 나왔다.

셋 다 대화를 뭔가로 바꿔서 저장한다.

- Mem0 : 사실을 뽑는다
- Zep : 엔티티랑 관계로 그래프를 만든다
- A-MEM : 노트를 만들고 서로 잇는다

차이는 바꾼 걸 어떻게 고치느냐였다. Mem0는 모순이면 지우고, Zep은 무효화 시각만 찍고 남기고, A-MEM은 이웃 노트의 설명을 덮어쓴다.

그때는 구현 취향 차이 정도로 보였을 것 같은데, 1년 반 뒤 Rate-Distortion 논문이 이걸 가역·비가역 문제로 정리하면서 큰 차이가 된다. 가역은 버리거나 고친 내용을 나중에 다시 꺼낼 수 있는 방식이고, 비가역은 한번 지우거나 덮어쓰면 끝인 방식이다.

같은 시기에 나온 From Human Memory to AI Memory는 아직 장기/단기를 분류 기준으로 뒀다. 8개월 뒤 서베이(Memory in the Age of AI Agents)가 그 이분법으로는 부족하다고 한다.

같은 무렵 Experience-Following은 실행 경험을 전부 넣으면 네 에이전트 모두에서 고정 메모리보다 나빠진다고 했다. 비슷한 과거를 그대로 따라 하니까 나쁜 경험도 따라 하기 때문이었다. 무엇을 남길지라는, 다음 시기의 질문을 먼저 꺼낸 논문이다.

### **3.3 무엇을 남길지 고민한 시기 (2025 하반기 ~ 2026 초)**

구조가 어느 정도 갖춰지고 나니 질문이 바뀌었다. 어떻게 저장하느냐에서 무엇을 저장하느냐로.

- What Deserves Memory : 중요도 점수 대신 "기존 지식으로 예측했는데 틀린 부분"만 남김
- SYNAPSE : 잘 찾는 것에 더해서, 못 찾았을 때 모른다고 답하게 만듦

그리고 이 시기 끝에 2025년 12월 서베이가 Forms(어디에 담나), Functions(무엇을 위해), Dynamics(어떻게 움직이나)로 전체를 다시 나눴다.

### **3.4 다시 검증하는 시기 (2026)**

올해 나온 논문들은 좀 다르다. 새 구조를 내놓기보다 지금까지 쌓은 게 맞았는지 따진다. 크게 넷으로 묶인다.

첫째, 평가를 의심한다.

Anatomy of Agentic Memory는 메모리 시스템 점수를 같은 모델에 대화 전체를 넣은 점수랑 나란히 놓으라고 한다. 내가 앞 리뷰 수치로 계산해보면 3개 중 2개(MemGPT, Mem0)가 마이너스다. MemFail은 점수 하나로 뭉뚱그리지 말고 요약·저장·검색 중 어디서 틀렸는지 나눠 보자고 한다.

둘째, 병목을 다시 찾는다.

MemMachine은 저장 방식 개선(+0.8%p)보다 검색 개선(검색 개수만 바꿔도 +4.2%p)이 크다고 한다. AMV-L은 느린 원인이 저장 용량보다 검색 후보가 계속 커지는 데 있다고 본다.

셋째, 구조 자체를 의심한다.

ReFind가 제일 멀리 갔다. 아무 구조도 안 만들고 원문 대화 위에 BM25 반복 검색과 채팅용 기능 몇 개만 얹어서 그래프·트리 기반 시스템을 전부 넘었다. Mem0랑 Zep은 BM25의 절반 정도였다.

넷째, 운영을 본다.

Memory Portability는 모델을 바꾸면 요약형 메모리가 방향에 따라 +9.91 ~ −13.28pp 흔들리고, 원본이 없으면 복구가 48건 전부 실패한다고 한다. The Past Is Prologue는 갱신을 무조건 받지 말고 옛 버전이랑 비교해서 나은 쪽을 남기자고 한다.

## **4. 여러 논문에서 겹친 결론**

21편이 각자 다른 걸 재는데, 결론이 겹치는 데가 네 군데 있었다.

### **4.1 원본을 남긴 쪽이 나았다**

제일 여러 번 나왔다.

- HippoRAG : 그래프는 색인, 답은 원래 구절
- Zep : 지우지 않고 무효화 시각만 기록
- Rate-Distortion : 같은 예산에서 가역 0.95 vs 비가역 0.33~0.56
- ReFind : 원문을 안 건드리고 1위
- Memory Portability : 원본 없으면 복구 48건 중 0건
- The Past Is Prologue : 옛 버전이 있어야 비교하고 되돌릴 수 있음

반대로 성적이 나빴던 쪽은 원본을 버렸다. Mem0는 사실만 남기고, 요약형 NOTES는 손실의 80%가 쓰기 단계에서 생겼다.

그렇다고 원본을 전부 검색 대상으로 두면 AMV-L에서 말한 문제가 생긴다. 남겨두되 검색 대상에서는 뺄 수 있게 두는 게 맞는 것 같다.

### **4.2 저장보다 검색이 자주 막힌다**

같은 방향으로 나온 게 네 개다.

- MemMachine : 검색 최적화 효과가 저장 최적화의 5배
- Memory Portability : RAG 손실의 81%가 검색 단계
- AMV-L : 병목은 검색 후보 크기
- ReFind : 원문 위에 BM25 반복 검색과 채팅용 기능 몇 개만 얹어서 1위

예외도 있다. What Deserves Memory는 검색을 단순하게 두고 저장 단계 증류만으로 이겼다.

이 둘은 MemFail로 보면 정리가 된다. 요약에서 틀리는 문제면 검색 개수를 늘려도 소용없고, 검색에서 틀리는 문제면 늘리면 좋아진다.

### **4.3 모델을 올려서는 안 풀린다**

MemFail은 더 강한 모델이 오히려 장황한 메모리를 만들어서 성능을 깎는다고 한다. Anatomy는 작은 모델에서 메모리 쓰기 형식 오류가 30%까지 오른다고 하고, Memory Portability는 모델을 바꾸면 기존 메모리 해석이 흔들린다고 한다.

-> 모델만 좋아지면 메모리도 알아서 좋아질 줄 알았는데 그렇지 않다.

### **4.4 대화 전체를 넣은 점수와 먼저 비교한다**

Anatomy에서 제안한 방식이다. 메모리 시스템 점수에서 같은 모델에 대화 전체를 넣은 점수를 빼본다. 이 차이가 없거나 마이너스면, 그 시스템은 정확도보다 지연이랑 비용 때문에 쓰게 된다.

이렇게 보면 Mem0는 LoCoMo에서 LLM-judge 점수(J)를 72.90 → 66.88로 6점 내주고, p95 지연(느린 쪽 5% 경계 지연)을 17.1초 → 1.44초로 12배 줄인 시스템이다. 나쁘다는 게 아니라 뭘 얻으려고 쓰는 시스템인지가 분명해진다.

## **5. 아직 비어 있는 곳**

21편을 서베이의 분류에 올려보면 몇 칸이 비어 있다.

- 파라미터에 기억을 넣는 방식 : 두 서베이 모두 분류 기준으로 세웠는데, 21편 중에 실제로 구현한 시스템이 없음
- 경험 기억 평가 : 대부분 사용자에 대한 사실을 재고, 에이전트가 경험에서 배우는지 재는 건 LongMemEval-V2, Experience-Following, Janus 정도
- 어시스턴트가 한 말 기억하기 : Zep(−17.7%)이랑 What Deserves Memory(−5.3점) 둘 다 대화 전체를 넣은 경우보다 짐
- 반복 갱신의 누적 효과 : A-MEM의 진화나 Mem0의 갱신을 수천 번 돌린 결과를 잰 논문이 없음

어시스턴트 발화 쪽은 추출하면서 안내나 추천 같은 발화가 흐려지는 것 같다. 반복 갱신은 Rate-Distortion이 작은 규모로 위험을 보여준 정도다.

-> 마지막 건 실제로 오래 돌리는 서비스라면 제일 먼저 궁금할 부분인데 아무도 안 쟀다??

## **6. 다시 만든다면**

21편을 읽고 나서 메모리를 다시 만든다면 이 순서로 해볼 것 같다.

1. 대화 전체를 넣은 경우를 먼저 재서 메모리가 정말 필요한지부터 본다
2. 원문은 남기고, 요약이나 추출은 그 위에 색인으로 얹는다
3. 저장 구조는 그대로 두고 검색부터 손본다 (검색 개수, 이웃 턴, BM25+벡터, 시간 감쇠)
4. MemFail 네 과제(조건부 사실, 같이 있는 선호, 없는 사람, 흩어진 사실 잇기)로 어디서 틀리는지 본다
5. 그다음에 구조를 얹는다 (삭제 대신 무효화, 갱신 전에 옛 버전과 비교)
6. 임베딩 모델을 바꿀 때는 전부 다시 만들고, LLM을 바꿀 때는 방향별로 메모리를 다시 잰다

처음 이 시리즈를 시작할 때는 어떤 메모리 구조를 고를지가 궁금했다.

다 읽고 나니 질문이 바뀌었다. 구조를 고르기 전에, 원문 남기고 검색만 제대로 해도 어디까지 가는지부터 봐야겠다.
