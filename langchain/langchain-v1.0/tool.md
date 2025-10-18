# Tool

## 1. Tool Calling 개념 <a href="#id-1-tool-calling" id="id-1-tool-calling"></a>

**Tool Calling**은 LLM이 특정 기능(함수)를 호출할 수 있도록 하는 기술로 모델이 직접 함수를 실행하는 것이 아니라, 함수 호출에 필요한 인자를 생성하고, 실제 실행은 사용자가 처리한다.

### 핵심 구성요소

* **Tool Schema**: 함수의 이름, 설명, 인자 정의​
* **bind\_tools()**: Tool을 모델에 바인딩하는 메서드​
* **tool\_calls**: 모델이 생성한 Tool 호출 정보​
* **ToolMessage**: Tool 실행 결과를 담는 메시지​

## 2. Tool 생성 방법 <a href="#id-2-tool" id="id-2-tool"></a>

### 2-1. @tool 데코레이터 (권장)

가장 간단하고 권장되는 방법.​

```python
from langchain_core.tools import tool
from typing import List

@tool
def add(a: int, b: int) -> int:
    """두 정수를 더합니다.
    
    Args:
        a: 첫 번째 정수
        b: 두 번째 정수
    """
    return a + b

@tool
def multiply(a: int, b: int) -> int:
    """두 정수를 곱합니다.
    
    Args:
        a: 첫 번째 정수
        b: 두 번째 정수
    """
    return a * b

@tool
def search_database(query: str, limit: int = 5) -> List[dict]:
    """데이터베이스에서 정보를 검색합니다.
    
    Args:
        query: 검색 쿼리
        limit: 반환할 최대 결과 수
    """
    return [{"id": 1, "title": f"Result for {query}"}]
```

### 2-2. Pydantic 스키마 활용

복잡한 Tool은 Pydantic으로 스키마를 명시적으로 정의할 수 있다.​

```python
from pydantic import BaseModel, Field
from typing import Optional, List

class WeatherInput(BaseModel):
    """날씨 조회 입력 스키마"""
    city: str = Field(description="도시 이름 (예: Seoul, Busan)")
    country: str = Field(default="KR", description="국가 코드")
    units: str = Field(default="metric", description="온도 단위")

@tool("weather-lookup", args_schema=WeatherInput)
def get_weather(city: str, country: str = "KR", units: str = "metric") -> dict:
    """지정된 도시의 현재 날씨 정보를 조회합니다."""
    return {
        "city": city,
        "temperature": 22,
        "condition": "맑음",
        "humidity": 65
    }
```

### 2-3. StructuredTool &#x20;

동기/비동기 버전을 모두 제공할 때 유용.​

```python
from langchain_core.tools import StructuredTool

def calculate_statistics(numbers: List[float], operation: str) -> dict:
    """숫자 리스트의 통계를 계산합니다."""
    if operation == "mean":
        return {"result": sum(numbers) / len(numbers)}
    elif operation == "sum":
        return {"result": sum(numbers)}
    return {"error": "Unknown operation"}

async def acalculate_statistics(numbers: List[float], operation: str) -> dict:
    return calculate_statistics(numbers, operation)

class StatisticsInput(BaseModel):
    numbers: List[float] = Field(description="숫자 리스트")
    operation: str = Field(description="수행할 연산 (mean, sum, max, min)")

statistics_tool = StructuredTool.from_function(
    func=calculate_statistics,
    coroutine=acalculate_statistics,
    name="statistics_calculator",
    description="숫자 리스트의 통계를 계산합니다",
    args_schema=StatisticsInput
)
```

## 3. OpenAI를 사용한 Tool Calling <a href="#id-3-openai--tool-calling" id="id-3-openai--tool-calling"></a>

### 3-1.  기본 구현

```python
import os
from langchain.chat_models import init_chat_model
from langchain_core.tools import tool
from langchain_core.messages import HumanMessage, SystemMessage

os.environ["OPENAI_API_KEY"] = "your-api-key"

@tool
def calculator(operation: str, a: float, b: float) -> float:
    """간단한 계산을 수행합니다.
    
    Args:
        operation: 연산 종류 (add, subtract, multiply, divide)
        a: 첫 번째 숫자
        b: 두 번째 숫자
    """
    operations = {
        "add": lambda: a + b,
        "subtract": lambda: a - b,
        "multiply": lambda: a * b,
        "divide": lambda: a / b if b != 0 else "Error"
    }
    return operations.get(operation, lambda: "Unknown")()

# init_chat_model로 OpenAI 초기화
llm = init_chat_model(
    "gpt-4o-mini",
    model_provider="openai",
    temperature=0
)

# Tool 바인딩
llm_with_tools = llm.bind_tools([calculator])

# Tool 호출
messages = [
    SystemMessage(content="당신은 수학 계산을 도와주는 AI입니다."),
    HumanMessage(content="15 곱하기 24는 얼마인가요?")
]

response = llm_with_tools.invoke(messages)
print("Tool calls:", response.tool_calls)

# Tool 실행
if response.tool_calls:
    for tool_call in response.tool_calls:
        result = calculator.invoke(tool_call["args"])
        print(f"결과: {result}")
```

