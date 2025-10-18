# Streaming

LangGraph v1.0의 **Streaming** 기능은 LLM 호출 결과를 **실시간으로** 클라이언트에 전달하여 **반응형(reactive)** UX를 구현할 수 있도록 지원한다. &#x20;

***

## 1. Streaming 모드 개요 <a href="#id-1-streaming" id="id-1-streaming"></a>

LangGraph v1.0은 다음 5가지 스트리밍 모드를 지원합니다:​

* **values**: 각 노드 실행 완료 시 전체 State
* **updates**: 각 단계의 상태 변경(델타)
* **messages**: LLM 토큰 스트리밍 + 메타데이터
* **custom**: 사용자 정의 이벤트 데이터
* **debug**: 내부 실행 로그 및 타이밍 정보

`stream_mode`에 리스트를 전달하여 **동시 다중 모드**도 가능하며, 각 스트림은 `(mode, chunk)` 형태로 반환됩니다.

***

## 2. `init_chat_model` 초기화 <a href="#undefined" id="undefined"></a>

```
pythonbuilder.init_chat_model(
    provider="openai",
    model="gpt-4",
    api_key="OPENAI_API_KEY",
    stream=True         # OpenAI 스트리밍 활성화
)
builder.init_chat_model(
    provider="ollama",
    model="llama2",
    host="http://localhost:11434",
    stream=True         # Ollama 스트리밍 활성화
)
```

***

## 3. Graph API 예제 <a href="#id-3-graph-api" id="id-3-graph-api"></a>

### 3.1. 간단 메시지 스트리밍

```
pythonfrom langgraph.graph import StateGraph, START, END
from typing import TypedDict
from langchain.messages import SystemMessage

class StreamState(TypedDict):
    prompt: str
    assistant: str

builder = StateGraph(StreamState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="...", stream=True)

def chat(state: StreamState) -> dict:
    # 스트리밍으로 LLM 호출
    return {"assistant": builder.chat_model.invoke([
        SystemMessage(content=state["prompt"])
    ]).content}

builder.add_node("chat", chat)
builder.add_edge(START, "chat")
builder.add_edge("chat", END)

graph = builder.compile()

# 스트리밍 수신
for mode, chunk in graph.stream({"prompt":"Hello"}, stream_mode=["messages"]):
    # mode == "messages", chunk.tokens 리스트
    print("Received token:", chunk.token)
```

### 3.2. 멀티 모드 스트리밍

```
pythonbuilder.init_chat_model(provider="ollama", model="llama2", host="http://localhost:11434", stream=True)

# 동일한 graph 정의...

# messages + updates 모드 동시 사용
for mode, chunk in graph.stream({"prompt":"LangGraph 스트리밍"}, stream_mode=["messages","updates"]):
    if mode == "messages":
        print("Token:", chunk.token)
    else:
        print("State update:", chunk)
```

***

## 4. Functional API 예제 <a href="#id-4-functional-api" id="id-4-functional-api"></a>

```python
from langgraph.func import entrypoint, task, init_chat_model

# 모델 초기화
init_chat_model(provider="openai", model="gpt-4", api_key="...", stream=True)
init_chat_model(provider="ollama", model="llama2", host="http://localhost:11434", stream=True)

@task
def ask_llm(prompt: str) -> str:
    # 스트리밍 결과를 하나의 문자열로 합칠 수도 있고
    response = builder.chat_model.invoke([{"role":"user","content":prompt}])
    return response.content

@entrypoint
def streaming_agent(prompt: str) -> str:
    return ask_llm(prompt)

# 스트리밍 호출
agent = streaming_agent.stream("안녕하세요?", stream_mode=["messages","values"])
for mode, chunk in agent:
    if mode == "messages":
        print("Token:", chunk.token)
    else:
        print("State:", chunk)
```

***

## 5. Custom 스트리밍 이벤트 <a href="#id-5-custom" id="id-5-custom"></a>

```python
from langgraph.graph import StateGraph, START, END
from langgraph.types import Command

class CustomState(TypedDict):
    prompt: str
    progress: int

builder = StateGraph(CustomState)
builder.init_chat_model(provider="openai", model="gpt-4", api_key="...", stream=True)

def track_progress(state: CustomState) -> Command[Literal["END"]]:
    # 진행률 0→100 전송
    return Command(
        update={"progress": state.get("progress",0) + 10},
        emit_custom={"progress": state.get("progress",0) + 10},
        goto="END" if state["progress"] >= 100 else "track_progress"
    )

builder.add_node("track_progress", track_progress)
builder.add_edge(START, "track_progress")
builder.add_edge("track_progress", END)
graph = builder.compile()

for mode, chunk in graph.stream({"prompt":""}, stream_mode=["custom","values"]):
    if mode == "custom":
        print("Custom event:", chunk)
    else:
        print("State snapshot:", chunk)
```

