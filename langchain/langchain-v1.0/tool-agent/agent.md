# Agent

## 1. 핵심 포인트 <a href="#undefined" id="undefined"></a>

1. **`init_chat_model`** 사용으로 OpenAI와 Ollama를 통일된 방식으로 초기화​
2. \*\*`create_agent`\*\*가 v1.0의 표준 Agent 생성 방법​
3. **Middleware**를 통해 Agent의 모든 단계를 커스터마이즈 가능​
4. **`@tool` 데코레이터**로 간단하게 커스텀 도구 생성​
5. **TypedDict**로 커스텀 상태 정의 필수 (v1.0 요구사항)​
6. **ReAct 패턴**으로 추론과 행동을 반복하며 문제 해결​
7. 모든 패턴은 **LangGraph 기반**으로 persistence, streaming, human-in-the-loop 자동 지원​

### 패턴 1. Basic Agent (OpenAI) <a href="#id-1-basic-agent-openai" id="id-1-basic-agent-openai"></a>

가장 기본적인 Agent 패턴으로 `init_chat_model`과 `create_agent`를 사용합니다.​

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.tools import tool

# init_chat_model로 OpenAI 모델 초기화
model = init_chat_model(
    "openai:gpt-4o",  # 또는 "gpt-4o"만 입력해도 자동 추론
    temperature=0.7,
    max_tokens=1000
)

# 커스텀 도구 정의
@tool
def search_web(query: str) -> str:
    """웹 검색을 수행합니다. 최신 정보가 필요할 때 사용하세요."""
    # 실제 검색 API 호출
    return f"검색 결과: {query}에 대한 최신 정보..."

@tool
def calculator(expression: str) -> str:
    """수학 계산을 수행합니다. 예: '2 + 2' 또는 '10 * 5'"""
    try:
        result = eval(expression)
        return f"계산 결과: {result}"
    except Exception as e:
        return f"계산 오류: {str(e)}"

# Agent 생성
agent = create_agent(
    model=model,
    tools=[search_web, calculator],
    system_prompt="당신은 유능한 AI 어시스턴트입니다. 필요한 도구를 사용하여 정확한 답변을 제공하세요."
)

# Agent 실행
result = agent.invoke({
    "messages": [{"role": "user", "content": "2025년 AI 트렌드를 검색하고, 관련 기업 수를 10배로 계산해줘"}]
})

print(result["messages"][-1].content)
```

### 패턴 2: Basic Agent (Ollama) <a href="#id-2-basic-agent-ollama" id="id-2-basic-agent-ollama"></a>

Ollama를 사용한 로컬 LLM Agent 구현​

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.tools import tool

# init_chat_model로 Ollama 모델 초기화
model = init_chat_model(
    "ollama:llama3.2",  # 또는 "ollama:qwen2.5:7b"
    temperature=0.7,
    base_url="http://localhost:11434"  # Ollama 서버 주소
)

# 도구 정의
@tool
def get_weather(city: str) -> str:
    """특정 도시의 날씨를 가져옵니다."""
    weather_data = {
        "서울": "맑음, 15도",
        "부산": "흐림, 18도"
    }
    return weather_data.get(city, "날씨 정보 없음")

@tool
def translate_text(text: str, target_lang: str = "en") -> str:
    """텍스트를 번역합니다. target_lang: en, ko, ja"""
    # 간단한 번역 로직 (실제로는 번역 API 사용)
    return f"번역됨: {text} (언어: {target_lang})"

# Agent 생성
agent = create_agent(
    model=model,
    tools=[get_weather, translate_text],
    system_prompt="You are a helpful assistant. Use tools when needed."
)

# Agent 실행
result = agent.invoke({
    "messages": [{"role": "user", "content": "서울 날씨 알려주고 영어로 번역해줘"}]
})

print(result["messages"][-1].content)
```

### 패턴 3: Tool-Calling Agent with Multiple Tools <a href="#id-3-tool-calling-agent-with-multiple-tools" id="id-3-tool-calling-agent-with-multiple-tools"></a>

여러 도구를 활용하는 고급 Agent 패턴.​

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.tools import tool
from datetime import datetime
import json

# 모델 초기화 (OpenAI 또는 Ollama)
model = init_chat_model("openai:gpt-4o", temperature=0)
# model = init_chat_model("ollama:llama3.2", temperature=0)

# 복잡한 도구들 정의
@tool
def search_database(query: str, filters: dict = None) -> str:
    """데이터베이스를 검색합니다. query: 검색어, filters: 필터 조건(dict)"""
    # 실제 DB 쿼리
    results = [
        {"id": 1, "name": "Product A", "price": 100},
        {"id": 2, "name": "Product B", "price": 200}
    ]
    return json.dumps(results, ensure_ascii=False)

