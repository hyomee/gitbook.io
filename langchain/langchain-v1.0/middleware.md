# Middleware

## LangChain v1.0 Middleware  <a href="#langchain-v10-middleware" id="langchain-v10-middleware"></a>

LangChain v1.0에서 도입된 **Middleware**는 에이전트의 핵심 루프(모델 호출 + 도구 실행)에 대한 세밀한 제어를 제공하는 강력한 기능 Middleware를 사용하면 모델 호출 전후, 도구 실행, 프롬프트 수정 등 다양한 시점에 커스텀 로직을 삽입할 수 있습니다.​

## 1. Middleware의 핵심 개념

Middleware는 에이전트 실행의 특정 지점에서 실행되는 **3가지 핵심 Hook**을 제공한다.

**1. before\_model Hook**

* 모델 호출 전에 실행
* 상태 업데이트, 입력 검증, 조건부 라우팅 가능
* `jump_to`를 사용해 실행 흐름 제어 가능

**2. modify\_model\_request Hook**

* 모델 요청 직전에 실행
* 프롬프트, 도구, 모델 파라미터 동적 수정
* 영구 상태 변경 불가 (요청만 수정)

**3. after\_model Hook**

* 모델 응답 후, 도구 실행 전에 실행
* 응답 검증, 후처리, 상태 업데이트
* `jump_to`로 흐름 제어 가능

## 2. 기본 구현 패턴

### 2-1. 클래스 기반 Middleware (추천)

```python
from langchain.agents.middleware import AgentMiddleware, AgentState, ModelRequest
from langgraph.runtime import Runtime
from typing import Any, Callable

class CustomMiddleware(AgentMiddleware):
    """기본 Middleware 클래스 구조"""
    
    def before_model(self, state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
        """모델 호출 전 실행"""
        print(f"Processing {len(state['messages'])} messages")
        
        # 조건부 종료
        if len(state['messages']) > 50:
            return {"jump_to": "end"}
        
        return None
    
    def modify_model_request(self, request: ModelRequest, state: AgentState) -> ModelRequest:
        """모델 요청 수정"""
        # 시스템 프롬프트 동적 생성
        user_id = state.get("user_id", "guest")
        request.system_prompt = f"You are assisting user {user_id}"
        
        # 도구 필터링
        request.tools = [t for t in request.tools if self.is_allowed(t, user_id)]
        
        return request
    
    def after_model(self, state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
        """모델 호출 후 실행"""
        last_message = state["messages"][-1]
        
        # 응답 검증
        if "BLOCKED" in last_message.content:
            return {
                "messages": [AIMessage("I cannot respond to that.")],
                "jump_to": "end"
            }
        
        return None
```

### 2-2. Decorator 기반 Middleware (간단한 경우)

```python
from langchain.agents.middleware import before_model, after_model, wrap_model_call
from langchain.agents.middleware import ModelRequest, ModelResponse
from typing import Callable

# Node-style: 로깅
@before_model
def log_before_model(state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
    print(f"Messages: {len(state['messages'])}")
    return None

# Node-style: 검증
@after_model(can_jump_to=["end"])
def validate_output(state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
    last_msg = state["messages"][-1]
    if "BLOCKED" in last_msg.content:
        return {"jump_to": "end"}
    return None

# Wrap-style: 재시도 로직
@wrap_model_call
def retry_model(
    request: ModelRequest,
    handler: Callable[[ModelRequest], ModelResponse]
) -> ModelResponse:
    for attempt in range(3):
        try:
            return handler(request)
        except Exception as e:
            if attempt == 2:
                raise
            print(f"Retry {attempt + 1}/3 after error: {e}")
```

## 3. 예시

### 3-1. OpenAI API 사용 예제

```python
from langchain.agents import create_agent
from langchain.agents.middleware import AgentMiddleware, ModelRequest
from langchain_openai import ChatOpenAI
from langchain.messages import HumanMessage, AIMessage
from langgraph.checkpoint.memory import InMemorySaver

class OpenAIMiddleware(AgentMiddleware):
    """OpenAI 모델을 위한 커스텀 Middleware"""
    
    def before_model(self, state: AgentState, runtime: Runtime):
        print(f"[OpenAI] Processing {len(state['messages'])} messages")
        return None
    
    def modify_model_request(self, request: ModelRequest, state: AgentState):
        # 대화 길이에 따라 모델 동적 선택
        msg_count = len(state['messages'])
        
        if msg_count > 10:
            request.model = ChatOpenAI(model="gpt-4o", temperature=0.7)
            print("[OpenAI] Using GPT-4o for complex conversation")
        else:
            request.model = ChatOpenAI(model="gpt-4o-mini", temperature=0.5)
            print("[OpenAI] Using GPT-4o-mini for simple query")
        
        # 커스텀 시스템 프롬프트
        request.system_prompt = """You are a helpful AI assistant. 
        Be concise and professional in your responses."""
        
        return request
    
    def after_model(self, state: AgentState, runtime: Runtime):
        last_msg = state["messages"][-1]
        print(f"[OpenAI] Response generated: {len(last_msg.content)} chars")
        return None

# Agent 생성
agent = create_agent(
    model=ChatOpenAI(model="gpt-4o"),  # 기본 모델
    tools=[],  # 도구 리스트
    middleware=[OpenAIMiddleware()],
    checkpointer=InMemorySaver()
)

# 사용
result = agent.invoke({
    "messages": [HumanMessage(content="What is machine learning?")]
})
print(result["messages"][-1].content)
```

