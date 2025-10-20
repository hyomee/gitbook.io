# Guardrails

## LangChain v1.0 Guardrails  <a href="#langchain-v10-guardrails" id="langchain-v10-guardrails"></a>

## 1. Guardrails란? <a href="#guardrails" id="guardrails"></a>

Guardrails는 AI 애플리케이션을 안전하고 규정을 준수하도록 만들기 위해 에이전트 실행의 주요 지점에서 콘텐츠를 검증하고 필터링하는 보호 장치로 Guardrails는 민감한 정보 감지, 콘텐츠 정책 시행, 출력 검증, 그리고 위험한 동작을 사전에 방지하는 역할을 수행하여  Guardrails 시스템은 Middleware를 통해 강력하고 유연한 AI 안전성 제어를 제공하여 필요에 따라 다층 보안 시스템을 구축할 수 있다.​

**핵심 요점:**

* **Before/After Hooks**: 순차적 검증 및 상태 업데이트에 사용​
* **Wrap Hooks**: 실행 제어, 재시도, 폴백에 사용​
* **내장 Guardrails**: PII, HITL 등 즉시 사용 가능​
* **커스텀 Guardrails**: 비즈니스 로직에 맞게 확장 가능​
* **Decorator 패턴**: 간단한 경우 빠른 구현 가능

### 1-1. 주요 사용 사례

* **PII(개인식별정보) 유출 방지**: 이메일, 신용카드 번호 등의 민감 정보 탐지 및 처리​
* **프롬프트 인젝션 공격 탐지 및 차단**: 악의적인 프롬프트 시도 방지​
* **부적절하거나 유해한 콘텐츠 차단**: 독성 콘텐츠, 폭력적 표현 등 필터링​
* **비즈니스 규칙 및 컴플라이언스 요구사항 시행**: 산업별 규제 준수​
* **출력 품질 및 정확성 검증**: LLM 응답의 품질 보장​

### 1-2. Guardrails의 두 가지 접근 방식 <a href="#guardrails" id="guardrails"></a>

#### 1-2-1. Deterministic Guardrails (규칙 기반)

정규 표현식 패턴, 키워드 매칭, 명시적 체크와 같은 규칙 기반 로직을 사용한.​

**장점:**

* 빠르고 예측 가능함
* 비용 효율적
* 명확한 규칙으로 디버깅 용이

**단점:**

* 미묘한 위반 사항을 놓칠 수 있음
* 복잡한 맥락 이해 불가

#### 1-2-2. Model-based Guardrails (모델 기반)

LLM이나 분류기를 사용하여 의미적 이해로 콘텐츠를 평가한다.​

**장점:**

* 규칙이 놓치는 미묘한 문제 포착
* 맥락적 이해 가능
* 유연한 적용

**단점:**

* 더 느리고 비용이 많이 듦
* 예측 가능성이 낮음

## 2. LangChain v1.0의 Middleware 시스템 <a href="#langchain-v10-middleware" id="langchain-v10-middleware"></a>

LangChain v1.0에서는 Guardrails를 **Middleware**를 통해 한다.. Middleware는 에이전트 실행 흐름의 특정 지점에서 실행되는 훅(hook)을 제공한다.​

### 2-1. Middleware Hook 종류

#### 2-1-1. Node-style Hooks (순차 실행)

* `before_agent`: 에이전트 시작 전 (호출당 1회)​
* `before_model`: 각 모델 호출 전​
* `after_model`: 각 모델 응답 후​
* `after_agent`: 에이전트 완료 후 (호출당 최대 1회)​

#### 2-1-2. Wrap-style Hooks (실행 제어)

* `wrap_model_call`: 각 모델 호출을 감싸서 제어​
* `wrap_tool_call`: 각 도구 호출을 감싸서 제어​

## 3. 구현 패턴 <a href="#id-1--pii-detection-guardrails" id="id-1--pii-detection-guardrails"></a>

### 3-1. 내장 PII Detection Guardrails <a href="#id-1--pii-detection-guardrails" id="id-1--pii-detection-guardrails"></a>

#### 3-1-1. OpenAI 사용 예제

```python
from langchain.agents import create_agent
from langchain.agents.middleware import PIIMiddleware
from langchain_openai import ChatOpenAI

# OpenAI 모델 설정
model = ChatOpenAI(
    model="gpt-4o",
    api_key="your-openai-api-key",
    temperature=0
)

# PII 탐지 Guardrails를 적용한 에이전트 생성
agent = create_agent(
    model=model,
    tools=[search_tool, email_tool],
    middleware=[
        # 입력에서 이메일 Redact (숨김 처리)
        PIIMiddleware(
            "email",
            strategy="redact",  # [REDACTED_EMAIL]로 대체
            apply_to_input=True,
        ),
        # 입력에서 신용카드 번호 Mask (일부만 표시)
        PIIMiddleware(
            "credit_card",
            strategy="mask",  # ****-****-****-1234 형태로 표시
            apply_to_input=True,
        ),
        # API 키 탐지 시 차단
        PIIMiddleware(
            "api_key",
            detector=r"sk-[a-zA-Z0-9]{32}",  # 정규식 패턴
            strategy="block",  # 에러 발생
            apply_to_input=True,
        ),
        # 출력에서 IP 주소 해싱
        PIIMiddleware(
            "ip",
            strategy="hash",  # 결정론적 해시로 대체
            apply_to_output=True,
        ),
    ],
)

# 사용 예시
result = agent.invoke({
    "messages": [{
        "role": "user", 
        "content": "My email is john@example.com and my card is 4532-1234-5678-9010"
    }]
})
print(result)
```

