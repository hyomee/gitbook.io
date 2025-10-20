# Streaming

**Streaming**은 LLM 응답을 실시간으로 점진적으로 표시하여 사용자 경험을 크게 향상시키는 핵심 기술로 LangChain v1.0은 다양한 스트리밍 API와 모드를 제공하여 토큰 스트리밍, 에이전트 진행 상황, 사용자 정의 업데이트 등을 실시간으로 전달할 수 있다.​

***

## 1. Streaming 핵심 개념 <a href="#streaming" id="streaming"></a>

### Streaming이 중요한 이유

LLM은 응답 생성에 수 초에서 수십 초가 걸릴 수 있습니다. Streaming을 사용하면:​

* **응답성 향상**: 사용자가 즉시 피드백을 받음
* **지연 시간 감소**: 체감 대기 시간 단축
* **투명성 증대**: 진행 상황을 실시간으로 표시

### Streaming API

LangChain v1.0은 세 가지 주요 스트리밍 API를 제공합니다:​

1. **`stream()`**: 동기 스트리밍 - 최종 출력만
2. **`astream()`**: 비동기 스트리밍 - 최종 출력만
3. **`astream_events()`**: 비동기 이벤트 스트리밍 - 중간 단계 + 최종 출력

### LangGraph Stream Modes

LangGraph는 5가지 스트리밍 모드를 지원합니다:​

| 모드           | 설명             | 사용 예시          |
| ------------ | -------------- | -------------- |
| **values**   | 각 단계 후 전체 상태   | 전체 그래프 상태 모니터링 |
| **updates**  | 각 단계 후 상태 변경만  | 네트워크 대역폭 절약    |
| **messages** | LLM 토큰 + 메타데이터 | 토큰별 UI 업데이트    |
| **custom**   | 사용자 정의 데이터     | 도구 진행률 표시      |
| **debug**    | 상세 디버깅 정보      | 개발 및 디버깅       |

***

### 1. 설치 및 기본 설정 <a href="#id-1" id="id-1"></a>

```python
# 필수 패키지 설치
pip install --pre -U langchain
pip install -U langchain-openai langchain-ollama
pip install -U langgraph

# 환경 변수 설정
import os
os.environ["OPENAI_API_KEY"] = "your-openai-api-key"
# Ollama는 로컬에서 실행 (http://localhost:11434)
```

***

## 2. 기본 패턴: Token Streaming (OpenAI) <a href="#id-2---token-streaming-openai" id="id-2---token-streaming-openai"></a>

### 2.1 Simple Token Streaming

```python
from langchain.chat_models import init_chat_model

# init_chat_model로 OpenAI 모델 초기화
model = init_chat_model("openai:gpt-4o", temperature=0.7)

# ===== Sync Streaming =====
print("="*70)
print("Sync Streaming (stream)")
print("="*70)

for chunk in model.stream("Write a short poem about AI"):
    print(chunk.content, end="", flush=True)

# ===== Async Streaming =====
import asyncio

async def async_streaming():
    print("\n\n" + "="*70)
    print("Async Streaming (astream)")
    print("="*70)
    
    async for chunk in model.astream("Explain quantum computing in simple terms"):
        print(chunk.content, end="", flush=True)

asyncio.run(async_streaming())
```

### 2.2 Chain Streaming with Prompt (OpenAI)

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# Prompt 템플릿
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant that explains concepts clearly."),
    ("human", "{topic}")
])

# Chain 구성
chain = prompt | model | StrOutputParser()

# Streaming
print("\n" + "="*70)
print("Chain Streaming")
print("="*70)

for chunk in chain.stream({"topic": "What is machine learning?"}):
    print(chunk, end="", flush=True)
```

***

## 3. 기본 패턴: Token Streaming (Ollama) <a href="#id-3---token-streaming-ollama" id="id-3---token-streaming-ollama"></a>

### 3.1 Ollama Token Streaming

```python
from langchain.chat_models import init_chat_model

# init_chat_model로 Ollama 모델 초기화
model = init_chat_model("ollama:llama3.1", temperature=0.7)

print("="*70)
print("Ollama Token Streaming")
print("="*70)

for chunk in model.stream("Explain the concept of neural networks"):
    print(chunk.content, end="", flush=True)
