# Context Engineering

### Context Engineering이란?

에이전트가 실패하는 주요 원인은 LLM이 잘못된 응답을 생성하기 때문이며, 이는 두 가지로 나뉩니다:​

* **모델 자체의 한계**: 더 나은 모델 선택 필요
* **적절한 컨텍스트 부재**: Context Engineering으로 해결

### Context 타입

LangChain v1.0은 다섯 가지 컨텍스트 타입을 지원합니다:​

1. **Instructions**: System prompts (정적/동적)
2. **Tools**: 에이전트가 사용 가능한 도구
3. **Session context**: 대화 메시지, 세션 상태 (Short-term memory)
4. **Long-term memory**: 세션 간 지속되는 정보​
5. **Runtime context**: User ID, DB 연결 등 불변 설정​

***

## 1. 기본 설정 및 설치 <a href="#id-1" id="id-1"></a>

```python
# 필수 패키지 설치
pip install --pre -U langchain
pip install -U langchain-openai langchain-ollama
pip install -U langgraph-checkpoint langgraph

# 환경 변수 설정
import os
os.environ["OPENAI_API_KEY"] = "your-openai-api-key"
# Ollama는 로컬에서 실행 (http://localhost:11434)
```

***

## 2. init\_chat\_model을 사용한 기본 Agent 생성 <a href="#id-2-initchatmodel---agent" id="id-2-initchatmodel---agent"></a>

### 2.1 OpenAI 사용

```python
pythonfrom langchain.agents import create_agent
from langchain.chat_models import init_chat_model
from langchain.tools import tool

# init_chat_model로 OpenAI 모델 초기화
model = init_chat_model(
    "openai:gpt-4o",
    temperature=0.7,
    max_tokens=1000
)

# 간단한 도구 정의
@tool
def get_weather(city: str) -> str:
    """Get weather for a given city."""
    return f"It's sunny in {city}!"

# Agent 생성
agent = create_agent(
    model=model,
    tools=[get_weather],
    system_prompt="You are a helpful weather assistant."
)

# 실행
result = agent.invoke({
    "messages": [{"role": "user", "content": "What's the weather in Seoul?"}]
})

print(result["messages"][-1].content)
```

### 2.2 Ollama 사용

```python
from langchain.agents import create_agent
from langchain.chat_models import init_chat_model
from langchain.tools import tool

# init_chat_model로 Ollama 모델 초기화
model = init_chat_model(
    "ollama:llama3.1",
    temperature=0.7,
    base_url="http://localhost:11434"
)

@tool
def calculate(expression: str) -> float:
    """Calculate a mathematical expression."""
    return eval(expression)

agent = create_agent(
    model=model,
    tools=[calculate],
    system_prompt="You are a helpful math assistant."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "What is 25 * 4 + 10?"}]
})

print(result["messages"][-1].content)
```

***

## 3. Context Engineering 패턴: Dynamic Prompts <a href="#id-3-context-engineering--dynamic-prompts" id="id-3-context-engineering--dynamic-prompts"></a>

### 3.1 Decorator 기반 Dynamic Prompt (OpenAI)

```python
from dataclasses import dataclass
from langchain.agents import create_agent
from langchain.agents.middleware import dynamic_prompt, ModelRequest
from langchain.chat_models import init_chat_model

@dataclass
class UserContext:
    user_id: str
    user_role: str
    language: str = "ko"

# Dynamic prompt 미들웨어
@dynamic_prompt
def personalized_prompt(request: ModelRequest) -> str:
    """사용자 컨텍스트에 따라 동적으로 프롬프트 생성"""
    user_role = request.runtime.context.user_role
    language = request.runtime.context.language
    
    base_prompt = "You are a helpful assistant."
    
    if user_role == "expert":
        base_prompt += "\nProvide detailed technical responses with code examples."
    elif user_role == "beginner":
        base_prompt += "\nExplain concepts simply and avoid jargon."
    
    if language == "ko":
        base_prompt += "\nRespond in Korean."
    
    # 세션 상태 기반 조정
    message_count = len(request.state["messages"])
    if message_count > 10:
        base_prompt += "\nThis is a long conversation - be concise."
    
    return base_prompt

# OpenAI 모델로 Agent 생성
model = init_chat_model("openai:gpt-4o", temperature=0.7)

agent = create_agent(
    model=model,
    tools=[],
    middleware=[personalized_prompt],
    context_schema=UserContext
)

# Expert 사용자로 실행
result = agent.invoke(
    {"messages": [{"role": "user", "content": "Explain async programming"}]},
    context=UserContext(user_id="user123", user_role="expert", language="ko")
)

print(result["messages"][-1].content)
```

