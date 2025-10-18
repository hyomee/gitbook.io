# Persistence

LangGraph v1.0의 **Persistence(영속성)** 기능은 장기 실행 워크플로우에서 **상태 유실 방지**, **오류 복구**, **이력 조회**를 가능하게 한다. 체크포인터(checkpointer)와 스토어(store)로 다양한 저장소에 상태를 저장하고 불러올 수 있으며, OpenAI 및 Ollama API를 사용하는 워크플로우에 손쉽게 통합된다.

***

## 1. Persistence 구성 요소 <a href="#id-1-persistence" id="id-1-persistence"></a>

1. **Checkpointer**
   * Super-step마다 그래프의 State를 저장
   * 복구, 시간여행, 재실행 지원
2. **Store**
   * 여러 thread 간 공유 메타데이터 저장
   * 사용자 세션, 영구 메모리 관리

LangGraph v1.0은 공식적으로 제공하는 Saver 구현체들:​

* **MemorySaver**: 메모리 기반 (테스트/실험용)
* **SqliteSaver**: SQLite 기반 (로컬 개발용)
* **PostgresSaver**: PostgreSQL 기반 (프로덕션용)

***

## 2. `init_chat_model` 초기화 <a href="#undefined" id="undefined"></a>

Persistence 예제들 모두 **init\_chat\_model**로 OpenAI와 Ollama 챗 모델을 초기화하여 일관된 방식으로 LLM을 호출한다.​

```python
builder.init_chat_model(
    provider="openai",
    model="gpt-4",
    api_key="OPENAI_API_KEY",
    temperature=0.5
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

### 3.1. MemorySaver (메모리 저장소)

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict

class MemState(TypedDict):
    prompt: str
    answer: str

# 체크포인터 설정
memory_saver = MemorySaver()
builder = StateGraph(MemState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="...")

# 노드 정의
def ask(state: MemState) -> dict:
    msg = builder.chat_model.invoke([{"role":"user","content": state["prompt"]}])
    return {"answer": msg.content}

builder.add_node("ask", ask)
builder.add_edge(START, "ask")
builder.add_edge("ask", END)

# 그래프 컴파일 및 실행
graph = builder.compile(checkpointer=memory_saver)
result = graph.invoke({"prompt": "Hello, LangGraph!"})
print(result["answer"])
```

### 3.2. SqliteSaver (로컬 SQLite)

```python
from langgraph.checkpoint.sqlite import SqliteSaver

# SQLite 파일 기반 저장소
sqlite_saver = SqliteSaver(db_path="langgraph.db")
builder = StateGraph(MemState)
builder.init_chat_model(provider="ollama", model="llama2", host="http://localhost:11434")

# 동일하게 노드 및 엣지 정의...
graph = builder.compile(checkpointer=sqlite_saver)
# 대화 시도 1
graph.invoke({"prompt": "첫 번째 입력"})
# 대화 시도 2 (이전 상태 누적)
graph.invoke({"prompt": "두 번째 입력"})
```

### 3.3. PostgresSaver (프로덕션 PostgreSQL)

```python
from langgraph.checkpoint.postgres import PostgresSaver

DB_URI = "postgresql://user:pass@localhost:5432/langgraph"
postgres_saver = PostgresSaver.from_conn_string(DB_URI)

builder = StateGraph(MemState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="...")

with postgres_saver as saver:
    saver.setup()  # 테이블 생성 등 초기 설정
    graph = builder.compile(checkpointer=saver)
    graph.invoke({"prompt": "프로덕션 테스트"})
```

***

### 3.4. Store(크로스 스레드 메모리) 패턴 <a href="#id-4-store" id="id-4-store"></a>

```python
from langgraph.store import InMemoryStore

# Store 설정
store = InMemoryStore()
builder = StateGraph(MemState, store=store)
builder.init_chat_model(provider="ollama", model="llama2", host="http://localhost:11434")

def personalize(state: MemState) -> dict:
    user_name = store.get("user_name") or "Guest"
    msg = builder.chat_model.invoke(
        [{"role":"user","content": f"{user_name}, {state['prompt']}"}]
    )
    return {"answer": msg.content}

# 초기 사용자 설정
store.set("user_name", "Alice")

builder.add_node("personalize", personalize)
builder.add_edge(START, "personalize")
builder.add_edge("personalize", END)

graph = builder.compile()
print(graph.invoke({"prompt": "안녕하세요?"})["answer"])
```

***

### 3.5. 시간여행(Time Travel)과 복구 패턴 <a href="#id-5-time-travel" id="id-5-time-travel"></a>

1.  **이력 조회**

    ```python
    history = graph.get_state_history({"configurable":{"thread_id":"t1"}})
    for h in history:
        print(h.configurable["checkpoint_id"])
    ```
2.  **과거 체크포인트에서 재실행**

    ```python
    cfg = {"configurable":{"thread_id":"t1","checkpoint_id": history[1].configurable["checkpoint_id"]}}
    past_result = graph.invoke({"prompt":"복구 테스트"}, cfg)
    print(past_result["answer"])
    ```

***

### 3.6. Functional API에서 Persistence <a href="#id-6-functional-api-persistence" id="id-6-functional-api-persistence"></a>

```python
from langgraph.func import entrypoint, task, init_chat_model
from langgraph.checkpoint.postgres import PostgresSaver

# 모델 및 체크포인터 초기화
init_chat_model(provider="openai", model="gpt-4", api_key="...")
postgres_saver = PostgresSaver.from_conn_string("postgresql://user:pass@/db")

@task
def ask_llm(prompt: str) -> str:
    return builder.chat_model.invoke([{"role":"user","content":prompt}]).content

@entrypoint(checkpointer=postgres_saver, thread_id="user123")
def my_workflow(prompt: str) -> str:
    return ask_llm(prompt)

# 실행
print(my_workflow("영속성 테스트"))
```
