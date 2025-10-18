# Agent init\_chat\_model

### 1. 예제 <a href="#id-1--agent---initchatmodel" id="id-1--agent---initchatmodel"></a>

### 1-1. 기본 Agent - init\_chat\_model 사용 <a href="#id-1--agent---initchatmodel" id="id-1--agent---initchatmodel"></a>

#### 1-1-1. Ollama 버전

```python
from langchain.chat_models import init_chat_model
from langchain.tools import tool
from langchain.agents import create_agent

# init_chat_model로 Ollama 모델 초기화
llm = init_chat_model(
    model="llama3.1:8b",
    model_provider="ollama",
    temperature=0.7,
    base_url="http://localhost:11434"  # 로컬 서버
)

# 원격 Ollama 서버 사용 시
llm_remote = init_chat_model(
    model="llama3.1:8b",
    model_provider="ollama",
    temperature=0.7,
    base_url="http://192.168.1.100:11434"  # 원격 서버 IP:포트
)

@tool
def get_weather(city: str) -> str:
    """특정 도시의 날씨 정보를 가져옵니다."""
    return f"{city}의 날씨는 맑고 온도는 22도입니다!"

# Agent 생성
agent = create_agent(
    model=llm,
    tools=[get_weather],
    system_prompt="당신은 친절한 날씨 정보 제공 비서입니다."
)

# 실행
result = agent.invoke({
    "messages": [{"role": "user", "content": "서울 날씨 알려줘"}]
})

print(result["messages"][-1].content)
```

#### 1-1-2. OpenAI 버전

```python
from langchain.chat_models import init_chat_model
from langchain.tools import tool
from langchain.agents import create_agent
from dotenv import load_dotenv
import os

# 환경 변수 로드
load_dotenv()

# init_chat_model로 OpenAI 모델 초기화
llm = init_chat_model(
    model="gpt-4o",
    model_provider="openai",
    temperature=0.7,
    api_key=os.getenv("OPENAI_API_KEY")
)

# 또는 model_provider 생략 (자동 추론)
llm = init_chat_model(
    "gpt-4o",  # gpt- 접두사로 openai 자동 인식
    temperature=0.7
)

@tool
def get_weather(city: str) -> str:
    """특정 도시의 날씨 정보를 가져옵니다."""
    return f"{city}의 날씨는 맑고 온도는 22도입니다!"

agent = create_agent(
    model=llm,
    tools=[get_weather],
    system_prompt="당신은 친절한 날씨 정보 제공 비서입니다."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "서울 날씨 알려줘"}]
})

print(result["messages"][-1].content)
```

***

### 1-2. 다중 Tool Agent <a href="#id-2--tool-agent" id="id-2--tool-agent"></a>

#### 1-2-1. Ollama 버전

```python
from langchain.chat_models import init_chat_model
from langchain.tools import tool
from langchain.agents import create_agent

llm = init_chat_model(
    "llama3.1:8b",
    model_provider="ollama",
    temperature=0,
    base_url="http://localhost:11434"
)

@tool
def search(query: str) -> str:
    """정보를 검색합니다."""
    return f"'{query}'에 대한 검색 결과입니다."

@tool
def calculate(expression: str) -> str:
    """수식을 계산합니다."""
    try:
        result = eval(expression)
        return str(result)
    except Exception as e:
        return f"계산 오류: {str(e)}"

@tool
def get_weather(location: str) -> str:
    """특정 위치의 날씨를 조회합니다."""
    return f"{location}의 날씨: 맑음, 기온 20°C"

agent = create_agent(
    model=llm,
    tools=[search, calculate, get_weather],
    system_prompt="당신은 다양한 도구를 활용할 수 있는 유용한 비서입니다."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "15 곱하기 7은 얼마야?"}]
})

print(result["messages"][-1].content)
```

#### 1-2-2. OpenAI 버전