### 3-2. Ollama API 사용 예제

```python
from langchain.agents import create_agent
from langchain.agents.middleware import AgentMiddleware, ModelRequest
from langchain_ollama import ChatOllama
from langchain.messages import HumanMessage
from langgraph.checkpoint.memory import InMemorySaver

class OllamaMiddleware(AgentMiddleware):
    """Ollama 로컬 모델을 위한 커스텀 Middleware"""
    
    def before_model(self, state: AgentState, runtime: Runtime):
        print(f"[Ollama] Processing {len(state['messages'])} messages")
        
        # 메시지가 너무 많으면 요약
        if len(state['messages']) > 30:
            print("[Ollama] Message history getting long, consider summarizing")
        
        return None
    
    def modify_model_request(self, request: ModelRequest, state: AgentState):
        last_msg = state['messages'][-1]
        query_text = str(last_msg.content).lower()
        
        # 쿼리 복잡도에 따라 모델 선택
        if any(keyword in query_text for keyword in ['complex', 'detailed', 'explain']):
            # 복잡한 질문: 큰 모델 사용
            request.model = ChatOllama(
                model="llama3.1:70b",
                temperature=0.7,
                num_predict=2048
            )
            print("[Ollama] Using llama3.1:70b for complex query")
        else:
            # 간단한 질문: 작은 모델 사용
            request.model = ChatOllama(
                model="llama3.2:3b",
                temperature=0.5,
                num_predict=512
            )
            print("[Ollama] Using llama3.2:3b for simple query")
        
        # Ollama 특정 설정
        request.model_kwargs = {
            "top_k": 40,
            "top_p": 0.9,
            "repeat_penalty": 1.1
        }
        
        return request
    
    def after_model(self, state: AgentState, runtime: Runtime):
        last_msg = state["messages"][-1]
        response_preview = last_msg.content[:100]
        print(f"[Ollama] Response: {response_preview}...")
        return None

# Agent 생성
agent = create_agent(
    model=ChatOllama(model="llama3.1:8b"),  # 기본 모델
    tools=[],
    middleware=[OllamaMiddleware()],
    checkpointer=InMemorySaver()
)

# 사용
result = agent.invoke({
    "messages": [HumanMessage(content="Explain quantum computing in detail")]
})
print(result["messages"][-1].content)
```

## 4. 고급 구현 패턴

### 4-1. 도구 접근 제어 Middleware

```python
class ToolAccessControlMiddleware(AgentMiddleware):
    """사용자 역할에 따른 도구 접근 제어"""
    
    def __init__(self, permissions_config: dict):
        super().__init__()
        self.permissions = permissions_config
    
    def modify_model_request(self, request: ModelRequest, state: AgentState):
        user_role = state.get("user_role", "guest")
        
        # 역할별 허용 도구 필터링
        allowed_tools = []
        for tool in request.tools:
            tool_name = tool.name if hasattr(tool, 'name') else str(tool)
            
            if self._is_tool_allowed(tool_name, user_role):
                allowed_tools.append(tool)
            else:
                print(f"[Access] Blocked tool '{tool_name}' for role '{user_role}'")
        
        request.tools = allowed_tools
        return request
    
    def _is_tool_allowed(self, tool_name: str, role: str) -> bool:
        allowed = self.permissions.get(role, [])
        return "all" in allowed or tool_name in allowed

# 사용 예제
permissions = {
    "admin": ["all"],
    "user": ["search", "calculator", "weather"],
    "guest": ["search"]
}

agent = create_agent(
    model="openai:gpt-4o",
    tools=[search_tool, calculator_tool, email_tool, database_tool],
    middleware=[ToolAccessControlMiddleware(permissions)],
    checkpointer=InMemorySaver()
)
```

### 4-2. 응답 검증 및 PII 제거 Middleware