#### 3-1-2. Ollama 사용 예제

```python
from langchain.agents import create_agent
from langchain.agents.middleware import PIIMiddleware
from langchain_ollama import ChatOllama

# Ollama 모델 설정 (로컬에서 실행)
model = ChatOllama(
    model="llama3.1",
    base_url="http://localhost:11434",
    temperature=0
)

# PII 탐지 Guardrails를 적용한 에이전트 생성
agent = create_agent(
    model=model,
    tools=[search_tool, database_tool],
    middleware=[
        # 이메일 주소 탐지 및 숨김
        PIIMiddleware(
            "email",
            strategy="redact",
            apply_to_input=True,
            apply_to_output=True,
        ),
        # 전화번호 탐지 (커스텀 정규식)
        PIIMiddleware(
            "phone_number",
            detector=r"\d{3}-\d{4}-\d{4}",  # 한국 전화번호 패턴
            strategy="mask",
            apply_to_input=True,
        ),
        # 주민등록번호 탐지 및 차단
        PIIMiddleware(
            "ssn_korea",
            detector=r"\d{6}-\d{7}",
            strategy="block",
            apply_to_input=True,
        ),
    ],
)

# 사용 예시
result = agent.invoke({
    "messages": [{
        "role": "user",
        "content": "제 이메일은 kim@example.com이고 전화번호는 010-1234-5678입니다"
    }]
})
```

#### 3-1-3. PII 전략 비교표

| 전략       | 설명                    | 예시                 |
| -------- | --------------------- | ------------------ |
| `redact` | `[REDACTED_TYPE]`로 대체 | `[REDACTED_EMAIL]` |
| `mask`   | 부분적으로 숨김              | `**-**-****-1234`  |
| `hash`   | 결정론적 해시로 대체           | `a8f5f167...`      |
| `block`  | 탐지 시 예외 발생            | Error thrown       |

### 3-2. Human-in-the-Loop Guardrails <a href="#id-2-human-in-the-loop-guardrails" id="id-2-human-in-the-loop-guardrails"></a>

#### 3-2-1. OpenAI + HITL 예제

```python
from langchain.agents import create_agent
from langchain.agents.middleware import HumanInTheLoopMiddleware
from langchain_openai import ChatOpenAI
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.types import Command

# OpenAI 모델 설정
model = ChatOpenAI(model="gpt-4o", api_key="your-api-key")

# HITL을 위한 Checkpointer 필요
checkpointer = InMemorySaver()

agent = create_agent(
    model=model,
    tools=[search_tool, send_email_tool, delete_database_tool],
    middleware=[
        HumanInTheLoopMiddleware(
            interrupt_on={
                # 민감한 작업은 승인 필요
                "send_email": True,
                "delete_database": True,
                # 안전한 작업은 자동 승인
                "search": False,
            }
        ),
    ],
    checkpointer=checkpointer,
)

# Thread ID가 필요 (상태 유지용)
config = {"configurable": {"thread_id": "conversation-123"}}

# 1단계: 에이전트 호출 (이메일 전송 시도)
result = agent.invoke(
    {"messages": [{"role": "user", "content": "팀에게 이메일을 보내줘"}]},
    config=config
)
# 에이전트가 일시 중지되고 승인 대기

# 2단계: 승인 후 재개
result = agent.invoke(
    Command(resume={"decisions": [{"type": "approve"}]}),
    config=config  # 동일한 thread_id로 재개
)
print(result)
```

### 3-2-2. Ollama + HITL 예제 (고급 설정)

```python
from langchain.agents import create_agent
from langchain.agents.middleware import HumanInTheLoopMiddleware
from langchain_ollama import ChatOllama
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.types import Command

# Ollama 모델 설정
model = ChatOllama(
    model="llama3.1",
    base_url="http://localhost:11434"
)

checkpointer = InMemorySaver()

def custom_description(tool_call):
    """커스텀 도구 호출 설명"""
    return f"도구: {tool_call['name']}, 인자: {tool_call['args']}"

agent = create_agent(
    model=model,
    tools=[read_email_tool, send_email_tool, update_database_tool],
    middleware=[
        HumanInTheLoopMiddleware(
            interrupt_on={
                # 세밀한 제어 설정
                "send_email": {
                    "allowed_decisions": ["approve", "edit", "reject"],
                    "description": custom_description,
                },
                # 기본 승인만 허용
                "update_database": {
                    "allowed_decisions": ["approve", "reject"],
                },
                # 자동 승인
                "read_email": False,
            },
            description_prefix="[승인 필요]"
        ),
    ],
    checkpointer=checkpointer,
)

config = {"configurable": {"thread_id": "session-456"}}

# 호출 및 승인
result = agent.invoke(
    {"messages": [{"role": "user", "content": "데이터베이스를 업데이트해줘"}]},
    config=config
)

# 수정(edit)이 필요한 경우
result = agent.invoke(
    Command(resume={
        "decisions": [{
            "type": "edit",
            "tool_call": {
                "name": "send_email",
                "args": {"to": "modified@example.com", "body": "수정된 내용"}
            }
        }]
    }),
    config=config
)
```

