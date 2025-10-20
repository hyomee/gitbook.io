# Time travel

LangGraph v1.0의 **Time Travel** 기능은 실행 중 생성된 체크포인트를 탐색하고, 과거 특정 시점으로 돌아가 워크플로우를 재실행하거나 디버깅할 수 있도록 지원한다. OpenAI와 Ollama API를 `init_chat_model`로 초기화하여 Graph API 및 Functional API 양쪽에서 활용하는 패턴 및 예제.

***

## 1. `init_chat_model` 초기화 <a href="#undefined" id="undefined"></a>

```python
from langgraph.graph import StateGraph

builder = StateGraph(MyState)
# OpenAI GPT-4
builder.init_chat_model(
    provider="openai",
    model="gpt-4",
    api_key="OPENAI_API_KEY",
    stream=False
)
# Ollama llama2
builder.init_chat_model(
    provider="ollama",
    model="llama2",
    host="http://localhost:11434",
    stream=False
)
```

***

## 2. Graph API 패턴 <a href="#id-2-graph-api" id="id-2-graph-api"></a>

### 2.1. 시간여행 기초

```python
from langgraph.graph import StateGraph, START, END
from langchain.messages import SystemMessage
from typing import TypedDict

class TTState(TypedDict):
    step: int
    history: list[str]

builder = StateGraph(TTState)
# init_chat_model 설정

def step_node(state: TTState) -> dict:
    msg = builder.chat_model.invoke([
        SystemMessage(content=f"Step {state['step']}: 진행하세요")
    ])
    return {
        "history": [msg.content],
        "step": state["step"] + 1
    }

builder.add_node("step", step_node)
builder.add_edge(START, "step")
builder.add_edge("step", END)
graph = builder.compile()

# 1) 워크플로우 실행, 체크포인트 생성
res1 = graph.invoke({"step": 1, "history": []}, {"configurable":{"thread_id":"tt1"}})

# 2) 상태 이력 조회
history = graph.get_state_history({"configurable":{"thread_id":"tt1"}})
# history: list of checkpoint objects with checkpoint_id and state

# 3) 과거로 돌아가기: 두 번째 체크포인트에서 재실행
cp_id = history[1].configurable["checkpoint_id"]
config = {"configurable":{"thread_id":"tt1","checkpoint_id":cp_id}}
res2 = graph.invoke(None, config)
```

### 2.2. 분기 시나리오 재실행

```python
def decide_node(state: TTState) -> Literal["branch_a","branch_b"]:
    return "branch_a" if state["step"] % 2 == 0 else "branch_b"

def branch_a(state: TTState) -> dict:
    return {"history":[*state["history"], "A"], "step": state["step"]+1}

def branch_b(state: TTState) -> dict:
    return {"history":[*state["history"], "B"], "step": state["step"]+1}

builder = StateGraph(TTState)
# init_chat_model 설정...

builder.add_node("step", step_node)
builder.add_node("branch_a", branch_a)
builder.add_node("branch_b", branch_b)
builder.add_conditional_edges("step", decide_node, ["branch_a","branch_b"])
builder.add_edge("branch_a", END)
builder.add_edge("branch_b", END)
graph = builder.compile()

# 실행 후 체크포인트 선택하여 다른 분기로 재실행 가능
```

***

## 3. Functional API 패턴 <a href="#id-3-functional-api" id="id-3-functional-api"></a>

```python
from langgraph.func import entrypoint, task, init_chat_model

# init_chat_model 설정

@task
def do_step(step: int) -> tuple[int,str]:
    msg = builder.chat_model.invoke([{"role":"user","content":f"Step{step}"}])
    return step+1, msg.content

@entrypoint(thread_id="tt2")
def workflow(step: int) -> str:
    # 첫 단계
    new_step, out = do_step(step)
    # 체크포인트 자동 생성됨
    # 클라이언트가 과거 checkpoint_id로 재호출 가능
    return out

# 재실행: workflow.invoke(None, {"configurable":{"thread_id":"tt2","checkpoint_id":cp_id}})
```

***

## 4. 타임트래블 활용 예제 <a href="#id-4" id="id-4"></a>

1. **디버깅**: 분기 로직 오류 분석
2. **“what-if” 시나리오**: 다른 입력/변수로 재실행
3. **버전 간 비교**: 여러 체크포인트 상태 비교

## 5. 통합예제

LangGraph v1.0의 **Time Travel** 기능을 FastAPI와 통합하여, 실행 이력을 조회하고 과거 특정 체크포인트로 돌아가 워크플로우를 재실행할 수 있는 완전한 예제로  OpenAI와 Ollama API를 `init_chat_model`로 초기화하며, 클래스 구조로 캡슐화했다.