### 3.2 Class 기반 Dynamic Prompt (Ollama)

```python
from langchain.agents.middleware import AgentMiddleware, ModelRequest, ModelResponse
from typing import Callable

class AdaptivePromptMiddleware(AgentMiddleware):
    """대화 복잡도에 따라 프롬프트 조정"""
    
    def wrap_model_call(
        self,
        request: ModelRequest,
        handler: Callable[[ModelRequest], ModelResponse]
    ) -> ModelResponse:
        # 메시지 길이 기반 프롬프트 조정
        message_count = len(request.messages)
        
        # 기존 시스템 프롬프트 가져오기
        system_prompt = request.system_prompt or "You are a helpful assistant."
        
        if message_count > 15:
            system_prompt += "\n[IMPORTANT] Keep responses very brief due to context length."
        elif message_count > 10:
            system_prompt += "\nKeep responses concise."
        
        # 수정된 프롬프트 적용
        request.system_prompt = system_prompt
        
        return handler(request)

# Ollama 모델로 Agent 생성
model = init_chat_model("ollama:llama3.1")

agent = create_agent(
    model=model,
    tools=[],
    middleware=[AdaptivePromptMiddleware()],
    system_prompt="You are a coding assistant."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "Explain Python decorators"}]
})

print(result["messages"][-1].content)
```

***

## 4. Context Engineering 패턴: Message Trimming <a href="#id-4-context-engineering--message-trimming" id="id-4-context-engineering--message-trimming"></a>

### 4.1 Before Model Hook으로 메시지 관리 (OpenAI)

```python
from langchain.agents.middleware import before_model, AgentState
from langchain.messages import RemoveMessage
from langgraph.graph.message import REMOVE_ALL_MESSAGES
from langgraph.runtime import Runtime
from typing import Any

@before_model
def trim_messages(state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
    """긴 대화에서 오래된 메시지 제거"""
    messages = state["messages"]
    
    # 10개 이하면 trim 불필요
    if len(messages) <= 10:
        return None
    
    # 시스템 메시지 + 최근 8개 메시지만 유지
    return {
        "messages": [
            RemoveMessage(id=REMOVE_ALL_MESSAGES),
            messages[0],  # System message
            *messages[-8:]  # Recent messages
        ]
    }

model = init_chat_model("openai:gpt-4o-mini")

agent = create_agent(
    model=model,
    tools=[],
    middleware=[trim_messages],
    system_prompt="You are a helpful assistant."
)

# 많은 메시지로 테스트
messages = [{"role": "user", "content": f"Message {i}"} for i in range(15)]
result = agent.invoke({"messages": messages})

print(f"Original: {len(messages)}, After trim: {len(result['messages'])}")
```

### 4.2 SummarizationMiddleware 사용 (Ollama)

```python
from langchain.agents.middleware import SummarizationMiddleware

# Ollama 모델 초기화
model = init_chat_model("ollama:qwen2:7b")

agent = create_agent(
    model=model,
    tools=[],
    middleware=[
        SummarizationMiddleware(
            model=init_chat_model("ollama:llama3.1"),  # 요약용 모델
            max_tokens_before_summary=2000,
            messages_to_keep=15
        )
    ],
    system_prompt="You are a conversational AI."
)

# 긴 대화 시뮬레이션
long_conversation = [
    {"role": "user", "content": f"Tell me about topic {i}"}
    for i in range(20)
]

result = agent.invoke({"messages": long_conversation})
```

***

## 5. Context Engineering 패턴: Contextual Tools <a href="#id-5-context-engineering--contextual-tools" id="id-5-context-engineering--contextual-tools"></a>

