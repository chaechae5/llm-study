# LangGraph 개념 정리

---

## 목차

1. [LangGraph 개요](#1-langgraph-개요)
2. [그래프 기본 개념](#2-그래프-기본-개념)
3. [핵심 구성요소](#3-핵심-구성요소)
4. [State 설계](#4-state-설계)
5. [Node와 Edge](#5-node와-edge)
6. [왜 Node와 Edge를 명시적으로 추가해야 하는가](#6-왜-node와-edge를-명시적으로-추가해야-하는가)
7. [compile()과 CompiledStateGraph](#7-compile과-compiledstategraph)
8. [Conditional Routing (조건부 분기)](#8-conditional-routing-조건부-분기)
9. [Tool 통합](#9-tool-통합)
10. [Memory 관리](#10-memory-관리)
11. [Human-in-the-Loop](#11-human-in-the-loop)
12. [LangGraph Agent 작성 순서](#12-langgraph-agent-작성-순서)
13. [핵심 요약](#13-핵심-요약)

---

## 1. LangGraph 개요

### LangGraph란?

**장기간 실행되는 상태 기반 에이전트를 구축, 관리 및 배포**하기 위한 저수준 오케스트레이션 프레임워크이자 런타임.

에이전트의 실행 흐름을 **그래프(Graph)** 구조로 표현함으로써, 복잡한 분기 로직·루프·병렬 처리를 명확하게 모델링할 수 있음.

### 주요 특징

| 항목 | 내용 |
|------|------|
| 추상화 수준 | 저수준(Low-level) — 세밀한 제어 가능 |
| LangChain 의존성 | 없음 (독립적으로 사용 가능) |
| 핵심 집중 영역 | 에이전트 오케스트레이션 |
| 주요 기능 | 내구성 있는 실행, 스트리밍, Human-in-the-loop |

### LangGraph vs LangChain

🤔 **랭체인이랑 랭그래프가 뭐가 달라?**

랭체인은 **LLM 호출·프롬프트·도구를 연결하는 파이프라인 도구**고,
랭그래프는 **그 위에서 에이전트의 실행 흐름(루프·분기·상태)을 제어하는 오케스트레이션 도구**임.

| | 비유 | 역할 |
|--|------|------|
| **LangChain** | 레고 블록 | LLM·프롬프트·도구 등 부품을 만들고 연결 |
| **LangGraph** | 레고로 만든 기계의 작동 방식 | 부품들이 어떤 순서로, 어떤 조건에서 움직일지 제어 |

```
LangChain이 잘하는 것
→ "LLM 불러서 → 프롬프트 넣고 → 결과 파싱"
   단순한 선형 파이프라인

LangGraph가 잘하는 것
→ "LLM 결과 보고 → 도구 쓸지 판단 → 도구 실행 → 다시 LLM → 조건에 따라 분기"
   루프·분기·상태가 있는 복잡한 흐름
```

> 둘은 경쟁 관계가 아니라 **같이 쓰는 관계**. LangGraph 안에서 LangChain 컴포넌트(LLM, 도구 등)를 부품으로 가져다 씀.

---

### 실제 코드로 비교

**LangChain만 쓸 때** — 단순 체인, 한 번 실행하고 끝

```python
chain = prompt | llm | output_parser
result = chain.invoke({"input": "안녕"})
# 한 번 실행하고 끝. 루프 없음
```

**LangGraph 쓸 때** — LLM 결과에 따라 루프·분기

```python
# LLM 결과에 따라 도구를 쓸지 판단하고
# 도구 실행 후 다시 LLM으로 돌아오는 루프
llm → (도구 필요?) → 도구 실행 → llm → (도구 필요?) → ... → 종료
```

---

### 🤔 Tool을 많이 정의해야 LangGraph가 효율적인 거 아니야?

Tool 개수보다 **"흐름이 복잡한가"** 가 더 중요한 기준임.
Tool이 없어도 LangGraph가 필요한 경우가 있음.

```
1. 루프가 필요할 때
   → LLM 결과를 보고 "충분한가?" 판단 후 재시도 반복

2. 조건 분기가 필요할 때
   → 입력에 따라 완전히 다른 경로로 실행

3. 상태를 유지해야 할 때
   → 여러 단계를 거치면서 중간 결과를 쌓아가야 할 때

4. Human-in-the-Loop
   → 특정 시점에 사람의 승인을 받아야 할 때

5. 멀티 에이전트
   → 여러 에이전트가 협력하는 구조
```

### 언제 뭘 써야 해?

| 상황 | 추천 |
|------|------|
| LLM 한 번 호출하고 끝 | LangChain만으로 충분 |
| Tool 많은데 선형으로 실행 | LangChain만으로 충분 |
| Tool 1개뿐인데 루프·분기 필요 | LangGraph 필요 |
| Tool 많고 + 흐름도 복잡 | LangGraph 필요 |

> **핵심 질문 → "실행 흐름을 내가 제어해야 하는가?"**
> YES → LangGraph / NO (한 번 실행하고 끝) → LangChain으로 충분

---

## 2. 그래프 기본 개념

LangGraph의 핵심 모델은 **방향성 그래프(Directed Graph)**.

```
Graph = Vertex(Node) + Edge

Vertex(Node): 객체의 추상적 표현 (점이나 원으로 표기)
Edge(Link):   관계를 표현 (직선이나 곡선으로 표기, 방향성 가짐)
```

### 그래프 유형

| 유형 | 설명 | LangGraph 적용 |
|------|------|---------------|
| DAG (비순환 방향 그래프) | 사이클 없는 방향 그래프 | 단순 파이프라인 구현 |
| Cyclic Graph (순환 그래프) | 사이클 허용 | 루프 기반 에이전트 구현 (LangGraph의 핵심 강점) |

LangGraph는 **사이클(Cycle)을 허용**하기 때문에 "관찰 → 재계획 → 재실행"의 반복 루프를 자연스럽게 구현할 수 있음.

---

## 3. 핵심 구성요소

### 기본 구조 코드

```python
from langgraph.graph import StateGraph, MessagesState, START, END

def mock_llm(state: MessagesState):
    return {"messages": [{"role": "ai", "content": "hello world"}]}

# 그래프 정의
graph = StateGraph(MessagesState)
graph.add_node(mock_llm)           # 노드 추가
graph.add_edge(START, "mock_llm")  # 시작 엣지
graph.add_edge("mock_llm", END)    # 종료 엣지
graph = graph.compile()            # 컴파일

# 실행
result = graph.invoke({"messages": [{"role": "user", "content": "hi!"}]})
```

**그래프 흐름:**
```
__start__
    ↓
 mock_llm
    ↓
 __end__
```

### 핵심 개념 정리

| 개념 | 설명 |
|------|------|
| `StateGraph` | Node들 간 공유 상태(State) 값을 읽고 써서 통신하는 그래프 |
| `MessagesState` | TypedDict로 구현된 사전 정의 상태 스키마. 사용자·AI·도구 간 대화 관리 표준화 |
| `START` | 그래프의 시작을 나타내는 특수 노드 |
| `END` | 그래프의 끝을 나타내는 특수 노드 |
| `CompiledStateGraph` | `compile()` 후 `invoke()`, `stream()` 호출이 가능한 실행 가능 그래프 |

---

## 4. State 설계

State는 그래프 내 모든 Node가 공유하는 **중앙 데이터 저장소**. Node는 State를 읽어 작업을 수행하고, 업데이트된 State를 반환함.

---

### TypedDict

🤔 **그냥 dict 쓰면 안 돼? 왜 TypedDict야?**

`dict`는 어떤 키에 어떤 타입이 들어오는지 코드만 봐서는 알 수 없음.
`TypedDict`는 **각 키의 타입을 미리 선언**해두기 때문에 IDE가 오타·타입 불일치를 코드 작성 시점에 잡아줌.

```python
from typing import TypedDict

# ❌ dict 방식 — 뭐가 들어오는지 코드만 봐서는 모름
state = {"messages": [], "user_id": ""}

# ✅ TypedDict 방식 — 구조가 명확하게 보임
class State(TypedDict):
    messages: list
    user_id: str
```

| 항목 | dict | TypedDict |
|------|------|-----------|
| 타입 검사 시점 | 런타임 (검사 없음) | 정적 (코드 작성 시 IDE가 사전 감지) |
| 키/값 타입 지정 | 불명확 | 각 키마다 구체적 타입 지정 |
| 유연성 | 런타임에 키 추가/제거 가능 | 정의된 구조 준수 필수 |
| 안정성 | 낮음 (오타·타입 불일치 → 런타임 오류) | 높음 (코드 작성 시 조기 감지) |

---

### Annotated

🤔 **Annotated는 왜 쓰는 거야?**

Python의 타입 힌트는 기본적으로 타입 정보만 담을 수 있음.
`Annotated`를 쓰면 타입 힌트에 **추가 메타 정보(리듀서 함수 등)** 를 같이 실어 보낼 수 있음.

```python
from typing import Annotated
# Annotated[실제 타입, 추가 메타 정보]

messages: Annotated[list, add_messages]
#                   ^^^^  ^^^^^^^^^^^^
#                 타입    리듀서 함수 (어떻게 업데이트할지)
```

LangGraph는 State 필드를 업데이트할 때 `Annotated`에 붙은 리듀서 함수를 읽고, 그 규칙대로 State를 병합함.

---

### 리듀서 함수 (Reducer)

🤔 **리듀서 함수가 뭐야?**

**"두 값을 어떻게 합칠지" 규칙을 정의한 함수**.

노드가 새 값을 반환했을 때 LangGraph는 이런 상황에 놓임.

> "기존 State에 값이 있고, 노드가 새 값을 반환했어. 그럼 어떻게 합치지?"

이때 그 **합치는 규칙**이 리듀서 함수.

```python
# 리듀서 없음 → 그냥 덮어씀
messages = ["안녕"]
새_값    = ["반가워"]
결과     = ["반가워"]   # 기존 값 사라짐 ❌

# add_messages 리듀서 있음 → 누적시킴
messages = ["안녕"]
새_값    = ["반가워"]
결과     = ["안녕", "반가워"]  # 기존 값 보존 ✅
```

🤔 **커스텀 리듀서도 만들 수 있어?**

`add_messages` 말고 직접 규칙을 정의할 수 있음.

```python
# 항상 최신 메시지 1개만 유지하는 리듀서
def keep_latest(old, new):
    return new[-1:]

class State(TypedDict):
    messages: Annotated[list, keep_latest]

# 숫자를 합산하는 리듀서
def sum_values(old: int, new: int) -> int:
    return old + new

class State(TypedDict):
    total_tokens: Annotated[int, sum_values]
    # 노드가 100을 반환하면 기존 값에 더해짐
    # 200 + 100 = 300
```

| 리듀서 | 동작 | 활용 사례 |
|--------|------|----------|
| `add_messages` (기본 제공) | 메시지 누적, 동일 ID면 교체 | 대화 히스토리 관리 |
| 커스텀: `keep_latest` | 최신 값만 유지 | 최근 검색 결과만 보관 |
| 커스텀: `sum_values` | 값을 합산 | 토큰 사용량 누적 집계 |

---

### add_messages

🤔 **add_messages는 뭐야?**

두 메시지 리스트를 **"덮어쓰기" 없이 병합**하는 리듀서 함수.
`Annotated`와 함께 써서 State의 `messages` 필드에 새 메시지를 누적(append)시킴.

```python
from typing import Annotated, TypedDict
from langgraph.graph.message import add_messages

class State(TypedDict):
    messages: Annotated[list, add_messages]
```

🤔 **add_messages 없으면 어떻게 돼?**

```python
# ❌ 리듀서 없음 → 노드가 반환한 값으로 통째로 덮어씀
class State(TypedDict):
    messages: list
# 결과: 이전 대화 내용이 싹 날아감

# ✅ add_messages 있음 → 기존 메시지에 새 메시지를 추가
class State(TypedDict):
    messages: Annotated[list, add_messages]
# 결과: 대화 히스토리가 쌓임
```

🤔 **같은 ID의 메시지가 들어오면?**

단순 append가 아니라 **동일 ID면 교체**, 새 ID면 추가하는 방식으로 동작함.

```python
existing = [HumanMessage("안녕", id="1"), AIMessage("반갑습니다", id="2")]
new      = [AIMessage("수정된 응답", id="2"), HumanMessage("감사합니다", id="3")]

result = add_messages(existing, new)
# → [HumanMessage("안녕", id="1"),
#    AIMessage("수정된 응답", id="2"),  ← id="2"가 교체됨
#    HumanMessage("감사합니다", id="3")]
```

Tool 결과를 업데이트하거나 스트리밍 중 메시지를 수정할 때 활용됨.

---

## 5. Node와 Edge

### Node (노드)

그래프에서 **실제 작업을 수행하는 단위**. Python 함수로 정의됨.

- 입력: 현재 `State`
- 출력: 업데이트할 State의 딕셔너리 (부분 업데이트)

```python
def my_node(state: State) -> dict:
    # state를 읽어 작업 수행
    new_value = process(state["some_key"])
    # 업데이트할 필드만 반환
    return {"some_key": new_value}
```

**Node의 종류:**

| 종류 | 설명 |
|------|------|
| LLM Node | LLM을 호출하여 응답 생성 |
| Tool Node | 도구(API, DB 등)를 실행 |
| Router Node | 조건을 평가하여 다음 경로 결정 |
| Human Node | 사람의 입력을 기다리는 대기 노드 |

### Edge (엣지)

Node 간의 **연결 및 실행 순서**를 정의.

```python
# 일반 엣지: A → B 항상 실행
graph.add_edge("node_a", "node_b")

# 조건부 엣지: 조건에 따라 분기
graph.add_conditional_edges(
    "node_a",
    routing_function,   # 다음 노드 이름을 반환하는 함수
    {
        "path_1": "node_b",
        "path_2": "node_c"
    }
)
```

---

## 6. 왜 Node와 Edge를 명시적으로 추가해야 하는가

🤔 **함수 만들었는데 왜 또 add_node로 등록해야 해?**

함수가 존재한다고 그래프가 자동으로 알아채지 않음.
`add_node`로 등록해야 비로소 **그래프의 실행 단위**로 인식됨.

```python
graph.add_node(mock_llm)  # 이걸 해야 "아, 이 함수 쓸 거구나" 인식
```

---

🤔 **노드 등록했으면 됐지, 왜 add_edge도 해야 해?**

노드만 있으면 "작업이 있다"는 것만 알 뿐, **실행 순서를 모름**.
Edge가 있어야 `START → mock_llm → END` 흐름이 완성됨.

```python
graph.add_edge(START, "mock_llm")  # 여기서 시작해서
graph.add_edge("mock_llm", END)    # 여기서 끝내
```

공항(Node)이 있어도 항공 노선(Edge)이 없으면 비행기가 어디로 가야 할지 모르는 것과 같음.

| 개념 | 비유 |
|------|------|
| Node | 공항 (목적지/경유지) |
| Edge | 항공 노선 |
| Graph | 전체 노선도 |

---

🤔 **그냥 함수 순서대로 호출하면 안 돼? 왜 굳이 이렇게 해?**

단순 함수 호출로는 조건 분기·루프·병렬 처리를 표현하기 어려움.
이렇게 선언해두면 **실행 흐름 자체가 데이터**가 돼서, 조건에 따라 경로를 바꾸거나 루프를 도는 복잡한 흐름을 깔끔하게 제어할 수 있음.

```
add_node → "이런 작업이 있어"
add_edge → "이 순서로 실행해"

둘 다 없으면 → 그래프 실행 불가
```

---

## 7. compile()과 CompiledStateGraph

🤔 **왜 compile()을 해야 해? 그냥 바로 실행하면 안 돼?**

`StateGraph`는 **설계도**일 뿐이라 실행 능력이 없음.
`compile()` 하는 순간 LangGraph가 내부적으로 아래를 수행하고 **실행 가능한 객체**로 변환해줌.

```python
graph = graph.compile()
# 이 시점에 일어나는 일:
#   1. 유효성 검사  — 연결 안 된 노드, 도달 불가한 노드 없는지 확인
#   2. 실행 계획 수립 — 어떤 순서로 노드를 실행할지 내부 계획 생성
#   3. 체크포인터 연결 — memory/checkpointer 설정이 있으면 연결
#   4. 스트리밍 설정  — stream() 지원을 위한 내부 구조 준비
```

| 단계 | 비유 |
|------|------|
| `add_node` / `add_edge` | 건축 설계도 그리기 |
| `compile()` | 설계도 검토 + 실제 건물 짓기 |
| `invoke()` | 건물 입주 후 실제 사용 |

---

🤔 **왜 invoke()는 compile() 전엔 못 써?**

`StateGraph`랑 `CompiledStateGraph`는 **아예 다른 클래스**임.
`StateGraph`엔 `invoke()` 메서드 자체가 없고, `compile()` 하면 실행 메서드가 붙은 새 클래스가 반환되는 구조.

```python
graph = StateGraph(MessagesState)
# → add_node, add_edge만 가능 (설계 전용 클래스)

graph = graph.compile()
# → CompiledStateGraph 반환 (실행 전용 클래스)
#   invoke(), stream(), get_state() 사용 가능
```

만약 compile 없이 invoke를 허용했다면?
- 유효성 검사 없이 실행 → 실행 도중 오류 발생
- 체크포인터 없이 실행 → 메모리 기능 동작 안 함
- invoke 호출 때마다 유효성 검사 반복 → 성능 낭비

```
StateGraph           →  설계 단계 (invoke 불가)
    ↓ compile()
    ↓ 유효성 검사 + 실행 계획 수립 + 기능 연결
CompiledStateGraph   →  실행 단계 (invoke 가능)
```

---

## 8. Conditional Routing (조건부 분기)

조건에 따라 그래프의 실행 흐름을 분기하는 기능.

### 기본 예시: 날씨 분기

```python
from typing import Literal

# State 정의
class State(TypedDict):
    input: str
    result: str

# 라우팅 함수 — 다음 노드 이름을 반환
def weather_routing_fn(state: State) -> Literal["goto_rainy_node", "goto_sunny_node"]:
    user_input = state['input']
    if user_input.lower() == 'rainy':
        return "goto_rainy_node"
    elif user_input.lower() == 'sunny':
        return "goto_sunny_node"
    else:
        raise ValueError(f"알 수 없는 날씨 조건: {user_input}")

# 조건부 엣지 등록
graph_builder.add_conditional_edges(
    "weather_node",          # 분기가 시작되는 노드
    weather_routing_fn,      # 라우팅 함수
    {
        "goto_rainy_node": "rainy_node",   # 반환값 → 실제 노드 매핑
        "goto_sunny_node": "sunny_node"
    }
)
```

**그래프 흐름:**
```
   __start__
       ↓
 weather_node
   ↙         ↘
rainy_node  sunny_node
   ↘         ↙
   __end__
```

### 실전 예시: LLM 응답 기반 라우팅

```python
def should_use_tool(state: State) -> Literal["call_tool", "end"]:
    last_message = state["messages"][-1]
    # LLM이 tool_calls를 포함한 응답을 반환했다면 도구 실행
    if hasattr(last_message, "tool_calls") and last_message.tool_calls:
        return "call_tool"
    return "end"

graph.add_conditional_edges(
    "llm_node",
    should_use_tool,
    {
        "call_tool": "tool_node",
        "end": END
    }
)
```

이 패턴을 통해 **LLM이 도구 호출이 필요하다고 판단할 때만 tool_node로 분기**, 그렇지 않으면 종료하는 ReAct 루프를 구현할 수 있음.

---

## 9. Tool 통합

### `@tool` 데코레이터

Python 함수를 LangChain Tool로 변환.

```python
from langchain_core.tools import tool

# 방법 1: 인라인 주석으로 설명
@tool
def search_api(query: str) -> str:
    # 외부 API를 통해 검색 결과를 반환한다.
    return f"Search result for: {query}"

# 방법 2: Docstring으로 설명 (권장 — LLM이 도구를 이해하는 데 사용)
@tool
def multiply(a: int, b: int) -> int:
    """두 정수 a와 b를 곱한 결과를 반환한다.

    Args:
        a: 첫 번째 정수
        b: 두 번째 정수
    """
    return a * b
```

> 💡 **Docstring이 중요한 이유**: LLM은 Docstring을 읽고 "언제, 어떻게 이 도구를 쓸지"를 판단함. 명확하고 구체적인 설명이 Agent의 도구 선택 정확도를 높임.

### ToolNode

여러 도구를 묶어 그래프의 Node로 등록하는 편의 클래스.

```python
from langgraph.prebuilt import ToolNode

tools = [search_api, multiply]
tool_node = ToolNode(tools)

# 그래프에 노드로 추가
graph.add_node("tool_node", tool_node)
```

### LLM에 도구 바인딩

```python
from langchain_anthropic import ChatAnthropic

model = ChatAnthropic(model="claude-sonnet-4-20250514")

# 도구를 LLM에 바인딩 → LLM이 tool_calls 생성 가능
model_with_tools = model.bind_tools(tools)
```

### tool_calls — LLM이 도구 호출을 담는 내장 속성

🤔 **`tool_calls`는 내가 선언한 변수야?**

아니. `AIMessage` 객체에 **내장된 속성(예약 필드)** 임. `bind_tools`로 도구를 등록한 LLM이 응답할 때, 도구가 필요하다고 판단하면 자동으로 채워줌.

```python
llm_with_tools = llm.bind_tools([add, multiply])
response = llm_with_tools.invoke("1 더하기 2는?")

# AIMessage 안에 자동으로 생김
print(response.tool_calls)
# [{"name": "add", "args": {"a": 1, "b": 2}, "id": "abc123"}]
```

**`AIMessage` 구조:**

```python
AIMessage(
    content="",              # LLM 텍스트 응답
    tool_calls=[...],        # ← 내장 속성 (도구 호출 시 자동으로 채워짐)
    invalid_tool_calls=[],   # 잘못된 도구 호출
    usage_metadata={...}     # 토큰 사용량
)
```

**도구 호출 없을 때 vs 있을 때:**

```python
# 도구 호출 없을 때
AIMessage(content="안녕하세요!", tool_calls=[])  # 빈 리스트

# 도구 호출 있을 때
AIMessage(content="", tool_calls=[
    {"name": "add",      "args": {"a": 1, "b": 2}, "id": "abc"},
    {"name": "multiply", "args": {"a": 3, "b": 4}, "id": "def"},
])
```

> `tool_call_id`는 LLM이 여러 도구를 동시에 호출할 때 어떤 결과가 어떤 호출의 것인지 매칭하는 데 쓰임.

**`should_continue`에서 이렇게 활용:**

```python
def should_continue(state):
    if state["messages"][-1].tool_calls:  # 비어있으면 False → END
        return "tool_node"
    return END
```

**직접 구현한 `tool_node`에서 활용:**

```python
def tool_node(state: dict):
    """도구 호출 수행"""
    result = []
    for tool_call in state["messages"][-1].tool_calls:
        tool = tools_by_name[tool_call["name"]]          # 내가 정의한 툴에서 찾기
        observation = tool.invoke(tool_call["args"])      # 실제 실행
        result.append(ToolMessage(
            content=observation,          # 실행 결과를 LLM에게 전달
            tool_call_id=tool_call["id"]  # 어떤 호출의 결과인지 매칭
        ))
    return {"messages": result}
```

**흐름 정리:**

```
내가 정의한 툴: [add, multiply, search, ...]
                    ↓ bind_tools로 등록
LLM이 판단:    "add랑 multiply 써야겠다" → tool_calls에 자동으로 저장
                    ↓
tool_node:     tool_calls 순회하며 실제 실행 → ToolMessage로 결과 반환
                    ↓
LLM:           결과 보고 최종 답변 생성
```

**전체 ReAct 루프 예시:**

```python
from langgraph.graph import StateGraph, MessagesState, START, END
from langgraph.prebuilt import ToolNode

def call_llm(state: MessagesState):
    response = model_with_tools.invoke(state["messages"])
    return {"messages": [response]}

def should_continue(state: MessagesState):
    last = state["messages"][-1]
    if last.tool_calls:
        return "tools"
    return END

graph = StateGraph(MessagesState)
graph.add_node("llm", call_llm)
graph.add_node("tools", ToolNode(tools))
graph.add_edge(START, "llm")
graph.add_conditional_edges("llm", should_continue, {"tools": "tools", END: END})
graph.add_edge("tools", "llm")  # 도구 실행 후 다시 LLM으로

app = graph.compile()
```

```
__start__ → llm ─→ (tool_calls 있음?) ─→ tools → llm (반복)
                ↘ (tool_calls 없음?) ─→ __end__
```

---

## 10. Memory 관리

🤔 **Agent가 대화 내용을 기억하려면 어떻게 해?**

메모리는 범위에 따라 두 가지로 나뉨.

| 메모리 유형 | 범위 | 용도 | 구현 방법 |
|------------|------|------|----------|
| **Short-term** | 에이전트 실행 중 | 현재 대화의 컨텍스트 | State의 `messages` 필드 |
| **Long-term** | 세션 간 영속 | 사용자 정보, 선호도, 이전 기억 | 외부 DB, Vector Store |

---

🤔 **Short-term 메모리는 어떻게 동작해?**

State의 `messages` 필드가 곧 단기 메모리. 노드가 실행될 때마다 `add_messages` 리듀서가 메시지를 누적시켜 대화 흐름을 유지함.

```python
class State(TypedDict):
    messages: Annotated[list, add_messages]  # 대화가 쌓이는 곳
```

실행할 때마다 이전 메시지가 State에 남아있어서 LLM이 문맥을 이어받을 수 있음.

---

🤔 **세션이 끊겨도 대화를 이어가려면?**

`Checkpointer`를 쓰면 됨. State를 외부에 저장해두고, 다음 호출 때 불러오는 방식.

```python
from langgraph.checkpoint.memory import MemorySaver

memory = MemorySaver()
app = graph.compile(checkpointer=memory)  # compile 시점에 연결

# thread_id로 대화 세션 구분
config = {"configurable": {"thread_id": "user_123"}}
result = app.invoke({"messages": [("user", "안녕")]}, config=config)

# 같은 thread_id로 다시 호출 → 이전 대화 이어받음
result2 = app.invoke({"messages": [("user", "아까 뭐라고 했지?")]}, config=config)
```

🤔 **thread_id가 뭐야?**

대화 세션을 구분하는 키. 같은 `thread_id`면 이전 State를 불러오고, 다른 `thread_id`면 새 대화로 시작함.

```
thread_id: "user_123"  → A 사용자의 대화 세션
thread_id: "user_456"  → B 사용자의 대화 세션 (완전히 별개)
```

🤔 **MemorySaver 말고 다른 Checkpointer도 있어?**

| Checkpointer | 특징 |
|-------------|------|
| `MemorySaver` | 인메모리 저장 — 프로세스 종료 시 사라짐. 테스트용 |
| `SqliteSaver` | SQLite 파일에 저장 — 로컬 영속 저장 |
| `PostgresSaver` | PostgreSQL에 저장 — 프로덕션 환경에 적합 |

```python
# 프로덕션 예시 (PostgreSQL)
from langgraph.checkpoint.postgres import PostgresSaver

checkpointer = PostgresSaver.from_conn_string("postgresql://...")
app = graph.compile(checkpointer=checkpointer)
```

---

## 11. Human-in-the-Loop

Agent가 중요한 결정을 내리기 전에 **사람의 검토/승인을 요청**하는 패턴.

### interrupt 활용

```python
from langgraph.types import interrupt

def review_node(state: State):
    # 현재 계획을 사람에게 보여주고 승인을 요청
    human_approval = interrupt({
        "question": "이 작업을 실행할까요?",
        "plan": state["plan"]
    })
    
    if human_approval == "approve":
        return {"approved": True}
    else:
        return {"approved": False, "feedback": human_approval}
```

### 주요 활용 사례

| 사례 | 설명 |
|------|------|
| 결제/구매 승인 | 일정 금액 이상의 트랜잭션 실행 전 사람 확인 |
| 외부 발행 검토 | 이메일·SNS 등 외부에 발행되는 콘텐츠 최종 검토 |
| 위험 작업 확인 | DB 삭제, 시스템 변경 등 되돌리기 어려운 작업 전 확인 |
| 불확실한 상황 처리 | Agent가 확신하지 못하는 경우 사람에게 판단 위임 |

---

## 12. LangGraph Agent 작성 순서

1. **시스템 프롬프트 작성** — Agent의 역할, 제약, 행동 지침을 명확히 정의
2. **도구(Tool) 정의** — `@tool` 데코레이터로 외부 데이터·시스템과 연동되는 함수 작성
3. **모델 구성** — LLM 선택 및 도구 바인딩 (`model.bind_tools(tools)`)
4. **구조화된 출력 설정** — 예측 가능한 결과를 위해 Pydantic 모델 등으로 출력 스키마 정의
5. **State 설계** — 에이전트 실행 중 필요한 정보를 TypedDict로 정의
6. **그래프 설계** — Node, Edge, Conditional Edge로 실행 흐름 정의
7. **Memory/Checkpointer 추가** — 대화 지속성 및 세션 관리 설정
8. **Human-in-the-Loop 설정** — 필요 시 사람의 검토 포인트 삽입
9. **컴파일 및 실행** — `graph.compile()` 후 `invoke()` 또는 `stream()`으로 실행

---

## 13. 핵심 요약

| 항목 | 내용 |
|------|------|
| **정의** | 상태 기반 에이전트를 그래프 구조로 오케스트레이션하는 프레임워크 |
| **핵심 모델** | 방향성 순환 그래프 (Node + Edge + State) |
| **State** | 모든 Node가 공유하는 중앙 데이터. TypedDict + Annotated로 정의 |
| **Node** | 실제 작업 수행 단위 (LLM 호출, 도구 실행, 라우팅 등) |
| **Edge** | Node 간 연결. 일반 엣지 vs 조건부 엣지(Conditional Edge) |
| **add_messages** | 메시지를 누적(append)하는 리듀서. 동일 ID는 대체 |
| **Checkpointer** | 대화 상태 영속화. 중단/재개, 롤백 지원 |
| **Human-in-the-Loop** | `interrupt`로 중요 결정 시점에 사람의 검토 삽입 |
| **Tool 통합** | `@tool` + `ToolNode` + `bind_tools`로 LLM과 도구를 연결 |
| **선택 기준** | 복잡한 흐름 제어, 루프, 병렬 처리가 필요할 때 LangGraph 선택 |

---