## 4. Ollama를 사용한 Tool Calling <a href="#id-4-ollama--tool-calling" id="id-4-ollama--tool-calling"></a>

Ollama는 Tool Calling을 지원하는 모델(llama3.1, mistral, qwen 등)을 사용해야 한다.​

### 4-1. 기본 구현

```python
from langchain.chat_models import init_chat_model
from langchain_core.tools import tool
from langchain_core.messages import HumanMessage, SystemMessage, ToolMessage

@tool
def get_current_time(timezone: str = "Asia/Seoul") -> str:
    """현재 시간을 조회합니다.
    
    Args:
        timezone: 타임존
    """
    from datetime import datetime
    import pytz
    
    tz = pytz.timezone(timezone)
    return datetime.now(tz).strftime("%Y-%m-%d %H:%M:%S %Z")

@tool
def search_info(topic: str) -> str:
    """주제에 대한 정보를 검색합니다."""
    info_db = {
        "python": "Python은 고급 프로그래밍 언어입니다.",
        "langchain": "LangChain은 LLM 애플리케이션 프레임워크입니다.",
    }
    return info_db.get(topic.lower(), f"{topic}에 대한 정보 없음")

# Ollama 모델 초기화
llm = init_chat_model(
    model="llama3.1",
    model_provider="ollama",
    temperature=0
)

llm_with_tools = llm.bind_tools([get_current_time, search_info])

messages = [
    SystemMessage(content="당신은 유용한 AI 어시스턴트입니다."),
    HumanMessage(content="서울의 현재 시간은?")
]

response = llm_with_tools.invoke(messages)

# Tool 실행 및 결과 반영
if response.tool_calls:
    messages.append(response)
    
    for tool_call in response.tool_calls:
        if tool_call["name"] == "get_current_time":
            result = get_current_time.invoke(tool_call["args"])
        else:
            result = search_info.invoke(tool_call["args"])
        
        messages.append(
            ToolMessage(
                content=str(result),
                tool_call_id=tool_call["id"]
            )
        )
    
    final_response = llm_with_tools.invoke(messages)
    print("최종 응답:", final_response.content)
```

## 5. Agent 패턴 구현 <a href="#id-5-agent" id="id-5-agent"></a>

Tool을 자동으로 실행하는 반복 루프를 구현한 Agent 패턴.​

```python
from langchain.chat_models import init_chat_model
from langchain_core.tools import tool
from langchain_core.messages import HumanMessage, SystemMessage, ToolMessage

@tool
def add(a: int, b: int) -> int:
    """두 수를 더합니다."""
    return a + b

@tool
def multiply(a: int, b: int) -> int:
    """두 수를 곱합니다."""
    return a * b

tools = [add, multiply]
tools_by_name = {tool.name: tool for tool in tools}

llm = init_chat_model("gpt-4o-mini", model_provider="openai", temperature=0)
llm_with_tools = llm.bind_tools(tools)

def run_agent(query: str, max_iterations: int = 5) -> str:
    """Tool을 사용하는 Agent 실행"""
    
    messages = [
        SystemMessage(content="당신은 유용한 AI입니다. 필요시 도구를 사용하세요."),
        HumanMessage(content=query)
    ]
    
    for iteration in range(max_iterations):
        print(f"\n--- Iteration {iteration + 1} ---")
        
        response = llm_with_tools.invoke(messages)
        messages.append(response)
        
        # Tool call이 없으면 종료
        if not response.tool_calls:
            print("최종 응답:", response.content)
            return response.content
        
        # Tool 실행
        for tool_call in response.tool_calls:
            tool_name = tool_call["name"]
            tool_args = tool_call["args"]
            
            tool = tools_by_name[tool_name]
            result = tool.invoke(tool_args)
            
            print(f"{tool_name}({tool_args}) = {result}")
            
            messages.append(
                ToolMessage(
                    content=str(result),
                    tool_call_id=tool_call["id"],
                    name=tool_name
                )
            )
    
    return "최대 반복 횟수 도달"

# 실행
result = run_agent("5와 7을 더한 후, 그 결과에 3을 곱하세요.")
```

## 6. 완전한 통합 예제 <a href="#id-6" id="id-6"></a>

