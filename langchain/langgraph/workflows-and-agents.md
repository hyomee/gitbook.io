# Workflows and agents

LangGraph v1.0의 **Workflows and Agents** 모듈은 복잡한 LLM 기반 워크플로우를 **재사용 가능한 에이전트 단위**로 정의하고 체계적으로 실행하기 위한 고수준 인터페이스를 제공한다. `init_chat_model`을 통해 OpenAI와 Ollama API를 직관적으로 초기화할 수 있으며, Graph API와 Functional API 양쪽에서 일관된 패턴으로 활용할 수 있다.​

***

## 1. Workflows and Agents 개요 <a href="#id-1-workflows-and-agents" id="id-1-workflows-and-agents"></a>

* **Workflow**: 여러 단계(task)를 순차 또는 병렬로 연결해 특정 목표를 달성하는 유닛
* **Agent**: Workflow를 캡슐화한 실행 가능한 단위로, 입력을 받아 상태(state)를 관리하면서 결과를 반환
* **패턴**
  * **Sequential**: 단계별 순차 실행
  * **Conditional**: 상태 기반 분기
  * **Parallel**: `Send` API를 통한 병렬 워커 실행
  * **Tool Integration**: 에이전트 내에서 외부 도구(tool)를 호출

Workflows and Agents는 Graph API와 Functional API에서 동일한 개념적 추상화를 제공한다.​

***

## 2. `init_chat_model`을 통한 API 초기화 <a href="#undefined" id="undefined"></a>

LangGraph v1.0에서는 `builder.init_chat_model` 또는 `init_chat_model` 헬퍼로 OpenAI와 Ollama 챗 모델을 간단히 초기화할 수 있다.​

```python
from langgraph.graph import StateGraph

builder = StateGraph(MyState)
# OpenAI GPT-4
builder.init_chat_model(
    provider="openai",
    model="gpt-4",
    api_key="OPENAI_API_KEY",
    temperature=0.7
)

# Ollama llama2
builder.init_chat_model(
    provider="ollama",
    model="llama2",
    host="http://localhost:11434",
    stream=True
)
```

* `provider`: "openai" 또는 "ollama"
* `model`: 사용할 모델 이름
* 추가 파라미터로 인증 키, 스트리밍 여부 등 설정

***

## 3. Graph API 예시 <a href="#id-3-graph-api" id="id-3-graph-api"></a>

### 3.1. 간단한 순차 워크플로우

```python
from langgraph.graph import StateGraph, START, END
from langchain.messages import SystemMessage

class MyState(TypedDict):
    query: str
    response: str

def ask_llm(state: MyState) -> dict:
    msg = builder.chat_model.invoke(
        [SystemMessage(content=state["query"])]
    )
    return {"response": msg.content}

builder = StateGraph(MyState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="...")
builder.add_node("ask", ask_llm)
builder.add_edge(START, "ask")
builder.add_edge("ask", END)

graph = builder.compile()
result = graph.invoke({"query": "LangGraph란 무엇인가요?"})
print(result["response"])
```

### 3.2. 조건부 분기와 Ollama

```python
from typing import Literal

def decide_agent(state: MyState) -> Literal["ask_openai", "ask_ollama"]:
    return "ask_openai" if "OpenAI" in state["query"] else "ask_ollama"

def ask_ollama(state: MyState) -> dict:
    msg = builder.chat_model.invoke(
        [{"role":"user","content":state["query"]}]
    )
    return {"response": msg.content}

builder = StateGraph(MyState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="...")
builder.init_chat_model(provider="ollama", model="llama2", host="http://localhost:11434")

builder.add_node("ask_openai", ask_llm)
builder.add_node("ask_ollama", ask_ollama)
builder.add_conditional_edges(START, decide_agent, ["ask_openai","ask_ollama"])
builder.add_edge("ask_openai", END)
builder.add_edge("ask_ollama", END)

graph = builder.compile()
```

