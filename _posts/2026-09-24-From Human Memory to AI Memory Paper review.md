---
date: 2026-09-24 09:00:00 +0900
title: "From Human Memory to AI Memory Paper review"
excerpt: "대상·형태·시간 세 축으로 AI 메모리를 8분면에 나눈다. 2026년 서베이와 교차 검증해보면 1년 사이 무엇이 달라졌는지가 보인다."
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

From Human Memory to AI Memory: A Survey on Memory Mechanisms in the Era of LLMs

[https://arxiv.org/abs/2504.15965](https://arxiv.org/abs/2504.15965)

From Human Memory to AI Memory는 Huawei Noah's Ark Lab에서 쓴 LLM 메모리 서베이 논문이다.

사람의 기억 분류에서 출발해서, AI 메모리를 대상·형태·시간 세 축으로 8분면에 나눈다.

2025년 4월 22일에 나왔고 4월 23일에 v2가 올라왔다. 저자 8명, 26쪽이다.

뒤에서 볼 [2026년 서베이](https://momozzing.github.io/paper%20review/Memory-in-the-Age-of-AI-Agents-Paper-review/)보다 8개월 먼저 나왔다. 같은 대상을 다르게 나눈다. 뒤의 지금 관점 절에서 두 서베이를 비교해서 1년 사이에 뭐가 달라졌는지 정리했다.

좀 더 자세히 알아보자.

## **1. Introduction**

기존 리뷰들이 메모리 메커니즘은 자세히 정리했지만 빠진 게 있다고 한다.

LLM 기반 AI 시스템의 메모리와 사람 기억이 어떤 관계인지, 사람 기억에서 아이디어를 얻어 더 나은 메모리를 어떻게 만들 수 있는지를 정리한 리뷰가 아직 없다는 것이다.

그래서 사람 기억 분류부터 시작해서 AI 메모리와 연결한다.

## **2. Overview**

### **2.1 Human Memory**

인간 기억 쪽부터 본다.

Atkinson-Shiffrin 다중저장 모델로 단기와 장기를 나눈다.

단기 기억은 적은 양의 정보를 짧게(초~분) 들고 있는 임시 저장이다. 두 가지로 나뉜다.

- 감각 기억 : 바깥에서 들어온 감각 정보를 잠깐 저장. 시각(iconic), 청각(echoic), 촉각(haptic). 수 밀리초에서 수 초
- 작업 기억 : 문제 풀기나 학습 같은 걸 하려고 정보를 직접 처리하고 조작

뒤에서 볼 [2026 서베이](https://momozzing.github.io/paper%20review/Memory-in-the-Age-of-AI-Agents-Paper-review/)도 7.8절에서 Atkinson-Shiffrin과 Tulving을 가져오는데, 거기서는 마지막 절에 두고 여기서는 시작점에 둔다.

### **2.2 Memory of LLM-driven AI Systems**

논문은 먼저 사람 기억 범주를 AI 메모리에 하나씩 짝지어 그림으로 보여준다.

![사람 기억과 AI 메모리의 대응 (논문 Figure 1)](https://momozzing.github.io/assets/images/human-to-ai-memory/fig1-human-ai-memory-parallels.png)

왼쪽이 사람 기억, 오른쪽이 LLM 기반 AI 메모리다. 감각 기억은 텍스트·이미지·오디오·비디오 입력으로, 작업 기억은 대화·CoT·프롬프트 캐시로 이어진다.

장기 기억 쪽은 일화 기억이 비파라미터 검색으로, 의미 기억이 파라미터 주입으로, 절차 기억이 태스크와 스킬 학습으로 대응된다.

#### **2.2.1 3D-8Q Memory Taxonomy**

세 축으로 나눈다.

1. Object (대상) : personal / system
2. Form (형태) : non-parametric / parametric
3. Time (시간) : short-term / long-term

2×2×2 = 여덟 분면이다.

- I (Personal·비파라미터·단기, Working) : 세션 안에서 실시간으로 맥락을 보충
- II (Personal·비파라미터·장기, Episodic) : 세션을 넘어서 보존. 과거 사용자 대화를 떠올려서 개인화
- III (Personal·파라미터·단기, Working) : 지금 대화의 맥락 이해를 잠깐 강화
- IV (Personal·파라미터·장기, Semantic) : 새로 얻은 지식을 모델에 계속 넣음
- V (System·비파라미터·단기, Working) : CoT 프롬프트 같은 중간 출력을 저장해서 복잡한 추론·결정을 도움
- VI (System·비파라미터·장기, Procedural) : 과거 경험과 자기반성에서 얻은 걸 담아 추론·문제해결 능력을 다듬음
- VII (System·파라미터·단기, Working) : KV 캐시 같은 임시 파라미터 저장으로 추론 속도 최적화
- VIII (System·파라미터·장기, Semantic + Procedural) : 모델 파라미터에 들어 있는 기반 지식

personal과 system 두 개념은 이렇게 나눈다.

Personal memory는 모델이 환경에서 보고 들은 개별 데이터이고, system memory는 과제를 하면서 만들어진 중간 결과처럼 시스템 안에서 생긴 메모리라고 한다.

2026 서베이에는 이 구분이 없다. 거기는 Forms(어디에 담는가)와 Functions(무엇을 위해)로 나누고, 누구의 것인지는 축으로 두지 않는다.

-> 실무에서는 이 구분이 쓸모 있을 것 같다. 사용자 정보와 에이전트 자기 작업 기록은 수명도 삭제 정책도 다르다. 개인정보 삭제 요청이 오면 personal만 지우면 되고 system은 남긴다.

이 시리즈 논문들을 8분면에 넣으면 어떨까.

이 시리즈에서 다루는 논문들을 넣어보면 이렇다. 아직 안 본 논문도 섞여 있는데, 뒤에서 하나씩 나온다.

- I (personal·비파라미터·단기) : MemGPT의 FIFO 큐, 세션 히스토리
- II (personal·비파라미터·장기) : Mem0, Zep, A-MEM, MemMachine. 제일 많이 몰린 칸
- III, IV (personal·파라미터) : 거의 비어 있음
- V (system·비파라미터·단기) : ReAct의 중간 추론, MemGPT working context
- VI (system·비파라미터·장기) : Experience-Following, Janus, NEMORI
- VII (system·파라미터·단기) : KV 캐시 압축 (SnapKV, H2O)
- VIII (system·파라미터·장기) : 모델 가중치, 지식 편집

이 시리즈 논문들 대부분이 II와 VI 두 칸에 몰려 있다. III·IV(개인화된 파라미터 메모리)는 아직 비어 있다.

-> 이렇게 칸을 나눠두니 어디가 비어 있는지 보인다.

## **3. Open Problems and Future Directions**

`From X to Y` 형식으로 다섯 가지를 제시한다.

1. Unimodal → Multimodal : 텍스트만 다루던 데서 이미지·음성·영상·센서 데이터까지. 의료 예시를 드는데, 진료 기록(텍스트) + 의료 영상 + 의사-환자 대화(음성)를 합치면 더 정확하게 진단할 수 있다고 한다
2. Static → Stream : 정적 메모리는 배치 처리다. 묶음으로 정보를 쌓고 정해진 때에 처리·저장·검색한다. 스트림 메모리는 연속적으로 실시간으로 돈다
3. Specific → Comprehensive : 지금 메모리 구조는 바로 추론하는 데 쓰는 단기 메모리나 도메인 지식 저장처럼 좁은 부분에만 집중한다. 그래서 전체적인 유연성, 일반화, 적응성이 떨어진다고 한다
4. Exclusive → Shared : 지금은 시스템마다 메모리가 따로 논다. 의료 모델이 금융 모델과 메모리를 공유하는 식으로 도메인끼리 지식을 옮기는 걸 전망한다
5. Individual Privacy → Collective Privacy : 데이터 공유가 늘면서 프라이버시 보호가 개인에서 집단 쪽으로 옮겨간다

## **4. 지금 관점: 2026 서베이와 비교**

두 서베이를 나란히 놓으면 8개월 사이에 달라진 게 보인다.

- 분류 축 : 이 논문(2025-04)은 대상 / 형태 / 시간, 2026 서베이(2025-12)는 Forms / Functions / Dynamics
- 시간 축 : 이 논문은 short/long을 축 하나로 세웠고, 2026 서베이는 "이분법으로는 다양성을 못 담는다"고 한다
- 인간 기억 : 이 논문은 시작점, 2026 서베이는 7.8절 마지막
- 분량 : 26쪽 vs 107쪽
- 벤치마크·프레임워크 : 이 논문은 없고, 2026 서베이는 6장 전체(25종 표)

제일 큰 차이는 시간 축이다.

이 논문은 단기/장기를 세 축 중 하나로 세운다. 2026 서베이는 반대로 long/short-term 이분법으로는 요즘 시스템을 다 담을 수 없다고 하고, 그 자리에 Functions(factual/experiential/working)를 넣는다.

이 논문이 나오고 8개월 만에 그 축이 빠진 셈이다.

-> 그래도 이 논문이 틀렸다고 보긴 어려울 것 같다. 시간만으로 나눈 게 아니라 대상·형태랑 섞어서 8분면을 만들었으니 단순 이분법보다는 세밀하다. 2026 서베이가 비판한 건 시간만으로 나누는 쪽이다.

남은 것과 빠진 것을 보면 이렇다.

1. personal vs system 구분은 남는다. 2026 서베이에는 이 축이 없는데 실무에서는 필요하다. 삭제 정책, 접근 제어, 보관 기간이 다 다르다. 나중에 볼 [Memory Portability 리뷰](https://momozzing.github.io/paper%20review/Memory-Portability-Paper-review/)에서 "전체 이력 보존은 프라이버시·보안·보관·삭제 의무를 만든다"고 하는데, 그게 personal 쪽에만 걸린다.
2. 시간 축은 밀려났다. 2026 서베이는 하나의 저장소 안에서 쓰임새에 따라 장기·단기가 갈린다고 정의한다.
3. 두 서베이 모두 parametric memory를 하나의 축으로 다룬다. 이 논문의 III·IV·VII·VIII 네 분면, 2026 서베이의 Forms 3.2절이다. 그런데 이 시리즈 논문들 중에서 parametric을 실제로 구현한 시스템은 하나도 없다.

-> 서베이는 축으로 세웠는데 실제 연구는 거기 없다. 왜 그런지는 잘 모르겠다??

## **5. Conclusion**

conclusion 부분을 보면, 사람 기억과 LLM 기반 AI 메모리의 관계를 정리하고, 사람의 인지 원리가 더 효율적이고 유연한 메모리 구조를 만드는 데 어떻게 도움이 될 수 있는지 살펴봤다고 한다.

지각 기억, 작업 기억, 장기 기억 같은 사람 기억 범주를 AI 메모리 모델과 비교하고, 그 위에 대상·형태·시간 세 차원으로 8분면 분류를 만들었다.

여태까지 메모리 분류가 장기/단기나 저장 형태 하나로 나눴다면, 이 논문은 누구의 기억인지(personal/system)까지 축으로 넣어서 나눈다.

2026 서베이가 주 좌표계라면 이건 보조 좌표계 정도로 보면 될 것 같다. personal/system 구분과 8분면에서 빈칸이 보인다는 게 쓸모 있었다.

한계는 벤치마크도 프레임워크 비교도 없다는 점이다. 2026 서베이가 6장 전체를 실제로 쓸 수 있는 자원 정리에 쓴 것과 다르다. 분류만 있고 "그래서 뭘 쓰면 되나"는 없다.

-> 26쪽이라 분량 때문에 뺀 것 같다.

다음은 [Experience-Following](https://momozzing.github.io/paper%20review/Experience-Following-Paper-review/)이다. 메모리에 무엇을 넣고 무엇을 지우는지가 에이전트 행동을 어떻게 바꾸는지를 네 에이전트로 잰 논문이다.
