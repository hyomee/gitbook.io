# Add and manage memory

LangGraph v1.0의 **Memory** 기능은 **단기 메모리(checkpointer)** 와 **장기 메모리(store)** 를 통해 대화 컨텍스트를 유지하고, 사용자별 정보를 영속적으로 관리할 수 있도록 한다. OpenAI와 Ollama API를 `init_chat_model`로 초기화하여 Graph API와 Functional API 양쪽에서 활용하는 다양한 메모리 패턴​

***

## 1. Memory 구성 요소 <a href="#id-1-memory" id="id-1-memory"></a>

### 1.1. Checkpointer (단기 메모리)

* **역할**: 각 super-step마다 상태를 저장하여 대화 히스토리 유지
* **구현체**: MemorySaver, SqliteSaver, PostgresSaver​

### 1.2. Store (장기 메모리)

* **역할**: 여러 thread 간 공유 데이터 저장 (사용자 프로필, 선호도 등)
* **구현체**: InMemoryStore, PostgresStore​

***

## 2. `init_chat_model` 초기화 <a href="#undefined" id="undefined"></a>

```
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
    host="http://localhost:11434"
)
```

***

## 3. Graph API 패턴 <a href="#id-3-graph-api" id="id-3-graph-api"></a>

### 3.1. 기본 Checkpointer (대화 히스토리)

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import MemorySaver
from langchain.messages import HumanMessage, AIMessage
from typing import TypedDict, Annotated
from langgraph.graph.message import add_messages

class MemoryState(TypedDict):
    messages: Annotated[list, add_messages]

builder = StateGraph(MemoryState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="...")

def chat_node(state: MemoryState) -> dict:
    response = builder.chat_model.invoke(state["messages"])
    return {"messages": [response]}

builder.add_node("chat", chat_node)
builder.add_edge(START, "chat")
builder.add_edge("chat", END)

# MemorySaver로 대화 히스토리 유지
memory = MemorySaver()
graph = builder.compile(checkpointer=memory)

# 첫 번째 대화
config1 = {"configurable": {"thread_id": "user_123"}}
result1 = graph.invoke(
    {"messages": [HumanMessage(content="내 이름은 Alice야")]},
    config1
)
print(result1["messages"][-1].content)

# 두 번째 대화 (이전 메시지 자동 로드)
result2 = graph.invoke(
    {"messages": [HumanMessage(content="내 이름이 뭐라고 했지?")]},
    config1
)
print(result2["messages"][-1].content)  # "Alice라고 하셨습니다"
```

### 3.2. Store를 사용한 장기 메모리

```python
from langgraph.store import InMemoryStore
from langchain.messages import SystemMessage

class UserState(TypedDict):
    user_id: str
    messages: Annotated[list, add_messages]

store = InMemoryStore()
builder = StateGraph(UserState)
builder.init_chat_model(provider="ollama", model="llama2", host="http://localhost:11434")

def personalized_chat(state: UserState) -> dict:
    # Store에서 사용자 정보 조회
    user_data = store.get(("users", state["user_id"]))
    user_name = user_data.get("name", "Guest") if user_data else "Guest"
    
    # 시스템 메시지에 사용자 정보 포함
    messages = [
        SystemMessage(content=f"사용자 이름: {user_name}")
    ] + state["messages"]
    
    response = builder.chat_model.invoke(messages)
    return {"messages": [response]}

builder.add_node("chat", personalized_chat)
builder.add_edge(START, "chat")
builder.add_edge("chat", END)

memory = MemorySaver()
graph = builder.compile(checkpointer=memory)

# Store에 사용자 정보 저장
store.put(("users", "user_123"), "name", {"name": "Alice", "preferences": {"lang": "ko"}})

# 실행
config = {"configurable": {"thread_id": "thread_1"}}
result = graph.invoke(
    {"user_id": "user_123", "messages": [HumanMessage(content="안녕하세요")]},
    config
)
print(result["messages"][-1].content)  # "안녕하세요, Alice님!"
```

### 3.3. PostgreSQL을 사용한 프로덕션 메모리

```python
from langgraph.checkpoint.postgres import PostgresSaver

DB_URI = "postgresql://user:pass@localhost:5432/langgraph"
postgres_saver = PostgresSaver.from_conn_string(DB_URI)

builder = StateGraph(MemoryState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="...")

# 동일한 노드 정의...

with postgres_saver as saver:
    saver.setup()
    graph = builder.compile(checkpointer=saver)
    
    config = {"configurable": {"thread_id": "prod_user_1"}}
    result = graph.invoke(
        {"messages": [HumanMessage(content="프로덕션 테스트")]},
        config
    )
```

### 3.4. 대화 요약 메모리 (압축)

```python
from langchain.schema import SystemMessage

class SummaryState(TypedDict):
    messages: Annotated[list, add_messages]
    summary: str

builder = StateGraph(SummaryState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="...")