### 3-3. Custom Guardrails - Before Agent Hook <a href="#id-3-custom-guardrails---before-agent-hook" id="id-3-custom-guardrails---before-agent-hook"></a>

#### 3-3-1. OpenAI + 콘텐츠 필터 예제

```python
from typing import Any
from langchain.agents.middleware import AgentMiddleware, AgentState, hook_config
from langgraph.runtime import Runtime
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI

class ContentFilterMiddleware(AgentMiddleware):
    """Deterministic Guardrail: 금지된 키워드가 포함된 요청 차단"""
    
    def __init__(self, banned_keywords: list[str]):
        super().__init__()
        self.banned_keywords = [kw.lower() for kw in banned_keywords]
    
    @hook_config(can_jump_to=["end"])
    def before_agent(self, state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
        # 첫 번째 사용자 메시지 확인
        if not state["messages"]:
            return None
        
        first_message = state["messages"][0]
        if first_message.type != "human":
            return None
        
        content = first_message.content.lower()
        
        # 금지된 키워드 체크
        for keyword in self.banned_keywords:
            if keyword in content:
                # 모든 처리 전에 실행 차단
                return {
                    "messages": [{
                        "role": "assistant",
                        "content": "부적절한 콘텐츠가 포함된 요청은 처리할 수 없습니다. 다시 작성해주세요."
                    }],
                    "jump_to": "end"
                }
        
        return None

# OpenAI 모델과 함께 사용
model = ChatOpenAI(model="gpt-4o", api_key="your-api-key")

agent = create_agent(
    model=model,
    tools=[search_tool, calculator_tool],
    middleware=[
        ContentFilterMiddleware(
            banned_keywords=["hack", "exploit", "malware", "jailbreak"]
        ),
    ],
)

# 차단될 요청
result = agent.invoke({
    "messages": [{"role": "user", "content": "How do I hack into a database?"}]
})
print(result)  # "부적절한 콘텐츠가 포함된 요청은 처리할 수 없습니다..."
```

### 3-3-2. Ollama + 언어 감지 Guardrail

```python
from typing import Any
from langchain.agents.middleware import AgentMiddleware, AgentState, hook_config
from langgraph.runtime import Runtime
from langchain.agents import create_agent
from langchain_ollama import ChatOllama
import re

class LanguageDetectionMiddleware(AgentMiddleware):
    """허용된 언어만 처리하는 Guardrail"""
    
    def __init__(self, allowed_languages: list[str]):
        super().__init__()
        self.allowed_languages = allowed_languages
        # 간단한 언어 감지 (한국어, 영어)
        self.korean_pattern = re.compile(r'[가-힣]')
        self.english_pattern = re.compile(r'[a-zA-Z]')
    
    def detect_language(self, text: str) -> str:
        korean_count = len(self.korean_pattern.findall(text))
        english_count = len(self.english_pattern.findall(text))
        
        if korean_count > english_count:
            return "ko"
        elif english_count > 0:
            return "en"
        return "unknown"
    
    @hook_config(can_jump_to=["end"])
    def before_agent(self, state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
        if not state["messages"]:
            return None
        
        first_message = state["messages"][0]
        if first_message.type != "human":
            return None
        
        detected_lang = self.detect_language(first_message.content)
        
        if detected_lang not in self.allowed_languages:
            return {
                "messages": [{
                    "role": "assistant",
                    "content": f"지원하지 않는 언어입니다. 허용된 언어: {', '.join(self.allowed_languages)}"
                }],
                "jump_to": "end"
            }
        
        return None

# Ollama 모델과 함께 사용
model = ChatOllama(
    model="llama3.1",
    base_url="http://localhost:11434"
)

agent = create_agent(
    model=model,
    tools=[search_tool, translation_tool],
    middleware=[
        LanguageDetectionMiddleware(
            allowed_languages=["ko", "en"]
        ),
    ],
)

# 사용 예시
result = agent.invoke({
    "messages": [{"role": "user", "content": "안녕하세요, 검색해주세요"}]
})
```

### 3-4. Custom Guardrails - After Agent Hook (모델 기반) <a href="#id-4-custom-guardrails---after-agent-hook" id="id-4-custom-guardrails---after-agent-hook"></a>

#### 3-4-1. OpenAI + 안전성 검증 Guardrail

