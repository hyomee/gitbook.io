# Subgraphs

LangGraph v1.0의 **Subgraphs**는 복잡한 워크플로우를 재사용 가능한 작은 그래프로 분리해 모듈화와 유지 보수성을 높이는 기능이다. `init_chat_model`을 통해 OpenAI 및 Ollama API를 일관되게 초기화하고, 메인 그래프 내에서 서브그래프를 호출하거나 여러 에이전트 간 핸드오프를 구현할 수 있다.​

***

## 1. Subgraphs 개념 <a href="#id-1-subgraphs" id="id-1-subgraphs"></a>

* **모듈화:** 공통 기능(예: 채팅, 요약, 검증)을 별도 그래프로 정의
* **재사용성:** 여러 워크플로우에서 동일 서브그래프를 반복 활용
* **Multi-Agent:** 서로 다른 에이전트 간 상태 및 제어권 전환

Subgraph는 **StateGraph 인스턴스**로 정의되며, 부모 그래프에 노드로 추가된다.​

***

## 2. `init_chat_model` 초기화 <a href="#undefined" id="undefined"></a>

```python
from langgraph.graph import StateGraph

builder = StateGraph(SubState)
# OpenAI
builder.init_chat_model(
    provider="openai", model="gpt-4",
    api_key="OPENAI_API_KEY", stream=False
)
# Ollama
builder.init_chat_model(
    provider="ollama", model="llama2",
    host="http://localhost:11434", stream=False
)
```

***

## 3. Graph API 예제 <a href="#id-3-graph-api" id="id-3-graph-api"></a>

### 3.1. 서브그래프 정의

```python
from langgraph.graph import StateGraph, START, END
from typing import TypedDict
from langchain.messages import SystemMessage

# 서브그래프 상태
class ChatSubState(TypedDict):
    prompt: str
    response: str

# 서브그래프 빌더
sub_builder = StateGraph(ChatSubState)
sub_builder.init_chat_model(provider="openai", model="gpt-4", api_key="...")
def chat_node(state: ChatSubState) -> dict:
    msg = sub_builder.chat_model.invoke([
        SystemMessage(content=state["prompt"])
    ])
    return {"response": msg.content}
sub_builder.add_node("chat", chat_node)
sub_builder.add_edge(START, "chat")
sub_builder.add_edge("chat", END)
chat_subgraph = sub_builder.compile()
```

### 3.2. 부모 그래프에 서브그래프 통합

```python
from langgraph.graph import StateGraph, START, END
from typing import TypedDict, Literal
from langgraph.types import Command

class MainState(TypedDict):
    query: str
    answer: str

builder = StateGraph(MainState)
builder.init_chat_model(provider="ollama", model="llama2", host="http://localhost:11434")

# 1) 서브그래프 노드로 추가
builder.add_node("use_chat", chat_subgraph)

# 2) 상태 매핑(서브→부모)
def map_substate(state: MainState, substate: ChatSubState) -> dict:
    return {"answer": substate["response"]}

builder.add_edge(START, "use_chat", map_substate)

builder.add_edge("use_chat", END)
graph = builder.compile()

# 실행
result = graph.invoke({"query":"Hello Subgraph"})
print(result["answer"])
```

***

## 4. Conditional Subgraph 호출 <a href="#id-4-conditional-subgraph" id="id-4-conditional-subgraph"></a>

```python
def choose_subgraph(state: MainState) -> Literal["use_chat","END"]:
    return "use_chat" if "chat" in state["query"] else "END"

builder.add_conditional_edges(START, choose_subgraph, ["use_chat", "END"])
```

***

## 5. Multi-Agent 핸드오프 <a href="#id-5-multi-agent" id="id-5-multi-agent"></a>

### 5.1. 추가 서브그래프 정의