### 5.1 Runtime Context 접근하는 도구 (OpenAI)

```python
from dataclasses import dataclass
from langchain.tools import tool, ToolRuntime

@dataclass
class AppContext:
    user_id: str
    api_key: str
    region: str = "kr"

@tool
def search_documents(
    query: str,
    runtime: ToolRuntime[AppContext]
) -> str:
    """Search documents with user context."""
    # Runtime context 접근
    user_id = runtime.context.user_id
    region = runtime.context.region
    
    # Session state 접근
    conversation_length = len(runtime.state["messages"])
    
    return f"Searched '{query}' for user {user_id} in {region} (conv: {conversation_length} msgs)"

@tool
def get_user_preferences(
    runtime: ToolRuntime[AppContext]
) -> str:
    """Get user preferences from store (long-term memory)."""
    user_id = runtime.context.user_id
    
    # Long-term memory 접근 (store)
    if runtime.store:
        prefs = runtime.store.get(("users",), user_id)
        if prefs:
            return f"User preferences: {prefs.value}"
    
    return "No preferences found"

model = init_chat_model("openai:gpt-4o")

agent = create_agent(
    model=model,
    tools=[search_documents, get_user_preferences],
    context_schema=AppContext,
    system_prompt="You are a search assistant."
)

result = agent.invoke(
    {"messages": [{"role": "user", "content": "Search for AI papers"}]},
    context=AppContext(user_id="user789", api_key="secret", region="us")
)

print(result["messages"][-1].content)
```

### 5.2 Custom State Schema 사용 (Ollama)

```python
from langchain.agents.middleware import AgentState
from typing_extensions import NotRequired

class CustomState(AgentState):
    """커스텀 상태 스키마"""
    user_preferences: NotRequired[dict]
    search_history: NotRequired[list]
    interaction_count: NotRequired[int]

@tool
def save_preference(
    key: str,
    value: str,
    runtime: ToolRuntime[None, CustomState]
) -> str:
    """Save user preference to session state."""
    # Custom state 업데이트
    prefs = runtime.state.get("user_preferences", {})
    prefs[key] = value
    
    # State는 직접 수정 불가, return으로 업데이트
    return f"Saved preference: {key}={value}"

@tool
def get_interaction_stats(
    runtime: ToolRuntime[None, CustomState]
) -> str:
    """Get interaction statistics."""
    count = runtime.state.get("interaction_count", 0)
    history = runtime.state.get("search_history", [])
    
    return f"Interactions: {count}, Searches: {len(history)}"

model = init_chat_model("ollama:llama3.1")

agent = create_agent(
    model=model,
    tools=[save_preference, get_interaction_stats],
    system_prompt="You are a personalization assistant."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "Save my language preference as Korean"}],
    "user_preferences": {},
    "search_history": [],
    "interaction_count": 1
})
```

***

## 6. Context Engineering 패턴: Dynamic Tool Selection <a href="#id-6-context-engineering--dynamic-tool-selection" id="id-6-context-engineering--dynamic-tool-selection"></a>

### 6.1 권한 기반 도구 필터링 (OpenAI)

```python
from langchain.agents.middleware import wrap_model_call, ModelRequest, ModelResponse
from typing import Callable

@dataclass
class UserContext:
    user_id: str
    user_role: str  # "admin", "editor", "viewer"

@wrap_model_call
def permission_based_tools(
    request: ModelRequest,
    handler: Callable[[ModelRequest], ModelResponse]
) -> ModelResponse:
    """사용자 권한에 따라 도구 필터링"""
    user_role = request.runtime.context.user_role
    
    if user_role == "admin":
        # Admin은 모든 도구 접근
        pass
    elif user_role == "editor":
        # Editor는 삭제 도구 제외
        request.tools = [t for t in request.tools if t.name != "delete_data"]
    else:
        # Viewer는 읽기 전용 도구만
        request.tools = [t for t in request.tools if t.name.startswith("read_")]
    
    return handler(request)

@tool
def read_data(query: str) -> str:
    """Read data from database."""
    return f"Data for query: {query}"

@tool
def write_data(data: str) -> str:
    """Write data to database."""
    return f"Written: {data}"

@tool
def delete_data(id: str) -> str:
    """Delete data from database."""
    return f"Deleted: {id}"

model = init_chat_model("openai:gpt-4o")

agent = create_agent(
    model=model,
    tools=[read_data, write_data, delete_data],
    middleware=[permission_based_tools],
    context_schema=UserContext
)

# Viewer로 실행
result = agent.invoke(
    {"messages": [{"role": "user", "content": "Show me user data"}]},
    context=UserContext(user_id="user456", user_role="viewer")
)

print(result["messages"][-1].content)
```

