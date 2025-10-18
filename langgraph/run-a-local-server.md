# Run a local server

LangGraph v1.0의 **로컬 서버 실행** 기능은 컴파일된 그래프를 HTTP API로 배포해, 외부 애플리케이션이 REST 호출만으로 에이전트를 사용할 수 있도록 한다. OpenAI 및 Ollama API를 `init_chat_model`로 초기화하며, **Graph API** 방식을 기준으로 FastAPI 없이도 간단한 서버를 구동하거나, FastAPI로 래핑해 확장하는 패턴을 모두 제공한다.​

***

### 1. Graph API: 내장 서버 실행 <a href="#id-1-graph-api" id="id-1-graph-api"></a>

LangGraph v1.0은 `local_server` 모듈을 통해 **Flask** 기반 경량 HTTP 서버를 바로 띄울 수 있다.

```python
# server.py
import os
from langgraph.graph import StateGraph, START, END
from langgraph.graph.local_server import serve_graph
from langchain.messages import SystemMessage
from typing import TypedDict

# 1) 상태 스키마 정의
class ChatState(TypedDict):
    prompt: str
    response: str

# 2) 그래프 빌더 및 챗 모델 초기화
builder = StateGraph(ChatState)
builder.init_chat_model(
    provider="openai", model="gpt-4",
    api_key=os.getenv("OPENAI_API_KEY"),
    stream=False
)
builder.init_chat_model(
    provider="ollama", model="llama2",
    host=os.getenv("OLLAMA_HOST","http://localhost:11434"),
    stream=False
)

# 3) 노드 정의
def chat_node(state: ChatState) -> dict:
    msg = builder.chat_model.invoke([SystemMessage(content=state["prompt"])])
    return {"response": msg.content}

builder.add_node("chat", chat_node)
builder.add_edge(START, "chat")
builder.add_edge("chat", END)

# 4) 그래프 컴파일
graph = builder.compile()

# 5) 로컬 서버 실행 (기본 포트 8001)
if __name__ == "__main__":
    serve_graph(graph, host="0.0.0.0", port=8001)
```

#### 사용

```
bashpython server.py
```

* `POST /invoke` 엔드포인트:\
   JSON 바디: `{"inputs": {"prompt":"Hello"}}`\
   응답: `{"outputs": {"response":"Hi there!"}}`

***

### 2. FastAPI 래핑 패턴 <a href="#id-2-fastapi" id="id-2-fastapi"></a>

내장 서버 대신 FastAPI로 더욱 세부 제어가 필요할 때 사용한다.

```python
# fastapi_server.py
import os
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from langgraph.graph import StateGraph, START, END

class ChatStateModel(BaseModel):
    prompt: str

class ChatResponseModel(BaseModel):
    response: str

# 1) 그래프 구성 (위 server.py와 동일)
class ChatState(dict):
    prompt: str
    response: str

builder = StateGraph(ChatState)
builder.init_chat_model(
    provider="openai", model="gpt-4",
    api_key=os.getenv("OPENAI_API_KEY"), stream=False
)
builder.init_chat_model(
    provider="ollama", model="llama2",
    host=os.getenv("OLLAMA_HOST","http://localhost:11434"), stream=False
)

def chat_node(state: ChatState) -> dict:
    from langchain.messages import SystemMessage
    msg = builder.chat_model.invoke([SystemMessage(content=state["prompt"])])
    return {"response": msg.content}

builder.add_node("chat", chat_node)
builder.add_edge(START, "chat")
builder.add_edge("chat", END)
graph = builder.compile()

# 2) FastAPI 앱 정의
app = FastAPI()

@app.post("/invoke", response_model=ChatResponseModel)
def invoke(request: ChatStateModel):
    try:
        result = graph.invoke({"prompt": request.prompt})
        return {"response": result["response"]}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

#### 실행

```bash
uvicorn fastapi_server:app --reload --port 8002
```

* `POST http://localhost:8002/invoke`\
  Body: `{"prompt":"Hi"}`
* 응답: `{"response":"Hello!"}`

***

### 3. 확장 패턴: 상태·스트리밍 지원 <a href="#id-3" id="id-3"></a>

* **체크포인터 통합**: `compile(checkpointer=...)` 파라미터로 장기·단기 메모리 유지
* **스트리밍 엔드포인트**: `graph.stream(...)`을 FastAPI `StreamingResponse`로 래핑
* **인증·로깅**: FastAPI 미들웨어 활용 가능