```python
# 요약용 서브그래프
class SummSubState(TypedDict):
    text: str
    summary: str

summ_builder = StateGraph(SummSubState)
summ_builder.init_chat_model(provider="openai", model="gpt-4", api_key="...")
def summarize_node(state: SummSubState) -> dict:
    msg = summ_builder.chat_model.invoke([
        SystemMessage(content=f"요약: {state['text']}")
    ])
    return {"summary": msg.content}
summ_builder.add_node("summarize", summarize_node)
summ_builder.add_edge(START, "summarize")
summ_builder.add_edge("summarize", END)
summ_subgraph = summ_builder.compile()
```

### 5.2. 핸드오프 구현

```python
from langgraph.types import Command

class HandoffState(TypedDict):
    content: str
    summary: str

builder = StateGraph(HandoffState)
builder.init_chat_model(provider="ollama", model="llama2", host="http://localhost:11434")

# 첫 번째 에이전트: 챗
builder.add_node("chat", chat_subgraph)

# 두 번째 에이전트: 요약
builder.add_node("summ", summ_subgraph)

# 핸드오프 노드
def handoff_node(state: HandoffState) -> Command:
    return Command(
        update={"text": state["content"]},
        goto="summ"  # summ 서브그래프로 전환
    )

builder.add_node("handoff", handoff_node)
builder.add_edge(START, "chat")
builder.add_edge("chat", "handoff")
builder.add_edge("handoff", "summ")
builder.add_edge("summ", END)

graph = builder.compile()
```

***

## 6. Functional API에서 Subgraphs <a href="#id-6-functional-api-subgraphs" id="id-6-functional-api-subgraphs"></a>

```python
from langgraph.func import entrypoint, task, init_chat_model, use_subgraph

# 서브그래프(task) 정의
@task
def chat_task(prompt: str) -> str:
    return builder.chat_model.invoke([{"role":"user","content":prompt}]).content

@task
def summar_task(text: str) -> str:
    return builder.chat_model.invoke([{"role":"user","content":f"요약: {text}"}]).content

# 메인 워크플로우에서 서브그래프 호출
@entrypoint
def workflow(query: str) -> str:
    answer = chat_task(query)
    summary = summar_task(answer)
    return summary

# 실행
print(workflow("LangGraph Subgraphs Example"))
```

## 7. 통합 예제

LangGraph v1.0의 **Subgraphs** 기능을 활용하여, 채팅과 요약 기능을 각각 서브그래프로 분리한 뒤 FastAPI 서버와 통합하는 예제로  OpenAI와 Ollama 챗 모델을 `init_chat_model`로 초기화하고, 부모 그래프에서 서브그래프를 호출하여 모듈화된 워크플로우를 구현한다