```python
from langchain.chat_models import init_chat_model
from langchain.tools import tool
from langchain.agents import create_agent
import os
from dotenv import load_dotenv

load_dotenv()

llm = init_chat_model(
    "gpt-4o-mini",  # 비용 절감 모델
    temperature=0
)

@tool
def search(query: str) -> str:
    """정보를 검색합니다."""
    return f"'{query}'에 대한 검색 결과입니다."

@tool
def calculate(expression: str) -> str:
    """수식을 계산합니다."""
    try:
        result = eval(expression)
        return str(result)
    except Exception as e:
        return f"계산 오류: {str(e)}"

@tool
def get_weather(location: str) -> str:
    """특정 위치의 날씨를 조회합니다."""
    return f"{location}의 날씨: 맑음, 기온 20°C"

agent = create_agent(
    model=llm,
    tools=[search, calculate, get_weather],
    system_prompt="당신은 다양한 도구를 활용할 수 있는 유용한 비서입니다."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "15 곱하기 7은 얼마야?"}]
})

print(result["messages"][-1].content)
```

***

### 1-3. Custom State Schema with Middleware <a href="#id-3-custom-state-schema-with-middleware" id="id-3-custom-state-schema-with-middleware"></a>

#### 1-3-1. Ollama 버전

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent, AgentState
from langchain.agents.middleware import AgentMiddleware
from langchain.tools import tool
from typing import Any
from typing_extensions import NotRequired

llm = init_chat_model(
    "qwen2:7b-instruct",
    model_provider="ollama",
    base_url="http://localhost:11434"
)

class CustomState(AgentState):
    user_name: NotRequired[str]
    conversation_count: NotRequired[int]

class ConversationCounterMiddleware(AgentMiddleware[CustomState]):
    state_schema = CustomState
    
    def before_model(self, state: CustomState, runtime) -> dict[str, Any] | None:
        count = state.get("conversation_count", 0)
        print(f"현재 대화 횟수: {count}")
        
        if count > 5:
            print("대화 횟수 제한 도달")
            return {"jump_to": "end"}
        return None
    
    def after_model(self, state: CustomState, runtime) -> dict[str, Any] | None:
        current_count = state.get("conversation_count", 0)
        return {"conversation_count": current_count + 1}

@tool
def greeting_tool(name: str) -> str:
    """사용자에게 인사합니다."""
    return f"안녕하세요, {name}님!"

agent = create_agent(
    model=llm,
    tools=[greeting_tool],
    middleware=[ConversationCounterMiddleware()],
    system_prompt="당신은 친절한 대화 상대입니다."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "안녕"}],
    "user_name": "홍길동",
    "conversation_count": 0
})

print(result["messages"][-1].content)
```

#### 1-3-2. OpenAI 버전

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent, AgentState
from langchain.agents.middleware import AgentMiddleware
from langchain.tools import tool
from typing import Any
from typing_extensions import NotRequired
import os
from dotenv import load_dotenv

load_dotenv()

llm = init_chat_model(
    "gpt-4o",
    temperature=0.7
)

class CustomState(AgentState):
    user_name: NotRequired[str]
    conversation_count: NotRequired[int]

class ConversationCounterMiddleware(AgentMiddleware[CustomState]):
    state_schema = CustomState
    
    def before_model(self, state: CustomState, runtime) -> dict[str, Any] | None:
        count = state.get("conversation_count", 0)
        print(f"현재 대화 횟수: {count}")
        
        if count > 5:
            print("대화 횟수 제한 도달")
            return {"jump_to": "end"}
        return None
    
    def after_model(self, state: CustomState, runtime) -> dict[str, Any] | None:
        current_count = state.get("conversation_count", 0)
        return {"conversation_count": current_count + 1}

@tool
def greeting_tool(name: str) -> str:
    """사용자에게 인사합니다."""
    return f"안녕하세요, {name}님!"

agent = create_agent(
    model=llm,
    tools=[greeting_tool],
    middleware=[ConversationCounterMiddleware()],
    system_prompt="당신은 친절한 대화 상대입니다."
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "안녕"}],
    "user_name": "홍길동",
    "conversation_count": 0
})

print(result["messages"][-1].content)
```

***

