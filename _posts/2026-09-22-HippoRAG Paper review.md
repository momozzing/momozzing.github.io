---
date: 2026-09-22 12:00:00 +0900
title: "HippoRAG Paper review"
excerpt: "해마 색인 이론을 RAG에 옮겼다. LLM으로 스키마 없는 지식그래프를 만들고 Personalized PageRank를 돌려 단발 검색으로 다중홉을 푼다."
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


HippoRAG: Neurobiologically Inspired Long-Term Memory for Large Language Models

[https://arxiv.org/abs/2405.14831](https://arxiv.org/abs/2405.14831)

HippoRAG는 Ohio State University와 Stanford에서 만든 RAG 방법이다.

사람 뇌의 해마 색인 이론을 가져와서, LLM으로 지식그래프를 만들고 Personalized PageRank(PPR)로 한 번의 검색에 다중홉을 푼다.

2024년 5월 23일에 올라왔고 2025년 1월 14일에 v3가 나왔다. 저자 5명에 31쪽이다.

뒤에서 볼 [서베이](https://momozzing.github.io/paper%20review/Memory-in-the-Age-of-AI-Agents-Paper-review/)에서는 RAG 쪽이랑 메모리 쪽 양쪽에서 다 인용되는 논문으로 꼽고, 나중에 볼 [ReFind 표](https://momozzing.github.io/paper%20review/ReFind-Paper-review/)에서는 구조화 메모리 중에 유일하게 BM25-RAG를 넘는다(53.2 vs 48.8).

왜 이것만 다른지는 5장에서 정리했다. 좀 더 자세히 알아보자.

## **1. Introduction**

다중홉 질문을 두 가지로 나눈다.

- Path-following : "Alhandra는 어느 구역에서 태어났나?"처럼 정해진 경로를 따라가면 되는 질문
- Path-finding : "알츠하이머 신경과학을 연구하는 스탠퍼드 교수는?"처럼 갈 수 있는 경로가 많은데 그중 맞는 걸 찾아야 하는 질문

첫 번째는 반복 검색으로 풀린다. Alhandra → Vila de Xira → Portugal 순으로 따라가면 된다.

두 번째는 다르다. "스탠퍼드"에서 뻗는 경로도 많고 "알츠하이머"에서 뻗는 경로도 많다. 둘이 만나는 지점을 찾아야 하는데, 질의 임베딩 하나로는 그 지점이 안 보인다.

그래서 여러 단계 RAG를 완벽하게 돌려도 이런 지식 통합 문제에는 부족할 때가 많다고 한다.

## **2. HippoRAG**

### **2.1 The Hippocampal Memory Indexing Theory**

Teyler와 Discenna(1986)의 해마 색인 이론에서 설계를 가져왔다.

사람의 장기기억은 두 가지를 해낸다고 본다.

- Pattern separation : 서로 다른 경험이 서로 다른 표현으로 저장되게 함
- Pattern completion : 일부 단서만으로 전체 기억을 되살림

저장할 때는 분리가 일어난다. 신피질이 감각 자극을 고수준 특징으로 바꾸고, 해마주위 영역(PHR)을 거쳐서 해마가 색인을 만든다. 해마에서는 두드러진 신호가 색인에 들어가고 서로 연결된다.

꺼낼 때는 완성이 일어난다. 해마가 PHR에서 일부 신호를 받으면 맥락에 따라 전체 기억을 되살린다.

### **2.2 Overview**

이 구조를 그대로 옮겼다.

![HippoRAG 방법론 (논문 Figure 2)](https://momozzing.github.io/assets/images/hipporag/fig2-method.png)

- 신피질 → LLM (입력 처리)
- 해마 색인 → 스키마 없는 지식그래프
- 해마주위 영역 → 검색 인코더
- pattern completion → Personalized PageRank

### **2.3 Detailed Methodology**

먼저 오프라인 색인이다.

명령어 튜닝 LLM `L`과 검색 인코더 `M`으로 구절들을 처리한다.

1. OpenIE로 명사구 노드와 관계 엣지를 뽑는다. 1-shot 프롬프팅이다
2. 두 단계로 나눈다. 먼저 개체명을 뽑고, 그 개체명을 OpenIE 프롬프트에 넣어서 트리플을 뽑는다. 트리플에는 개체명 말고 개념(명사구)도 들어간다
3. 인코더로 동의어 엣지를 추가한다. 두 엔티티의 코사인 유사도가 임계값 `τ`를 넘으면 잇는다

두 단계로 나눈 건 일반성과 개체명 편향 사이에서 균형을 맞추려고 그랬다고 한다.

동의어 엣지는 색인에 엣지를 더 넣어서 pattern completion이 더 잘 되게 한다고 본다.

다음은 온라인 검색이다.

1. 질의에서 개체명 `C_q`를 1-shot 프롬프트로 뽑는다 (Figure 2 예시에서는 "Stanford"와 "Alzheimer's")
2. 같은 인코더로 인코딩해서 그래프에서 코사인 유사도가 제일 높은 노드를 질의 노드 `R_q`로 고른다
3. 질의 노드를 시작점으로 PPR을 돌린다
4. PPR 노드 확률을 구절 단위로 합쳐서 순위를 매긴다

PPR이 그래프 경로를 탐색하고 관련 부분그래프를 찾아주니까, 한 번의 검색 안에서 다중홉 추론을 하는 셈이라고 한다.

뒤에서 볼 [SYNAPSE 리뷰](https://momozzing.github.io/paper%20review/SYNAPSE-Paper-review/)의 활성 확산이랑 비슷한 계열이다. SYNAPSE는 PageRank를 전역 사전확률로 쓰는데, 여기서는 질의 노드에서 출발하는 PPR로 쓴다.

## **3. Results**

단일 단계 검색부터 보자.

| 방법 | MuSiQue R@5 | 2Wiki R@5 | HotpotQA R@5 | 평균 R@5 |
|---|---:|---:|---:|---:|
| BM25 | 41.2 | 61.9 | 72.2 | 58.4 |
| Contriever | 46.6 | 57.5 | 75.5 | 59.9 |
| GTR | 49.1 | 67.9 | 73.3 | 63.4 |
| ColBERTv2 | 49.2 | 68.2 | 79.3 | 65.6 |
| RAPTOR (ColBERTv2) | 46.5 | 64.7 | 75.6 | 62.3 |
| Proposition (ColBERTv2) | 50.1 | 64.9 | 78.1 | 64.4 |
| HippoRAG (ColBERTv2) | 51.9 | 89.1 | 77.7 | 72.9 |

2Wiki에서 68.2 → 89.1로 20.9점 올랐다. abstract의 "최대 20%"가 여기서 나온 거다.

HotpotQA에서는 진다(77.7 vs 79.3). 논문은 HotpotQA가 지식 통합이 별로 필요 없는 데이터셋이고, 개념과 맥락 사이의 절충 문제도 있다고 한다.

-> [ReFind 표](https://momozzing.github.io/paper%20review/ReFind-Paper-review/)에서는 HippoRAG 2가 SH-QA 76.0으로 높은데 여기서는 HotpotQA가 약하다. 근데 과제가 다르다. 여기 HotpotQA는 위키 다중홉이고 ReFind의 SH-QA는 대화 단일홉이다.

반복 검색과 결합하면 더 오른다.

| 방법 | 평균 R@5 |
|---|---:|
| IRCoT + BM25 | 66.4 |
| IRCoT + ColBERTv2 | 70.0 |
| IRCoT + HippoRAG (ColBERTv2) | 78.2 |

반복 검색이랑 경쟁하는 게 아니라 같이 쓸 수 있다. IRCoT의 반복 검색 안에서 검색기로 HippoRAG를 쓰면 2Wiki에서 R@5가 18% 더 오른다고 한다.

QA 성능은 이렇다.

| 검색기 | 평균 EM | 평균 F1 |
|---|---:|---:|
| None | 24.6 | 35.5 |
| ColBERTv2 | 30.8 | 42.5 |
| HippoRAG (ColBERTv2) | 35.9 | 48.1 |
| IRCoT (ColBERTv2) | 33.3 | 44.7 |
| IRCoT + HippoRAG | 38.4 | 51.7 |

단일 단계 HippoRAG가 IRCoT보다 높다(48.1 vs 44.7).

그러면서 온라인 검색이 10~30배 싸고 6~13배 빠르다고 한다. 반복 검색만큼의 정확도를 한 번의 검색으로 내는 거다.

## **4. Discussions**

### **4.1 What Makes HippoRAG Work?**

먼저 OpenIE 모델을 바꿔 봤다.

| OpenIE | 평균 R@5 |
|---|---:|
| REBEL (전용 모델) | 58.4 |
| Llama-3.1-8B-Instruct | 67.8 |
| Llama-3.1-70B-Instruct | 72.5 |
| GPT-3.5 (기본) | 72.9 |

전용 OpenIE 모델(REBEL)을 쓰면 크게 떨어진다.

GPT-3.5가 REBEL보다 트리플을 두 배 많이 만든다고 한다. REBEL은 일반 개념이 들어간 트리플을 잘 안 만들어서 쓸모 있는 연결을 많이 놓친다는 것이다.

Llama-3.1-70B는 GPT-3.5랑 거의 비슷하고, 8B도 2Wiki만 빼면 괜찮다. 큰 코퍼스를 색인할 때 더 싼 대안이 될 수 있다고 한다.

PPR이 실제로 기여하는지도 봤다.

| 방식 | 평균 R@5 |
|---|---:|
| `R_q` 노드만 | 56.2 |
| `R_q` 노드 + 이웃 | 59.2 |
| PPR (기본) | 72.9 |

PPR을 빼면 16점 넘게 떨어진다.

그리고 PPR 없이 `R_q` 노드에 이웃을 더하는 건 질의 노드만 쓰는 것보다 나쁘다고 한다.

-> 이웃을 막 넓히면 잡음이 늘어서, 얼마나 퍼질지 조절하는 PPR이 필요하다는 것 같다. [SYNAPSE](https://momozzing.github.io/paper%20review/SYNAPSE-Paper-review/)에서는 fan effect와 측면 억제로 같은 문제를 풀었다.

나머지 구성요소도 하나씩 뺐다.

| 제거 | 평균 R@5 |
|---|---:|
| w/o Node Specificity | 70.9 |
| w/o Synonymy Edges | 70.5 |
| 전체 | 72.9 |

둘 다 2점 정도다. PPR에 비하면 작다.

### **4.2 HippoRAG's Advantage: Single-Step Multi-Hop Retrieval**

All-Recall로 본다. 일부만 찾았는지가 아니라 근거 구절을 다 찾았는지를 잰다.

| 방법 | MuSiQue AR@5 | 2Wiki AR@5 | 평균 AR@5 |
|---|---:|---:|---:|
| ColBERTv2 | 16.1 | 37.1 | 37.4 |
| HippoRAG | 22.4 | 75.7 | 52.0 |

2Wiki에서 37.1 → 75.7로 두 배다.

논문은 이 개선이 부분 검색을 한 질문이 늘어서가 아니라, 근거 문서를 전부 찾은 질문이 늘어서 생긴 거라고 한다.

-> 다중홉은 근거 하나만 빠져도 못 푸니까, R@5보다 AR@5가 실제 능력에 더 가까워 보인다.

### **4.3 HippoRAG's Potential: Path-Finding Multi-Hop Retrieval**

앞에서 본 "알츠하이머 연구하는 스탠퍼드 교수?" 질문에 대한 상위 3개 결과는 이렇다.

- HippoRAG : Thomas Südhof, Karl Deisseroth, Robert Sapolsky
- ColBERTv2 : Brian Knutson, Eric Knudsen, Lisa Giocomo
- IRCoT : ColBERTv2와 동일

ColBERTv2랑 IRCoT가 같은 답을 낸다. 반복 검색을 해도 못 찾는다는 거다.

## **5. 지금 관점: 왜 이것만 이겼나**

[ReFind 표](https://momozzing.github.io/paper%20review/ReFind-Paper-review/)에서 구조화 메모리 중 HippoRAG 2만 BM25-RAG를 넘었다. 이 논문을 읽고 나서 생각해 본 이유는 세 가지다.

1. 원본 구절을 안 버린다. 그래프를 만들긴 하지만 검색 결과는 원래 구절이다. PPR 노드 확률을 구절로 모아서 순위를 매긴다.
2. 스키마가 없다. 논문이 `schemaless knowledge graph`라고 적었다. 스키마를 안 정하니까 뭘 버릴지 미리 정하지 않는다.
3. 검색 비용이 낮다. 색인은 오프라인이고 질의할 때는 PPR만 돈다. IRCoT보다 6~13배 빠르다.

1번은 뒤에서 볼 Mem0가 사실만 남기고, A-MEM이 노트로 바꾸는 것과 다르다. [Rate-Distortion](https://momozzing.github.io/paper%20review/Rate-Distortion-Memory-Compaction-Paper-review/)의 P-rev도 만족한다. 그래프는 색인이고 저장소가 아니다.

해마도 기억을 직접 저장하지 않고 신피질의 기억을 가리키는 색인만 갖고 있다고 하니까, 이론을 그대로 옮긴 결과다.

2번은 뒤에서 볼 Zep이 엔티티 타입을 정하고 Mem0g가 노드 타입을 나누는 것과 다르다.

-> 그런데 나중에 볼 [Memory Portability 리뷰](https://momozzing.github.io/paper%20review/Memory-Portability-Paper-review/)에서는 반대로 고정 스키마 KG가 모델 교체에 강했다(∆ 0.0004). 스키마가 없으면 OpenIE를 돌린 모델에 결과가 묶인다. 실제로 REBEL이랑 GPT-3.5의 트리플 수가 두 배 차이 난다. 스키마가 없으면 검색은 좋아지는데 모델을 바꾸면 그래프를 다시 만들어야 하는 거 아닌가??

3번은 뒤에서 볼 [Anatomy 리뷰](https://momozzing.github.io/paper%20review/Anatomy-of-Agentic-Memory-Paper-review/)에서 잰 MemoryOS의 31초 검색이랑 차이가 크다.

-> 다중홉 질문을 볼 때는 R@k보다 AR(all-recall)을 같이 보는 게 좋을 것 같다.

-> PPR은 그래프 구조가 필요해서 Qdrant 위에 바로 얹기는 어렵다. 질의에서 엔티티를 뽑고 그 엔티티로 다시 검색하는 2단 구조는 그래프 없이도 되니까, path-finding 일부는 그걸로도 나아지지 않을까?

## **6. Conclusions & Limitations**

논문은 신경생물학 원리를 가져온 단순한 방법으로 기존 RAG의 한계를 넘으면서도 파라미터 메모리보다 나은 점은 유지할 수 있다는 걸 보였다고 한다.

path-following 다중홉 QA에서 좋은 결과, path-finding에서의 가능성, 큰 효율 개선, 그리고 계속 갱신할 수 있다는 점 때문에 HippoRAG를 기존 RAG와 파라미터 메모리 사이의 중간쯤에 있는 방법으로 본다.

한계도 적었다.

- 모든 구성요소를 학습 없이 기성품으로 썼다. 오류를 분석해 보니 대부분 NER과 OpenIE에서 나와서, 파인튜닝하면 좋아질 여지가 크다고 한다
- 나머지 오류는 그래프 탐색에서 나온다. 단순 PPR보다 나은 방법이 있을 수 있다고 한다

첫 번째는 뒤에서 볼 리뷰들이랑 이어진다. 쓰기 단계에서 LLM 추출 품질이 전체를 좌우한다는 것이고, [Anatomy](https://momozzing.github.io/paper%20review/Anatomy-of-Agentic-Memory-Paper-review/)에서 형식 오류율 30%로 잰 부분이다.

여태까지 다중홉은 검색을 여러 번 반복해서 풀었다면, 이 방법은 그래프 위에 PPR을 돌려서 한 번의 검색으로 푼다.

다음은 [LongMemEval](https://momozzing.github.io/paper%20review/LongMemEval-Paper-review/)이다. 장기기억을 다섯 능력으로 쪼개고, 메모리 설계를 indexing·retrieval·reading 세 단계 네 제어점으로 나눈 벤치마크다.
