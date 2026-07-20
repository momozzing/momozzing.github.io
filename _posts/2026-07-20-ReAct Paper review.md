---
title: "ReAct Paper review"
excerpt: "지금 모든 LLM Agent의 뼈대가 된 논문. Thought → Action → Observation 루프부터 LangChain create_agent까지 뜯어본다."
categories:
  - Paper review
tags:
  - Large Language Model
  - NLP
  - Agent
  - Paper review
mathjax: true
toc: true
toc_sticky: true
---

ReAct: Synergizing Reasoning and Acting in Language Models

[https://arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629)

ReAct는 프린스턴 + Google Brain에서 나온 논문이다. (ICLR 2023)

지금 나오는 LLM Agent들의 근본이 되는 논문이다.

LangChain, LangGraph의 ReAct agent가 다 여기서 나왔다.

어떻게 동작하는지 알아보자.

## **Introduction**

LLM은 reasoning(CoT prompting)과 acting(action plan 생성)이 각각 따로 연구가 되어왔다.

CoT는 모델 내부 지식만으로 생각한다. 외부와 단절된(closed) 상태라 hallucination이 생기고, 초반 reasoning이 틀리면 계속 틀린 방향으로 간다. (error propagation)

Act-only는 계획과 목표 추적 없이 행동만 하니까 복잡한 task를 못 푼다.

본 논문은 action space를 확장해서 언어로 된 "thought"도 하나의 action처럼 생성하게 한다.

thought는 환경을 바꾸지 않고 context만 업데이트한다는 것이 포인트다.

## **ReAct: Synergizing Reasoning + Acting**

Thought → Action → Observation 루프를 반복한다.

1. Thought: 현재 context 보고 뭘 해야할지 reasoning (목표 분해, 계획 수정, 예외 처리)
2. Action: 외부 환경에 액션 실행
3. Observation: 액션 결과를 context에 추가하고 다시 Thought로

형식적으로 보면 원래 action space $A$에 언어 공간 $L$을 더해 $\hat{A} = A \cup L$로 확장한 것이다. thought $\hat{a} \in L$은 환경에 아무 영향을 주지 않고 context만 갱신한다. 대신 $L$은 무한한 공간이라 이 확장된 space에서의 학습이 어렵고, 강한 언어 prior를 가진 LLM이 있어야 동작한다.

논문이 예시로 드는 thought의 역할은 다양하다.

- 목표를 subgoal로 분해하고 action plan 수립
- 태스크에 필요한 상식 주입 ("후추통은 캐비닛이나 조리대에 있을 것")
- observation에서 중요한 부분 추출
- 진행 상황 추적과 plan 전환
- 예외 처리와 plan 수정

![4가지 프롬프팅 방법 비교 (논문 Figure 1)](https://momozzing.github.io/assets/images/react/fig1-react-comparison.png)

CoT(1b)는 그럴듯하게 추론하다가 환각으로 틀리고, Act-only(1c)는 검색만 하다가 답을 못 찾는다. ReAct(1d)는 검색 결과를 보고 생각을 수정해가며 정답에 도달한다.

(2)의 ALFWorld도 같은 패턴이다. Act-only(2a)는 후추통을 찾으려고 서랍과 싱크대를 뒤지다가 안 되는 행동만 반복한다("Nothing happens"). ReAct(2b)는 Think로 "후추통은 캐비닛이나 조리대에 있을 확률이 높다"고 위치부터 추론해서 조리대에서 찾아내고, 찾은 뒤엔 "이제 서랍에 넣어야지"라고 다음 subgoal을 세워서 성공한다. 생각 없이 행동만 하면 같은 곳을 계속 맴돈다는 것을 보여주는 예시다.

few-shot 예시는 사람이 직접 작성한 trajectory(궤적 — Thought/Action/Observation으로 문제를 푸는 전체 풀이 과정 기록) 몇 개가 전부다. (HotpotQA 6개, FEVER 3개, ALFWorld 2개, WebShop 1개)

모델은 PaLM-540B 사용. 학습 없이 prompting만으로 동작.

task 성격에 따라 thought 배치를 다르게 한다.

- 지식 task(QA): 매 스텝마다 thought-action 교차 (dense)
- 의사결정 task(ALFWorld): 필요할 때만 thought 생성 (sparse) — 모델이 스스로 언제 생각할지 결정

논문은 이 설계가 주는 특징을 4가지로 정리한다.

- **A) Intuitive and easy to design**: 프롬프트 설계가 쉽다. 어노테이터가 자신이 한 행동 위에 생각을 언어로 적기만 하면 된다. 특별한 포맷 설계나 예시 선정 기법도 쓰지 않았다
- **B) General and flexible**: thought space가 자유로워서 QA, 사실 검증, 텍스트 게임, 웹 탐색처럼 액션 스페이스가 전혀 다른 태스크에 모두 적용된다
- **C) Performant and robust**: in-context 예시 1~6개만으로 새로운 태스크 인스턴스에 일반화되고, reasoning만 하거나 acting만 하는 베이스라인을 도메인 불문하고 이긴다
- **D) Human aligned and controllable**: 추론 과정이 사람이 읽을 수 있는 형태라 reasoning과 사실 여부를 그대로 검사할 수 있고, 중간에 thought를 수정해서(thought editing) 에이전트의 행동을 실시간으로 교정할 수도 있다