@tool
def get_current_time() -> str:
    """현재 시간을 반환합니다."""
    return datetime.now().strftime("%Y-%m-%d %H:%M:%S")

@tool
def send_notification(message: str, recipient: str) -> str:
    """알림을 전송합니다. message: 메시지 내용, recipient: 수신자"""
    # 실제 알림 전송 로직
    return f"알림 전송 완료: {recipient}에게 '{message}' 전송됨"

@tool
def analyze_data(data: str) -> str:
    """데이터를 분석하고 인사이트를 제공합니다."""
    # 간단한 분석 로직
    try:
        items = json.loads(data)
        total = sum(item.get('price', 0) for item in items)
        return f"총 {len(items)}개 항목, 총액: {total}원"
    except:
        return "데이터 분석 실패"

# Agent 생성
agent = create_agent(
    model=model,
    tools=[search_database, get_current_time, send_notification, analyze_data],
    system_prompt="""당신은 데이터 분석 전문가입니다. 
    사용자의 요청을 분석하고 필요한 도구들을 순서대로 사용하여 작업을 완료하세요.
    여러 도구를 조합하여 복잡한 작업을 수행할 수 있습니다."""
)

# 복잡한 작업 실행
result = agent.invoke({
    "messages": [{
        "role": "user", 
        "content": "데이터베이스에서 상품을 검색하고, 분석한 다음, 결과를 관리자에게 알림으로 보내줘"
    }]
})

print(result["messages"][-1].content)
```

### 패턴 4: Dynamic Model Selection Middleware <a href="#id-4-dynamic-model-selection-middleware" id="id-4-dynamic-model-selection-middleware"></a>

런타임에 모델을 동적으로 선택하는 패턴.​

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.agents.middleware import wrap_model_call, ModelRequest, ModelResponse
from langchain.tools import tool

# 다양한 모델 초기화
gpt_nano = init_chat_model("openai:gpt-4o-mini", temperature=0)
gpt_advanced = init_chat_model("openai:gpt-4o", temperature=0)
ollama_local = init_chat_model("ollama:llama3.2", temperature=0)

@tool
def complex_analysis(data: str) -> str:
    """복잡한 데이터 분석을 수행합니다."""
    return f"분석 완료: {data}"

# 동적 모델 선택 미들웨어
@wrap_model_call
def dynamic_model_selector(request: ModelRequest, handler) -> ModelResponse:
    """대화 복잡도와 메시지 수에 따라 모델을 동적으로 선택"""
    message_count = len(request.state["messages"])
    
    # 최근 메시지 내용 분석
    last_message = request.state["messages"][-1].content if request.state["messages"] else ""
    
    # 복잡한 작업이거나 긴 대화
    if message_count > 10 or "복잡한" in last_message or "분석" in last_message:
        print(f"[Middleware] 고급 모델 선택: GPT-4o (메시지 수: {message_count})")
        request.model = gpt_advanced
    # 간단한 작업은 로컬 모델 사용
    elif "간단한" in last_message or message_count < 3:
        print(f"[Middleware] 로컬 모델 선택: Llama3.2 (메시지 수: {message_count})")
        request.model = ollama_local
    # 기본적으로 경량 모델 사용
    else:
        print(f"[Middleware] 경량 모델 선택: GPT-4o-mini (메시지 수: {message_count})")
        request.model = gpt_nano
    
    return handler(request)

# Agent 생성
agent = create_agent(
    model=gpt_nano,  # 기본 모델
    tools=[complex_analysis],
    middleware=[dynamic_model_selector]
)

# 테스트
print("=== 간단한 질문 ===")
result1 = agent.invoke({
    "messages": [{"role": "user", "content": "간단한 질문: 안녕하세요?"}]
})

print("\n=== 복잡한 질문 ===")
result2 = agent.invoke({
    "messages": [{"role": "user", "content": "복잡한 데이터 분석이 필요한 작업입니다."}]
})
```

### 패턴 5: Custom State Agent <a href="#id-5-custom-state-agent" id="id-5-custom-state-agent"></a>

커스텀 상태를 관리하는 Agent 패턴.​