***

### 4. Functional API 예시 <a href="#id-4-functional-api" id="id-4-functional-api"></a>

Functional API에서는 Python 함수와 데코레이터로 에이전트를 정의한다.​

```python
from langgraph.func import entrypoint, task, init_chat_model

# 모델 초기화 (글로벌 설정)
init_chat_model(provider="openai", model="gpt-4", api_key="...")
init_chat_model(provider="ollama", model="llama2", host="http://localhost:11434")

@task
def ask_openai(query: str) -> str:
    return builder.chat_model.invoke([{"role":"user","content":query}]).content

@task
def ask_ollama(query: str) -> str:
    return builder.chat_model.invoke([{"role":"user","content":query}]).content

@entrypoint
def my_agent(query: str) -> str:
    if "Ollama" in query:
        return ask_ollama(query)
    return ask_openai(query)

# 실행
response = my_agent("Ollama API로 질문합니다.")
print(response)
```

***

## 5. 패턴   <a href="#id-5" id="id-5"></a>

### 5-1. Sequential Processing <a href="#id-1-sequential-processing" id="id-1-sequential-processing"></a>

여러 작업(task)을 순차적으로 실행하는 가장 기본 패턴.​

```python
from langgraph.graph import StateGraph, START, END
from typing import TypedDict
from langchain.messages import SystemMessage

class SeqState(TypedDict):
    query: str
    draft: str
    final: str

builder = StateGraph(SeqState)
# 모델 초기화
builder.init_chat_model(provider="openai", model="gpt-4", api_key="OPENAI_API_KEY")
builder.init_chat_model(provider="ollama", model="llama2", host="http://localhost:11434")

# 1단계: 초안 생성 (OpenAI)
def draft_step(state: SeqState) -> dict:
    msg = builder.chat_model.invoke(
        [SystemMessage(content=state["query"])]
    )
    return {"draft": msg.content}

# 2단계: 초안 개선 (Ollama)
def refine_step(state: SeqState) -> dict:
    msg = builder.chat_model.invoke(
        [{"role":"user","content":state["draft"]}]
    )
    return {"final": msg.content}

builder.add_node("draft", draft_step)
builder.add_node("refine", refine_step)
builder.add_edge(START, "draft")
builder.add_edge("draft", "refine")
builder.add_edge("refine", END)

graph = builder.compile()
result = graph.invoke({"query": "LangGraph 소개 문장 작성"})
print(result["final"])
```

***

### 5-2. Conditional Routing <a href="#id-2-conditional-routing" id="id-2-conditional-routing"></a>

상태(state)를 검사해 분기 조건에 따라 다른 경로를 타는 패턴.​

```python
from typing import Literal

class CondState(TypedDict):
    question: str
    answer: str

builder = StateGraph(CondState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="OPENAI_API_KEY")
builder.init_chat_model(provider="ollama", model="llama2", host="http://localhost:11434")

def choose_model(state: CondState) -> Literal["use_openai","use_ollama"]:
    return "use_openai" if "OpenAI" in state["question"] else "use_ollama"

def use_openai(state: CondState) -> dict:
    msg = builder.chat_model.invoke([{"role":"user","content":state["question"]}])
    return {"answer": msg.content}

def use_ollama(state: CondState) -> dict:
    msg = builder.chat_model.invoke([{"role":"user","content":state["question"]}])
    return {"answer": msg.content}

builder.add_node("use_openai", use_openai)
builder.add_node("use_ollama", use_ollama)
builder.add_conditional_edges(START, choose_model, ["use_openai","use_ollama"])
builder.add_edge("use_openai", END)
builder.add_edge("use_ollama", END)

graph = builder.compile()
```

***

### 5-3. Parallel Calls (`Send` API) <a href="#undefined" id="undefined"></a>

목록 기반의 동시 다발 작업을 병렬 처리하고 결과를 병합하는 패턴.​