### 6.2 LLM 기반 도구 선택 (Ollama)

```python
from langchain.agents.middleware import LLMToolSelectorMiddleware

@tool
def web_search(query: str) -> str:
    """Search the web."""
    return f"Web results for: {query}"

@tool
def database_query(sql: str) -> str:
    """Query database."""
    return f"DB results for: {sql}"

@tool
def send_email(to: str, subject: str) -> str:
    """Send email."""
    return f"Email sent to {to}"

@tool
def create_report(topic: str) -> str:
    """Create a report."""
    return f"Report created: {topic}"

@tool
def analyze_data(data: str) -> str:
    """Analyze data."""
    return f"Analysis of: {data}"

# Ollama 모델로 도구 선택
model = init_chat_model("ollama:llama3.1")

agent = create_agent(
    model=model,
    tools=[web_search, database_query, send_email, create_report, analyze_data],
    middleware=[
        LLMToolSelectorMiddleware(
            model=init_chat_model("ollama:qwen2:7b"),  # 가벼운 모델로 선택
            max_tools=2,  # 최대 2개 도구만 선택
            always_include=["web_search"]  # 항상 포함
        )
    ],
    system_prompt="You are a research assistant."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "Find information about LangChain"}]
})
```

***

## 7. Context Engineering 패턴: Dynamic Model Selection <a href="#id-7-context-engineering--dynamic-model-selection" id="id-7-context-engineering--dynamic-model-selection"></a>

### 7.1 대화 길이 기반 모델 전환 (OpenAI)

```python
from langchain.agents.middleware import wrap_model_call

@wrap_model_call
def adaptive_model(
    request: ModelRequest,
    handler: Callable[[ModelRequest], ModelResponse]
) -> ModelResponse:
    """대화 길이에 따라 모델 동적 선택"""
    message_count = len(request.messages)
    
    if message_count > 20:
        # 긴 대화 - 큰 컨텍스트 윈도우 모델
        request.model = init_chat_model("openai:gpt-4o")
    elif message_count > 10:
        # 중간 길이 - 표준 모델
        request.model = init_chat_model("openai:gpt-4o-mini")
    else:
        # 짧은 대화 - 효율적 모델
        request.model = init_chat_model("openai:gpt-3.5-turbo")
    
    return handler(request)

agent = create_agent(
    model=init_chat_model("openai:gpt-4o-mini"),  # 기본 모델
    tools=[],
    middleware=[adaptive_model],
    system_prompt="You are a conversational assistant."
)

# 긴 대화로 테스트
long_conversation = [
    {"role": "user", "content": f"Question {i}: Tell me something interesting"}
    for i in range(25)
]

result = agent.invoke({"messages": long_conversation})
```

### 7.2 OpenAI와 Ollama 간 Fallback

```python
from langchain.agents.middleware import ModelFallbackMiddleware

agent = create_agent(
    model=init_chat_model("openai:gpt-4o"),  # Primary
    tools=[],
    middleware=[
        ModelFallbackMiddleware(
            init_chat_model("openai:gpt-4o-mini"),  # Fallback 1
            init_chat_model("ollama:llama3.1")      # Fallback 2 (local)
        )
    ],
    system_prompt="You are a resilient assistant."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "Tell me about AI"}]
})
```

***

## 8. 고급 패턴: Long-term Memory (Store) <a href="#id-8---long-term-memory-store" id="id-8---long-term-memory-store"></a>

### 8.1 Long-term Memory 사용 (OpenAI + Ollama)