```python
from typing import TypedDict, Any
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent, AgentState
from langchain.agents.middleware import AgentMiddleware
from langchain.tools import tool, InjectedToolArg
from typing_extensions import Annotated

# 커스텀 상태 정의 (TypedDict 사용 필수)
class CustomState(AgentState):
    user_preferences: dict
    conversation_context: dict
    interaction_count: int

# 상태를 활용하는 도구
@tool
def personalized_search(
    query: str,
    state: Annotated[CustomState, InjectedToolArg]
) -> str:
    """사용자 선호도를 반영한 맞춤 검색"""
    preferences = state.get("user_preferences", {})
    style = preferences.get("style", "일반")
    return f"[{style} 스타일] {query}에 대한 검색 결과..."

@tool
def update_context(
    key: str, 
    value: str,
    state: Annotated[CustomState, InjectedToolArg]
) -> str:
    """대화 컨텍스트를 업데이트합니다."""
    context = state.get("conversation_context", {})
    context[key] = value
    return f"컨텍스트 업데이트 완료: {key}={value}"

# 상태 관리 미들웨어
class StateManagementMiddleware(AgentMiddleware):
    state_schema = CustomState
    tools = [personalized_search, update_context]
    
    def before_model(self, state: CustomState, runtime) -> dict[str, Any] | None:
        """모델 호출 전 상태 로깅"""
        count = state.get("interaction_count", 0)
        print(f"[Middleware] 상호작용 횟수: {count}")
        return {"interaction_count": count + 1}
    
    def after_model(self, state: CustomState, runtime) -> dict[str, Any] | None:
        """모델 응답 후 처리"""
        print(f"[Middleware] 현재 선호도: {state.get('user_preferences', {})}")
        return None

# Agent 생성
model = init_chat_model("openai:gpt-4o", temperature=0)
agent = create_agent(
    model=model,
    tools=[],
    middleware=[StateManagementMiddleware()],
    system_prompt="사용자 선호도를 고려하여 맞춤형 답변을 제공하세요."
)

# 상태와 함께 Agent 실행
result = agent.invoke({
    "messages": [{"role": "user", "content": "기술적인 설명으로 AI 트렌드를 검색해줘"}],
    "user_preferences": {"style": "기술적", "detail_level": "상세"},
    "conversation_context": {"topic": "AI"},
    "interaction_count": 0
})

print(result["messages"][-1].content)
```

### 패턴 6: Summarization Middleware <a href="#id-6-summarization-middleware" id="id-6-summarization-middleware"></a>

긴 대화를 자동으로 요약하는 패턴입니다.​

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.agents.middleware import SummarizationMiddleware
from langchain.tools import tool

model = init_chat_model("openai:gpt-4o", temperature=0)

@tool
def search_knowledge_base(topic: str) -> str:
    """지식 베이스에서 정보를 검색합니다."""
    return f"{topic}에 대한 상세 정보..."

# 요약 미들웨어 설정
summarization_middleware = SummarizationMiddleware(
    model="openai:gpt-4o-mini",  # 요약용 경량 모델
    max_tokens_before_summary=500,  # 500 토큰 초과 시 요약
    messages_to_keep=10,  # 최근 10개 메시지 유지
    summary_prompt="이전 대화를 핵심 내용 중심으로 간결하게 요약하세요."
)

# Agent 생성
agent = create_agent(
    model=model,
    tools=[search_knowledge_base],
    middleware=[summarization_middleware],
    system_prompt="당신은 상세한 설명을 제공하는 전문가입니다."
)

# 긴 대화 시뮬레이션
messages = [{"role": "user", "content": f"주제 {i}에 대해 설명해줘"} for i in range(15)]

for msg in messages:
    result = agent.invoke({"messages": [msg]})
    print(f"응답: {result['messages'][-1].content[:50]}...")
```

### 패턴 7: Human-in-the-Loop Agent <a href="#id-7-human-in-the-loop-agent" id="id-7-human-in-the-loop-agent"></a>

중요한 작업 실행 전 사람의 승인을 받는 패턴.​

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.agents.middleware import HumanInTheLoopMiddleware
from langchain.tools import tool
from langgraph.checkpoint.memory import MemorySaver

model = init_chat_model("openai:gpt-4o", temperature=0)

@tool
def send_email(recipient: str, subject: str, body: str) -> str:
    """이메일을 전송합니다."""
    # 실제 이메일 전송 로직
    return f"이메일 전송 완료: {recipient}에게 '{subject}' 전송"

@tool
def execute_payment(amount: int, account: str) -> str:
    """결제를 실행합니다."""
    # 실제 결제 로직
    return f"결제 완료: {account}에 {amount}원 전송"

@tool
def safe_search(query: str) -> str:
    """안전한 검색 (승인 불필요)"""
    return f"{query} 검색 결과..."

# Human-in-the-Loop 미들웨어 설정
hitl_middleware = HumanInTheLoopMiddleware(
    tool_configs={
        "send_email": {"require_approval": True},
        "execute_payment": {"require_approval": True}
    }
)

# Agent 생성 (checkpointer 필수)
checkpointer = MemorySaver()
agent = create_agent(
    model=model,
    tools=[send_email, execute_payment, safe_search],
    middleware=[hitl_middleware],
    checkpointer=checkpointer
)

# Agent 실행
thread_id = "user-123"
config = {"configurable": {"thread_id": thread_id}}

# 민감한 작업 요청
result = agent.invoke({
    "messages": [{"role": "user", "content": "john@example.com에게 '안녕하세요' 제목으로 이메일 보내줘"}]
}, config=config)

# 중단된 경우 (승인 대기 중)
if result.get("next"):
    print("승인 대기 중...")
    # 승인 후 재개
    result = agent.invoke(None, config=config)
```