```python
from langchain.agents.middleware import AgentMiddleware, AgentState, hook_config
from langgraph.runtime import Runtime
from langchain_core.messages import AIMessage
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from typing import Any

class SafetyGuardrailMiddleware(AgentMiddleware):
    """Model-based Guardrail: LLM을 사용하여 응답 안전성 평가"""
    
    def __init__(self):
        super().__init__()
        # 안전성 평가용 경량 모델
        self.safety_model = init_chat_model("openai:gpt-4o-mini")
    
    @hook_config(can_jump_to=["end"])
    def after_agent(self, state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
        # 마지막 AI 응답 가져오기
        if not state["messages"]:
            return None
        
        last_message = state["messages"][-1]
        if not isinstance(last_message, AIMessage):
            return None
        
        # 모델을 사용하여 안전성 평가
        safety_prompt = f"""다음 응답이 안전하고 적절한지 평가하세요.
'SAFE' 또는 'UNSAFE'로만 응답하세요.

응답: {last_message.content}"""
        
        result = self.safety_model.invoke([{
            "role": "user", 
            "content": safety_prompt
        }])
        
        if "UNSAFE" in result.content:
            return {
                "messages": [{
                    "role": "assistant",
                    "content": "죄송합니다. 해당 응답을 제공할 수 없습니다. 다시 요청해주세요."
                }],
                "jump_to": "end"
            }
        
        return None

# OpenAI 메인 모델
main_model = init_chat_model("openai:gpt-4o")

agent = create_agent(
    model=main_model,
    tools=[search_tool, calculator_tool],
    middleware=[SafetyGuardrailMiddleware()],
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "How do I make explosives?"}]
})
```

#### 3-4-2. Ollama + 품질 검증 Guardrail

```python
from langchain.agents.middleware import AgentMiddleware, AgentState, hook_config
from langgraph.runtime import Runtime
from langchain_core.messages import AIMessage
from langchain_ollama import ChatOllama
from langchain.agents import create_agent
from typing import Any

class QualityValidationMiddleware(AgentMiddleware):
    """응답 품질을 검증하는 Guardrail"""
    
    def __init__(self, min_length: int = 10, max_retries: int = 2):
        super().__init__()
        self.min_length = min_length
        self.max_retries = max_retries
        self.retry_count = 0
    
    @hook_config(can_jump_to=["model"])
    def after_agent(self, state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
        if not state["messages"]:
            return None
        
        last_message = state["messages"][-1]
        if not isinstance(last_message, AIMessage):
            return None
        
        # 품질 체크: 최소 길이
        if len(last_message.content) < self.min_length:
            if self.retry_count < self.max_retries:
                self.retry_count += 1
                # 모델로 다시 점프하여 재생성
                return {
                    "messages": state["messages"] + [{
                        "role": "user",
                        "content": "좀 더 자세하게 설명해주세요."
                    }],
                    "jump_to": "model"
                }
            else:
                return {
                    "messages": [{
                        "role": "assistant",
                        "content": "충분한 정보를 제공할 수 없습니다."
                    }],
                    "jump_to": "end"
                }
        
        self.retry_count = 0
        return None

# Ollama 모델
model = ChatOllama(
    model="llama3.1",
    base_url="http://localhost:11434",
    temperature=0.7
)

agent = create_agent(
    model=model,
    tools=[search_tool],
    middleware=[
        QualityValidationMiddleware(
            min_length=50,
            max_retries=2
        )
    ],
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "인공지능이 뭐야?"}]
})
```

### 3-5. Wrap-style Guardrails <a href="#id-5-wrap-style-guardrails" id="id-5-wrap-style-guardrails"></a>

#### 3-5-1. OpenAI + 모델 호출 재시도 Guardrail

```python
from langchain.agents.middleware import AgentMiddleware, ModelRequest, ModelResponse
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI
from typing import Callable
import time

class RetryMiddleware(AgentMiddleware):
    """모델 호출 실패 시 재시도"""
    
    def __init__(self, max_retries: int = 3, backoff_factor: float = 2.0):
        super().__init__()
        self.max_retries = max_retries
        self.backoff_factor = backoff_factor
    
    def wrap_model_call(
        self,
        request: ModelRequest,
        handler: Callable[[ModelRequest], ModelResponse],
    ) -> ModelResponse:
        for attempt in range(self.max_retries):
            try:
                return handler(request)
            except Exception as e:
                if attempt == self.max_retries - 1:
                    raise
                wait_time = (self.backoff_factor ** attempt)
                print(f"재시도 {attempt + 1}/{self.max_retries}, {wait_time}초 대기: {e}")
                time.sleep(wait_time)

# OpenAI 모델
model = ChatOpenAI(
    model="gpt-4o",
    api_key="your-api-key",
    request_timeout=10
)

agent = create_agent(
    model=model,
    tools=[search_tool],
    middleware=[
        RetryMiddleware(max_retries=3, backoff_factor=2.0)
    ],
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "검색해줘"}]
})
```

#### 3-5-2. Ollama + 동적 모델 선택 Guardrail