```python
from langgraph.types import Send
from typing import TypedDict

class ParallelState(TypedDict):
    prompts: list[str]
    results: list[str]

builder = StateGraph(ParallelState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="OPENAI_API_KEY")

def fan_out(state: ParallelState):
    # prompts 리스트의 각 항목에 대해 병렬 워커 생성
    return [Send("generate", {"query": q}) for q in state["prompts"]]

def generate(state: dict) -> dict:
    msg = builder.chat_model.invoke([{"role":"user","content": state["query"]}])
    return {"results": [msg.content]}

builder.add_node("fan_out", fan_out)
builder.add_node("generate", generate)
builder.add_edge(START, "fan_out")
builder.add_edge("generate", END)

graph = builder.compile()
result = graph.invoke({"prompts": ["A","B","C"]})
print(result["results"])
```

***

### 5-4. Tool Integration <a href="#id-4-tool-integration" id="id-4-tool-integration"></a>

ToolNode를 사용해 LLM이 요청한 외부 도구를 자동 호출하는 패턴.​

```python
from langgraph.graph import StateGraph, START, END
from langgraph.nodes import ToolNode
from typing import TypedDict

class ToolState(TypedDict):
    messages: list
    output: str

builder = StateGraph(ToolState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="OPENAI_API_KEY")

# 외부 도구 예제
def search_tool(query: str) -> str:
    return f"검색 결과 for {query}"

# ToolNode 등록
tool_node = ToolNode(
    llm=builder.chat_model,
    tools={"search": search_tool}
)

# 워크플로우
builder.add_node("chat", lambda s: {"messages": s["messages"]})
builder.add_node("tools", tool_node)
builder.add_edge(START, "chat")
builder.add_edge("chat", "tools")
builder.add_edge("tools", END)

graph = builder.compile()
```

***

### 5-5. Human-in-the-Loop <a href="#id-5-human-in-the-loop" id="id-5-human-in-the-loop"></a>

`interrupt`를 통해 실행 중단 후 사람의 피드백을 받아 재개하는 패턴.​

```python
from langgraph.types import interrupt, Command
from langgraph.graph import StateGraph, START, END
from typing import TypedDict

class HumanState(TypedDict):
    draft: str
    feedback: str
    final: str

builder = StateGraph(HumanState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="OPENAI_API_KEY")

def draft(state: HumanState) -> dict:
    msg = builder.chat_model.invoke([{"role":"user","content":"초안 생성"}])
    return {"draft": msg.content}

def get_feedback(state: HumanState) -> dict:
    # 사용자 입력 대기
    resp = interrupt({"draft": state["draft"]})
    return {"feedback": resp["data"]}

def finalize(state: HumanState) -> dict:
    msg = builder.chat_model.invoke(
        [{"role":"user","content": state["feedback"]}]
    )
    return {"final": msg.content}

builder.add_node("draft", draft)
builder.add_node("get_feedback", get_feedback)
builder.add_node("finalize", finalize)
builder.add_edge(START, "draft")
builder.add_edge("draft", "get_feedback")
builder.add_edge("get_feedback", "finalize")
builder.add_edge("finalize", END)

graph = builder.compile()
```

***

### 5-6. Time-Travel Debugging <a href="#id-6-time-travel-debugging" id="id-6-time-travel-debugging"></a>

체크포인트를 탐색하고 과거 상태에서 재실행하는 패턴.​

```python
# 기존 그래프(graph) 컴파일 및 실행 후
history = graph.get_state_history({"configurable":{"thread_id":"t1"}})
# 특정 checkpoint 선택
checkpoint_id = history[-2].configurable["checkpoint_id"]

# 과거 상태에서 재실행
config = {"configurable":{"thread_id":"t1","checkpoint_id":checkpoint_id}}
result = graph.invoke(None, config)
print("Rolled back result:", result)
```