### 패턴 8: Structured Output Agent <a href="#id-8-structured-output-agent" id="id-8-structured-output-agent"></a>

특정 형식의 출력을 생성하는 패턴.​

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.agents.structured_output import ToolStrategy, ProviderStrategy
from langchain.tools import tool
from pydantic import BaseModel, Field

model = init_chat_model("openai:gpt-4o", temperature=0)

# 출력 스키마 정의
class ContactInfo(BaseModel):
    """연락처 정보"""
    name: str = Field(description="이름")
    email: str = Field(description="이메일 주소")
    phone: str = Field(description="전화번호")
    company: str = Field(description="회사명")

class ProductAnalysis(BaseModel):
    """제품 분석 결과"""
    product_name: str = Field(description="제품명")
    price: float = Field(description="가격")
    rating: float = Field(description="평점 (1-5)")
    pros: list[str] = Field(description="장점 목록")
    cons: list[str] = Field(description="단점 목록")

@tool
def search_product(query: str) -> str:
    """제품 정보를 검색합니다."""
    return """
    제품명: 노트북 A
    가격: 1500000원
    평점: 4.5/5
    장점: 고성능, 경량, 긴 배터리 수명
    단점: 높은 가격, 제한된 포트
    """

# ToolStrategy 사용 (모든 tool-calling 모델에서 동작)
agent_tool_strategy = create_agent(
    model=model,
    tools=[search_product],
    response_format=ToolStrategy(ProductAnalysis)
)

result = agent_tool_strategy.invoke({
    "messages": [{"role": "user", "content": "노트북 A를 검색하고 분석해줘"}]
})

# 구조화된 출력 접근
structured_output = result["structured_response"]
print(f"제품: {structured_output.product_name}")
print(f"가격: {structured_output.price}원")
print(f"평점: {structured_output.rating}/5")
print(f"장점: {', '.join(structured_output.pros)}")

# ProviderStrategy 사용 (OpenAI 네이티브 지원)
agent_provider_strategy = create_agent(
    model=model,
    tools=[],
    response_format=ProviderStrategy(ContactInfo)
)

result2 = agent_provider_strategy.invoke({
    "messages": [{
        "role": "user", 
        "content": "이름: 홍길동, 이메일: hong@example.com, 전화: 010-1234-5678, 회사: ABC Corp"
    }]
})

contact = result2["structured_response"]
print(f"\n연락처: {contact.name}, {contact.email}, {contact.phone}, {contact.company}")
```

### 패턴 비교표 <a href="#undefined" id="undefined"></a>

| 패턴                | OpenAI 지원 | Ollama 지원 | 주요 용도      | 복잡도 |
| ----------------- | --------- | --------- | ---------- | --- |
| Basic Agent       | ✅         | ✅         | 간단한 작업 자동화 | ⭐   |
| Tool-calling      | ✅         | ✅         | 다중 도구 활용   | ⭐⭐  |
| Middleware        | ✅         | ✅         | 실행 흐름 제어   | ⭐⭐⭐ |
| Dynamic Model     | ✅         | ✅         | 비용 최적화     | ⭐⭐⭐ |
| Custom State      | ✅         | ✅         | 상태 관리      | ⭐⭐⭐ |
| Summarization     | ✅         | ✅         | 긴 대화 관리    | ⭐⭐  |
| Human-in-the-Loop | ✅         | ✅         | 승인 워크플로우   | ⭐⭐⭐ |
| Structured Output | ✅         | ⚠️        | 데이터 추출     | ⭐⭐  |

**참고**: Ollama의 Structured Output 지원은 모델에 따라 다름

### &#x20;<a href="#undefined" id="undefined"></a>