### 1-4. Dynamic Prompt Middleware <a href="#id-4-dynamic-prompt-middleware" id="id-4-dynamic-prompt-middleware"></a>

#### 1-4-1. Ollama 버전

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.agents.middleware import dynamic_prompt, ModelRequest
from langchain.tools import tool
from dataclasses import dataclass

llm = init_chat_model(
    "llama3.1:8b",
    model_provider="ollama",
    temperature=0.7,
    base_url="http://localhost:11434"
)

@dataclass
class Context:
    user_role: str
    user_name: str

@dynamic_prompt
def personalized_prompt(request: ModelRequest) -> str:
    """사용자 역할에 따라 시스템 프롬프트를 동적으로 생성"""
    user_role = request.runtime.context.user_role
    user_name = request.runtime.context.user_name
    
    base_prompt = f"당신은 {user_name}님을 돕는 AI 비서입니다."
    
    if user_role == "expert":
        return f"{base_prompt} 전문적이고 기술적인 답변을 제공하세요."
    elif user_role == "beginner":
        return f"{base_prompt} 쉽고 친절하게 설명해주세요."
    else:
        return base_prompt

@tool
def analyze_data(data: str) -> str:
    """데이터를 분석합니다."""
    return f"'{data}' 데이터 분석 완료"

agent = create_agent(
    model=llm,
    tools=[analyze_data],
    middleware=[personalized_prompt],
    context_schema=Context
)

# Expert 사용자로 실행
result = agent.invoke(
    {"messages": [{"role": "user", "content": "머신러닝 설명해줘"}]},
    context=Context(user_role="expert", user_name="김철수")
)

print(result["messages"][-1].content)
```

#### 1-4-2. OpenAI 버전

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.agents.middleware import dynamic_prompt, ModelRequest
from langchain.tools import tool
from dataclasses import dataclass
import os
from dotenv import load_dotenv

load_dotenv()

llm = init_chat_model(
    "gpt-4o",
    temperature=0.7
)

@dataclass
class Context:
    user_role: str
    user_name: str

@dynamic_prompt
def personalized_prompt(request: ModelRequest) -> str:
    """사용자 역할에 따라 시스템 프롬프트를 동적으로 생성"""
    user_role = request.runtime.context.user_role
    user_name = request.runtime.context.user_name
    
    base_prompt = f"당신은 {user_name}님을 돕는 AI 비서입니다."
    
    if user_role == "expert":
        return f"{base_prompt} 전문적이고 기술적인 답변을 제공하세요."
    elif user_role == "beginner":
        return f"{base_prompt} 쉽고 친절하게 설명해주세요."
    else:
        return base_prompt

@tool
def analyze_data(data: str) -> str:
    """데이터를 분석합니다."""
    return f"'{data}' 데이터 분석 완료"

agent = create_agent(
    model=llm,
    tools=[analyze_data],
    middleware=[personalized_prompt],
    context_schema=Context
)

# Beginner 사용자로 실행
result = agent.invoke(
    {"messages": [{"role": "user", "content": "머신러닝 설명해줘"}]},
    context=Context(user_role="beginner", user_name="이영희")
)

print(result["messages"][-1].content)
```

***

### 1-5. ToolRuntime으로 Context 접근 <a href="#id-5-toolruntime-context" id="id-5-toolruntime-context"></a>

#### 1-5-1. Ollama 버전

```python
from langchain.chat_models import init_chat_model
from dataclasses import dataclass
from langchain.tools import tool, ToolRuntime
from langchain.agents import create_agent

llm = init_chat_model(
    "llama3.1:8b",
    model_provider="ollama",
    base_url="http://localhost:11434"
)

@dataclass
class Context:
    user_id: str
    api_key: str

@tool
def fetch_user_data(runtime: ToolRuntime[Context]) -> str:
    """사용자 데이터를 조회합니다."""
    user_id = runtime.context.user_id
    api_key = runtime.context.api_key
    
    return f"사용자 {user_id}의 데이터를 조회했습니다. (API 키: {api_key[:5]}...)"

@tool
def write_to_memory(data: str, runtime: ToolRuntime[Context]) -> str:
    """장기 메모리에 데이터를 저장합니다."""
    if runtime.store:
        user_id = runtime.context.user_id
        runtime.store.put(("user_data",), user_id, {"data": data})
        return f"데이터가 저장되었습니다."
    return "Store를 사용할 수 없습니다."

agent = create_agent(
    model=llm,
    tools=[fetch_user_data, write_to_memory],
    context_schema=Context
)

result = agent.invoke(
    {"messages": [{"role": "user", "content": "내 데이터 조회해줘"}]},
    context=Context(user_id="user_123", api_key="sk-abc123def456")
)

print(result["messages"][-1].content)
```