```python
from langchain.agents.middleware import AgentMiddleware, ModelRequest, ModelResponse
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain_ollama import ChatOllama
from typing import Callable

class DynamicModelMiddleware(AgentMiddleware):
    """대화 길이에 따라 다른 모델 사용"""
    
    def __init__(self):
        super().__init__()
        self.light_model = ChatOllama(
            model="llama3.1:8b",
            base_url="http://localhost:11434"
        )
        self.heavy_model = ChatOllama(
            model="llama3.1:70b",
            base_url="http://localhost:11434"
        )
    
    def wrap_model_call(
        self,
        request: ModelRequest,
        handler: Callable[[ModelRequest], ModelResponse],
    ) -> ModelResponse:
        # 대화 길이에 따라 모델 선택
        if len(request.messages) > 10:
            print("복잡한 대화 → 큰 모델 사용 (70b)")
            request.model = self.heavy_model
        else:
            print("간단한 대화 → 작은 모델 사용 (8b)")
            request.model = self.light_model
        
        return handler(request)

# 기본 모델
base_model = ChatOllama(
    model="llama3.1:8b",
    base_url="http://localhost:11434"
)

agent = create_agent(
    model=base_model,
    tools=[calculator_tool, search_tool],
    middleware=[DynamicModelMiddleware()],
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "계산해줘: 123 * 456"}]
})
```

#### 3-5-3. OpenAI + Ollama 하이브리드 모델 Fallback

```python
from langchain.agents.middleware import AgentMiddleware, ModelRequest, ModelResponse
from langchain_openai import ChatOpenAI
from langchain_ollama import ChatOllama
from langchain.agents import create_agent
from typing import Callable

class HybridModelFallbackMiddleware(AgentMiddleware):
    """OpenAI 실패 시 Ollama로 폴백"""
    
    def __init__(self):
        super().__init__()
        self.fallback_model = ChatOllama(
            model="llama3.1",
            base_url="http://localhost:11434"
        )
    
    def wrap_model_call(
        self,
        request: ModelRequest,
        handler: Callable[[ModelRequest], ModelResponse],
    ) -> ModelResponse:
        try:
            # 먼저 OpenAI 시도
            print("OpenAI 모델 호출 시도...")
            return handler(request)
        except Exception as e:
            # 실패 시 Ollama로 폴백
            print(f"OpenAI 실패: {e}")
            print("Ollama 로컬 모델로 폴백...")
            request.model = self.fallback_model
            return handler(request)

# 주 모델: OpenAI
primary_model = ChatOpenAI(
    model="gpt-4o",
    api_key="your-api-key"
)

agent = create_agent(
    model=primary_model,
    tools=[search_tool],
    middleware=[HybridModelFallbackMiddleware()],
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "최신 뉴스 검색해줘"}]
})
```

### 3-6. 도구 호출 모니터링 Guardrail <a href="#id-6----guardrail" id="id-6----guardrail"></a>

#### 3-6-1. OpenAI + 도구 실행 로깅

```python
from langchain.tools.tool_node import ToolCallRequest
from langchain.agents.middleware import AgentMiddleware
from langchain_core.messages import ToolMessage
from langgraph.types import Command
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI
from typing import Callable
import json

class ToolMonitoringMiddleware(AgentMiddleware):
    """도구 호출 모니터링 및 로깅"""
    
    def __init__(self, log_file: str = "tool_calls.log"):
        super().__init__()
        self.log_file = log_file
    
    def wrap_tool_call(
        self,
        request: ToolCallRequest,
        handler: Callable[[ToolCallRequest], ToolMessage | Command],
    ) -> ToolMessage | Command:
        tool_name = request.tool_call['name']
        tool_args = request.tool_call['args']
        
        print(f"[도구 실행] {tool_name}")
        print(f"[인자] {json.dumps(tool_args, ensure_ascii=False, indent=2)}")
        
        try:
            result = handler(request)
            print(f"[성공] {tool_name} 완료")
            
            # 로그 기록
            with open(self.log_file, 'a', encoding='utf-8') as f:
                f.write(f"[SUCCESS] {tool_name}: {tool_args}\n")
            
            return result
        except Exception as e:
            print(f"[실패] {tool_name}: {e}")
            
            # 실패 로그 기록
            with open(self.log_file, 'a', encoding='utf-8') as f:
                f.write(f"[FAILED] {tool_name}: {tool_args} - Error: {e}\n")
            
            raise

# OpenAI 모델
model = ChatOpenAI(model="gpt-4o", api_key="your-api-key")

agent = create_agent(
    model=model,
    tools=[search_tool, calculator_tool, email_tool],
    middleware=[
        ToolMonitoringMiddleware(log_file="agent_tools.log")
    ],
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "123 곱하기 456을 계산하고 결과를 검색해줘"}]
})
```

#### 3-6-2. Ollama + 도구 사용 제한 Guardrail

