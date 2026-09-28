---
date: 2026-09-27 09:00:00 +0900
title: "LongMemEval-V2 Paper review"
excerpt: "V1이 사용자 이력을 물었다면 V2는 환경 경험을 묻는다. 웹 에이전트가 숙련된 동료가 되는가를 재는 벤치마크, 최대 1억 1500만 토큰."
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

LongMemEval-V2: Evaluating Long-Term Agent Memory Toward Experienced Colleagues

[https://arxiv.org/abs/2605.12493](https://arxiv.org/abs/2605.12493)

LongMemEval-V2는 UCLA에서 만든 에이전트 메모리 벤치마크다.

웹 에이전트가 같은 환경에서 반복해서 일한 경험을 메모리로 잘 쌓아서, 숙련된 동료처럼 되는지를 잰다.

2026년 5월 12일에 나왔고, 저자 7명에 32쪽이다. [V1](https://momozzing.github.io/paper%20review/LongMemEval-Paper-review/)과 1저자(Di Wu)가 같다.

평가 기준을 V1에서 V2로 옮겨야 할지 보려고 읽었다. 읽어보니 V2는 V1의 후속이라기보다 다른 문제를 잰다. 둘 중 하나를 고르는 게 아니라 무엇을 재려는지에 따라 다르다.

좀 더 자세히 알아보자.

## **1. 무엇이 달라졌나**

V1은 사용자-어시스턴트 대화에서 사용자에 대한 사실을 기억하는지 물었다.

V2는 메모리 시스템이 에이전트를 맞춤 환경을 잘 다루는 숙련자로 만들어주는지를 묻는다.

기존 에이전트 메모리 벤치마크는 대부분 사용자 이력, 짧은 궤적, 다운스트림 태스크 성공률을 봤고, 메모리 시스템이 환경마다 다른 경험을 제대로 익히는지를 직접 재는 방법은 없었다고 한다.

### **V1 vs V2**

| | LongMemEval-V1 | LongMemEval-V2 |
|---|---|---|
| 도메인 | 사용자-어시스턴트 대화 | 웹 에이전트 |
| 세션 수 | 48–475 | 100–498 |
| 토큰 | 115k–1.5M | 25M–115M |
| 문항 | 500 | 451 |
| 멀티모달 | ✗ | ✓ |
| 능력 축 | 5개 (IE/MR/KU/TR/ABS) | 5개 (다른 5개) |

토큰이 100배 늘었다. V1은 1.5M, V2는 115M이다.

논문 표에 따르면 비교한 벤치마크 중에 다섯 능력을 전부 다루는 건 V2 하나라고 한다.

## **2. 다섯 가지 능력**

한 환경에서 반복해서 일하고 나면 숙련된 동료는 뭘 익히게 되는가? 라는 질문에서 시작한다.

다섯 가지로 나눈다.

1. Static State Recall : 중요한 랜드마크, 페이지 레이아웃, 모듈 기능, 상태 사이의 작은 차이를 기억한다
2. Dynamic State Tracking : 환경의 월드 모델처럼 동작한다. 상태와 행동이 주어지면 환경이 어떻게 바뀌는지 안다
3. Workflow Knowledge : 그 환경에서 자주 하는 작업의 단계를 안다
4. Environment Gotchas : 그 환경에서 반복되는 함정을 알고 피한다
5. Premise Awareness : 다른 환경에서는 맞지만 지금 환경에서는 틀린 전제를 알아챈다

V1의 다섯 가지(정보 추출, 다중 세션 추론, 지식 갱신, 시간 추론, 회피)랑 겹치는 게 거의 없다. V1은 사실을 묻고 V2는 절차와 함정을 묻는다.

[서베이](https://momozzing.github.io/paper%20review/Memory-in-the-Age-of-AI-Agents-Paper-review/)의 Functions 분류로 보면 V1은 factual memory를, V2는 experiential memory를 잰다. 앞에서 본 메모리 논문들은 거의 다 factual 쪽이었다.

-> Gotchas랑 Premise Awareness는 처음 보는 축이다. "이 환경에서 자주 깨지는 것", "다른 데서는 됐는데 여기선 안 되는 것"을 묻는 벤치마크는 못 봤다.

문항은 WorkArena-ServiceNow(46.8%), WebArena-CMS(20.2%), WebArena-OneStopShop(18.4%), WebArena-Reddit(14.6%)에서 나온다. 형식은 단답(50.1%), 자유형(34.6%), 객관식(15.3%)이다.

## **3. 평가 형식**

평가를 맥락 수집(context gathering) 과제로 둔다.

메모리 시스템은 API 두 개를 지원해야 한다.

```
Insert(h)   — 궤적을 순차 삽입
Query(q)    — 최종 메모리에 질의
```

궤적 건초더미를 순서대로 전부 넣고, 질문으로 질의해서 나온 맥락을 받는다. 그걸 고정된 리더 모델(Qwen3.5-9B)이 읽고 답한다.

메모리가 쓸 만한 증거를 돌려주는지를 리더 성능과 떼어서 재려는 것이다. 논문도 한계 절에서 이건 일부러 한 설계라고 밝힌다. 엔드투엔드 태스크 성공률은 재지 않는다.

앞에서 본 [Anatomy 리뷰](https://momozzing.github.io/paper%20review/Anatomy-of-Agentic-Memory-Paper-review/)에서는 메모리를 재는 건지 컨텍스트 길이를 재는 건지 따졌는데, 여기서는 리더를 고정해서 차이가 메모리에서만 나오게 했다.

## **4. 난이도 검증**

1. 궤적 없이 풀 수 있나?

질문만 주고 프런티어 LLM에 물어봤다. 제일 좋은 모델이 14.1%다.

공개 지식이나 파라미터 지식만으로는 대부분 못 푼다고 한다.

2. 정답이 들어 있는 궤적을 주면 풀리나?

오라클 접근을 주면 long-context 프롬프팅 점수가 많이 오르지만, 그래도 한계가 있다고 한다. 궤적이 모델 컨텍스트 창보다 크기 때문이다.

-> 25M~115M 토큰이니 그럴 만하다. 컨텍스트를 늘려서 풀 수 있는 벤치마크는 아닌 것 같다.

## **5. AgentRunbook**

논문이 같이 내놓은 메모리 방법 두 개다.

### **5.1 AgentRunbook-R (RAG)**

넣을 때 구조화된 메모리 항목을 뽑아두고 질의할 때 검색한다. 단위가 다른 지식 풀 세 개를 둔다.

1. raw state slice : 궤적 상태를 중심으로 자른 창. 국소 UI 관측이랑 주변 행동. 세밀한 시각·텍스트 증거를 그대로 남김
2. state transition event : 연속된 상태에서 뽑은 이벤트. 행동이 환경을 어떻게 바꾸는지. 환경 월드 모델의 증거를 쌓음
3. procedure and hint note : 궤적 단위 노트. 재사용할 수 있는 워크플로, 탐색 패턴, 환경별 함정

질의할 때는 LLM 컨트롤러가 질의랑 지금 메모리 스냅샷을 보고 풀마다 검색 질의를 만든다.

뒤에서 볼 [Rate-Distortion 리뷰](https://momozzing.github.io/paper%20review/Rate-Distortion-Memory-Compaction-Paper-review/)의 다중 충실도(P-fid)를 구현한 모양이다. 원본 조각이랑 추상 노트를 같이 둔다.

### **5.2 AgentRunbook-C (코딩 에이전트)**

이쪽은 방식이 다르다.

검색을 고정된 벡터 검색 파이프라인으로 하지 않고, 궤적을 파일로 그대로 저장한 다음 코딩 에이전트가 질의할 때 직접 찾아보고 골라내게 한다.

기성 코딩 에이전트는 메모리 모듈로 쓰라고 만든 게 아니라서 너무 많이 찾거나, 너무 적게 찾거나, 비효율적으로 본다고 한다. 그래서 가벼운 장치 세 개를 붙인다.

- workflow 문서 : 메모리 모듈로 행동하라는 지시와 증거 수집 단계
- query-time manifest : 지금 메모리 구조 요약. 자세히 보기 전에 관련 궤적을 추리는 데 씀
- helper script : 상태 구간 보기, 궤적 안 검색 같은 자주 쓰는 연산

나중에 볼 [ReFind](https://momozzing.github.io/paper%20review/ReFind-Paper-review/)는 원본을 두고 에이전트가 검색하게 하는데, 여기서는 원본을 파일시스템에 두고 코딩 에이전트에게 맡긴다.

## **6. 결과**

리더는 항상 Qwen3.5-9B다.

| 방법 | Small 전체 | Medium 전체 | 지연 |
|---|---:|---:|---:|
| No retrieval | 0.013 | 0.013 | 0s |
| RAG: query→slice | 0.428 | 0.381 | 0.1s |
| RAG: query→slice + notes | 0.510 | 0.459 | 0.1s |
| AgentRunbook-R | 0.586 | 0.570 | ~26s |
| Vanilla Codex | 0.699 | 0.687 | — |
| AgentRunbook-C | 0.749 | 0.701 | 높음 |

표를 보면,

1. no-retrieval이 1.3%다. 메모리 없이는 리더가 못 푼다.
2. 노트를 더하면 8점 넘게 오른다. slice만 쓰면 0.428, slice+notes는 0.510이다. 국소 관측만으로는 부족하다.
3. AgentRunbook-R이 RAG 중 제일 좋은 것보다 높다. 0.586 / 0.570이다. abstract에 나온 48.5%는 전체 평균 기준이다.
4. 코딩 에이전트가 제일 높다. 평균 72.5%로 vanilla Codex(69.3%)보다 높다. 붙인 장치들이 +3점 정도 기여한다.

ablation을 보면 풀마다 역할이 다르다고 한다.

- raw slice 풀은 static 질문에 중요
- event 풀을 빼면 static, dynamic, gotchas가 전부 나빠짐
- Workflow 질문은 국소 관측만 찾는 것보다, 궤적 경험을 재사용할 수 있는 이벤트랑 노트로 묶어둘 때 좋아짐

### **6.1 정확도-지연 절충**

메모리 컨트롤러의 reasoning effort가 전체 질의 지연에 크게 영향을 준다고 한다.

- AgentRunbook-R : 정확도는 중간, 지연은 낮음. 약 26초이고 thinking을 끄면 훨씬 낮아짐. 질의 효율이 중요하면 이쪽
- AgentRunbook-C : 정확도를 더 올리지만 지연이 큼

코딩 에이전트는 워크플로 안내, manifest, 궤적 검사 도구랑 같이 줄 때 메모리 컨트롤러로 더 잘 동작한다고 한다.

## **7. 지금 관점: V1과 V2 중 무엇을 쓸 것인가**

둘은 갈아타는 관계가 아니고 재는 대상이 다르다.

V1은 사용자에 대한 사실을 기억하는지를 115k–1.5M 토큰 규모에서 잰다. 개인화 챗봇이나 선호 추적에 맞고, 앞 리뷰들에서 Mem0, Zep, A-MEM이 겨룬 곳이다.

V2는 환경에 대한 경험을 익히는지를 25M–115M 토큰 규모에서 잰다. 웹·도구 에이전트나 반복 업무 자동화에 맞고, 여기서 평가된 시스템은 거의 없다.

업무용 챗봇이라면 V1이 여전히 기준선일 것 같다. 사용자 정보를 기억하고 갱신하는 게 주로 필요한 거라서, V1의 KU(지식 갱신)랑 ABS(회피)가 바로 해당된다.

다만 V2가 보는 축은 앞에서 본 논문들에서 거의 비어 있었다. experiential memory를 제대로 잰 건 앞에서 본 [Experience-Following](https://momozzing.github.io/paper%20review/Experience-Following-Paper-review/) 하나였고, 그것도 직접 만든 태스크였다.

-> Gotchas랑 Premise Awareness는 도구 호출을 반복하는 에이전트에 바로 해당하는 것 같다. 예를 들면 MCP 도구가 특정 조건에서 실패하는 패턴을 에이전트가 익히는지 같은 것.

AgentRunbook-R의 3풀 구조는 규모가 작아도 가져와 볼 만하다. 원본 조각, 상태 전이 이벤트, 절차 노트를 따로 저장하고 따로 검색한다. ablation을 보면 인덱스 하나에 다 넣는 것보다 낫고, 풀마다 맡는 질문 유형이 다르다.

26초 지연도 참고할 숫자다. AgentRunbook-R이 "지연이 낮은" 쪽인데도 26초다.

-> thinking을 끄면 내려간다고는 하지만, 이 규모(25M+ 토큰)에서 실시간 응답은 어려울 것 같다.

## **8. Conclusion**

conclusion 부분을 보면, 메모리 시스템은 에이전트가 특정 환경을 잘 다루는 숙련자가 되도록 도와야 한다고 한다.

다섯 능력을 다 다루고, 멀티모달 웹 에이전트 이력으로 1억 토큰이 넘는 맥락 깊이까지 벤치마크를 키웠다.

논문이 밝힌 한계는 세 가지다.

1. 웹 에이전트만 다룬다. 코딩 에이전트, 컴퓨터 사용 에이전트, 도메인 특화 기업 에이전트는 포함하지 않는다
2. 미리 모아둔 궤적 이력으로 평가한다. 재현성과 통제된 비교를 위한 설계지만, 에이전트 자신의 행동이 바뀌면서 생기는 분포 변화는 담지 못한다
3. 엔드투엔드 태스크 성공률이 아니다. 고정 리더에게 쓸 만한 증거를 돌려주는지만 잰다. 메모리만 떼어서 보려고 일부러 그렇게 했다

2번은 앞에서 본 [Experience-Following 리뷰](https://momozzing.github.io/paper%20review/Experience-Following-Paper-review/)와 겹친다. 거기서는 에이전트가 만든 경험이 다시 에이전트 행동을 바꾸는 루프를 쟀는데, V2는 그 루프를 끊고 고정된 이력으로 평가한다.

여태까지 메모리 벤치마크가 사용자에 대한 사실을 기억하는지를 봤다면, V2는 에이전트가 환경에서 겪은 경험을 익히는지를 본다.

다음은 [MemFail](https://momozzing.github.io/paper%20review/MemFail-Paper-review/)이다. 메모리 시스템을 요약·저장·검색 세 연산으로 나눠서, 틀린 답이 어느 단계에서 나왔는지 찾아내는 진단 벤치마크다.