특히 D는 지금 관점으로 보면 agent 관측가능성(observability)과 human-in-the-loop의 원형이다. 에이전트가 왜 그 행동을 했는지 로그로 추적할 수 있는 이유가 thought를 언어로 남기기 때문이다.

요즘의 function calling agent가 이 구조를 그대로 사용하고 있다. tool call → result → 다시 생각하는 구조다.

## **Knowledge-Intensive Reasoning Tasks (HotpotQA, FEVER)**

HotpotQA는 위키 문서 두 개 이상을 넘나들어야 답이 나오는 multi-hop QA고, FEVER는 주장에 대해 SUPPORTS / REFUTES / NOT ENOUGH INFO를 판정하는 사실 검증 태스크다. 둘 다 질문(주장)만 주어지는 세팅이라, 모델이 근거를 직접 검색해서 찾아야 한다.

액션은 Wikipedia API 3개가 전부다.

- `search[entity]`: 해당 entity 페이지 첫 5문장 or 유사 페이지 top-5
- `lookup[string]`: 페이지 내 문자열 검색 (Ctrl+F 같은거)
- `finish[answer]`: 답 제출

HotpotQA EM 기준: Standard 28.7 / CoT 29.4 / Act-only 25.7 / ReAct 27.4

![PaLM-540B 프롬프팅 결과 (논문 Table 1)](https://momozzing.github.io/assets/images/react/table1-hotpotqa-fever.png)

ReAct 단독은 HotpotQA에서 CoT보다 오히려 낮다. 이 결과를 숨기지 않고 분석한 것이 이 논문의 정직한 부분이다. 반대로 FEVER에서는 ReAct(60.9)가 CoT(56.3)를 이기는데, 사실 검증은 최신의 정확한 지식을 가져오는 것이 결정적이기 때문이다.

이유는 error analysis에 나온다.

- CoT 실패의 56%가 hallucination. 대신 reasoning 구조는 유연함
- ReAct는 사실 기반(hallucination 6% vs CoT 14%)이지만, 검색 결과가 안좋으면 reasoning이 같이 망가지고(실패의 23%가 search error), 같은 thought-action을 반복하는 루프에 빠지기도 함

![ReAct vs CoT 성공/실패 모드 분석 (논문 Table 2)](https://momozzing.github.io/assets/images/react/table2-error-analysis.png)

그래서 둘을 섞는 방법을 제안한다. ReAct → CoT-SC (ReAct가 정해진 스텝(HotpotQA 7, FEVER 5) 안에 답을 못 찾으면 CoT-SC로 fallback), CoT-SC → ReAct (CoT-SC 다수결 confidence 낮으면 ReAct로 전환)

이 조합이 HotpotQA 34.2 / FEVER 64.6으로 prompting 방법 중 최고.

![CoT-SC 샘플 수에 따른 조합 방법 성능 (논문 Figure 2)](https://momozzing.github.io/assets/images/react/fig2-cotsc-combo.png)

CoT-SC 샘플 3~5개만 써도 순수 CoT-SC 21개 샘플 성능을 넘는다.

내부지식과 외부지식을 상황에 따라 골라 쓰는 것이 답이었다.

### **Finetuning**

HotpotQA에서 ReAct trajectory 3,000개로 작은 모델(PaLM-8B, 62B)을 finetuning 해봤다.

- prompting에서는 작은 모델일수록 ReAct가 4가지 방법 중 최하위 (형식 따라하기가 어려움)
- finetuning 하면 역전됨. finetuned PaLM-8B ReAct가 prompted PaLM-62B를 이기고, finetuned 62B가 prompted 540B를 이김
- Standard/CoT를 finetuning하는 건 지식 암기를 학습하는 거라 효과가 적고, ReAct finetuning은 "지식을 찾는 방법"을 학습하는 거라 일반화가 잘된다

![프롬프팅 vs 파인튜닝 스케일링 (논문 Figure 3)](https://momozzing.github.io/assets/images/react/fig3-finetuning-scaling.png)

요즘 많이 쓰는 작은 모델 + agent trajectory SFT 조합의 근거가 여기에 있다.

## **Decision Making Tasks (ALFWorld, WebShop)**

ALFWorld(텍스트 기반 집안일 시뮬레이터): ReAct 71% vs Act-only 45% vs BUTLER(IL, 학습 기반) 37%

WebShop(쇼핑 사이트 시뮬레이터): ReAct 40% vs IL/RL 기반 ~29%

in-context 예시 1~2개 프롬프팅으로 학습 기반 방법을 이겼다.

![ALFWorld / WebShop 결과 (논문 Table 3, 4)](https://momozzing.github.io/assets/images/react/table34-alfworld-webshop.png)

2022년 기준으로는 충격적인 결과다. thought가 goal을 subgoal로 분해하고 진행 상황을 추적해주는 것이 결정적이었다. Act-only는 중간에 자신이 무엇을 하고 있었는지 잊어버린다.

ablation으로 Inner Monologue 스타일(ReAct-IM)과도 비교한다. IM처럼 "환경 상태 관찰 + 목표 확인" 수준의 생각만 하게 하면 ALFWorld가 71 → 53으로 떨어진다. 목표를 subgoal로 분해하는 것과 물건이 어디 있을지 상식으로 추론하는 것이 사라지기 때문이다. 그냥 생각을 시키는 것이 아니라 어떤 생각을 하게 하느냐가 중요하다는 증거다.

다만 WebShop에서 인간 전문가(점수 82.1 / 성공률 59.6)와는 갭이 크다. 사람은 상품 탐색과 질의 재구성을 훨씬 능동적으로 한다. 2022년의 프롬프팅으로는 아직 못 따라가는 부분이었다.

부록(A.1)에서 GPT-3(text-davinci-002)로도 재현하는데, HotpotQA 30.8 vs PaLM 29.4, ALFWorld 78.4 vs 70.9로 오히려 더 잘 된다. instruction following으로 파인튜닝된 모델이라 그럴 것으로 추정한다. ReAct가 특정 모델에 종속된 방법이 아니라는 근거다.

## **LangChain의 create_agent는 실제로 어떻게 동작하는가**

논문의 Thought → Action → Observation 루프가 코드로는 어떻게 구현되는지 보자.

LangChain v1의 `create_agent`로 논문의 HotpotQA 세팅을 흉내내면 이렇다.

```python
from langchain.agents import create_agent
from langchain_core.tools import tool

@tool
def search(entity: str) -> str:
    """위키피디아에서 entity를 검색해 첫 5문장을 반환한다."""
    return wiki_api.search(entity)

@tool
def lookup(keyword: str) -> str:
    """현재 페이지에서 keyword가 포함된 다음 문장을 반환한다."""
    return wiki_api.lookup(keyword)

agent = create_agent(
    model="anthropic:claude-sonnet-4-6",
    tools=[search, lookup],
    system_prompt="질문에 답하기 위해 검색 도구를 사용해라.",
)

result = agent.invoke(
    {"messages": [{"role": "user", "content": "Apple Remote와 호환되는 다른 기기는?"}]}
)
```

논문의 `finish[answer]`는 tool로 만들지 않아도 된다. 이유는 뒤에서 설명한다.

### **내부 루프**

create_agent는 model 노드와 tools 노드를 가진 graph를 만든다.

model 노드가 메시지 리스트로 LLM을 호출하고, 응답 AIMessage에 tool_calls가 있으면 tools 노드가 실행되어 결과를 ToolMessage로 메시지 리스트에 추가한다. 그리고 다시 model 노드 호출.

tool_calls가 없는 응답이 나올 때까지 반복한다.

의사코드로 표현하면 이게 전부다.

```python
def agent_loop(messages):
    while True:
        ai_msg = model.invoke(messages)      # Thought + Action 생성
        messages.append(ai_msg)

        if not ai_msg.tool_calls:            # Action이 없으면 = finish
            return messages

        for tc in ai_msg.tool_calls:         # Action 실행
            result = tools[tc["name"]].invoke(tc["args"])
            messages.append(ToolMessage(result, tool_call_id=tc["id"]))
                                             # Observation 추가
```

### **논문 ↔ 구현 매핑**

| ReAct 논문 | create_agent |
|---|---|
| Thought | AIMessage의 `content` (tool_calls와 같이 생성됨) |
| Action | AIMessage의 `tool_calls` |
| Observation | `ToolMessage` |
| `finish[answer]` | tool_calls 없는 AIMessage = 루프 종료 조건 |
| trajectory (context 누적) | `messages` 리스트 |

### **논문과 달라진 점**

원논문(2022)은 function calling이 없던 시절이라 순수 텍스트로 동작했다.

```
Thought 1: Apple Remote를 검색해서 호환 기기를 찾아야겠다.
Act 1: Search[Apple Remote]
Obs 1: The Apple Remote is a remote control ...
```

이 텍스트를 정규식으로 파싱해서 `Search[...]`를 뽑아 실행했다. 그래서 모델이 형식을 조금만 틀리면 파싱이 깨졌다. (finetuning 섹션에서 작은 모델이 ReAct 형식을 못 따라했던 이유)

지금은 모델의 native function calling으로 Action이 구조화된 JSON(`tool_calls`)으로 나오니 파싱이 필요없다.

sparse thought도 자연스럽게 구현된다. 모델이 `content` 없이 `tool_calls`만 뱉으면 Act-only 스텝이고, `content`를 채우면 Thought가 있는 스텝이다. 논문에서 "모델이 스스로 언제 생각할지 결정한다"고 했던 그 부분.

결국 create_agent는 ReAct 루프에서 텍스트 파싱을 function calling으로 교체하고 graph로 감싼 것이다. 우리가 만드는 agent들도 본질은 2022년 이 논문의 while문이다.

## **Conclusion**

생각만 하면 환각이 생기고, 행동만 하면 계획이 없다. 둘을 섞으니 서로 보완이 된다.

단독으로 만능은 아니며(HotpotQA에서는 CoT에 지기도 한다), 내부지식 fallback이나 finetuning으로 보완해야 한다.

2022년의 이 단순한 while문이 4년이 지난 지금도 모든 LLM Agent의 뼈대다. 프레임워크가 아무리 화려해져도 결국 Thought → Action → Observation 루프 위에 서 있다. Agent를 만들다 길을 잃으면 이 논문으로 돌아오면 된다.

덧붙이면 1저자 Shunyu Yao는 이후 Tree of Thoughts를 내고 SWE-bench에도 참여한다. 이 사람의 논문은 계속 따라가 볼 가치가 있다.