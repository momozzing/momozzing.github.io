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

### **LangGraph의 Reflexion 에이전트**

ReAct는 `create_agent` 한 줄로 구현이 된다. 하지만 Reflexion는 다르다. agent 루프 **바깥에** 평가와 반성을 두는 구조라 LangGraph로 그래프를 직접 짠다.

LangChain이 공식 블로그 [Reflection Agents](https://www.langchain.com/blog/reflection-agents)에서 이 계열을 세 단계로 정리해뒀다. 단순한 것부터 Basic Reflection(생성 ↔ 비평 반복), **Reflexion**(비평을 구조화하고 외부 근거로 grounding), LATS(트리 탐색까지 확장, 이것도 Shunyu Yao 계보다) 순서다.

그중 Reflexion 에이전트는 세 노드로 구성된다. 이름만으로는 감이 안 오니 노드별 실제 구현을 보자. (코드는 블로그가 링크한 공식 노트북에서 핵심만 추렸다)

**0. 준비물** — 모델, 검색 도구, 그리고 세 노드가 공유하는 프롬프트다.

```python
from datetime import datetime
from pydantic import BaseModel, Field
from langchain_anthropic import ChatAnthropic
from langchain_community.tools.tavily_search import TavilySearchResults
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

llm = ChatAnthropic(model="claude-sonnet-4-6")
tavily_tool = TavilySearchResults(max_results=5)

actor_prompt = ChatPromptTemplate.from_messages([
    ("system",
     "You are expert researcher. Current time: {time}\n"
     "1. {first_instruction}\n"
     "2. Reflect and critique your answer. Be severe to maximize improvement.\n"
     "3. Recommend search queries to research information and improve your answer."),
    MessagesPlaceholder(variable_name="messages"),
]).partial(time=lambda: datetime.now().isoformat())
```

**1. Responder** — 이 구현의 심장은 프롬프트가 아니라 출력 스키마다. 답변만 생성하는 것이 아니라 자기 비평과 검색 쿼리까지 하나의 구조체로 강제한다.

```python
class Reflection(BaseModel):
    missing: str = Field(description="Critique of what is missing.")
    superfluous: str = Field(description="Critique of what is superfluous")

class AnswerQuestion(BaseModel):
    """답변, 자기 비평, 개선용 검색 쿼리를 한 번에 생성한다."""
    answer: str = Field(description="~250 word detailed answer to the question.")
    reflection: Reflection = Field(description="Your reflection on the initial answer.")
    search_queries: list[str] = Field(
        description="1-3 search queries for researching improvements "
        "to address the critique of your current answer."
    )

class ResponderWithRetries:
    """스키마 검증에 실패하면 에러를 보여주고 재시도시키는 래퍼"""
    def __init__(self, runnable, validator):
        self.runnable, self.validator = runnable, validator

    def respond(self, state: list):
        for attempt in range(3):
            response = self.runnable.invoke({"messages": state})
            try:
                self.validator.invoke(response)
                return response
            except ValidationError as e:
                state = state + [response, ToolMessage(
                    content=f"{repr(e)}\n\nPay close attention to the function schema.",
                    tool_call_id=response.tool_calls[0]["id"])]
        return response

initial_chain = actor_prompt.partial(
    first_instruction="Provide a detailed ~250 word answer."
) | llm.bind_tools(tools=[AnswerQuestion])

first_responder = ResponderWithRetries(
    runnable=initial_chain,
    validator=PydanticToolsParser(tools=[AnswerQuestion]),
)
```

한 번의 호출에서 "답변 + 뭐가 부족한지(missing) + 뭐가 과한지(superfluous) + 그래서 뭘 검색할지"가 전부 나온다. 반성을 자유 텍스트에 맡기지 않고 스키마로 강제한 것이 포인트다.

**2. Execute Tools** — Responder가 뽑은 쿼리를 실제 검색으로 실행해서 외부 근거를 가져온다.

```python
def run_queries(search_queries: list[str], **kwargs):
    """Run the generated queries."""
    return tavily_tool.batch([{"query": query} for query in search_queries])

execute_tools = ToolNode([
    StructuredTool.from_function(run_queries, name=AnswerQuestion.__name__),
    StructuredTool.from_function(run_queries, name=ReviseAnswer.__name__),
])
```

**3. Revisor** — Responder와 같은 스키마를 상속받되, 인용(references)을 추가로 강제한다.

```python
class ReviseAnswer(AnswerQuestion):
    """검색 근거를 인용하며 이전 답변을 수정한다."""
    references: list[str] = Field(
        description="Citations motivating your updated answer."
    )

revise_instructions = """Revise your previous answer using the new information.
- 이전 비평을 반영해 부족한 정보를 채워라
- 반드시 번호 인용([1], [2])을 달아 검증 가능하게 하라
- 과잉 정보를 걷어내고 250단어를 넘지 마라"""

revision_chain = actor_prompt.partial(
    first_instruction=revise_instructions
) | llm.bind_tools(tools=[ReviseAnswer])

revisor = ResponderWithRetries(
    runnable=revision_chain,
    validator=PydanticToolsParser(tools=[ReviseAnswer]),
)
```

인용을 스키마 레벨에서 강제하는 것이 이 구현의 grounding 장치다.

이 세 노드를 그래프로 조립한다.

```python
from langgraph.graph import END, MessageGraph

MAX_ITERATIONS = 5
builder = MessageGraph()
builder.add_node("draft", first_responder.respond)
builder.add_node("execute_tools", execute_tools)
builder.add_node("revise", revisor.respond)

builder.add_edge("draft", "execute_tools")
builder.add_edge("execute_tools", "revise")

def event_loop(state: list) -> str:
    # 도구 실행 횟수 = 지금까지의 반복 수
    num_iterations = sum(isinstance(m, ToolMessage) for m in state)
    if num_iterations > MAX_ITERATIONS:
        return END
    return "execute_tools"

builder.add_conditional_edges("revise", event_loop)
builder.set_entry_point("draft")
graph = builder.compile()
```

draft → 검색 → revise를 MAX_ITERATIONS까지 돌리는, 논문 Algorithm 1의 while문 그대로다.

### **논문 ↔ 구현 매핑**

| Reflexion 논문 | LangGraph Reflexion 에이전트 |
|---|---|
| Actor (M_a) | `draft` / `revise` 노드 (Responder, Revisor) |
| Self-Reflection (M_sr) | Responder가 구조화 출력으로 강제 생성하는 self-critique |
| 반성의 근거 | `execute_tools`의 웹 검색 (논문 코딩 세팅에서는 unit test) |
| mem (장기 기억) | 누적되는 메시지 리스트 — 이전 비평과 검색 결과가 다음 revise의 컨텍스트 |
| max trials | `MAX_ITERATIONS` + `event_loop` 분기 |

논문과 다른 점도 보인다. 논문은 Evaluator가 별도 모듈인데, 이 구현은 비평을 Actor의 구조화 출력 필드로 합쳐버렸다. 대신 근거를 웹 검색으로 잡는다. Table 3의 교훈("근거 없는 반성은 해롭다")을 여기 대입하면, 이 구현에서 반성의 품질을 지탱하는 것은 검색 결과다.

위 코드는 [공식 노트북](https://github.com/langchain-ai/langgraph/blob/23961cff61a42b52525f3b20b4094d8d2fba1744/docs/docs/tutorials/reflexion/reflexion.ipynb)을 블로그 분량에 맞게 정리한 것이다. 임포트 일부를 생략했으니 전체 실행 코드는 노트북을 참고하자. 한 가지 주의할 점은 `MessageGraph`가 블로그 당시의 API라는 것 — 지금 LangGraph에서는 `StateGraph`에 메시지 리스트를 담는 방식으로 같은 그래프를 만든다. 구조는 동일하다.

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