def summarize_node(state: SummaryState) -> dict:
    # 메시지가 10개 이상이면 요약
    if len(state["messages"]) > 10:
        summary_prompt = "다음 대화를 요약하세요:\n" + \
            "\n".join([m.content for m in state["messages"]])
        summary = builder.chat_model.invoke([
            SystemMessage(content=summary_prompt)
        ])
        return {
            "summary": summary.content,
            "messages": state["messages"][-5:]  # 최근 5개만 유지
        }
    return {}

def chat_node(state: SummaryState) -> dict:
    # 요약이 있으면 컨텍스트에 포함
    context = []
    if state.get("summary"):
        context.append(SystemMessage(content=f"이전 대화 요약: {state['summary']}"))
    
    response = builder.chat_model.invoke(context + state["messages"])
    return {"messages": [response]}

builder.add_node("summarize", summarize_node)
builder.add_node("chat", chat_node)
builder.add_edge(START, "summarize")
builder.add_edge("summarize", "chat")
builder.add_edge("chat", END)

graph = builder.compile(checkpointer=MemorySaver())
```

***

## 4. Functional API 패턴 <a href="#id-4-functional-api" id="id-4-functional-api"></a>

### 4.1. 기본 메모리

```python
from langgraph.func import entrypoint, task, init_chat_model
from langgraph.checkpoint.sqlite import SqliteSaver

init_chat_model(provider="openai", model="gpt-4", api_key="...")
sqlite_saver = SqliteSaver(db_path="memory.db")

@task
def chat(messages: list) -> str:
    response = builder.chat_model.invoke(messages)
    return response.content

@entrypoint(checkpointer=sqlite_saver, thread_id="user_456")
def memory_workflow(user_input: str, history: list) -> str:
    messages = history + [{"role": "user", "content": user_input}]
    return chat(messages)

# 실행
result = memory_workflow("안녕하세요", [])
```

### 4.2. Store 통합

```python
from langgraph.store import InMemoryStore

store = InMemoryStore()

@task
def get_user_context(user_id: str) -> dict:
    return store.get(("users", user_id)) or {}

@entrypoint(store=store, thread_id="thread_789")
def personalized_workflow(user_id: str, query: str) -> str:
    user_ctx = get_user_context(user_id)
    prompt = f"User: {user_ctx.get('name', 'Guest')}, Query: {query}"
    return builder.chat_model.invoke([{"role": "user", "content": prompt}]).content

# Store에 데이터 저장
store.put(("users", "u1"), "name", {"name": "Bob", "age": 30})

# 실행
result = personalized_workflow("u1", "날씨 알려줘")
```

***

## 5. 메모리 관리 패턴 <a href="#id-5" id="id-5"></a>

### 5.1. 메모리 삭제

```python
# 특정 thread 삭제
graph.delete_thread({"configurable": {"thread_id": "user_123"}})

# Store에서 데이터 삭제
store.delete(("users", "user_123"))
```

### 5.2. 메모리 만료 정책

```python
import time
from datetime import datetime, timedelta

class ExpiringMemory:
    def __init__(self, ttl_seconds=3600):
        self.store = InMemoryStore()
        self.ttl = ttl_seconds
    
    def put(self, key, value):
        self.store.put(key, "data", {"value": value, "timestamp": time.time()})
    
    def get(self, key):
        data = self.store.get(key)
        if data and time.time() - data["timestamp"] < self.ttl:
            return data["value"]
        return None
```

### 5.3. 크로스 스레드 메모리 공유

```python
# Thread A에서 저장
store.put(("global", "summary"), "data", {"text": "전체 요약"})

# Thread B에서 조회
global_summary = store.get(("global", "summary"))
```

***

## 6. 메모리 최적화 전략 <a href="#id-6" id="id-6"></a>

1. **슬라이딩 윈도우**: 최근 N개 메시지만 유지
2. **요약 압축**: 긴 대화를 주기적으로 요약
3. **계층적 메모리**: 단기(checkpointer) + 장기(store) 분리
4. **선택적 로드**: 필요한 컨텍스트만 로드

## 7. 통합 예제

OpenAI 및 Ollama API를 `init_chat_model`로 초기화하며, FastAPI와 통합해 메모리 기반 대화·조회·삭제 기능을 제공하는 완성형 클래스 예제

```python
# app.py
import os
from typing import TypedDict, Any, Dict, List
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from fastapi.responses import JSONResponse
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import MemorySaver
from langgraph.store import InMemoryStore
from langchain.schema import HumanMessage, SystemMessage
from langgraph.graph.message import add_messages

# 1) 상태 스키마
class ChatState(TypedDict):
    user_id: str
    messages: add_messages[list[Any]]
    # 영구 메모리(예: 사용자 이름) 로드
    user_name: str

# 2) 요청/응답 모델
class ChatRequest(BaseModel):
    user_id: str
    content: str

class MemoryResponse(BaseModel):
    user_id: str
    user_name: str
    messages: List[Dict[str, Any]]