```python
from langchain.tools.tool_node import ToolCallRequest
from langchain.agents.middleware import AgentMiddleware
from langchain_core.messages import ToolMessage
from langgraph.types import Command
from langchain.agents import create_agent
from langchain_ollama import ChatOllama
from typing import Callable
from collections import defaultdict

class ToolUsageLimitMiddleware(AgentMiddleware):
    """도구 사용 횟수 제한"""
    
    def __init__(self, tool_limits: dict[str, int]):
        super().__init__()
        self.tool_limits = tool_limits
        self.usage_count = defaultdict(int)
    
    def wrap_tool_call(
        self,
        request: ToolCallRequest,
        handler: Callable[[ToolCallRequest], ToolMessage | Command],
    ) -> ToolMessage | Command:
        tool_name = request.tool_call['name']
        
        # 사용 횟수 증가
        self.usage_count[tool_name] += 1
        
        # 제한 확인
        if tool_name in self.tool_limits:
            limit = self.tool_limits[tool_name]
            current_count = self.usage_count[tool_name]
            
            if current_count > limit:
                # 제한 초과 시 에러 메시지 반환
                return ToolMessage(
                    content=f"도구 '{tool_name}' 사용 제한 초과 ({current_count}/{limit})",
                    tool_call_id=request.tool_call.get('id', 'unknown')
                )
        
        return handler(request)

# Ollama 모델
model = ChatOllama(
    model="llama3.1",
    base_url="http://localhost:11434"
)

agent = create_agent(
    model=model,
    tools=[search_tool, database_query_tool, api_call_tool],
    middleware=[
        ToolUsageLimitMiddleware(
            tool_limits={
                "search_tool": 5,           # 검색 최대 5회
                "database_query_tool": 10,  # DB 쿼리 최대 10회
                "api_call_tool": 3,         # API 호출 최대 3회
            }
        )
    ],
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "여러 가지 검색해줘"}]
})
```

### 3-7: 다중 Guardrails 레이어 (종합) <a href="#id-7--guardrails" id="id-7--guardrails"></a>

#### 3-7-1. OpenAI 다층 보안 시스템

```python
from langchain.agents import create_agent
from langchain.agents.middleware import PIIMiddleware, HumanInTheLoopMiddleware
from langchain_openai import ChatOpenAI
from langgraph.checkpoint.memory import InMemorySaver

# 커스텀 Guardrails 임포트
from content_filter import ContentFilterMiddleware
from safety_check import SafetyGuardrailMiddleware
from tool_monitor import ToolMonitoringMiddleware

model = ChatOpenAI(model="gpt-4o", api_key="your-api-key")
checkpointer = InMemorySaver()

agent = create_agent(
    model=model,
    tools=[search_tool, send_email_tool, database_tool],
    middleware=[
        # Layer 1: Deterministic 입력 필터 (before_agent)
        ContentFilterMiddleware(
            banned_keywords=["hack", "exploit", "jailbreak"]
        ),
        
        # Layer 2: PII 보호 (입력/출력 모두)
        PIIMiddleware("email", strategy="redact", apply_to_input=True),
        PIIMiddleware("email", strategy="redact", apply_to_output=True),
        PIIMiddleware("credit_card", strategy="mask", apply_to_input=True),
        PIIMiddleware("ip", strategy="hash", apply_to_output=True),
        
        # Layer 3: 도구 실행 모니터링 (wrap_tool_call)
        ToolMonitoringMiddleware(log_file="security_audit.log"),
        
        # Layer 4: 민감한 도구 실행 시 인간 승인 (HITL)
        HumanInTheLoopMiddleware(
            interrupt_on={
                "send_email": True,
                "database_tool": True,
                "search_tool": False,
            }
        ),
        
        # Layer 5: Model-based 안전성 체크 (after_agent)
        SafetyGuardrailMiddleware(),
    ],
    checkpointer=checkpointer,
)

# 사용 예시
config = {"configurable": {"thread_id": "secure-session-001"}}

result = agent.invoke({
    "messages": [{
        "role": "user",
        "content": "john@example.com에게 데이터베이스 정보를 이메일로 보내줘"
    }]
}, config=config)
```

#### 3-7-2. Ollama 다층 보안 시스템

```python
from langchain.agents import create_agent
from langchain_ollama import ChatOllama
from langgraph.checkpoint.memory import InMemorySaver

# 커스텀 Guardrails
from language_detection import LanguageDetectionMiddleware
from quality_validation import QualityValidationMiddleware
from tool_usage_limit import ToolUsageLimitMiddleware

model = ChatOllama(
    model="llama3.1",
    base_url="http://localhost:11434"
)

checkpointer = InMemorySaver()

agent = create_agent(
    model=model,
    tools=[search_tool, translation_tool, summarization_tool],
    middleware=[
        # Layer 1: 언어 감지 및 제한 (before_agent)
        LanguageDetectionMiddleware(
            allowed_languages=["ko", "en"]
        ),
        
        # Layer 2: 도구 사용 횟수 제한 (wrap_tool_call)
        ToolUsageLimitMiddleware(
            tool_limits={
                "search_tool": 10,
                "translation_tool": 5,
            }
        ),
        
        # Layer 3: 응답 품질 검증 (after_agent)
        QualityValidationMiddleware(
            min_length=20,
            max_retries=2
        ),
    ],
    checkpointer=checkpointer,
)

config = {"configurable": {"thread_id": "ollama-session-001"}}

result = agent.invoke({
    "messages": [{"role": "user", "content": "인공지능의 역사를 검색하고 요약해줘"}]
}, config=config)
```

