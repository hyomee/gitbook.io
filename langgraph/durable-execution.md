# Durable execution

LangGraph v1.0의 **Durable Execution** 기능은 **장시간 대기**, **장애 복구**, **재시도 로직**, **일관된 상태 관리**를 보장하여 프로덕션 환경에서 **안정적이고 신뢰 가능한** 에이전트 워크플로우를 구축할 수 있게 한다. &#x20;

***

## 1. Durable Execution 개념 <a href="#id-1-durable-execution" id="id-1-durable-execution"></a>

Durable Execution은 다음을 지원합니다:

* **장기 실행**: 수시간\~수일에 걸친 워크플로우
* **장애 복구**: 인스턴스 다운 시 자동 복원
* **자동 재시도**: 실패 단계에 대한 재시도
* **상태 일관성**: 체크포인터 기반 원자적 상태 저장\
  체크포인터(checkpointer)와 **관찰성(Observability)**, **재시도 전략**을 결합하여 구현됩니다.​

***

## 2. `init_chat_model` 초기화 <a href="#undefined" id="undefined"></a>

```python
builder.init_chat_model(
    provider="openai",
    model="gpt-4",
    api_key="OPENAI_API_KEY",
    temperature=0.7
)
builder.init_chat_model(
    provider="ollama",
    model="llama2",
    host="http://localhost:11434",
    stream=False
)
```

***

## 3. Graph API 패턴 <a href="#id-3-graph-api" id="id-3-graph-api"></a>

### 3.1. 체크포인터 기반 복원 및 재시도

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.postgres import PostgresSaver
from langchain.messages import SystemMessage

class DurableState(TypedDict):
    prompt: str
    response: str
    attempts: int

# PostgreSQL 체크포인터
postgres_saver = PostgresSaver.from_conn_string("postgresql://user:pass@/db")

builder = StateGraph(DurableState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="...")
builder.init_chat_model(provider="ollama", model="llama2", host="http://localhost:11434")

def call_llm(state: DurableState) -> dict:
    # idempotent 호출: 이미 응답이 있으면 재호출하지 않음
    if state.get("response"):
        return {}
    try:
        msg = builder.chat_model.invoke(
            [SystemMessage(content=state["prompt"])]
        )
        return {"response": msg.content, "attempts": state.get("attempts",0) + 1}
    except Exception:
        # 실패 시 재시도 횟수 증가
        return {"attempts": state.get("attempts",0) + 1}

builder.add_node("call_llm", call_llm)
builder.add_edge(START, "call_llm")
builder.add_conditional_edges(
    "call_llm",
    lambda s: END if s.get("response") else "call_llm",
    [END, "call_llm"]
)

graph = builder.compile(checkpointer=postgres_saver)
# 장애 시에도 마지막 체크포인트부터 자동 복원 및 재시도
result = graph.invoke({"prompt":"Durable test"})
print(result["response"])
```

### 3.2. 장애 감지 및 알림

```python
from langgraph.types import Command
from langgraph.checkpoint.postgres import PostgresSaver

def notify_failure(state: DurableState) -> Command[Literal["END"]]:
    # 예: 슬랙 알림 전송 로직 삽입
    send_slack("LLM 호출 반복 실패")
    return Command(goto="END")

builder.add_node("notify", notify_failure)
# 최대 3회 시도 후 실패 처리
builder.add_conditional_edges(
    "call_llm",
    lambda s: "notify" if s["attempts"] >= 3 and not s.get("response") else (END if s.get("response") else "call_llm"),
    ["notify", END, "call_llm"]
)
```

***

## 4. Functional API 패턴 <a href="#id-4-functional-api" id="id-4-functional-api"></a>

```python
from langgraph.func import entrypoint, task, init_chat_model
from langgraph.checkpoint.sqlite import SqliteSaver

# 초기화
init_chat_model(provider="openai", model="gpt-4", api_key="...")
init_chat_model(provider="ollama", model="llama2", host="http://localhost:11434")

sqlite_saver = SqliteSaver(db_path="durable.db")

@task
def call_openai(prompt: str, response: str, attempts: int) -> tuple[str,int]:
    if response:
        return response, attempts
    try:
        msg = builder.chat_model.invoke([{"role":"user","content":prompt}])
        return msg.content, attempts + 1
    except:
        return "", attempts + 1

@entrypoint(checkpointer=sqlite_saver, thread_id="durable1")
def durable_workflow(prompt: str) -> str:
    response, attempts = call_openai(prompt, "", 0)
    # 재시도 로직
    while not response and attempts < 3:
        response, attempts = call_openai(prompt, response, attempts)
    if not response:
        raise RuntimeError("LLM 호출 실패")
    return response

# 실행
print(durable_workflow("Durable Execution Test"))
```

***

## 5. 대규모 병렬 Durable 워크플로우 <a href="#id-5---durable" id="id-5---durable"></a>

```python
from langgraph.types import Send
from langgraph.checkpoint.postgres import PostgresSaver

class BatchState(TypedDict):
    prompts: list[str]
    results: list[str]
    attempts: dict[int,int]

postgres = PostgresSaver.from_conn_string("postgresql://user:pass@/db")
builder = StateGraph(BatchState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="...")

def fan_out(state: BatchState):
    return [Send("process", {"idx": i, "prompt": p}) for i,p in enumerate(state["prompts"])]

def process_item(state: dict) -> dict:
    idx = state["idx"]
    prompt = state["prompt"]
    attempts = state["attempts"].get(idx,0)
    try:
        msg = builder.chat_model.invoke([{"role":"user","content":prompt}])
        return {"results": [(idx, msg.content)], "attempts": {idx: attempts+1}}
    except:
        return {"attempts": {idx: attempts+1}}

# 워크플로우 정의
builder.add_node("fan_out", fan_out)
builder.add_node("process", process_item)
builder.add_edge(START, "fan_out")
builder.add_edge("process", END)
graph = builder.compile(checkpointer=postgres)

# 호출
res = graph.invoke({"prompts":["A","B","C"]})
print(res["results"])
```