```python
import re
from langchain.agents.middleware import AgentMiddleware

class PIIRedactionMiddleware(AgentMiddleware):
    """개인정보 자동 제거 Middleware"""
    
    PII_PATTERNS = {
        'email': r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b',
        'phone': r'\b\d{3}[-.]?\d{3}[-.]?\d{4}\b',
        'ssn': r'\b\d{3}-\d{2}-\d{4}\b',
        'credit_card': r'\b\d{4}[- ]?\d{4}[- ]?\d{4}[- ]?\d{4}\b'
    }
    
    def after_model(self, state: AgentState, runtime: Runtime):
        last_msg = state["messages"][-1]
        original_content = last_msg.content
        
        # PII 검출 및 제거
        filtered_content = self._redact_pii(original_content)
        
        if filtered_content != original_content:
            print("[PII] Redacted sensitive information")
            last_msg.content = filtered_content
            return {"messages": state["messages"]}
        
        return None
    
    def _redact_pii(self, text: str) -> str:
        for pii_type, pattern in self.PII_PATTERNS.items():
            text = re.sub(pattern, f'[REDACTED_{pii_type.upper()}]', text)
        return text
```

### 4-3. Custom State를 사용하는 Middleware

```python
from typing_extensions import NotRequired

class CustomAgentState(AgentState):
    """커스텀 상태 스키마"""
    model_call_count: NotRequired[int]
    user_id: NotRequired[str]
    session_start_time: NotRequired[float]

class SessionManagementMiddleware(AgentMiddleware[CustomAgentState]):
    """세션 관리 및 호출 제한 Middleware"""
    
    state_schema = CustomAgentState
    
    def __init__(self, max_calls: int = 20):
        super().__init__()
        self.max_calls = max_calls
    
    def before_model(self, state: CustomAgentState, runtime: Runtime):
        # 호출 횟수 확인
        call_count = state.get("model_call_count", 0)
        
        if call_count >= self.max_calls:
            print(f"[Session] Call limit reached ({self.max_calls})")
            return {
                "messages": [AIMessage("Session limit reached. Please start a new session.")],
                "jump_to": "end"
            }
        
        return None
    
    def after_model(self, state: CustomAgentState, runtime: Runtime):
        # 호출 횟수 증가
        current_count = state.get("model_call_count", 0)
        return {"model_call_count": current_count + 1}

# 사용
agent = create_agent(
    model="openai:gpt-4o",
    tools=[],
    middleware=[SessionManagementMiddleware(max_calls=10)],
    checkpointer=InMemorySaver()
)

result = agent.invoke({
    "messages": [HumanMessage("Hello")],
    "model_call_count": 0,
    "user_id": "user-123"
})
```

## 5. Wrap-style Hooks (고급 패턴)

### 5-1. 재시도 Middleware

```python
import time
from typing import Callable

class RetryMiddleware(AgentMiddleware):
    """지수 백오프를 사용한 재시도 Middleware"""
    
    def __init__(self, max_retries: int = 3, backoff_factor: float = 2.0):
        super().__init__()
        self.max_retries = max_retries
        self.backoff_factor = backoff_factor
    
    def wrap_model_call(
        self,
        request: ModelRequest,
        handler: Callable[[ModelRequest], ModelResponse]
    ) -> ModelResponse:
        for attempt in range(self.max_retries):
            try:
                return handler(request)
            except Exception as e:
                if attempt == self.max_retries - 1:
                    print(f"[Retry] All attempts failed")
                    raise
                
                wait_time = self.backoff_factor ** attempt
                print(f"[Retry] Attempt {attempt + 1} failed, waiting {wait_time}s")
                time.sleep(wait_time)
```

### 5-2. 도구 호출 모니터링 Middleware

```python
from langchain.tools.tool_node import ToolCallRequest
from langchain_core.messages import ToolMessage

class ToolMonitoringMiddleware(AgentMiddleware):
    """도구 호출 모니터링 및 로깅"""
    
    def wrap_tool_call(
        self,
        request: ToolCallRequest,
        handler: Callable[[ToolCallRequest], ToolMessage]
    ) -> ToolMessage:
        tool_name = request.tool_call['name']
        tool_args = request.tool_call['args']
        
        start_time = time.time()
        print(f"[Tool] Executing: {tool_name}")
        print(f"[Tool] Arguments: {tool_args}")
        
        try:
            result = handler(request)
            elapsed = time.time() - start_time
            print(f"[Tool] Success in {elapsed:.2f}s")
            return result
        except Exception as e:
            elapsed = time.time() - start_time
            print(f"[Tool] Failed after {elapsed:.2f}s: {e}")
            raise
```

### 5-3. 여러 Middleware 조합 사용