```python
from langgraph.store.memory import InMemoryStore

# Store 초기화
def embed(texts: list[str]) -> list[list[float]]:
    # 실제 임베딩 함수로 교체
    return [[1.0, 2.0] for _ in texts]

store = InMemoryStore(index={"embed": embed, "dims": 2})

@dataclass
class UserContext:
    user_id: str

@tool
def save_user_preference(
    preference_type: str,
    value: str,
    runtime: ToolRuntime[UserContext]
) -> str:
    """Save user preference to long-term memory."""
    user_id = runtime.context.user_id
    
    if runtime.store:
        # namespace: (user_id, "preferences")
        namespace = ("users", user_id)
        
        # 기존 preferences 가져오기
        existing = runtime.store.get(namespace, "preferences")
        prefs = existing.value if existing else {}
        
        # 업데이트
        prefs[preference_type] = value
        
        # 저장
        runtime.store.put(namespace, "preferences", prefs)
        
        return f"Saved {preference_type}: {value}"
    
    return "Store not available"

@tool
def get_user_preference(
    preference_type: str,
    runtime: ToolRuntime[UserContext]
) -> str:
    """Get user preference from long-term memory."""
    user_id = runtime.context.user_id
    
    if runtime.store:
        namespace = ("users", user_id)
        prefs = runtime.store.get(namespace, "preferences")
        
        if prefs and preference_type in prefs.value:
            return f"{preference_type}: {prefs.value[preference_type]}"
    
    return "Preference not found"

# OpenAI 모델 사용
model = init_chat_model("openai:gpt-4o")

agent = create_agent(
    model=model,
    tools=[save_user_preference, get_user_preference],
    context_schema=UserContext,
    store=store,  # Store 연결
    system_prompt="You are a personalization assistant with long-term memory."
)

# 선호도 저장
result1 = agent.invoke(
    {"messages": [{"role": "user", "content": "Remember I prefer dark mode"}]},
    context=UserContext(user_id="user123")
)

# 선호도 조회 (새 세션)
result2 = agent.invoke(
    {"messages": [{"role": "user", "content": "What's my theme preference?"}]},
    context=UserContext(user_id="user123")
)

print(result2["messages"][-1].content)
```

***

## 9. 고급 패턴: Before/After Hooks <a href="#id-9---beforeafter-hooks" id="id-9---beforeafter-hooks"></a>

### 9.1 Logging Middleware

```python
from langchain.agents.middleware import before_model, after_model
import time

@before_model
def log_before_model(state: AgentState, runtime: Runtime) -> dict | None:
    """모델 호출 전 로깅"""
    print(f"[BEFORE MODEL] Messages: {len(state['messages'])}")
    print(f"[BEFORE MODEL] User: {getattr(runtime.context, 'user_id', 'unknown')}")
    
    # state에 타임스탬프 추가
    runtime.temp_data = {"start_time": time.time()}
    
    return None

@after_model
def log_after_model(state: AgentState, runtime: Runtime) -> dict | None:
    """모델 응답 후 로깅"""
    elapsed = time.time() - runtime.temp_data.get("start_time", 0)
    last_msg = state["messages"][-1]
    
    print(f"[AFTER MODEL] Response length: {len(last_msg.content)}")
    print(f"[AFTER MODEL] Elapsed: {elapsed:.2f}s")
    
    return None

model = init_chat_model("openai:gpt-4o")

agent = create_agent(
    model=model,
    tools=[],
    middleware=[log_before_model, log_after_model],
    system_prompt="You are a helpful assistant."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "Explain quantum computing"}]
})
```

### 9.2 Validation Middleware

```python
from langchain.agents.middleware import after_model, hook_config
from langchain.messages import AIMessage

@after_model
@hook_config(can_jump_to=["end"])
def validate_output(state: AgentState, runtime: Runtime) -> dict | None:
    """응답 검증 및 조기 종료"""
    last_message = state["messages"][-1]
    
    # 금지된 단어 체크
    forbidden_words = ["BLOCKED", "ERROR", "UNSAFE"]
    if any(word in last_message.content.upper() for word in forbidden_words):
        return {
            "messages": [AIMessage("I cannot respond to that request.")],
            "jump_to": "end"
        }
    
    # 너무 짧은 응답 체크
    if len(last_message.content) < 10:
        print("[VALIDATION] Warning: Response too short")
    
    return None

model = init_chat_model("ollama:llama3.1")

agent = create_agent(
    model=model,
    tools=[],
    middleware=[validate_output],
    system_prompt="You are a safe assistant."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "Tell me about AI safety"}]
})
```