```

### 3.2 Ollama with RAG Streaming

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# 간단한 retriever 시뮬레이션
def fake_retriever(query):
    return [
        "LangChain is a framework for developing LLM applications.",
        "It provides tools for building agents, chains, and RAG systems."
    ]

def format_docs(docs):
    return "\n\n".join(docs)

# RAG Prompt
rag_prompt = ChatPromptTemplate.from_messages([
    ("system", "Use the following context to answer:\n\n{context}"),
    ("human", "{question}")
])

# RAG Chain
rag_chain = (
    {"context": fake_retriever | format_docs, "question": RunnablePassthrough()}
    | rag_prompt
    | model
    | StrOutputParser()
)

# Streaming
print("\n" + "="*70)
print("Ollama RAG Streaming")
print("="*70)

for chunk in rag_chain.stream("What is LangChain?"):
    print(chunk, end="", flush=True)
```

***

## 4. 고급 패턴: astream\_events (OpenAI) <a href="#id-4---astreamevents-openai" id="id-4---astreamevents-openai"></a>

### 4.1 Event Filtering

```python
import asyncio
from langchain.chat_models import init_chat_model
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

model = init_chat_model("openai:gpt-4o-mini")

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful AI assistant."),
    ("human", "{query}")
])

chain = prompt | model | StrOutputParser()

async def stream_events():
    print("="*70)
    print("astream_events - Filtering Events")
    print("="*70)
    
    async for event in chain.astream_events(
        {"query": "Explain the benefits of async programming"},
        version="v2"
    ):
        kind = event["event"]
        
        # 채팅 모델 스트리밍 이벤트만 필터링
        if kind == "on_chat_model_stream":
            content = event["data"]["chunk"].content
            if content:
                print(content, end="", flush=True)

asyncio.run(stream_events())
```

### 4.2 Multiple Event Types

```python
sync def stream_all_events():
    print("\n" + "="*70)
    print("astream_events - All Event Types")
    print("="*70)
    
    async for event in chain.astream_events(
        {"query": "What is LangChain?"},
        version="v2"
    ):
        kind = event["event"]
        
        if kind == "on_chain_start":
            print(f"\n[CHAIN START] {event['name']}")
        
        elif kind == "on_chat_model_start":
            print(f"\n[LLM START] Model: {event['name']}")
        
        elif kind == "on_chat_model_stream":
            content = event["data"]["chunk"].content
            if content:
                print(content, end="", flush=True)
        
        elif kind == "on_chat_model_end":
            print(f"\n[LLM END] Tokens: {event['data']['output'].response_metadata.get('token_usage', {})}")
        
        elif kind == "on_chain_end":
            print(f"\n[CHAIN END] {event['name']}")

asyncio.run(stream_all_events())
```

***

## 5. LangGraph 패턴: Stream Modes (OpenAI) <a href="#id-5-langgraph--stream-modes-openai" id="id-5-langgraph--stream-modes-openai"></a>

### 5.1 Values Mode - 전체 상태 스트리밍

```python
from typing import TypedDict
from langchain.chat_models import init_chat_model
from langgraph.graph import StateGraph, START, END

model = init_chat_model("openai:gpt-4o-mini")

class State(TypedDict):
    input: str
    step1_output: str
    step2_output: str
    final_output: str

def step1(state: State):
    """첫 번째 처리 단계"""
    return {"step1_output": f"Processed: {state['input']}"}

def step2(state: State):
    """두 번째 처리 단계"""
    return {"step2_output": f"Enhanced: {state['step1_output']}"}

def step3(state: State):
    """LLM 최종 처리"""
    response = model.invoke(f"Summarize: {state['step2_output']}")
    return {"final_output": response.content}

# 그래프 구축
builder = StateGraph(State)
builder.add_node("step1", step1)
builder.add_node("step2", step2)
builder.add_node("step3", step3)
builder.add_edge(START, "step1")
builder.add_edge("step1", "step2")
builder.add_edge("step2", "step3")
builder.add_edge("step3", END)

graph = builder.compile()

# Values Mode Streaming
print("="*70)
print("LangGraph - Values Mode (전체 상태)")
print("="*70)

for chunk in graph.stream(
    {"input": "LangChain streaming"},
    stream_mode="values"
):
    print(f"\n[STATE UPDATE]")
    for key, value in chunk.items():
        if value:
            print(f"  {key}: {value[:50]}..." if len(str(value)) > 50 else f"  {key}: {value}")
```

### 5.2 Updates Mode - 변경사항만 스트리밍

```python
print("\n" + "="*70)
print("LangGraph - Updates Mode (변경사항만)")
print("="*70)

for chunk in graph.stream(
    {"input": "LangChain updates mode"},
    stream_mode="updates"
):
    print(f"\n[NODE UPDATE]: {chunk}")
```

***