#### 1-5-2. OpenAI 버전

```python
from langchain.chat_models import init_chat_model
from dataclasses import dataclass
from langchain.tools import tool, ToolRuntime
from langchain.agents import create_agent
import os
from dotenv import load_dotenv

load_dotenv()

llm = init_chat_model(
    "gpt-4o",
    temperature=0
)

@dataclass
class Context:
    user_id: str
    api_key: str

@tool
def fetch_user_data(runtime: ToolRuntime[Context]) -> str:
    """사용자 데이터를 조회합니다."""
    user_id = runtime.context.user_id
    api_key = runtime.context.api_key
    
    return f"사용자 {user_id}의 데이터를 조회했습니다. (API 키: {api_key[:5]}...)"

@tool
def write_to_memory(data: str, runtime: ToolRuntime[Context]) -> str:
    """장기 메모리에 데이터를 저장합니다."""
    if runtime.store:
        user_id = runtime.context.user_id
        runtime.store.put(("user_data",), user_id, {"data": data})
        return f"데이터가 저장되었습니다."
    return "Store를 사용할 수 없습니다."

agent = create_agent(
    model=llm,
    tools=[fetch_user_data, write_to_memory],
    context_schema=Context
)

result = agent.invoke(
    {"messages": [{"role": "user", "content": "내 데이터 조회해줘"}]},
    context=Context(user_id="user_123", api_key="ollama-key-789")
)

print(result["messages"][-1].content)
```

***

### 1-6. Streaming <a href="#id-6-streaming" id="id-6-streaming"></a>

#### 1-6-1. Ollama 버전

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.tools import tool

llm = init_chat_model(
    "llama3.1:8b",
    model_provider="ollama",
    temperature=0.7,
    base_url="http://localhost:11434"
)

@tool
def get_news(topic: str) -> str:
    """특정 주제의 뉴스를 가져옵니다."""
    return f"{topic}에 관한 최신 뉴스: AI 기술 발전, 새로운 모델 출시 등"

agent = create_agent(
    model=llm,
    tools=[get_news],
    system_prompt="당신은 뉴스 요약 전문가입니다."
)

# Streaming 방식으로 실행
for chunk in agent.stream(
    {"messages": [{"role": "user", "content": "AI 뉴스 검색하고 요약해줘"}]},
    stream_mode="values"
):
    latest_message = chunk["messages"][-1]
    
    if latest_message.content:
        print(latest_message.content, end="", flush=True)
    elif hasattr(latest_message, 'tool_calls') and latest_message.tool_calls:
        tool_names = [tc['name'] for tc in latest_message.tool_calls]
        print(f"\n[Tool 호출 중: {tool_names}]\n")

print("\n\n=== 완료 ===")
```

#### 1-6-2. OpenAI 버전

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.tools import tool
import os
from dotenv import load_dotenv

load_dotenv()

llm = init_chat_model(
    "gpt-4o",
    temperature=0.7
)

@tool
def get_news(topic: str) -> str:
    """특정 주제의 뉴스를 가져옵니다."""
    return f"{topic}에 관한 최신 뉴스: AI 기술 발전, 새로운 모델 출시 등"

agent = create_agent(
    model=llm,
    tools=[get_news],
    system_prompt="당신은 뉴스 요약 전문가입니다."
)

# Streaming 방식으로 실행
for chunk in agent.stream(
    {"messages": [{"role": "user", "content": "AI 뉴스 검색하고 요약해줘"}]},
    stream_mode="values"
):
    latest_message = chunk["messages"][-1]
    
    if latest_message.content:
        print(latest_message.content, end="", flush=True)
    elif hasattr(latest_message, 'tool_calls') and latest_message.tool_calls:
        tool_names = [tc['name'] for tc in latest_message.tool_calls]
        print(f"\n[Tool 호출 중: {tool_names}]\n")

print("\n\n=== 완료 ===")
```