***

## 10. 고급 패턴: Tool Retry & Error Handling <a href="#id-10---tool-retry--error-handling" id="id-10---tool-retry--error-handling"></a>

### 10.1 Tool Retry Middleware

```python
from langchain.agents.middleware import ToolRetryMiddleware

@tool
def unreliable_api(query: str) -> str:
    """Simulated unreliable API."""
    import random
    if random.random() < 0.5:
        raise ConnectionError("API timeout")
    return f"Result for: {query}"

model = init_chat_model("openai:gpt-4o")

agent = create_agent(
    model=model,
    tools=[unreliable_api],
    middleware=[
        ToolRetryMiddleware(
            max_retries=3,
            backoff_factor=2.0,
            initial_delay=1.0,
            jitter=True
        )
    ],
    system_prompt="You are a resilient assistant."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "Call the API"}]
})
```

### 10.2 Custom Tool Error Handler

```python
from langchain.agents.middleware import wrap_tool_call
from langchain.tools.tool_node import ToolCallRequest
from langchain_core.messages import ToolMessage
from langgraph.types import Command

@wrap_tool_call
def tool_error_handler(
    request: ToolCallRequest,
    handler: Callable[[ToolCallRequest], ToolMessage | Command]
) -> ToolMessage | Command:
    """도구 에러 처리"""
    tool_name = request.tool_call['name']
    print(f"[TOOL CALL] Executing: {tool_name}")
    
    try:
        result = handler(request)
        print(f"[TOOL CALL] Success: {tool_name}")
        return result
    except Exception as e:
        print(f"[TOOL CALL] Error in {tool_name}: {e}")
        
        # 에러를 ToolMessage로 반환하여 LLM이 처리하도록
        return ToolMessage(
            content=f"Tool '{tool_name}' failed: {str(e)}. Please try another approach.",
            tool_call_id=request.tool_call['id']
        )

model = init_chat_model("ollama:llama3.1")

agent = create_agent(
    model=model,
    tools=[unreliable_api],
    middleware=[tool_error_handler],
    system_prompt="You are an error-handling assistant."
)
```

***

## 11. 통합 예제: 모든 패턴 결합 <a href="#id-11" id="id-11"></a>

### 완전한 Context Engineering Agent (OpenAI + Ollama)