OpenAI와 Ollama를 선택적으로 사용하는 완전한 Multi-Agent 구현

```python
import os
from langchain.chat_models import init_chat_model
from langchain_core.tools import tool
from langchain_core.messages import HumanMessage, SystemMessage, ToolMessage
from typing import Literal

os.environ["OPENAI_API_KEY"] = "your-api-key"

@tool
def get_weather(city: str, units: Literal["celsius", "fahrenheit"] = "celsius") -> dict:
    """도시의 날씨 정보를 조회합니다."""
    weather_data = {
        "seoul": {"temp": 22, "condition": "맑음"},
        "busan": {"temp": 24, "condition": "흐림"}
    }
    data = weather_data.get(city.lower(), {"error": "도시 없음"})
    
    if units == "fahrenheit" and "temp" in data:
        data["temp"] = (data["temp"] * 9/5) + 32
        data["units"] = "°F"
    else:
        data["units"] = "°C"
    
    return data

@tool
def calculate(expression: str) -> float:
    """수학 표현식을 계산합니다."""
    try:
        return float(eval(expression, {"__builtins__": {}}, {}))
    except Exception as e:
        return f"오류: {str(e)}"

tools = [get_weather, calculate]
tools_dict = {tool.name: tool for tool in tools}

class MultiModelAgent:
    """OpenAI와 Ollama를 선택적으로 사용하는 Agent"""
    
    def __init__(self, model_type: Literal["openai", "ollama"] = "openai"):
        self.model_type = model_type
        
        if model_type == "openai":
            self.llm = init_chat_model("gpt-4o-mini", model_provider="openai", temperature=0)
        else:
            self.llm = init_chat_model("llama3.1", model_provider="ollama", temperature=0)
        
        self.llm_with_tools = self.llm.bind_tools(tools)
    
    def run(self, user_query: str, max_iterations: int = 5) -> dict:
        """Agent 실행"""
        
        messages = [
            SystemMessage(content="당신은 유용한 AI입니다. 필요한 도구를 사용하세요."),
            HumanMessage(content=user_query)
        ]
        
        history = []
        
        for iteration in range(max_iterations):
            print(f"\n{'='*50}")
            print(f"Iteration {iteration + 1}/{max_iterations}")
            
            response = self.llm_with_tools.invoke(messages)
            messages.append(response)
            
            if not response.tool_calls:
                return {
                    "answer": response.content,
                    "history": history,
                    "iterations": iteration + 1,
                    "model": self.model_type
                }
            
            for tool_call in response.tool_calls:
                tool_name = tool_call["name"]
                tool_args = tool_call["args"]
                
                print(f"🔧 Tool 호출: {tool_name}({tool_args})")
                
                tool = tools_dict[tool_name]
                result = tool.invoke(tool_args)
                
                print(f"   결과: {result}")
                
                history.append({
                    "tool": tool_name,
                    "args": tool_args,
                    "result": result
                })
                
                messages.append(
                    ToolMessage(
                        content=str(result),
                        tool_call_id=tool_call["id"],
                        name=tool_name
                    )
                )
        
        return {
            "answer": "최대 반복 도달",
            "history": history,
            "iterations": max_iterations
        }

# 사용 예시
agent = MultiModelAgent(model_type="openai")
result = agent.run("서울 날씨를 알려주고, 온도에 5를 더한 값을 계산해주세요.")

print(f"\n모델: {result['model']}")
print(f"반복: {result['iterations']}")
print(f"답변: {result['answer']}")
```

## 7. 주요 패턴 및 베스트 프랙티스 <a href="#id-7" id="id-7"></a>

### 7-1. Tool 정의 시 권장사항

1. **명확한 docstring**: 모델이 Tool의 목적을 이해할 수 있도록 상세히 작성​
2. **타입 힌트**: 모든 인자와 반환 타입에 타입 힌트 제공​
3. **Field 설명**: Pydantic Field를 사용해 인자 설명 추가​

### 7-2. 실행 패턴

1. **단순 실행**: Tool을 한 번만 호출하고 결과 반환
2. **Agent 루프**: Tool 결과를 메시지에 추가하고 모델 재호출​
3. **에러 핸들링**: ToolException을 사용한 안전한 에러 처리​

## 8. 모델별 주의사항

**OpenAI**:​

* gpt-4, gpt-3.5-turbo 등 대부분의 모델이 Tool Calling 지원
* `bind_tools()`로 간단히 Tool 바인딩

**Ollama**:​

* llama3.1, mistral, qwen 등 Tool Calling 지원 모델 사용 필수
* OpenAI 호환 API 사용
* Tool Calling 품질이 모델에 따라 다를 수 있음