### 3-8: Decorator 기반 Guardrails (간단한 사용) <a href="#id-8-decorator--guardrails" id="id-8-decorator--guardrails"></a>

#### 3-8-1. OpenAI + Decorator Pattern

```python
from langchain.agents.middleware import before_model, after_model, wrap_model_call
from langchain.agents.middleware import AgentState, ModelRequest, ModelResponse
from langchain.messages import AIMessage
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI
from langgraph.runtime import Runtime
from typing import Any, Callable

# Node-style: 모델 호출 전 로깅
@before_model
def log_before_model(state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
    print(f"[로그] 모델 호출 - 메시지 수: {len(state['messages'])}")
    return None

# Node-style: 출력 검증
@after_model(can_jump_to=["end"])
def validate_output(state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
    last_message = state["messages"][-1]
    if "BLOCKED" in last_message.content or "차단" in last_message.content:
        return {
            "messages": [AIMessage("해당 요청에 응답할 수 없습니다.")],
            "jump_to": "end"
        }
    return None

# Wrap-style: 재시도 로직
@wrap_model_call
def retry_model(
    request: ModelRequest,
    handler: Callable[[ModelRequest], ModelResponse],
) -> ModelResponse:
    for attempt in range(3):
        try:
            return handler(request)
        except Exception as e:
            if attempt == 2:
                raise
            print(f"재시도 {attempt + 1}/3: {e}")

# OpenAI 모델
model = ChatOpenAI(model="gpt-4o", api_key="your-api-key")

agent = create_agent(
    model=model,
    tools=[search_tool, calculator_tool],
    middleware=[log_before_model, validate_output, retry_model],
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "계산해줘"}]
})
```

#### 3-8-2. Ollama + Dynamic Prompt Decorator

```python
from langchain.agents.middleware import dynamic_prompt, before_model
from langchain.agents.middleware import AgentState, ModelRequest
from langchain.agents import create_agent
from langchain_ollama import ChatOllama
from langgraph.runtime import Runtime
from typing import Any
from datetime import datetime

# 동적 시스템 프롬프트 생성
@dynamic_prompt
def personalized_prompt(request: ModelRequest) -> str:
    user_id = request.runtime.context.get("user_id", "guest")
    current_time = datetime.now().strftime("%Y-%m-%d %H:%M")
    
    return f"""당신은 사용자 {user_id}를 위한 친절한 AI 어시스턴트입니다.
현재 시각: {current_time}
간결하고 명확하게 답변해주세요."""

# 토큰 사용량 모니터링
@before_model
def monitor_tokens(state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
    total_length = sum(len(msg.content) for msg in state["messages"])
    print(f"[토큰 추정] 약 {total_length // 4} 토큰")
    
    if total_length > 10000:
        print("[경고] 토큰 제한에 근접했습니다")
    
    return None

# Ollama 모델
model = ChatOllama(
    model="llama3.1",
    base_url="http://localhost:11434"
)

agent = create_agent(
    model=model,
    tools=[search_tool],
    middleware=[personalized_prompt, monitor_tokens],
)

result = agent.invoke(
    {"messages": [{"role": "user", "content": "안녕하세요"}]},
    {"configurable": {"user_id": "user123"}}
)
```

### 3-9. Custom State Schema를 사용한 Guardrails <a href="#id-9-custom-state-schema--guardrails" id="id-9-custom-state-schema--guardrails"></a>

#### 3-9-1. OpenAI + 상태 기반 제어

```python
from langchain.agents.middleware import AgentState, AgentMiddleware
from typing_extensions import NotRequired
from typing import Any
from langchain.agents import create_agent
from langchain_openai import ChatOpenAI
from langchain_core.messages import AIMessage

class CustomState(AgentState):
    """커스텀 상태 스키마"""
    model_call_count: NotRequired[int]
    user_id: NotRequired[str]
    security_level: NotRequired[str]
    total_tokens_used: NotRequired[int]

class SecurityMiddleware(AgentMiddleware[CustomState]):
    """보안 레벨 기반 제어 Guardrail"""
    
    state_schema = CustomState
    
    def before_model(self, state: CustomState, runtime) -> dict[str, Any] | None:
        security_level = state.get("security_level", "low")
        
        # 높은 보안 레벨에서는 모델 호출 제한
        if security_level == "high":
            count = state.get("model_call_count", 0)
            if count > 5:
                return {
                    "messages": [AIMessage("보안 정책으로 인해 대화가 제한되었습니다.")],
                    "jump_to": "end"
                }
        
        return None
    
    def after_model(self, state: CustomState, runtime) -> dict[str, Any] | None:
        # 카운터 증가
        return {
            "model_call_count": state.get("model_call_count", 0) + 1,
            "total_tokens_used": state.get("total_tokens_used", 0) + 100
        }

# OpenAI 모델
model = ChatOpenAI(model="gpt-4o", api_key="your-api-key")

agent = create_agent(
    model=model,
    tools=[search_tool],
    middleware=[SecurityMiddleware()],
)

# 커스텀 상태와 함께 호출
result = agent.invoke({
    "messages": [{"role": "user", "content": "검색해줘"}],
    "model_call_count": 0,
    "user_id": "user-789",
    "security_level": "high",
    "total_tokens_used": 0,
})

print(f"총 모델 호출: {result.get('model_call_count', 0)}")
print(f"사용된 토큰: {result.get('total_tokens_used', 0)}")
```