## 6. LangGraph 패턴: Messages Mode (Ollama) <a href="#id-6-langgraph--messages-mode-ollama" id="id-6-langgraph--messages-mode-ollama"></a>

### 6.1 Token-level Streaming

```python
from langchain.chat_models import init_chat_model
from langchain.tools import tool
from langgraph.prebuilt import create_react_agent

model = init_chat_model("ollama:llama3.1")

@tool
def get_weather(city: str) -> str:
    """Get weather for a city."""
    return f"Weather in {city}: 22°C, Sunny"

@tool
def calculate(expression: str) -> float:
    """Calculate a math expression."""
    return eval(expression)

# Agent 생성
agent = create_react_agent(model, [get_weather, calculate])

# Messages Mode - 토큰 레벨 스트리밍
print("="*70)
print("LangGraph - Messages Mode (토큰 스트리밍)")
print("="*70)

for token, metadata in agent.stream(
    {"messages": [{"role": "user", "content": "What's 15 * 8 + 100?"}]},
    stream_mode="messages"
):
    node = metadata.get("langgraph_node", "unknown")
    content = token.content if hasattr(token, 'content') else str(token)
    
    if content:
        print(f"[{node}] {content}", end="", flush=True)
```

***

## 7. LangGraph 패턴: Custom Streaming (OpenAI) <a href="#id-7-langgraph--custom-streaming-openai" id="id-7-langgraph--custom-streaming-openai"></a>

### 7.1 도구에서 Custom Data 스트리밍

```python
from langchain.tools import tool
from langchain.chat_models import init_chat_model
from langgraph.prebuilt import create_react_agent
from langgraph.config import get_stream_writer
import time

model = init_chat_model("openai:gpt-4o")

@tool
def process_large_dataset(dataset_name: str) -> str:
    """Process a large dataset with progress updates."""
    writer = get_stream_writer()
    
    # 진행률 스트리밍
    writer(f"🔄 Starting processing of {dataset_name}")
    time.sleep(1)
    
    writer(f"📊 Loaded {dataset_name} - 1000 records")
    time.sleep(1)
    
    writer(f"🔍 Analyzing data - 25% complete")
    time.sleep(1)
    
    writer(f"🔍 Analyzing data - 50% complete")
    time.sleep(1)
    
    writer(f"🔍 Analyzing data - 75% complete")
    time.sleep(1)
    
    writer(f"✅ Processing complete - Found 42 anomalies")
    
    return f"Dataset {dataset_name} processed: 42 anomalies detected"

# Agent 생성
agent = create_react_agent(model, [process_large_dataset])

# Custom Mode Streaming
print("="*70)
print("LangGraph - Custom Streaming (진행률)")
print("="*70)

for chunk in agent.stream(
    {"messages": [{"role": "user", "content": "Process the sales_data_2024 dataset"}]},
    stream_mode="custom"
):
    print(chunk)
```

***

## 8. 고급 패턴: Multiple Stream Modes (Ollama) <a href="#id-8---multiple-stream-modes-ollama" id="id-8---multiple-stream-modes-ollama"></a>

### 8.1 Updates + Messages + Custom 동시 스트리밍

```python
from langchain.chat_models import init_chat_model
from langchain.tools import tool
from langgraph.prebuilt import create_react_agent
from langgraph.config import get_stream_writer

model = init_chat_model("ollama:llama3.1")

@tool
def search_database(query: str) -> str:
    """Search database."""
    writer = get_stream_writer()
    writer(f"🔍 Searching for: {query}")
    writer(f"📚 Found 25 results")
    return f"Results for '{query}': 25 documents"

agent = create_react_agent(model, [search_database])

print("="*70)
print("Multiple Stream Modes - updates + messages + custom")
print("="*70)

for stream_mode, chunk in agent.stream(
    {"messages": [{"role": "user", "content": "Search for LangChain documentation"}]},
    stream_mode=["updates", "messages", "custom"]
):
    if stream_mode == "updates":
        print(f"\n[UPDATES] {list(chunk.keys())}")
    
    elif stream_mode == "messages":
        token, metadata = chunk
        if hasattr(token, 'content') and token.content:
            print(token.content, end="", flush=True)
    
    elif stream_mode == "custom":
        print(f"\n[CUSTOM] {chunk}")
```

***

## 9. 실전 예제: FastAPI Integration (OpenAI) <a href="#id-9---fastapi-integration-openai" id="id-9---fastapi-integration-openai"></a>

### 9.1 StreamingResponse with astream

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from langchain.chat_models import init_chat_model
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
import asyncio

app = FastAPI()