# 3) 클래스 기반 에이전트
class MemoryAgent:
    def __init__(self):
        # 환경변수 로드
        openai_key = os.getenv("OPENAI_API_KEY", "")
        ollama_host = os.getenv("OLLAMA_HOST", "http://localhost:11434")
        
        # 단기 메모리: MemorySaver
        self.checkpointer = MemorySaver()
        # 장기 메모리: InMemoryStore
        self.store = InMemoryStore()

        # 그래프 빌더 및 상태 스키마
        self.builder = StateGraph(ChatState, store=self.store)
        # 챗 모델 초기화 (stream=False)
        self.builder.init_chat_model(
            provider="openai", model="gpt-4",
            api_key=openai_key, stream=False
        )
        self.builder.init_chat_model(
            provider="ollama", model="llama2",
            host=ollama_host, stream=False
        )

        # 3.1) 노드 정의: 대화 처리
        def chat_node(state: ChatState) -> dict:
            # 장기 메모리에서 사용자 이름 조회
            profile = self.store.get(("users", state["user_id"])) or {}
            name = profile.get("name", "Guest")
            # 시스템 메시지로 사용자 이름 포함
            messages = [SystemMessage(content=f"사용자: {name}")] + state["messages"]
            # LLM 호출
            response = self.builder.chat_model.invoke(messages)
            return {"messages": [response], "user_name": name}

        # 그래프 구성
        self.builder.add_node("chat", chat_node)
        self.builder.add_edge(START, "chat")
        self.builder.add_edge("chat", END)
        self.graph = self.builder.compile(checkpointer=self.checkpointer)

    def chat(self, user_id: str, content: str) -> ChatState:
        thread_id = f"thread_{user_id}"
        config = {"configurable": {"thread_id": thread_id}}
        # 이전 메시지 히스토리 로드
        state = {"user_id": user_id, "messages": [HumanMessage(content=content)], "user_name": ""}
        result = self.graph.invoke(state, config)
        return result

    def get_memory(self, user_id: str) -> Dict[str, Any]:
        # 메시지 히스토리 조회
        thread_id = f"thread_{user_id}"
        history = self.checkpointer.storage.get(thread_id, [])
        messages = [m.dict() for entry in history for m in entry.state["messages"]]
        # 프로필 조회
        profile = self.store.get(("users", user_id)) or {}
        return {"user_id": user_id, "user_name": profile.get("name", "Guest"), "messages": messages}

    def set_user_profile(self, user_id: str, profile: Dict[str, Any]):
        self.store.put(("users", user_id), "profile", profile)

    def clear_memory(self, user_id: str):
        thread_id = f"thread_{user_id}"
        self.checkpointer.storage.pop(thread_id, None)
        self.store.delete(("users", user_id))

# 4) FastAPI 앱 및 엔드포인트
app = FastAPI()
agent = MemoryAgent()

@app.post("/chat", response_model=MemoryResponse)
def chat_endpoint(req: ChatRequest):
    res = agent.chat(req.user_id, req.content)
    return JSONResponse(content={
        "user_id": res["user_id"],
        "user_name": res["user_name"],
        "messages": [m.dict() for m in res["messages"]]
    })

@app.get("/memory/{user_id}", response_model=MemoryResponse)
def get_memory(user_id: str):
    mem = agent.get_memory(user_id)
    return JSONResponse(content=mem)

@app.post("/profile/{user_id}")
def set_profile(user_id: str, profile: Dict[str, Any]):
    agent.set_user_profile(user_id, profile)
    return JSONResponse(content={"status": "ok"})

@app.delete("/memory/{user_id}")
def clear_memory(user_id: str):
    agent.clear_memory(user_id)
    return JSONResponse(content={"status": "cleared"})
```

```textile
# requirements.txt
fastapi
uvicorn[standard]
langchain
langgraph
openai
requests
```

#### 실행 방법 <a href="#undefined" id="undefined"></a>

1.  의존성 설치

    ```bash
    pip install -r requirements.txt
    ```
2.  환경 변수 설정

    ```bash
    export OPENAI_API_KEY="YOUR_OPENAI_KEY"
    export OLLAMA_HOST="http://localhost:11434"
    ```
3.  FastAPI 서버 실행

    ```bash
    uvicorn app:app --reload --port 8000
    ```

#### 사용 예시

1.  **프로필 설정**

    ```bash
    curl -X POST http://localhost:8000/profile/user1 \
      -H "Content-Type: application/json" \
      -d '{"name":"Alice","preferences":{"lang":"ko"}}'
    ```
2.  **대화 전송**

    ```bash
    curl -X POST http://localhost:8000/chat \
      -H "Content-Type: application/json" \
      -d '{"user_id":"user1","content":"안녕하세요"}'
    ```
3.  **메모리 조회**

    ```bash
    curl http://localhost:8000/memory/user1
    ```
4.  **메모리 초기화**

    ```bash
    curl -X DELETE http://localhost:8000/memory/user1
    ```