***

### 1-7. Checkpointer를 이용한 메모리 관리 <a href="#id-7-checkpointer" id="id-7-checkpointer"></a>

#### 1-7-1. Ollama 버전

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.tools import tool
from langgraph.checkpoint.memory import InMemorySaver

llm = init_chat_model(
    "llama3.1:8b",
    model_provider="ollama",
    base_url="http://localhost:11434"
)

@tool
def calculate_sum(numbers: list) -> int:
    """숫자 리스트의 합을 계산합니다."""
    return sum(numbers)

# Checkpointer와 함께 agent 생성
agent = create_agent(
    model=llm,
    tools=[calculate_sum],
    checkpointer=InMemorySaver(),
    system_prompt="당신은 수학 계산을 돕는 비서입니다."
)

# Thread ID를 사용하여 대화 세션 관리
thread_config = {"configurable": {"thread_id": "user_conversation_1"}}

# 첫 번째 메시지
result1 = agent.invoke(
    {"messages": [{"role": "user", "content": "내 이름은 김철수야"}]},
    thread_config
)

print("첫 번째 응답:", result1["messages"][-1].content)

# 두 번째 메시지 - 이전 대화를 기억함
result2 = agent.invoke(
    {"messages": [{"role": "user", "content": "내 이름이 뭐였지?"}]},
    thread_config
)

print("두 번째 응답:", result2["messages"][-1].content)
```

#### 1-7-2. OpenAI 버전

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.tools import tool
from langgraph.checkpoint.memory import InMemorySaver
import os
from dotenv import load_dotenv

load_dotenv()

llm = init_chat_model(
    "gpt-4o-mini",
    temperature=0
)

@tool
def calculate_sum(numbers: list) -> int:
    """숫자 리스트의 합을 계산합니다."""
    return sum(numbers)

# Checkpointer와 함께 agent 생성
agent = create_agent(
    model=llm,
    tools=[calculate_sum],
    checkpointer=InMemorySaver(),
    system_prompt="당신은 수학 계산을 돕는 비서입니다."
)

# Thread ID를 사용하여 대화 세션 관리
thread_config = {"configurable": {"thread_id": "user_conversation_1"}}

# 첫 번째 메시지
result1 = agent.invoke(
    {"messages": [{"role": "user", "content": "내 이름은 김철수야"}]},
    thread_config
)

print("첫 번째 응답:", result1["messages"][-1].content)

# 두 번째 메시지 - 이전 대화를 기억함
result2 = agent.invoke(
    {"messages": [{"role": "user", "content": "내 이름이 뭐였지?"}]},
    thread_config
)

print("두 번째 응답:", result2["messages"][-1].content)
```

***

### 1-8: 런타임 모델 전환 (Configurable Model) <a href="#id-8----configurable-model" id="id-8----configurable-model"></a>

**동적으로 모델을 전환**할 수 있는 강력한 기능​

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.tools import tool
import os
from dotenv import load_dotenv

load_dotenv()

# 기본값 없이 configurable 모델 생성
llm = init_chat_model(
    temperature=0,
    # model을 지정하지 않으면 자동으로 configurable
)

@tool
def search_info(query: str) -> str:
    """정보를 검색합니다."""
    return f"'{query}'에 대한 검색 결과입니다."

agent = create_agent(
    model=llm,
    tools=[search_info],
    system_prompt="당신은 유용한 검색 비서입니다."
)

# OpenAI로 실행
result_openai = agent.invoke(
    {"messages": [{"role": "user", "content": "파이썬이란?"}]},
    config={"configurable": {"model": "gpt-4o"}}
)