model = init_chat_model("openai:gpt-4o")
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("human", "{query}")
])
chain = prompt | model | StrOutputParser()

@app.get("/stream")
async def stream_response(query: str):
    """Stream LLM response"""
    
    async def generate():
        async for chunk in chain.astream({"query": query}):
            # SSE 형식
            yield f"data: {chunk}\n\n"
    
    return StreamingResponse(generate(), media_type="text/event-stream")

@app.get("/stream-json")
async def stream_json(query: str):
    """Stream as JSON"""
    import json
    
    async def generate():
        async for chunk in chain.astream({"query": query}):
            data = {"content": chunk, "done": False}
            yield json.dumps(data) + "\n"
        
        # 마지막 메시지
        yield json.dumps({"content": "", "done": True}) + "\n"
    
    return StreamingResponse(generate(), media_type="application/x-ndjson")

# 실행: uvicorn main:app --reload
# 테스트: http://localhost:8000/stream?query=Tell me about AI
```

### 9.2 Agent Streaming with FastAPI (Ollama)

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from langchain.chat_models import init_chat_model
from langchain.tools import tool
from langgraph.prebuilt import create_react_agent
import json

app = FastAPI()

model = init_chat_model("ollama:llama3.1")

@tool
def calculate(expression: str) -> float:
    """Calculate math expression."""
    return eval(expression)

agent = create_react_agent(model, [calculate])

@app.post("/agent-stream")
async def agent_stream(query: str):
    """Stream agent execution"""
    
    async def generate():
        async for token, metadata in agent.astream(
            {"messages": [{"role": "user", "content": query}]},
            stream_mode="messages"
        ):
            if hasattr(token, 'content') and token.content:
                data = {
                    "content": token.content,
                    "node": metadata.get("langgraph_node", "unknown")
                }
                yield f"data: {json.dumps(data)}\n\n"
    
    return StreamingResponse(generate(), media_type="text/event-stream")

# 실행: uvicorn main:app --reload
# 테스트: POST http://localhost:8000/agent-stream?query=What is 100 * 25?
```

***

## 10. 실전 예제: Complete Streaming System <a href="#id-10---complete-streaming-system" id="id-10---complete-streaming-system"></a>

### 10.1 모든 패턴 통합 (OpenAI + Ollama)