```python
from langchain.agents.middleware import (
    SummarizationMiddleware,
    HumanInTheLoopMiddleware,
    ModelCallLimitMiddleware
)

# 복합 Middleware 구성
agent = create_agent(
    model="openai:gpt-4o",
    tools=[search_tool, calculator_tool, email_tool, database_tool],
    middleware=[
        # 1. 대화 요약 (토큰 관리)
        SummarizationMiddleware(
            model="openai:gpt-4o-mini",
            max_tokens_before_summary=4000,
            messages_to_keep=20
        ),
        
        # 2. Human-in-the-loop (승인 필요)
        HumanInTheLoopMiddleware(
            interrupt_on={
                "email_tool": {
                    "allowed_decisions": ["approve", "edit", "reject"]
                },
                "database_tool": {
                    "allowed_decisions": ["approve", "reject"]
                },
                "search_tool": False  # 자동 승인
            }
        ),
        
        # 3. 호출 횟수 제한
        ModelCallLimitMiddleware(
            thread_limit=20,
            run_limit=5,
            exit_behavior="end"
        ),
        
        # 4. 커스텀 로깅
        CustomLoggingMiddleware(),
        
        # 5. PII 제거
        PIIRedactionMiddleware(),
        
        # 6. 도구 접근 제어
        ToolAccessControlMiddleware(permissions)
    ],
    checkpointer=InMemorySaver()
)
```

### 5-4. 실행 순서 이해하기

여러 Middleware를 사용할 때 실행 순서:​

```python
middleware=[middleware1, middleware2, middleware3]

# 실행 순서:
# 1. before_model: 1 → 2 → 3 (순서대로)
# 2. modify_model_request: 1 → 2 → 3 (순서대로)
# 3. wrap_model_call: 1 → 2 → 3 → MODEL → 3 → 2 → 1 (중첩)
# 4. after_model: 3 → 2 → 1 (역순)
```

### 5-5. Built-in Middleware 활용

LangChain v1.0은 다양한 내장 Middleware를 제공한다:​

```python
from langchain.agents.middleware import (
    SummarizationMiddleware,          # 대화 요약
    HumanInTheLoopMiddleware,         # 사람 승인
    ModelCallLimitMiddleware,         # 호출 제한
    ToolCallLimitMiddleware,          # 도구 제한
    ModelFallbackMiddleware,          # 모델 대체
    PIIMiddleware,                    # PII 검출/제거
    TodoListMiddleware,               # 작업 관리
    LLMToolSelectorMiddleware,        # 도구 선택
    ToolRetryMiddleware,              # 도구 재시도
    LLMToolEmulator,                  # 도구 에뮬레이션
    ContextEditingMiddleware,         # 컨텍스트 편집
)
```

## 6. Best Practices

**1. 단일 책임 원칙**: 각 Middleware는 하나의 명확한 역할만 수행​

**2. 적절한 Hook 선택**:​

* Node-style (`before_model`, `after_model`): 순차적 로직, 로깅, 검증
* Wrap-style (`wrap_model_call`, `wrap_tool_call`): 재시도, 캐싱, 제어 흐름

**3. 에러 처리**: Middleware 에러가 전체 시스템을 중단시키지 않도록 처리​

**4. 성능 고려**: 비용이 큰 연산은 캐싱하거나 초기화 시 수행​

**5. 테스트**: 각 Middleware를 독립적으로 단위 테스트​

**6. 문서화**: Custom state 속성 명확히 문서화​

## 7. 주요 차이점: OpenAI vs Ollama

| 특징            | OpenAI                        | Ollama                             |
| ------------- | ----------------------------- | ---------------------------------- |
| **모델 초기화**    | `ChatOpenAI(model="gpt-4o")`​ | `ChatOllama(model="llama3.1:8b")`​ |
| **API 엔드포인트** | 클라우드 (api.openai.com)         | 로컬 (localhost:11434)​              |
| **모델 선택**     | 문자열로 모델명 지정                   | Ollama에서 pull한 모델명​                |
| **비용**        | 토큰 기반 과금                      | 로컬 실행 (무료)​                        |
| **프라이버시**     | 데이터가 클라우드로 전송                 | 완전 로컬 (높은 프라이버시)​                  |
| **레이턴시**      | 네트워크 지연 존재                    | 로컬 실행으로 빠름​                        |
| **모델 크기**     | 제한 없음                         | 하드웨어 제약 존재​                        |

## 8. 설치 및 설정

```bash
# LangChain v1.0 alpha 설치
pip install --pre -U langchain

# OpenAI 사용
pip install -U langchain-openai

# Ollama 사용
pip install -U langchain-ollama
```

```
python# OpenAI API Key 설정
import os
os.environ["OPENAI_API_KEY"] = "your-api-key"

# Ollama 서버 시작 (터미널에서)
# ollama serve
# ollama pull llama3.1:8b
```
