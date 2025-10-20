# Observability

LangGraph v1.0의 **Observability** 기능은 에이전트 실행에 대한 **트레이싱**, **메트릭 수집**, **로그**, **LangSmith 통합**을 통해 워크플로우의 성능 및 상태를 모니터링하고 디버깅할 수 있도록 지원한다. &#x20;

***

## 1. init\_chat\_model 초기화 <a href="#id-1-initchatmodel" id="id-1-initchatmodel"></a>

```
pythonfrom langgraph.graph import StateGraph

builder = StateGraph(MyState)
builder.init_chat_model(
    provider="openai",
    model="gpt-4",
    api_key="OPENAI_API_KEY",
    stream=False
)
builder.init_chat_model(
    provider="ollama",
    model="llama2",
    host="http://localhost:11434",
    stream=False
)
```

***

## 2. LangSmith 트레이싱 설정 <a href="#id-2-langsmith" id="id-2-langsmith"></a>

LangChain의 LangSmith 플랫폼과 네이티브 통합해 **API 호출**, **노드 실행** 등을 자동으로 추적합니다.​

```bash
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY="YOUR_LANGSMITH_KEY"
```

```python
builder = StateGraph(MyState)
# init_chat_model 설정 (위)
# 그래프 컴파일
graph = builder.compile()

# 실행 시 메타데이터 추가
config = {
    "run_name": "observability_run",
    "tags": ["prod","chat"],
    "metadata": {"user_id":"123","feature":"observability"}
}
result = graph.invoke({"input":"Hello"}, config)
```

LangSmith 대시보드에서 각 **LLM 호출**과 **노드 실행 시간**, **입출력**을 시각화할 수 있다.

***

## 3. Graph API 패턴 <a href="#id-3-graph-api" id="id-3-graph-api"></a>

### 3.1. 노드별 로그

```python
import logging
from langgraph.graph import StateGraph, START, END
from langchain.messages import SystemMessage

logging.basicConfig(level=logging.INFO)

class ObsState(dict): pass

builder = StateGraph(ObsState)
# init_chat_model 설정

def logged_node(state: ObsState) -> dict:
    logging.info(f"Entering node with state: {state}")
    resp = builder.chat_model.invoke([SystemMessage(content=state.get("input",""))])
    logging.info(f"Received response: {resp.content}")
    return {"output": resp.content}

builder.add_node("logged", logged_node)
builder.add_edge(START, "logged")
builder.add_edge("logged", END)
graph = builder.compile()

graph.invoke({"input":"Test"})
```

이 패턴으로 **콘솔**, **파일**, **원격 로깅** 스테이크홀더에 기록할 수 있다.

### 3.2. 커스텀 메트릭

```python
from prometheus_client import Counter, start_http_server
from langgraph.graph import StateGraph, START, END
from langchain.messages import SystemMessage

# Prometheus 메트릭 정의
REQUEST_COUNT = Counter("lg_requests_total", "Total LLM requests")
ERROR_COUNT = Counter("lg_errors_total", "Total errors")

start_http_server(8000)  # Prometheus 서버

class MState(dict): pass
builder = StateGraph(MState)
# init_chat_model 설정

def metric_node(state: MState) -> dict:
    REQUEST_COUNT.inc()
    try:
        resp = builder.chat_model.invoke([SystemMessage(content=state["input"])])
        return {"output": resp.content}
    except Exception:
        ERROR_COUNT.inc()
        raise

builder.add_node("metric", metric_node)
builder.add_edge(START, "metric")
builder.add_edge("metric", END)
graph = builder.compile()
```

Prometheus가 `http://localhost:8000/metrics`에서 지표를 스크레이핑한다.

***

## 4. Functional API 패턴 <a href="#id-4-functional-api" id="id-4-functional-api"></a>

```python
from langgraph.func import entrypoint, task, init_chat_model
import logging

# init_chat_model 설정

@task
def process(input: str) -> str:
    logging.info(f"Processing input: {input}")
    return builder.chat_model.invoke([{"role":"user","content":input}]).content

@entrypoint
def workflow(input: str) -> str:
    result = process(input)
    logging.info(f"Workflow result: {result}")
    return result

# 실행
workflow("Observability Test")
```

***

## 5. FastAPI 통합 예제 <a href="#id-5-fastapi" id="id-5-fastapi"></a>

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
import os, logging
from langgraph.graph import StateGraph, START, END
from langchain.messages import SystemMessage

# 로깅 설정
logging.basicConfig(level=logging.INFO)
app = FastAPI()

# 상태 스키마 정의
class ObsState(dict): pass

# 그래프 초기화
builder = StateGraph(ObsState)
builder.init_chat_model(
    provider="openai", model="gpt-4",
    api_key=os.getenv("OPENAI_API_KEY"), stream=False
)
builder.init_chat_model(
    provider="ollama", model="llama2",
    host=os.getenv("OLLAMA_HOST"), stream=False
)

def node(state: ObsState) -> dict:
    logging.info(f"State before: {state}")
    resp = builder.chat_model.invoke([SystemMessage(content=state["input"])])
    logging.info(f"LLM output: {resp.content}")
    return {"output": resp.content}

builder.add_node("obs", node)
builder.add_edge(START, "obs")
builder.add_edge("obs", END)
graph = builder.compile()

@app.post("/invoke")
async def invoke(request: Request):
    data = await request.json()
    config = {"run_name":"api_obs","tags":["api"]}
    result = graph.invoke({"input": data["input"]}, config)
    return JSONResponse(content=result)
```