```python
# app.py
import os
import uuid
from typing import TypedDict, List, Dict, Any, Literal
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from fastapi.responses import JSONResponse
from langgraph.graph import StateGraph, START, END
from langchain.messages import SystemMessage

# 1) 상태 스키마
class ChatSubState(TypedDict):
    prompt: str
    response: str

class SummSubState(TypedDict):
    text: str
    summary: str

class MainState(TypedDict):
    query: str
    response: str
    summary: str

# 2) 요청/응답 모델
class QueryRequest(BaseModel):
    query: str

class ResultResponse(BaseModel):
    thread_id: str
    response: str
    summary: str

class TravelRequest(BaseModel):
    thread_id: str
    checkpoint_id: str

# 3) 클래스 기반 에이전트
class SubgraphAgent:
    def __init__(self):
        openai_key = os.getenv("OPENAI_API_KEY", "")
        ollama_host = os.getenv("OLLAMA_HOST", "http://localhost:11434")

        # 3.1) 채팅 서브그래프 정의
        chat_builder = StateGraph(ChatSubState)
        chat_builder.init_chat_model(
            provider="openai", model="gpt-4", api_key=openai_key, stream=False
        )
        chat_builder.init_chat_model(
            provider="ollama", model="llama2", host=ollama_host, stream=False
        )
        def chat_node(state: ChatSubState) -> dict:
            msg = chat_builder.chat_model.invoke([
                SystemMessage(content=state["prompt"])
            ])
            return {"response": msg.content}
        chat_builder.add_node("chat", chat_node)
        chat_builder.add_edge(START, "chat")
        chat_builder.add_edge("chat", END)
        self.chat_subgraph = chat_builder.compile()

        # 3.2) 요약 서브그래프 정의
        summ_builder = StateGraph(SummSubState)
        summ_builder.init_chat_model(
            provider="openai", model="gpt-4", api_key=openai_key, stream=False
        )
        summ_builder.init_chat_model(
            provider="ollama", model="llama2", host=ollama_host, stream=False
        )
        def sum_node(state: SummSubState) -> dict:
            msg = summ_builder.chat_model.invoke([
                SystemMessage(content=f"요약: {state['text']}")
            ])
            return {"summary": msg.content}
        summ_builder.add_node("summarize", sum_node)
        summ_builder.add_edge(START, "summarize")
        summ_builder.add_edge("summarize", END)
        self.summ_subgraph = summ_builder.compile()

        # 3.3) 부모 그래프 정의
        self.builder = StateGraph(MainState)
        self.builder.init_chat_model(
            provider="openai", model="gpt-4", api_key=openai_key, stream=False
        )
        self.builder.init_chat_model(
            provider="ollama", model="llama2", host=ollama_host, stream=False
        )
        # 부모 노드: 채팅 서브그래프
        self.builder.add_node("chat", self.chat_subgraph)
        # 부모 노드: 요약 서브그래프
        self.builder.add_node("summarize", self.summ_subgraph)
        # 엣지 정의
        self.builder.add_edge(START, "chat", lambda state, sub: {"response": sub["response"]})
        self.builder.add_edge("chat", "summarize", lambda state, sub: {"text": state["response"]})
        self.builder.add_edge("summarize", END, lambda state, sub: {"summary": sub["summary"]})
        self.graph = self.builder.compile()

    def run(self, query: str) -> Dict[str, Any]:
        thread_id = str(uuid.uuid4())
        config = {"configurable": {"thread_id": thread_id}}
        result = self.graph.invoke({"query": query, "response": "", "summary": ""}, config)
        return {"thread_id": thread_id, "response": result["response"], "summary": result["summary"]}

    def get_history(self, thread_id: str) -> List[Dict[str, Any]]:
        history = self.graph.get_state_history({"configurable": {"thread_id": thread_id}})
        return [{"checkpoint_id": h.configurable["checkpoint_id"], "state": h.state} for h in history]

    def travel(self, thread_id: str, checkpoint_id: str) -> Dict[str, Any]:
        config = {"configurable": {"thread_id": thread_id, "checkpoint_id": checkpoint_id}}
        return self.graph.invoke(None, config)

# 4) FastAPI 앱 및 엔드포인트
app = FastAPI()
agent = SubgraphAgent()

@app.post("/query", response_model=ResultResponse)
def handle_query(req: QueryRequest):
    res = agent.run(req.query)
    return JSONResponse(content=res)

@app.get("/history/{thread_id}")
def get_history(thread_id: str):
    return JSONResponse(content=agent.get_history(thread_id))

@app.post("/travel")
def travel(req: TravelRequest):
    try:
        res = agent.travel(req.thread_id, req.checkpoint_id)
        return JSONResponse(content=res)
    except:
        raise HTTPException(status_code=404, detail="Invalid thread or checkpoint")
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
3.  서버 실행

    ```bash
    uvicorn app:app --reload --port 8000
    ```

#### 사용 예시

1.  **쿼리 실행**

    ```bash
    curl -X POST http://localhost:8000/query \
      -H "Content-Type: application/json" \
      -d '{"query":"LangGraph Subgraphs 예시"}'
    ```
2.  **이력 조회**

    ```bash
    curl http://localhost:8000/history/{thread_id}
    ```
3.  **체크포인트 여행**

    ```bash
    curl -X POST http://localhost:8000/travel \
      -H "Content-Type: application/json" \
      -d '{"thread_id":"...","checkpoint_id":"..."}'
    ```