#### 3-9-2. Ollama + 대화 컨텍스트 관리

```python
from langchain.agents.middleware import AgentState, AgentMiddleware
from typing_extensions import NotRequired
from typing import Any
from langchain.agents import create_agent
from langchain_ollama import ChatOllama
from langchain_core.messages import AIMessage

class ConversationState(AgentState):
    """대화 상태 스키마"""
    conversation_length: NotRequired[int]
    topic_switches: NotRequired[int]
    last_topic: NotRequired[str]

class ConversationManagementMiddleware(AgentMiddleware[ConversationState]):
    """대화 흐름 관리 Guardrail"""
    
    state_schema = ConversationState
    
    def __init__(self, max_length: int = 20):
        super().__init__()
        self.max_length = max_length
    
    def before_model(self, state: ConversationState, runtime) -> dict[str, Any] | None:
        conv_length = state.get("conversation_length", 0)
        
        # 대화 길이 제한
        if conv_length >= self.max_length:
            return {
                "messages": [AIMessage(
                    "대화가 너무 길어졌습니다. 새로운 대화를 시작해주세요."
                )],
                "jump_to": "end"
            }
        
        return None
    
    def after_model(self, state: ConversationState, runtime) -> dict[str, Any] | None:
        return {
            "conversation_length": state.get("conversation_length", 0) + 1,
        }

# Ollama 모델
model = ChatOllama(
    model="llama3.1",
    base_url="http://localhost:11434"
)

agent = create_agent(
    model=model,
    tools=[search_tool, calculator_tool],
    middleware=[
        ConversationManagementMiddleware(max_length=15)
    ],
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "계산해줘"}],
    "conversation_length": 0,
    "topic_switches": 0,
})
```

## 4. 주요 설정 옵션 비교 <a href="#undefined" id="undefined"></a>

### 4-1. PIIMiddleware 옵션

| 파라미터                    | 설명                                     | 기본값      |
| ----------------------- | -------------------------------------- | -------- |
| `pii_type`              | PII 유형 (email, credit\_card, ip, etc.) | 필수       |
| `strategy`              | 처리 전략 (block, redact, mask, hash)      | "redact" |
| `detector`              | 커스텀 탐지 함수 또는 정규식                       | None     |
| `apply_to_input`        | 입력 메시지 체크                              | True     |
| `apply_to_output`       | 출력 메시지 체크                              | False    |
| `apply_to_tool_results` | 도구 결과 체크                               | False    |

### 4-2. HumanInTheLoopMiddleware 옵션

| 파라미터                 | 설명                             |
| -------------------- | ------------------------------ |
| `interrupt_on`       | 도구별 승인 설정 매핑                   |
| `description_prefix` | 액션 요청 설명 접두사                   |
| `allowed_decisions`  | 허용된 결정 (approve, edit, reject) |

## 5. Best Practices <a href="#best-practices" id="best-practices"></a>

### 5-1. Guardrails 실행 순서

```python
middleware=[
    # 1. 빠른 Deterministic 체크 먼저
    ContentFilterMiddleware(...),
    
    # 2. PII 보호
    PIIMiddleware(...),
    
    # 3. 도구 실행 제어
    ToolMonitoringMiddleware(...),
    HumanInTheLoopMiddleware(...),
    
    # 4. 마지막에 느린 Model-based 체크
    SafetyGuardrailMiddleware(...),
]
```

### 5-2. OpenAI vs Ollama 선택 기준

**OpenAI 사용 권장:**

* 높은 정확도가 필요한 경우​
* 도구 호출(tool calling) 기능 필요​
* 최신 모델 접근 필요

**Ollama 사용 권장:**

* 프라이버시가 중요한 경우​
* 비용 절감이 필요한 경우​
* 오프라인 환경​
* 빠른 프로토타이핑​

### 5-3. 에러 처리

```python
class SafeGuardrailMiddleware(AgentMiddleware):
    def after_agent(self, state, runtime):
        try:
            # Guardrail 로직
            ...
        except Exception as e:
            # 에러 발생 시 graceful degradation
            print(f"Guardrail 에러: {e}")
            return None  # 계속 진행
```

### 5-4. 성능 최적화

* Deterministic guardrails를 먼저 배치하여 빠른 차단​
* Model-based guardrails는 마지막에 배치​
* 경량 모델(gpt-4o-mini)을 안전성 체크에 사용​