```python
import asyncio
from typing import TypedDict
from langchain.chat_models import init_chat_model
from langchain.tools import tool
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langgraph.graph import StateGraph, START, END
from langgraph.prebuilt import create_react_agent
from langgraph.config import get_stream_writer

# ===== 모델 초기화 =====
openai_model = init_chat_model("openai:gpt-4o-mini", temperature=0.7)
ollama_model = init_chat_model("ollama:llama3.1", temperature=0.7)

# ===== 도구 정의 =====
@tool
def analyze_sentiment(text: str) -> str:
    """Analyze sentiment of text."""
    writer = get_stream_writer()
    writer(f"📊 Analyzing sentiment...")
    writer(f"✅ Analysis complete")
    return f"Sentiment of '{text[:30]}...': Positive (0.85)"

@tool
def translate_text(text: str, target_lang: str = "Korean") -> str:
    """Translate text to target language."""
    writer = get_stream_writer()
    writer(f"🌐 Translating to {target_lang}...")
    return f"Translation: [번역된 텍스트]"

# ===== OpenAI Agent =====
openai_agent = create_react_agent(openai_model, [analyze_sentiment, translate_text])

# ===== Ollama Chain =====
ollama_prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a concise AI assistant."),
    ("human", "{query}")
])
ollama_chain = ollama_prompt | ollama_model | StrOutputParser()

# ===== Complex Graph =====
class ComplexState(TypedDict):
    input: str
    openai_response: str
    ollama_response: str
    final_summary: str

def openai_node(state: ComplexState):
    """OpenAI 처리"""
    writer = get_stream_writer()
    writer("🤖 OpenAI processing...")
    
    response = openai_model.invoke(f"Analyze: {state['input']}")
    return {"openai_response": response.content}

def ollama_node(state: ComplexState):
    """Ollama 처리"""
    writer = get_stream_writer()
    writer("🦙 Ollama processing...")
    
    response = ollama_model.invoke(f"Summarize: {state['input']}")
    return {"ollama_response": response.content}

def combine_node(state: ComplexState):
    """결과 결합"""
    summary = f"OpenAI: {state['openai_response'][:50]}...\nOllama: {state['ollama_response'][:50]}..."
    return {"final_summary": summary}

# 그래프 구축
builder = StateGraph(ComplexState)
builder.add_node("openai", openai_node)
builder.add_node("ollama", ollama_node)
builder.add_node("combine", combine_node)

builder.add_edge(START, "openai")
builder.add_edge(START, "ollama")
builder.add_edge("openai", "combine")
builder.add_edge("ollama", "combine")
builder.add_edge("combine", END)

complex_graph = builder.compile()

# ===== 테스트 함수 =====
async def test_all_streaming():
    print("="*70)
    print("COMPLETE STREAMING SYSTEM TEST")
    print("="*70)
    
    # ===== 1. Simple Token Streaming (OpenAI) =====
    print("\n" + "="*70)
    print("1. OpenAI Token Streaming")
    print("="*70)
    
    for chunk in openai_model.stream("Explain AI in one sentence"):
        print(chunk.content, end="", flush=True)
    
    # ===== 2. Chain Streaming (Ollama) =====
    print("\n\n" + "="*70)
    print("2. Ollama Chain Streaming")
    print("="*70)
    
    async for chunk in ollama_chain.astream({"query": "What is machine learning?"}):
        print(chunk, end="", flush=True)
    
    # ===== 3. Agent with Custom Streaming (OpenAI) =====
    print("\n\n" + "="*70)
    print("3. OpenAI Agent - Custom Streaming")
    print("="*70)
    
    for chunk in openai_agent.stream(
        {"messages": [{"role": "user", "content": "Analyze the sentiment of 'LangChain is awesome'"}]},
        stream_mode="custom"
    ):
        print(chunk)
    
    # ===== 4. Agent - Messages Mode (Ollama) =====
    print("\n" + "="*70)
    print("4. Ollama Agent - Messages Mode")
    print("="*70)
    
    ollama_agent = create_react_agent(ollama_model, [analyze_sentiment])
    
    for token, metadata in ollama_agent.stream(
        {"messages": [{"role": "user", "content": "Analyze sentiment of 'This is great'"}]},
        stream_mode="messages"
    ):
        if hasattr(token, 'content') and token.content:
            print(token.content, end="", flush=True)
    
    # ===== 5. Complex Graph - Multiple Modes =====
    print("\n\n" + "="*70)
    print("5. Complex Graph - Updates + Custom")
    print("="*70)
    
    for mode, chunk in complex_graph.stream(
        {"input": "LangChain streaming system"},
        stream_mode=["updates", "custom"]
    ):
        if mode == "updates":
            print(f"\n[NODE COMPLETE]: {list(chunk.keys())}")
        elif mode == "custom":
            print(f"[PROGRESS]: {chunk}")
    
    # ===== 6. astream_events (OpenAI) =====
    print("\n\n" + "="*70)
    print("6. OpenAI astream_events - Event Filtering")
    print("="*70)
    
    async for event in ollama_chain.astream_events(
        {"query": "Explain neural networks"},
        version="v2"
    ):
        kind = event["event"]
        if kind == "on_chat_model_stream":
            content = event["data"]["chunk"].content
            if content:
                print(content, end="", flush=True)
    
    print("\n\n" + "="*70)
    print("ALL STREAMING TESTS COMPLETE")
    print("="*70)

# 실행
asyncio.run(test_all_streaming())
```

***

## 11. Best Practices <a href="#id-11-best-practices" id="id-11-best-practices"></a>

### Streaming 최적화

1. **Async 사용**: `astream()`과 `astream_events()`는 항상 async로 사용​
2. **적절한 모드 선택**: 필요한 데이터만 스트리밍하여 네트워크 대역폭 절약​
3. **Event Filtering**: `astream_events()`에서 필요한 이벤트만 필터링​
4. **Callback Propagation**: 커스텀 Runnable에서 콜백 전달 필수​

### 모드별 사용 시나리오

| Stream Mode  | 언제 사용?            |
| ------------ | ----------------- |
| **values**   | 전체 그래프 상태가 필요할 때  |
| **updates**  | 변경사항만 추적 (기본값)    |
| **messages** | 토큰 단위 UI 업데이트     |
| **custom**   | 진행률, 로깅 등 커스텀 데이터 |
| **debug**    | 개발 및 디버깅          |

### 프로덕션 체크리스트

1. **Error Handling**: 스트리밍 중 에러 처리 구현
2. **Timeout 설정**: 긴 스트리밍에 타임아웃 적용
3. **Backpressure 관리**: 클라이언트가 느릴 때 대응
4. **로깅**: 스트리밍 이벤트 기록
5. **모니터링**: 토큰 사용량, 지연 시간 추적