```python
from dataclasses import dataclass
from langchain.agents import create_agent
from langchain.agents.middleware import (
    dynamic_prompt, before_model, after_model, wrap_model_call, wrap_tool_call,
    ModelRequest, ModelResponse, AgentState
)
from langchain.chat_models import init_chat_model
from langchain.tools import tool, ToolRuntime
from langchain.messages import RemoveMessage, AIMessage
from langgraph.graph.message import REMOVE_ALL_MESSAGES
from langgraph.runtime import Runtime
from langgraph.store.memory import InMemoryStore
from typing import Callable, Any
import time

# ===== Context Schema =====
@dataclass
class ApplicationContext:
    user_id: str
    user_role: str  # "admin", "user"
    language: str = "ko"
    max_message_length: int = 10

# ===== Long-term Memory Store =====
def embed_function(texts: list[str]) -> list[list[float]]:
    return [[1.0, 2.0] for _ in texts]

store = InMemoryStore(index={"embed": embed_function, "dims": 2})

# ===== Tools with Context =====
@tool
def search_knowledge_base(
    query: str,
    runtime: ToolRuntime[ApplicationContext]
) -> str:
    """Search the knowledge base."""
    user_id = runtime.context.user_id
    
    # Long-term memory에서 검색 히스토리 가져오기
    if runtime.store:
        history = runtime.store.get(("users", user_id), "search_history")
        if history:
            print(f"[TOOL] Previous searches: {len(history.value)}")
    
    return f"Found results for: {query}"

@tool
def save_user_note(
    note: str,
    runtime: ToolRuntime[ApplicationContext]
) -> str:
    """Save a note to long-term memory."""
    user_id = runtime.context.user_id
    
    if runtime.store:
        namespace = ("users", user_id)
        existing = runtime.store.get(namespace, "notes")
        notes = existing.value if existing else []
        notes.append(note)
        runtime.store.put(namespace, "notes", notes)
        return f"Saved note: {note}"
    
    return "Store not available"

@tool
def admin_delete_data(
    data_id: str,
    runtime: ToolRuntime[ApplicationContext]
) -> str:
    """Delete data (admin only)."""
    return f"Deleted data: {data_id}"

# ===== Middleware 1: Dynamic Prompt =====
@dynamic_prompt
def context_aware_prompt(request: ModelRequest) -> str:
    """사용자 컨텍스트 기반 동적 프롬프트"""
    user_role = request.runtime.context.user_role
    language = request.runtime.context.language
    message_count = len(request.state["messages"])
    
    prompt = "You are an intelligent assistant."
    
    if user_role == "admin":
        prompt += "\nYou have admin privileges and can perform sensitive operations."
    
    if language == "ko":
        prompt += "\nRespond in Korean."
    
    if message_count > 8:
        prompt += "\nKeep responses concise due to long conversation."
    
    return prompt

# ===== Middleware 2: Message Trimming =====
@before_model
def trim_long_conversations(state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
    """메시지 수 제한"""
    max_length = runtime.context.max_message_length
    messages = state["messages"]
    
    if len(messages) <= max_length:
        return None
    
    print(f"[TRIM] Trimming from {len(messages)} to {max_length} messages")
    
    return {
        "messages": [
            RemoveMessage(id=REMOVE_ALL_MESSAGES),
            messages[0],  # System message
            *messages[-(max_length-1):]
        ]
    }

# ===== Middleware 3: Permission-based Tool Filtering =====
@wrap_model_call
def filter_tools_by_permission(
    request: ModelRequest,
    handler: Callable[[ModelRequest], ModelResponse]
) -> ModelResponse:
    """권한에 따라 도구 필터링"""
    user_role = request.runtime.context.user_role
    
    if user_role != "admin":
        # Non-admin은 admin 도구 접근 불가
        request.tools = [t for t in request.tools if "admin" not in t.name.lower()]
        print(f"[PERMISSION] Filtered tools for role: {user_role}")
    
    return handler(request)

# ===== Middleware 4: Performance Logging =====
@before_model
def log_model_call_start(state: AgentState, runtime: Runtime) -> dict | None:
    """모델 호출 시작 로깅"""
    runtime.temp_data = {"start_time": time.time()}
    print(f"[LOG] Model call starting - Messages: {len(state['messages'])}")
    return None

@after_model
def log_model_call_end(state: AgentState, runtime: Runtime) -> dict | None:
    """모델 응답 로깅"""
    elapsed = time.time() - runtime.temp_data.get("start_time", 0)
    print(f"[LOG] Model responded in {elapsed:.2f}s")
    return None

# ===== Middleware 5: Dynamic Model Selection =====
@wrap_model_call
def select_model_by_context(
    request: ModelRequest,
    handler: Callable[[ModelRequest], ModelResponse]
) -> ModelResponse:
    """대화 복잡도에 따라 모델 선택"""
    message_count = len(request.messages)
    
    if message_count > 15:
        # 긴 대화 - OpenAI 큰 모델
        request.model = init_chat_model("openai:gpt-4o")
        print("[MODEL] Switched to gpt-4o for long conversation")
    elif message_count > 10:
        # 중간 - OpenAI 작은 모델
        request.model = init_chat_model("openai:gpt-4o-mini")
        print("[MODEL] Using gpt-4o-mini")
    else:
        # 짧은 대화 - Ollama 로컬 모델
        request.model = init_chat_model("ollama:llama3.1")
        print("[MODEL] Using Ollama llama3.1")
    
    return handler(request)

# ===== Middleware 6: Tool Error Handling =====
@wrap_tool_call
def handle_tool_errors(
    request: ToolCallRequest,
    handler: Callable[[ToolCallRequest], ToolMessage | Command]
) -> ToolMessage | Command:
    """도구 에러 처리"""
    tool_name = request.tool_call['name']
    
    try:
        result = handler(request)
        print(f"[TOOL] {tool_name} succeeded")
        return result
    except Exception as e:
        print(f"[TOOL] {tool_name} failed: {e}")
        return ToolMessage(
            content=f"Tool '{tool_name}' encountered an error. Try another approach.",
            tool_call_id=request.tool_call['id']
        )

# ===== Agent 생성 =====
# 기본 모델 (동적으로 변경됨)
default_model = init_chat_model("openai:gpt-4o-mini", temperature=0.7)

agent = create_agent(
    model=default_model,
    tools=[search_knowledge_base, save_user_note, admin_delete_data],
    middleware=[
        context_aware_prompt,           # Dynamic prompt
        trim_long_conversations,        # Message trimming
        log_model_call_start,          # Logging
        filter_tools_by_permission,    # Tool filtering
        select_model_by_context,       # Model selection
        log_model_call_end,            # Logging
        handle_tool_errors,            # Error handling
    ],
    context_schema=ApplicationContext,
    store=store,
    system_prompt="Base system prompt (will be overridden by dynamic prompt)"
)

# ===== 실행 예제 =====

# 1. Admin 사용자로 실행
print("\n" + "="*60)
print("Example 1: Admin User")
print("="*60)

result1 = agent.invoke(
    {"messages": [{"role": "user", "content": "지식 베이스에서 LangChain 정보를 찾아줘"}]},
    context=ApplicationContext(
        user_id="admin001",
        user_role="admin",
        language="ko",
        max_message_length=10
    )
)

print("\n[RESULT]", result1["messages"][-1].content)

# 2. 일반 사용자로 실행
print("\n" + "="*60)
print("Example 2: Regular User")
print("="*60)

result2 = agent.invoke(
    {"messages": [{"role": "user", "content": "Save a note: Meeting at 3PM"}]},
    context=ApplicationContext(
        user_id="user456",
        user_role="user",
        language="en",
        max_message_length=10
    )
)

print("\n[RESULT]", result2["messages"][-1].content)

# 3. 긴 대화 시뮬레이션 (모델 자동 전환 테스트)
print("\n" + "="*60)
print("Example 3: Long Conversation (Model Switching)")
print("="*60)

long_messages = [
    {"role": "user", "content": f"Question {i}: Tell me something"}
    for i in range(12)
]

result3 = agent.invoke(
    {"messages": long_messages},
    context=ApplicationContext(
        user_id="user789",
        user_role="user",
        language="ko"
    )
)

print(f"\n[RESULT] Final message count: {len(result3['messages'])}")
```

***

## 12. Best Practices <a href="#id-12-best-practices" id="id-12-best-practices"></a>

### Context Engineering 모범 사례​

1. **단순하게 시작**: 정적 프롬프트와 도구로 시작하고, 필요할 때만 동적 기능 추가
2. **점진적 테스트**: 한 번에 하나의 컨텍스트 엔지니어링 기능 추가
3. **성능 모니터링**: 모델 호출, 토큰 사용량, 지연 시간 추적
4. **내장 미들웨어 활용**: `SummarizationMiddleware`, `ToolRetryMiddleware` 등 활용
5. **컨텍스트 전략 문서화**: 어떤 컨텍스트가 왜 전달되는지 명확히 문서화

### Middleware 실행 순서​

```python
# Middleware: [middleware1, middleware2, middleware3]

# Before hooks: 순서대로 실행
middleware1.before_model() 
→ middleware2.before_model() 
→ middleware3.before_model()

# Wrap hooks: 중첩 실행 (함수 호출처럼)
middleware1.wrap_model_call(
    middleware2.wrap_model_call(
        middleware3.wrap_model_call(model)
    )
)

# After hooks: 역순 실행
middleware3.after_model() 
→ middleware2.after_model() 
→ middleware1.after_model()
```