```python
# app.py
import os
import uuid
from typing import TypedDict, Any, Dict, List
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from fastapi.responses import JSONResponse
from langgraph.graph import StateGraph, START, END
from langchain.messages import SystemMessage

# 1) 상태 스키마
class TTState(TypedDict):
    step: int
    history: List[str]

# 2) 요청/응답 모델
class StartRequest(BaseModel):
    initial_step: int

class TravelRequest(BaseModel):
    thread_id: str
    checkpoint_id: str

class HistoryItem(BaseModel):
    checkpoint_id: str
    state: TTState

# 3) 클래스 기반 에이전트
class TimeTravelAgent:
    def __init__(self):
        openai_key = os.getenv("OPENAI_API_KEY", "")
        ollama_host = os.getenv("OLLAMA_HOST", "http://localhost:11434")

        self.builder = StateGraph(TTState)
        # 챗 모델 초기화 (stream=False)
        self.builder.init_chat_model(
            provider="openai", model="gpt-4",
            api_key=openai_key, stream=False
        )
        self.builder.init_chat_model(
            provider="ollama", model="llama2",
            host=ollama_host, stream=False
        )

        # 워크플로우 노드 정의
        def step_node(state: TTState) -> dict:
            msg = self.builder.chat_model.invoke([
                SystemMessage(content=f"Executing step {state['step']}")
            ])
            return {
                "step": state["step"] + 1,
                "history": state["history"] + [msg.content]
            }

        self.builder.add_node("step", step_node)
        self.builder.add_edge(START, "step")
        self.builder.add_edge("step", END)
        self.graph = self.builder.compile()

    def start(self, initial_step: int) -> Dict[str, Any]:
        thread_id = str(uuid.uuid4())
        config = {"configurable": {"thread_id": thread_id}}
        result = self.graph.invoke(
            {"step": initial_step, "history": []},
            config
        )
        checkpoint_id = result["configurable"]["checkpoint_id"]
        return {
            "thread_id": thread_id,
            "checkpoint_id": checkpoint_id,
            "state": result
        }

    def history(self, thread_id: str) -> List[Dict[str, Any]]:
        config = {"configurable": {"thread_id": thread_id}}
        history = self.graph.get_state_history(config)
        return [
            {"checkpoint_id": h.configurable["checkpoint_id"], "state": h.state}
            for h in history
        ]

    def travel(self, thread_id: str, checkpoint_id: str) -> Dict[str, Any]:
        config = {
            "configurable": {
                "thread_id": thread_id,
                "checkpoint_id": checkpoint_id
            }
        }
        result = self.graph.invoke(None, config)
        return result

# 4) FastAPI 앱 및 엔드포인트
app = FastAPI()
agent = TimeTravelAgent()

@app.post("/tt/start")
def tt_start(req: StartRequest):
    data = agent.start(req.initial_step)
    return JSONResponse(content=data)

@app.get("/tt/history/{thread_id}")
def tt_history(thread_id: str):
    hist = agent.history(thread_id)
    return JSONResponse(content=hist)

@app.post("/tt/travel")
def tt_travel(req: TravelRequest):
    try:
        data = agent.travel(req.thread_id, req.checkpoint_id)
        return JSONResponse(content=data)
    except Exception:
        raise HTTPException(status_code=404, detail="Invalid thread_id or checkpoint_id")
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

1.  `requirements.txt` 설치

    ```bash
    pip install -r requirements.txt
    ```
2.  환경 변수 설정

    ```
    bashexport OPENAI_API_KEY="YOUR_OPENAI_KEY"
    export OLLAMA_HOST="http://localhost:11434"
    ```
3.  FastAPI 서버 실행

    ```
    bashuvicorn app:app --reload --port 8000
    ```

#### 사용 예시

1.  **시작**

    ```bash
    curl -X POST http://localhost:8000/tt/start \
      -H "Content-Type: application/json" \
      -d '{"initial_step":1}'
    ```

    응답:

    ```
    json{
      "thread_id":"uuid-xxx",
      "checkpoint_id":"cp-yyy",
      "state":{"step":2,"history":["Executing step 1"]}
    }
    ```
2.  **이력 조회**

    ```
    bashcurl http://localhost:8000/tt/history/uuid-xxx
    ```

    응답:

    ```
    json[
      {"checkpoint_id":"cp-yyy","state":{"step":2,"history":["Executing step 1"]}},
      {"checkpoint_id":"cp-zzz","state":{/* 다음 super-step 상태 */}}
    ]
    ```
3.  **과거 지점으로 이동**

    ```bash
    curl -X POST http://localhost:8000/tt/travel \
      -H "Content-Type: application/json" \
      -d '{"thread_id":"uuid-xxx","checkpoint_id":"cp-yyy"}'
    ```

    응답:

    ```
    json{"step":2,"history":["Executing step 1"]}
    ```
