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
field: agent
---

Reflexion: Language Agents with Verbal Reinforcement Learning

[https://arxiv.org/abs/2303.11366](https://arxiv.org/abs/2303.11366)

Reflexion은 Northeastern + MIT + Princeton에서 나온 논문이다. (NeurIPS 2023)

[지난 ReAct 리뷰](https://momozzing.github.io/paper%20review/ReAct-Paper-review/) 마지막에 "Shunyu Yao의 논문은 계속 따라가 볼 가치가 있다"고 했는데, 그 Shunyu Yao가 공저자로 들어간 후속 연구다.

ReAct가 Thought → Action → Observation 루프를 만들었다면, Reflexion은 그 루프에 실패에서 배우는 능력을 넣는다.

좀 더 자세히 알아보자.

## **1. Introduction**

LLM 에이전트도 시행착오로 배우게 하고 싶다.

그런데 전통적인 강화학습(RL)로 하려면 문제가 있다. 샘플이 대량으로 필요하고, 모델 파인튜닝 비용이 비싸다.

LLM 에이전트는 보통 API 뒤에 있는 거대 모델이라 가중치 업데이트 자체가 어렵기도 하다.

그래서 이 논문은 가중치 대신 말로 강화하자고 한다.

실패하면 "왜 실패했는지"를 언어로 반성하고, 그 반성문을 메모리에 쌓아두고, 다음 시도의 컨텍스트에 넣어준다.

gradient 업데이트가 한 번도 없는데 시도할수록 잘해진다고 한다.

![세 가지 태스크에서의 Reflexion 동작 (논문 Figure 1)](https://momozzing.github.io/assets/images/reflexion/fig1-three-domains.png)

의사결정, 코딩, 추론 세 도메인 모두 같은 패턴이다.

시도 → 실패 신호 → 반성("팬이 stoveburner 1에 없었으니 2를 봐야 했다") → 다음 시도에서 교정.

## **2. Reflexion: reinforcement via verbal reflection**

프레임워크는 LLM 3개(역할 분담)와 메모리로 구성된다.

1. Actor : 텍스트와 행동을 생성하는 모델. CoT나 ReAct를 그대로 Actor로 쓴다
2. Evaluator : Actor가 만든 trajectory에 점수를 매기는 모델. 태스크에 따라 exact match, 휴리스틱 룰, 또는 LLM 판정을 쓴다
3. Self-Reflection : 실패 신호 + trajectory를 보고 "무엇이 잘못됐고 다음엔 어떻게 해야 하는지"를 언어로 생성하는 모델

메모리는 두 층이다.

- 단기 기억: 현재 trial의 trajectory
- 장기 기억: 지금까지 쌓인 반성문들 (최대 3개 정도로 제한)

![Reflexion 구조와 알고리즘 (논문 Figure 2)](https://momozzing.github.io/assets/images/reflexion/fig2-architecture.png)

알고리즘은 단순하다.

시도하고, 평가하고, 실패면 반성문을 메모리에 추가하고, 통과하거나 max trial에 걸릴 때까지 반복한다.

반성문이 스칼라 reward보다 정보량이 훨씬 많다고 한다.

RL의 reward는 "0점이었다"만 알려주지만, 반성문은 "어디서 무엇을 잘못했고 대신 뭘 해야 하는지"를 알려준다. 이게 논문 제목의 verbal reinforcement다.

피드백 소스도 여러 가지를 쓸 수 있다.

- 환경이 주는 binary reward (외부)
- 스스로 만든 unit test 결과 (내부)
- 사람이 주는 자유형 텍스트

전부 반성의 재료로 쓸 수 있다고 한다.

## **3. Experiments**

의사결정, 추론, 코딩 세 도메인에서 실험한다.

### **3.1 Sequential Decision Making: ALFWorld**

ReAct 리뷰에서 봤던 그 텍스트 집안일 시뮬레이터다. Actor로 ReAct를 쓰고 Reflexion을 얹었다.

![ALFWorld 성공률과 실패 원인 분석 (논문 Figure 3)](https://momozzing.github.io/assets/images/reflexion/fig3-alfworld.png)

12번의 trial을 거치며 134개 태스크 중 130개(97%)를 풀어낸다.

ReAct 단독은 75% 부근에서 멈추고 더 이상 나아지지 않는다.

오른쪽 그래프는 실패 원인을 나눠본 것이다.

ReAct 단독은 hallucination 비율이 22%에 수렴한 채 회복하지 못한다. Reflexion은 trial이 거듭될수록 hallucination과 비효율적 계획이 거의 사라진다.

장기 기억이 도움이 되는 경우는 두 가지라고 한다.

1. 긴 trajectory 초반의 실수를 반성으로 찾아내는 것
2. "어디를 이미 뒤져봤는지"를 여러 trial에 걸쳐 기억해서 방을 차례대로 수색하는 것

-> 같은 실수를 반복하지 않게 되는 것 같다.

### **3.2 Reasoning: HotpotQA**

100개의 multi-hop 질문으로 추론 능력이 좋아지는지 본다.

피드백은 exact match binary 신호뿐이고, 반성문이 이 빈약한 신호를 키워주는 구조다.

![HotpotQA 결과와 ablation (논문 Figure 4)](https://momozzing.github.io/assets/images/reflexion/fig4-hotpotqa.png)

baseline의 재시도를 보면, ReAct-only, CoT-only에 temperature 0.7로 재시도를 시켜도 한 번 틀린 문제는 끝까지 못 푼다.

-> 그냥 다시 굴리는 걸로는 안 되고, 뭐가 틀렸는지 말로 알려줘야 나아지는 것 같다.

ablation도 있다.

반성문 없이 직전 trajectory만 컨텍스트에 넣어주는 episodic memory(EPM)와 비교하면, self-reflection이 8%p를 더 얻는다고 한다.

-> 기억을 주는 것과 교훈을 주는 것은 다른 것 같다.

### **3.3 Programming**

HumanEval, MBPP, LeetcodeHardGym으로 본다.

코딩은 Reflexion에게 유리한 도메인이다. 스스로 unit test를 만들어 실행하면 근거 있는 내부 피드백을 얻을 수 있기 때문이다.

테스트 만드는 방법은 이렇다.

1. CoT로 테스트를 생성한다
2. AST로 문법 검증을 한다
3. 최대 6개를 추린다

정답 테스트를 참조하지 않으니 pass@1로 보고할 수 있다고 한다.

![프로그래밍 벤치마크 결과 (논문 Table 1)](https://momozzing.github.io/assets/images/reflexion/table1-programming.png)

HumanEval Python에서 pass@1 91.0으로 당시 SOTA였던 GPT-4(80.1)를 크게 넘었다.

Rust에서도 60.0 → 68.0으로 오른다. 언어에 종속된 방법은 아닌 것 같다.

GPT-4의 사전학습 컷오프 이후 출제된 Leetcode hard 문제에서도 7.5 → 15.0으로 두 배가 된다. LeetcodeHardGym은 이 논문이 새로 만든 벤치마크다.

MBPP Python은 77.1로 GPT-4(80.1)보다 낮다.

원인은 자기가 만든 테스트의 신뢰도라고 한다.

테스트를 다 통과했는데 실제로는 틀린 코드일 확률(false positive)이 HumanEval은 1.4%인데 MBPP는 16.3%나 된다.

-> 평가자가 부실하면 반성도 부실해진다.

![테스트 생성/반성 ablation (논문 Table 3)](https://momozzing.github.io/assets/images/reflexion/table3-ablation.png)

HumanEval Rust 최고 난도 50문제로 ablation을 했다.

테스트 실행 없이 반성만 시키면 0.60 → 0.52로 base보다 오히려 나빠진다.

테스트(근거)와 반성(해석)이 둘 다 있어야 0.68로 오른다.

-> 근거 없는 반성은 오히려 해로운 것 같다.

## **4. 요즘 코딩 에이전트가 이 구조 그대로다**

지금 보면 Reflexion 루프는 낯설지 않다.

코딩 에이전트가 테스트를 돌리고, 실패하면 에러 로그를 읽고, "아 이 부분에서 타입이 안 맞았구나"라고 정리한 뒤 코드를 고쳐서 다시 시도한다.

이게 Actor(코드 생성) → Evaluator(테스트 실행) → Self-Reflection(에러 분석) → 재시도다.

### **4.1 LangGraph의 Reflexion 에이전트**

ReAct는 `create_agent` 한 줄로 구현이 된다.

하지만 Reflexion는 다르다. agent 루프 바깥에 평가와 반성을 두는 구조라 LangGraph로 그래프를 직접 짠다.

LangChain이 공식 블로그 [Reflection Agents](https://www.langchain.com/blog/reflection-agents)에서 이 계열을 세 단계로 정리해뒀다. 단순한 것부터 순서대로,

1. Basic Reflection : 생성 ↔ 비평 반복
2. Reflexion : 비평을 구조화하고 외부 근거로 grounding
3. LATS : 트리 탐색까지 확장. 이것도 Shunyu Yao 계보다

그중 Reflexion 에이전트는 세 노드로 구성된다.

이름만으로는 감이 안 오니 노드별 실제 구현을 보자. 코드는 블로그가 링크한 공식 노트북에서 필요한 부분만 추렸다.

#### **4.1.1 준비물**

모델, 검색 도구, 그리고 세 노드가 같이 쓰는 프롬프트다.

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
     "너는 전문 리서처다. 현재 시각: {time}\n"
     "1. {first_instruction}\n"
     "2. 네 답변을 스스로 비평해라. 개선 폭을 최대화하도록 냉정하게.\n"
     "3. 답변을 개선하는 데 필요한 검색 쿼리를 제안해라."),
    MessagesPlaceholder(variable_name="messages"),
]).partial(time=lambda: datetime.now().isoformat())
```

#### **4.1.2 Responder**

이 구현에서 제일 중요한 건 프롬프트보다 출력 스키마다.

답변만 생성하는 게 아니라 자기 비평과 검색 쿼리까지 하나의 구조체로 강제한다.

```python
class Reflection(BaseModel):
    missing: str = Field(description="답변에서 부족한 점에 대한 비평")
    superfluous: str = Field(description="답변에서 불필요하게 과한 점에 대한 비평")

class AnswerQuestion(BaseModel):
    """답변, 자기 비평, 개선용 검색 쿼리를 한 번에 생성한다."""
    answer: str = Field(description="질문에 대한 250단어 내외의 상세한 답변")
    reflection: Reflection = Field(description="초안 답변에 대한 자기 비평")
    search_queries: list[str] = Field(
        description="비평을 해소하기 위해 조사할 검색 쿼리 1~3개"
    )

class ResponderWithRetries:
    """스키마 검증에 실패하면 에러를 보여주고 재시도시키는 래퍼"""
    def __init__(self, runnable, validator):
        self.runnable, self.validator = runnable, validator

    def respond(self, state: dict):
        messages = state["messages"]
        for attempt in range(3):
            response = self.runnable.invoke({"messages": messages})
            try:
                self.validator.invoke(response)
                break
            except ValidationError as e:
                messages = messages + [response, ToolMessage(
                    content=f"{repr(e)}\n\n함수 스키마를 다시 확인하고 검증 오류를 고쳐서 응답해라.",
                    tool_call_id=response.tool_calls[0]["id"])]
        return {"messages": [response]}

initial_chain = actor_prompt.partial(
    first_instruction="250단어 내외의 상세한 답변을 작성해라."
) | llm.bind_tools(tools=[AnswerQuestion])

first_responder = ResponderWithRetries(
    runnable=initial_chain,
    validator=PydanticToolsParser(tools=[AnswerQuestion]),
)
```

한 번의 호출에서 "답변 + 뭐가 부족한지(missing) + 뭐가 과한지(superfluous) + 그래서 뭘 검색할지"가 전부 나온다.

반성을 자유 텍스트에 맡기지 않고 스키마로 강제했다.

#### **4.1.3 Execute Tools**

Responder가 뽑은 쿼리를 실제 검색으로 실행해서 외부 근거를 가져온다.

```python
def run_queries(search_queries: list[str], **kwargs):
    """생성된 검색 쿼리를 실행한다."""
    return tavily_tool.batch([{"query": query} for query in search_queries])

execute_tools = ToolNode([
    StructuredTool.from_function(run_queries, name=AnswerQuestion.__name__),
    StructuredTool.from_function(run_queries, name=ReviseAnswer.__name__),
])
```

#### **4.1.4 Revisor**

Responder와 같은 스키마를 상속받고, 인용(references)을 추가로 강제한다.

```python
class ReviseAnswer(AnswerQuestion):
    """검색 근거를 인용하며 이전 답변을 수정한다."""
    references: list[str] = Field(
        description="수정된 답변의 근거가 되는 인용 출처들"
    )

revise_instructions = """새로 얻은 정보를 사용해 이전 답변을 수정해라.
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

인용을 스키마 레벨에서 강제하는 게 이 구현의 grounding 장치다.

#### **4.1.5 그래프 조립**

이 세 노드를 그래프로 조립한다.

```python
from typing import Annotated
from typing_extensions import TypedDict
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages

class State(TypedDict):
    messages: Annotated[list, add_messages]

MAX_ITERATIONS = 5
builder = StateGraph(State)
builder.add_node("draft", first_responder.respond)
builder.add_node("execute_tools", execute_tools)
builder.add_node("revise", revisor.respond)

builder.add_edge(START, "draft")
builder.add_edge("draft", "execute_tools")
builder.add_edge("execute_tools", "revise")

def event_loop(state: State):
    # 도구 실행 횟수 = 지금까지의 반복 수
    num_iterations = sum(m.type == "tool" for m in state["messages"])
    if num_iterations > MAX_ITERATIONS:
        return END
    return "execute_tools"

builder.add_conditional_edges("revise", event_loop, ["execute_tools", END])
graph = builder.compile()

result = graph.invoke(
    {"messages": [("user", "LLM 에이전트의 자기 개선 기법을 정리해줘")]}
)
```

draft → 검색 → revise를 MAX_ITERATIONS까지 돌린다. 논문 Algorithm 1의 while문 그대로다.

### **4.2 논문 ↔ 구현 매핑**

- Actor (M_a) : `draft` / `revise` 노드 (Responder, Revisor)
- Self-Reflection (M_sr) : Responder가 구조화 출력으로 강제 생성하는 self-critique
- 반성의 근거 : `execute_tools`의 웹 검색 (논문 코딩 세팅에서는 unit test)
- mem (장기 기억) : 누적되는 메시지 리스트. 이전 비평과 검색 결과가 다음 revise의 컨텍스트가 된다
- max trials : `MAX_ITERATIONS` + `event_loop` 분기

논문과 다른 점도 있다.

논문은 Evaluator가 별도 모듈인데, 이 구현은 비평을 Actor의 구조화 출력 필드로 합쳐버렸다. 대신 근거를 웹 검색으로 잡는다.

-> Table 3 결과("근거 없는 반성은 해롭다")를 여기 대입하면, 이 구현에서 반성의 품질을 받쳐주는 건 검색 결과인 것 같다.

블로그(2024)의 원본 코드는 지금은 사라진 `MessageGraph` 기반이다.

그래서 위 코드는 현재의 LangGraph API(`StateGraph` + `add_messages`)로 리팩토링한 것이다. 상태가 "메시지 리스트 하나"인 건 그대로고, state 스키마를 명시하는 형태로 바뀌었다.

원본과 전체 실행 코드는 [공식 노트북](https://github.com/langchain-ai/langgraph/blob/23961cff61a42b52525f3b20b4094d8d2fba1744/docs/docs/tutorials/reflexion/reflexion.ipynb)을 참고하자.

에이전트에 붙는 memory 설계도 비슷하다.

세션에서 얻은 교훈을 파일로 남겨두고 다음 세션 컨텍스트에 넣어주는 패턴이 Reflexion의 장기 기억 부분이다.

-> 가중치는 그대로인데 컨텍스트가 학습되는 것, in-context learning을 학습 루프로 쓰는 것이다.

Table 3 결과는 지금도 그대로 적용된다.

판정이 부정확하면(멋대로 만든 테스트, 어설픈 LLM-judge) self-correction은 오히려 성능을 깎는다.

-> 에이전트 루프를 만들 때 "얼마나 잘 반성하느냐"보다 "반성의 근거가 얼마나 확실하냐"를 먼저 봐야 할 것 같다.

## **5. Limitations**

- Evaluator의 정확도에 전체가 의존한다. 코딩의 false positive 문제가 대표적
- 반성문 메모리를 최대 3개 정도로 잘라서 쓴다. 더 긴 학습을 하려면 요약이나 검색이 필요할 것
- 국소최적에 빠질 수 있다. 반성이 잘못된 방향을 가리키면 그쪽으로 계속 판다

-> 반성이 잘못됐는지는 누가 판단하지??

## **6. Conclusion**

ReAct가 행동하면서 생각하는 법을 만들었다면, Reflexion은 실패에서 배우는 법을 더했다.

가중치를 하나도 안 바꾸고, 실패 경험을 반성문으로 바꿔 메모리에 쌓는 것만으로 시도할수록 잘해지는 에이전트가 된다고 한다.

여태까지 강화학습이 스칼라 reward로 가중치를 업데이트했다면, 이 방법은 언어를 학습 신호로 쓰는 verbal RL이다.

단, 반성은 근거가 있을 때만 효과가 있다. 근거 없는 반성은 안 하느니만 못하다는 게 Table 3 결과다.