print("OpenAI 응답:", result_openai["messages"][-1].content)

# Ollama로 실행 (같은 agent 객체 사용)
result_ollama = agent.invoke(
    {"messages": [{"role": "user", "content": "자바란?"}]},
    config={"configurable": {
        "model": "llama3.1:8b",
        "model_provider": "ollama",
        "base_url": "http://localhost:11434"
    }}
)

print("Ollama 응답:", result_ollama["messages"][-1].content)
```

***

### 1-9. 기본값이 있는 Configurable Model <a href="#id-9---configurable-model" id="id-9---configurable-model"></a>

```python
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent
from langchain.tools import tool
import os
from dotenv import load_dotenv

load_dotenv()

# 기본값은 OpenAI, 런타임에 변경 가능
llm = init_chat_model(
    model="gpt-4o",
    temperature=0,
    configurable_fields=("model", "model_provider", "temperature", "max_tokens"),
    config_prefix="llm"  # 여러 모델 사용 시 prefix로 구분
)

@tool
def translate(text: str, target_lang: str) -> str:
    """텍스트를 목표 언어로 번역합니다."""
    return f"'{text}'를 {target_lang}로 번역: (번역된 텍스트)"

agent = create_agent(
    model=llm,
    tools=[translate],
    system_prompt="당신은 번역 전문가입니다."
)

# 기본 모델(GPT-4o)로 실행
result1 = agent.invoke({
    "messages": [{"role": "user", "content": "Hello를 한국어로 번역해줘"}]
})

print("GPT-4o 응답:", result1["messages"][-1].content)

# 런타임에 Ollama로 변경
result2 = agent.invoke(
    {"messages": [{"role": "user", "content": "Goodbye를 한국어로 번역해줘"}]},
    config={
        "configurable": {
            "llm_model": "llama3.1:8b",
            "llm_model_provider": "ollama",
            "llm_temperature": 0.5,
            "llm_base_url": "http://localhost:11434"
        }
    }
)

print("Ollama 응답:", result2["messages"][-1].content)
```

***

### 1-10: Multi-Agent with init\_chat\_model <a href="#id-10-multi-agent-with-initchatmodel" id="id-10-multi-agent-with-initchatmodel"></a>

```python
from langchain.chat_models import init_chat_model
from langchain.tools import tool
from langchain.agents import create_agent

# 서브 에이전트 1: Ollama 사용
data_analysis_agent = create_agent(
    model=init_chat_model(
        "llama3.1:8b",
        model_provider="ollama",
        base_url="http://localhost:11434"
    ),
    tools=[],
    system_prompt="당신은 데이터 분석 전문가입니다."
)

# 서브 에이전트 2: OpenAI 사용
report_writing_agent = create_agent(
    model=init_chat_model("gpt-4o", temperature=0.7),
    tools=[],
    system_prompt="당신은 보고서 작성 전문가입니다."
)

# 서브 에이전트를 tool로 감싸기
@tool(
    name="analyze_data_tool",
    description="데이터 분석을 수행합니다."
)
def call_data_analyst(query: str) -> str:
    result = data_analysis_agent.invoke({
        "messages": [{"role": "user", "content": query}]
    })
    return result["messages"][-1].content

@tool(
    name="write_report_tool",
    description="분석 결과를 기반으로 보고서를 작성합니다."
)
def call_report_writer(analysis_result: str) -> str:
    result = report_writing_agent.invoke({
        "messages": [{"role": "user", "content": f"다음 분석 결과로 보고서 작성: {analysis_result}"}]
    })
    return result["messages"][-1].content

# 메인 컨트롤러 에이전트
main_agent = create_agent(
    model=init_chat_model("gpt-4o"),
    tools=[call_data_analyst, call_report_writer],
    system_prompt="당신은 작업을 관리하는 컨트롤러입니다. 필요에 따라 전문가들에게 작업을 위임하세요."
)

# 실행
result = main_agent.invoke({
    "messages": [{"role": "user", "content": "매출 데이터 분석하고 보고서 작성해줘"}]
})

print(result["messages"][-1].content)
```

***

