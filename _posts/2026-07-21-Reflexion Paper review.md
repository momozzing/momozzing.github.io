---
title: "Reflexion Paper review"
excerpt: "실패를 반성문으로 남겨 다음 시도에 써먹는 에이전트. 가중치 업데이트 없는 '말로 하는 강화학습'으로 HumanEval 91%를 찍었다."
categories:
  - Paper review
tags:
  - Large Language Model
  - NLP
  - Agent
  - Paper review
toc: true
toc_sticky: true
---

Reflexion: Language Agents with Verbal Reinforcement Learning

[https://arxiv.org/abs/2303.11366](https://arxiv.org/abs/2303.11366)

Reflexion은 Northeastern + MIT + Princeton에서 나온 논문이다. (NeurIPS 2023)

[지난 ReAct 리뷰](https://momozzing.github.io/paper%20review/ReAct-Paper-review/) 마지막에 "Shunyu Yao의 논문은 계속 따라가 볼 가치가 있다"고 했는데, 바로 그 Shunyu Yao가 공저자로 참여한 후속 연구다.

ReAct가 Thought → Action → Observation 루프를 만들었다면, Reflexion은 그 루프에 **실패에서 배우는 능력**을 넣는다.

어떻게 배우는지 알아보자.

## **Introduction**

LLM 에이전트도 시행착오로 배우게 하고 싶다. 그런데 전통적인 강화학습(RL)로 하려면 문제가 있다. 샘플이 대량으로 필요하고, 모델 파인튜닝 비용이 비싸다. LLM 에이전트는 보통 API 뒤에 있는 거대 모델이라 가중치 업데이트 자체가 어렵기도 하다.

Reflexion의 답은 **가중치가 아니라 말로 강화하자**는 것이다.

실패하면 "왜 실패했는지"를 언어로 반성하고, 그 반성문을 메모리에 쌓아두고, 다음 시도의 컨텍스트에 넣어준다. gradient 업데이트가 한 번도 없는데 시도할수록 잘해진다.

![세 가지 태스크에서의 Reflexion 동작 (논문 Figure 1)](https://momozzing.github.io/assets/images/reflexion/fig1-three-domains.png)

의사결정, 코딩, 추론 세 도메인 모두 같은 패턴이다. 시도 → 실패 신호 → 반성("팬이 stoveburner 1에 없었으니 2를 봐야 했다") → 다음 시도에서 교정.

## **Reflexion: reinforcement via verbal reflection**

프레임워크는 LLM 3개(역할 분담)와 메모리로 구성된다.

- **Actor**: 텍스트와 행동을 생성하는 모델. CoT나 ReAct를 그대로 Actor로 쓴다
- **Evaluator**: Actor가 만든 trajectory에 점수를 매기는 모델. 태스크에 따라 exact match, 휴리스틱 룰, 또는 LLM 판정을 쓴다
- **Self-Reflection**: 실패 신호 + trajectory를 보고 "무엇이 잘못됐고 다음엔 어떻게 해야 하는지"를 언어로 생성하는 모델

메모리는 두 층이다.

- 단기 기억: 현재 trial의 trajectory
- 장기 기억: 지금까지 쌓인 반성문들 (최대 3개 정도로 제한)

![Reflexion 구조와 알고리즘 (논문 Figure 2)](https://momozzing.github.io/assets/images/reflexion/fig2-architecture.png)

알고리즘은 단순하다. 시도하고, 평가하고, 실패면 반성문을 메모리에 추가하고, 통과하거나 max trial에 걸릴 때까지 반복한다.

핵심 통찰은 반성문이 **스칼라 reward보다 훨씬 정보량이 많다**는 것이다. RL의 reward는 "0점이었다"만 알려주지만, 반성문은 "어디서 무엇을 잘못했고 대신 뭘 해야 하는지"를 알려준다. 이게 논문 제목의 verbal reinforcement다.

피드백 소스도 유연하다. 환경이 주는 binary reward(외부)든, 스스로 만든 unit test 결과(내부)든, 사람이 주는 자유형 텍스트든 전부 반성의 재료로 쓸 수 있다.

## **Decision Making: ALFWorld**

ReAct 리뷰에서 봤던 그 텍스트 집안일 시뮬레이터다. Actor로 ReAct를 쓰고 Reflexion을 얹었다.

![ALFWorld 성공률과 실패 원인 분석 (논문 Figure 3)](https://momozzing.github.io/assets/images/reflexion/fig3-alfworld.png)

12번의 trial을 거치며 134개 태스크 중 130개(97%)를 풀어낸다. ReAct 단독은 75% 부근에서 정체되고 더 이상 나아지지 않는다.

오른쪽 그래프가 더 재미있다. 실패 원인을 분류해보면 ReAct 단독은 hallucination 비율이 22%에 수렴한 채 회복하지 못하는데, Reflexion은 trial이 거듭될수록 hallucination과 비효율적 계획이 거의 사라진다.

장기 기억이 도움이 되는 경우는 두 가지다. 긴 trajectory 초반의 실수를 반성으로 짚어내는 것, 그리고 "어디를 이미 뒤져봤는지"를 여러 trial에 걸쳐 기억해서 방을 체계적으로 수색하는 것. 같은 실수를 반복하지 않게 된다.

## **Reasoning: HotpotQA**

100개의 multi-hop 질문으로 추론 능력 개선을 테스트한다. 피드백은 exact match binary 신호뿐이고, 반성문이 이 빈약한 신호를 증폭하는 구조다.

![HotpotQA 결과와 ablation (논문 Figure 4)](https://momozzing.github.io/assets/images/reflexion/fig4-hotpotqa.png)

주목할 부분이 baseline의 재시도다. ReAct-only, CoT-only에 temperature 0.7로 재시도를 시켜도 **한 번 틀린 문제는 끝까지 못 푼다**. 그냥 다시 굴리는 것으로는 안 되고, 뭐가 틀렸는지 언어로 짚어줘야 개선된다는 것이다.

ablation도 이 지점을 판다. 반성문 없이 직전 trajectory만 컨텍스트에 넣어주는 episodic memory(EPM)와 비교하면, self-reflection이 8%p를 더 얻는다. 기억을 주는 것과 교훈을 주는 것은 다르다.

## **Programming: HumanEval, MBPP, LeetcodeHardGym**

코딩은 Reflexion에게 유리한 도메인이다. 스스로 unit test를 만들어 실행하면 **근거 있는 내부 피드백**을 얻을 수 있기 때문이다. CoT로 테스트를 생성하고, AST로 문법 검증을 거쳐 최대 6개를 추린다. 정답 테스트를 참조하지 않으므로 pass@1 자격을 유지한다.

![프로그래밍 벤치마크 결과 (논문 Table 1)](https://momozzing.github.io/assets/images/reflexion/table1-programming.png)

HumanEval Python에서 pass@1 91.0으로 당시 SOTA였던 GPT-4(80.1)를 크게 넘었다. Rust에서도 60.0 → 68.0으로 오르는 걸 보면 언어에 종속된 방법이 아니다. GPT-4의 사전학습 컷오프 이후 출제된 Leetcode hard 문제(LeetcodeHardGym, 이 논문이 새로 만든 벤치마크)에서도 7.5 → 15.0으로 두 배가 된다.

MBPP Python은 77.1로 GPT-4(80.1)보다 낮다. 이 논문도 안 되는 걸 숨기지 않는다. 원인은 자기가 만든 테스트의 신뢰도다. 테스트를 다 통과했는데 실제로는 틀린 코드일 확률(false positive)이 HumanEval은 1.4%인데 MBPP는 16.3%나 된다. 평가자가 부실하면 반성도 부실해진다.

![테스트 생성/반성 ablation (논문 Table 3)](https://momozzing.github.io/assets/images/reflexion/table3-ablation.png)

HumanEval Rust 최고 난도 50문제로 한 ablation이 이 논문에서 제일 흥미로운 표다. 테스트 실행 없이 반성만 시키면 0.60 → 0.52로 **base보다 오히려 나빠진다**. 근거 없는 반성은 해롭다는 것. 테스트(근거)와 반성(해석)이 둘 다 있어야 0.68로 오른다.

## **요즘 코딩 에이전트가 이 구조 그대로다**

지금 관점에서 보면 Reflexion 루프는 낯설지 않다.

코딩 에이전트가 테스트를 돌리고, 실패하면 에러 로그를 읽고, "아 이 부분에서 타입이 안 맞았구나"라고 정리한 뒤 코드를 고쳐서 다시 시도하는 것. 정확히 Actor(코드 생성) → Evaluator(테스트 실행) → Self-Reflection(에러 분석) → 재시도다.

### **LangGraph로 Reflexion 루프 짜보기**

ReAct 리뷰에서는 `create_agent` 한 줄이면 됐다. Reflexion은 사정이 다르다. agent 루프 **바깥에** 평가와 반성을 한 겹 더 감는 구조라 원버튼 API가 없고, LangGraph의 `StateGraph`로 그래프를 직접 짠다.

논문의 코딩 세팅(Actor 생성 → unit test 실행 → 반성 → 재시도)을 최소 구현하면 이렇다.

```python
from typing import TypedDict
from langgraph.graph import StateGraph, START, END

class State(TypedDict):
    task: str
    code: str
    test_result: str
    reflections: list[str]   # 논문의 mem (장기 기억)
    trials: int

def actor(state: State):
    lessons = "\n".join(state["reflections"]) or "없음"
    prompt = f"""문제: {state['task']}
지난 시도에서 얻은 교훈:
{lessons}
교훈을 반영해서 코드를 작성해라."""
    return {"code": llm.invoke(prompt).content, "trials": state["trials"] + 1}

def evaluator(state: State):
    return {"test_result": run_tests(state["code"])}   # 실제 실행 = 단단한 근거

def self_reflection(state: State):
    prompt = f"""코드:
{state['code']}
테스트 결과:
{state['test_result']}
무엇이 잘못됐고 다음 시도에서 어떻게 고쳐야 하는지 한 문단으로 정리해라."""
    return {"reflections": state["reflections"] + [llm.invoke(prompt).content]}

def should_retry(state: State) -> str:
    if "failed" not in state["test_result"] or state["trials"] >= 3:
        return "done"
    return "retry"

builder = StateGraph(State)
builder.add_node("actor", actor)
builder.add_node("evaluator", evaluator)
builder.add_node("self_reflection", self_reflection)

builder.add_edge(START, "actor")
builder.add_edge("actor", "evaluator")
builder.add_conditional_edges(
    "evaluator", should_retry, {"done": END, "retry": "self_reflection"}
)
builder.add_edge("self_reflection", "actor")

graph = builder.compile()
result = graph.invoke({
    "task": "두 문자열이 애너그램인지 판별하는 함수를 작성해라",
    "code": "", "test_result": "", "reflections": [], "trials": 0,
})
```

### **논문 ↔ 구현 매핑**

| Reflexion 논문 | LangGraph 구현 |
|---|---|
| Actor (M_a) | `actor` 노드 — 반성문을 프롬프트에 주입해서 생성 |
| Evaluator (M_e) | `evaluator` 노드 + `should_retry` 분기 |
| Self-Reflection (M_sr) | `self_reflection` 노드 |
| mem (장기 기억) | state의 `reflections` 리스트 |
| max trials | `trials` 카운터 (그래프 전체로는 `recursion_limit`) |
| 통과 시 종료 | conditional edge의 `END` |

verbal reinforcement가 코드로는 어디에 있는지 보면, `actor` 프롬프트에 `reflections`를 끼워 넣는 세 줄이 전부다. 가중치 업데이트는 어디에도 없고, 학습된 것은 전부 state에 쌓인 문장들이다.

이 패턴은 [LangGraph 공식 튜토리얼](https://langchain-ai.github.io/langgraph/tutorials/reflexion/reflexion/)에도 Reflexion이라는 이름 그대로 올라가 있다. 논문의 구조가 프레임워크의 표준 레시피가 된 셈이다.

에이전트에 붙는 memory 설계도 마찬가지다. 세션에서 얻은 교훈을 파일로 남겨두고 다음 세션 컨텍스트에 주입하는 패턴은 Reflexion의 장기 기억을 그대로 닮았다. 가중치는 그대로인데 컨텍스트가 학습되는 것, 요즘 말로 하면 in-context learning을 학습 루프로 쓰는 것이다.

Table 3의 교훈도 현재진행형이다. 판정이 부정확하면(멋대로 만든 테스트, 어설픈 LLM-judge) self-correction은 오히려 성능을 깎는다. 에이전트 루프를 설계할 때 "얼마나 잘 반성하느냐"보다 "반성의 근거가 얼마나 단단하냐"를 먼저 챙겨야 하는 이유다.

## **Limitations**

- Evaluator의 정확도에 전체가 의존한다. 코딩의 false positive 문제가 대표적
- 반성문 메모리를 최대 3개 정도로 잘라서 쓴다. 더 긴 학습을 하려면 요약이나 검색이 필요할 것
- 국소최적에 빠질 수 있다. 반성이 잘못된 방향을 가리키면 그쪽으로 계속 판다

## **Conclusion**

ReAct가 행동하면서 생각하는 법을 만들었다면, Reflexion은 실패에서 배우는 법을 더했다.

가중치를 하나도 안 바꾸고, 실패 경험을 반성문으로 바꿔 메모리에 쌓는 것만으로 시도할수록 잘해지는 에이전트가 된다. 스칼라 reward 대신 언어를 학습 신호로 쓰는 verbal RL이라는 프레임이 이 논문의 기여다.

단, 반성은 근거가 있을 때만 힘을 쓴다. 근거 없는 반성은 안 하느니만 못하다는 Table 3의 결과는 에이전트를 만드는 사람이라면 기억해둘 만하다.

Shunyu Yao 추적은 계속된다.