***

## 6. Debug 모드로 내부 실행 추적 <a href="#id-6-debug" id="id-6-debug"></a>

```python
for mode, chunk in graph.stream({"prompt":"Debugging"}, stream_mode=["debug"]):
    # chunk contains execution logs, timing, node names
    print(chunk)
```

## 7. 통합 예제

LangGraph v1.0의 **Streaming** 기능을 **클래스**로 캡슐화하고, **FastAPI** 서버와 통합한 완성도 높은 패턴으로 OpenAI와 Ollama 챗 모델을 `init_chat_model`로 초기화하고, SSE(Server-Sent Events) 스트리밍을 통해 클라이언트에 토큰과 상태 업데이트를 실시간 전송한다.

```python
# app.py
import os
from typing import TypedDict, AsyncGenerator
from fastapi import FastAPI, Request
from fastapi.responses import StreamingResponse
from langgraph.graph import StateGraph, START, END
from langgraph.types import Send
from langchain.messages import SystemMessage

class ChatState(TypedDict):
    prompt: str
    response_tokens: list[str]

class StreamingAgent:
    def __init__(self):
        # 환경변수 로드
        openai_key = os.getenv("OPENAI_API_KEY", "")
        ollama_host = os.getenv("OLLAMA_HOST", "http://localhost:11434")

        # 그래프 빌더 및 상태 스키마
        self.builder = StateGraph(ChatState)
        # 챗 모델 초기화 (스트리밍 활성화)
        self.builder.init_chat_model(
            provider="openai",
            model="gpt-4",
            api_key=openai_key,
            stream=True
        )
        self.builder.init_chat_model(
            provider="ollama",
            model="llama2",
            host=ollama_host,
            stream=True
        )
        # 노드 등록
        self.builder.add_node("chat", self.chat_node)
        self.builder.add_edge(START, "chat")
        self.builder.add_edge("chat", END)
        # 그래프 컴파일
        self.graph = self.builder.compile()

    def chat_node(self, state: ChatState) -> dict | AsyncGenerator[dict, None]:
        """
        스트리밍 LLM 호출을 처리.
        토큰 단위로 즉시 yield하고,
        최종 response_tokens 리스트를 반환.
        """
        # invoke_stream 으로 토큰 스트림 수신
        token_list: list[str] = []
        for chunk in self.builder.chat_model.invoke_stream([
            SystemMessage(content=state["prompt"])
        ]):
            token_list.append(chunk.token)
            # 토큰 이벤트
            yield {"mode": "messages", "token": chunk.token}
        # 전체 토큰 상태 업데이트
        return {"response_tokens": token_list}

    def stream_events(self, prompt: str) -> AsyncGenerator[str, None]:
        """
        FastAPI StreamingResponse용 이벤트 제너레이터.
        messages + values 모드로 스트리밍.
        """
        for mode, chunk in self.graph.stream(
            {"prompt": prompt},
            stream_mode=["messages", "values"]
        ):
            if mode == "messages":
                yield f"event: token\ndata: {chunk.token}\n\n"
            else:  # values
                # 전체 상태 스냅샷
                yield f"event: state\ndata: {chunk}\n\n"

# FastAPI 앱 인스턴스 생성
app = FastAPI()
agent = StreamingAgent()

@app.post("/chat/stream")
async def chat_stream(request: Request):
    body = await request.json()
    prompt = body.get("prompt", "")
    return StreamingResponse(
        agent.stream_events(prompt),
        media_type="text/event-stream"
    )
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

    ```
    bashpip install -r requirements.txt
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
4.  **클라이언트 예제** (JavaScript SSE)

    ```
    javascriptconst evtSource = new EventSource("/chat/stream", {
      method: "POST",
      body: JSON.stringify({ prompt: "Hello, world!" }),
      headers: { "Content-Type": "application/json" }
    });
    evtSource.addEventListener("token", e => {
      console.log("Token:", e.data);
    });
    evtSource.addEventListener("state", e => {
      console.log("State:", JSON.parse(e.data));
    });
    ```
