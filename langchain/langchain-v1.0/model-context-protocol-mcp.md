# Model Context Protocol (MCP)

\*\*Model Context Protocol (MCP)\*\*은 Anthropic이 개발한 오픈 프로토콜로, AI 애플리케이션이 LLM에 도구와 컨텍스트를 제공하는 방법을 표준화합니다. MCP는 마치 다양한 전자기기를 하나의 포트로 연결하는 **USB-C**처럼, AI 모델과 외부 시스템 간의 연결을 표준화​

***

## 1. MCP 핵심 개념 <a href="#mcp" id="mcp"></a>

### MCP란?

MCP는 세 가지 핵심 구성 요소를 제공합니다:​

1. **Tools**: LLM이 실행할 수 있는 작업 (RESTful의 POST와 유사)​
2. **Resources**: 읽기 전용 데이터 제공 (RESTful의 GET와 유사)​
3. **Prompts**: 재사용 가능한 메시지 템플릿​
4. **Context**: 세션 기능 접근 (로깅, HTTP 요청 등)​

### Transport 타입

MCP는 세 가지 전송 메커니즘을 지원합니다:​

* **stdio**: 로컬 서버를 subprocess로 실행, stdin/stdout으로 통신. 로컬 도구에 최적​
* **streamable\_http**: 독립 프로세스로 HTTP 서버 실행. 원격 연결과 다중 클라이언트 지원​
* **sse (Server-Sent Events)**: 실시간 스트리밍 통신에 최적화​

### Stateful vs Stateless

* **Stateless (기본)**: 각 도구 호출마다 새로운 `ClientSession` 생성. 독립적 요청에 적합​
* **Stateful**: 세션 간 상태 유지. 순차적 도구 호출이 필요한 경우 (예: Playwright의 `browser_navigate` → `browser_click`)​

***

### 1. 설치 및 기본 설정 <a href="#id-1" id="id-1"></a>

```python
# 필수 패키지 설치
pip install --pre -U langchain
pip install -U langchain-openai langchain-ollama
pip install -U langchain-mcp-adapters
pip install -U langgraph
pip install mcp  # MCP 서버 개발용

# 환경 변수 설정
import os
os.environ["OPENAI_API_KEY"] = "your-openai-api-key"
# Ollama는 로컬에서 실행 (http://localhost:11434)
```

***

## 2. MCP 서버 생성 (FastMCP) <a href="#id-2-mcp---fastmcp" id="id-2-mcp---fastmcp"></a>

### 2.1 기본 Math Server (stdio transport)

```python
# math_server.py
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Math")

@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b

@mcp.tool()
def multiply(a: int, b: int) -> int:
    """Multiply two numbers"""
    return a * b

@mcp.tool()
def subtract(a: int, b: int) -> int:
    """Subtract b from a"""
    return a - b

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

### 2.2 Weather Server (Streamable HTTP transport)

```python
# weather_server.py
from mcp.server.fastmcp import FastMCP
import asyncio

mcp = FastMCP("Weather")

@mcp.tool()
async def get_weather(location: str) -> str:
    """Get weather for location."""
    # 실제로는 API 호출
    weather_data = {
        "Seoul": "22°C, Sunny",
        "New York": "15°C, Cloudy",
        "Tokyo": "18°C, Rainy"
    }
    return weather_data.get(location, f"Weather data not available for {location}")

@mcp.tool()
async def get_forecast(location: str, days: int = 3) -> str:
    """Get weather forecast for location."""
    return f"{days}-day forecast for {location}: Mostly sunny"

if __name__ == "__main__":
    mcp.run(transport="streamable-http", port=8000)
```

**서버 실행:**

```bash
# Math server (stdio)
python math_server.py

# Weather server (HTTP) - 별도 터미널
python weather_server.py
```

***

## 3. 기본 패턴: MultiServerMCPClient + init\_chat\_model <a href="#id-3---multiservermcpclient--initchatmodel" id="id-3---multiservermcpclient--initchatmodel"></a>

### 3.1 OpenAI 사용 (Stateless)

```python
import asyncio
from langchain_mcp_adapters.client import MultiServerMCPClient
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent

async def openai_mcp_agent():
    # init_chat_model로 OpenAI 모델 초기화
    model = init_chat_model("openai:gpt-4o", temperature=0.7)
    
    # MultiServerMCPClient 설정
    client = MultiServerMCPClient({
        "math": {
            "transport": "stdio",
            "command": "python",
            "args": ["/absolute/path/to/math_server.py"]  # 절대 경로 필수
        },
        "weather": {
            "transport": "streamable_http",
            "url": "http://localhost:8000/mcp"
        }
    })
    
    # MCP 도구 가져오기 (LangChain Tool로 자동 변환)
    tools = await client.get_tools()
    print(f"Available tools: {[tool.name for tool in tools]}")
    
    # Agent 생성
    agent = create_agent(
        model=model,
        tools=tools,
        system_prompt="You are a helpful assistant with access to math and weather tools."
    )
    
    # Math 도구 사용
    result1 = await agent.ainvoke({
        "messages": [{"role": "user", "content": "What is (3 + 5) * 12?"}]
    })
    print("\n[Math Result]")
    print(result1["messages"][-1].content)
    
    # Weather 도구 사용
    result2 = await agent.ainvoke({
        "messages": [{"role": "user", "content": "What's the weather in Seoul?"}]
    })
    print("\n[Weather Result]")
    print(result2["messages"][-1].content)
    
    # 복합 질문
    result3 = await agent.ainvoke({
        "messages": [{"role": "user", "content": "Calculate 15 * 8 and then tell me the weather in Tokyo"}]
    })
    print("\n[Combined Result]")
    print(result3["messages"][-1].content)

# 실행
asyncio.run(openai_mcp_agent())
```

### 3.2 Ollama 사용 (Stateless)

```python
import asyncio
from langchain_mcp_adapters.client import MultiServerMCPClient
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent

async def ollama_mcp_agent():
    # init_chat_model로 Ollama 모델 초기화
    model = init_chat_model("ollama:llama3.1", temperature=0.7)
    
    # MultiServerMCPClient 설정
    client = MultiServerMCPClient({
        "math": {
            "transport": "stdio",
            "command": "python",
            "args": ["/absolute/path/to/math_server.py"]
        },
        "weather": {
            "transport": "streamable_http",
            "url": "http://localhost:8000/mcp"
        }
    })
    
    # MCP 도구 가져오기
    tools = await client.get_tools()
    
    # Agent 생성
    agent = create_agent(
        model=model,
        tools=tools,
        system_prompt="You are a helpful assistant. Use tools to answer questions accurately."
    )
    
    # 실행
    result = await agent.ainvoke({
        "messages": [{"role": "user", "content": "Add 100 and 250, then check weather in New York"}]
    })
    print(result["messages"][-1].content)

asyncio.run(ollama_mcp_agent())
```

***

## 4. 고급 패턴: Stateful Session <a href="#id-4---stateful-session" id="id-4---stateful-session"></a>

### 4.1 Stateful MCP Server 생성

```python
# file_manager_server.py
from mcp.server.fastmcp import FastMCP, Context
from typing import Dict

mcp = FastMCP("FileManager")

# 세션별 상태 저장
session_state: Dict[str, list] = {}

@mcp.tool()
def create_file(filename: str, ctx: Context) -> str:
    """Create a new file in the session."""
    session_id = id(ctx)  # 세션 식별
    
    if session_id not in session_state:
        session_state[session_id] = []
    
    session_state[session_id].append(filename)
    return f"Created file: {filename}"

@mcp.tool()
def list_files(ctx: Context) -> str:
    """List all files created in this session."""
    session_id = id(ctx)
    
    if session_id not in session_state:
        return "No files created yet"
    
    files = session_state[session_id]
    return f"Files in session: {', '.join(files)}"

@mcp.tool()
def delete_file(filename: str, ctx: Context) -> str:
    """Delete a file from the session."""
    session_id = id(ctx)
    
    if session_id in session_state and filename in session_state[session_id]:
        session_state[session_id].remove(filename)
        return f"Deleted file: {filename}"
    return f"File not found: {filename}"

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

### 4.2 Stateful Session 사용 (OpenAI)

```python
import asyncio
from langchain_mcp_adapters.client import MultiServerMCPClient
from langchain_mcp_adapters.tools import load_mcp_tools
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent

async def stateful_openai_agent():
    model = init_chat_model("openai:gpt-4o")
    
    client = MultiServerMCPClient({
        "file_manager": {
            "transport": "stdio",
            "command": "python",
            "args": ["/absolute/path/to/file_manager_server.py"]
        }
    })
    
    # Stateful session 생성
    async with client.session("file_manager") as session:
        # 세션에서 도구 로드
        tools = await load_mcp_tools(session)
        
        agent = create_agent(
            model=model,
            tools=tools,
            system_prompt="You are a file management assistant. Maintain file state across operations."
        )
        
        # 순차적 작업 1: 파일 생성
        result1 = await agent.ainvoke({
            "messages": [{"role": "user", "content": "Create files: report.txt, data.csv, notes.md"}]
        })
        print("\n[Create Files]")
        print(result1["messages"][-1].content)
        
        # 순차적 작업 2: 파일 목록 조회
        result2 = await agent.ainvoke({
            "messages": [{"role": "user", "content": "List all files"}]
        })
        print("\n[List Files]")
        print(result2["messages"][-1].content)
        
        # 순차적 작업 3: 파일 삭제
        result3 = await agent.ainvoke({
            "messages": [{"role": "user", "content": "Delete data.csv"}]
        })
        print("\n[Delete File]")
        print(result3["messages"][-1].content)
        
        # 순차적 작업 4: 다시 목록 조회
        result4 = await agent.ainvoke({
            "messages": [{"role": "user", "content": "List all files again"}]
        })
        print("\n[List Files After Delete]")
        print(result4["messages"][-1].content)
    
    # 세션 종료 후에는 상태가 유지되지 않음

asyncio.run(stateful_openai_agent())
```

### 4.3 Multiple Stateful Sessions (Ollama)

```python
import asyncio
from contextlib import AsyncExitStack
from langchain_mcp_adapters.client import MultiServerMCPClient
from langchain_mcp_adapters.tools import load_mcp_tools
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent

async def multi_stateful_ollama_agent():
    model = init_chat_model("ollama:llama3.1")
    
    client = MultiServerMCPClient({
        "math": {
            "transport": "stdio",
            "command": "python",
            "args": ["/absolute/path/to/math_server.py"]
        },
        "file_manager": {
            "transport": "stdio",
            "command": "python",
            "args": ["/absolute/path/to/file_manager_server.py"]
        }
    })
    
    # 여러 세션 동시 관리
    async with AsyncExitStack() as stack:
        # 두 서버에 대한 세션 생성
        session1 = await stack.enter_async_context(client.session("math"))
        session2 = await stack.enter_async_context(client.session("file_manager"))
        
        # 모든 도구 로드
        tools = [
            *await load_mcp_tools(session1),
            *await load_mcp_tools(session2)
        ]
        
        agent = create_agent(
            model=model,
            tools=tools,
            system_prompt="You have access to math and file management tools."
        )
        
        result = await agent.ainvoke({
            "messages": [{"role": "user", "content": "Calculate 50 * 3, then create a file called results.txt"}]
        })
        print(result["messages"][-1].content)

asyncio.run(multi_stateful_ollama_agent())
```

***

## 5. MCP Resources 패턴 <a href="#id-5-mcp-resources" id="id-5-mcp-resources"></a>

### 5.1 Resource 서버 생성

```python
# docs_server.py
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Documentation")

@mcp.resource("docs://guide/{topic}")
def get_guide(topic: str) -> str:
    """Get documentation guide for a topic."""
    guides = {
        "langchain": "LangChain is a framework for developing applications powered by LLMs...",
        "mcp": "Model Context Protocol standardizes how applications provide context to LLMs...",
        "agents": "Agents are systems that use LLMs to decide which actions to take..."
    }
    return guides.get(topic, f"No guide found for {topic}")

@mcp.resource("docs://api")
def get_api_docs() -> str:
    """Get API documentation."""
    return """
    API Documentation:
    - POST /tools: Execute a tool
    - GET /resources: List available resources
    - GET /prompts: List available prompts
    """

@mcp.resource("config://settings")
def get_settings() -> str:
    """Get application settings."""
    return '{"theme": "dark", "language": "en", "debug": false}'

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

### 5.2 Resources 사용 (OpenAI)

```python
import asyncio
from langchain_mcp_adapters.client import MultiServerMCPClient
from langchain_mcp_adapters.resources import load_mcp_resources
from langchain.chat_models import init_chat_model

async def openai_mcp_resources():
    model = init_chat_model("openai:gpt-4o")
    
    client = MultiServerMCPClient({
        "docs": {
            "transport": "stdio",
            "command": "python",
            "args": ["/absolute/path/to/docs_server.py"]
        }
    })
    
    # Resources 로드
    async with client.session("docs") as session:
        # 모든 리소스 목록 가져오기
        resources_list = await session.list_resources()
        print("Available resources:")
        for res in resources_list.resources:
            print(f"  - {res.uri}: {res.name}")
        
        # 특정 리소스 읽기
        langchain_guide = await session.read_resource("docs://guide/langchain")
        print("\n[LangChain Guide]")
        print(langchain_guide.contents[0].text)
        
        mcp_guide = await session.read_resource("docs://guide/mcp")
        print("\n[MCP Guide]")
        print(mcp_guide.contents[0].text)
        
        # LLM에 리소스 컨텍스트 제공
        api_docs = await session.read_resource("docs://api")
        
        response = await model.ainvoke([
            {"role": "system", "content": f"You have access to these API docs:\n{api_docs.contents[0].text}"},
            {"role": "user", "content": "How do I execute a tool via the API?"}
        ])
        print("\n[LLM Response with Resource Context]")
        print(response.content)

asyncio.run(openai_mcp_resources())
```

***

## 6. MCP Prompts 패턴 <a href="#id-6-mcp-prompts" id="id-6-mcp-prompts"></a>

### 6.1 Prompt 서버 생성

```python
# prompts_server.py
from mcp.server.fastmcp import FastMCP
from mcp.server.fastmcp.prompts import base

mcp = FastMCP("PromptTemplates")

@mcp.prompt(title="Code Review")
def review_code(code: str, language: str = "python") -> str:
    """Generate a code review prompt."""
    return f"""Please review this {language} code:

```

{code}

<pre class="language-textile"><code class="lang-textile"><strong>Provide feedback on:
</strong>1. Code quality and best practices
2. Potential bugs or issues
3. Performance optimizations
4. Suggestions for improvement
"""

@mcp.prompt(title="Debug Assistant")
def debug_error(error: str, context: str = "") -> list[base.Message]:
    """Generate a debugging conversation."""
    messages = [
        base.UserMessage("I'm seeing this error:"),
        base.UserMessage(error)
    ]
    
    if context:
        messages.append(base.UserMessage(f"Context: {context}"))
    
    messages.append(
        base.AssistantMessage("I'll help you debug that. What have you tried so far?")
    )
    
    return messages

@mcp.prompt(title="Explain Concept")
def explain_concept(concept: str, level: str = "beginner") -> str:
    """Generate a prompt to explain a concept."""
    level_instructions = {
        "beginner": "Explain in simple terms with examples, avoiding technical jargon.",
        "intermediate": "Provide a balanced explanation with some technical details.",
        "expert": "Give a deep technical explanation with advanced concepts."
    }
    
    instruction = level_instructions.get(level, level_instructions["beginner"])
    
    return f"Please explain '{concept}' to a {level} level audience.\n\n{instruction}"

if __name__ == "__main__":
    mcp.run(transport="stdio")
</code></pre>

### 6.2 Prompts 사용 (Ollama)

```python
import asyncio
from langchain_mcp_adapters.client import MultiServerMCPClient
from langchain_mcp_adapters.prompts import load_mcp_prompt
from langchain.chat_models import init_chat_model

async def ollama_mcp_prompts():
    model = init_chat_model("ollama:llama3.1")
    
    client = MultiServerMCPClient({
        "prompts": {
            "transport": "stdio",
            "command": "python",
            "args": ["/absolute/path/to/prompts_server.py"]
        }
    })
    
    async with client.session("prompts") as session:
        # 사용 가능한 프롬프트 목록
        prompts_list = await session.list_prompts()
        print("Available prompts:")
        for prompt in prompts_list.prompts:
            print(f"  - {prompt.name}: {prompt.description}")
        
        # 프롬프트 1: Code Review
        code_sample = """
def calculate_average(numbers):
    sum = 0
    for n in numbers:
        sum = sum + n
    return sum / len(numbers)
"""
        
        review_messages = await load_mcp_prompt(
            session,
            "review_code",
            arguments={"code": code_sample, "language": "python"}
        )
        
        response1 = await model.ainvoke(review_messages)
        print("\n[Code Review]")
        print(response1.content)
        
        # 프롬프트 2: Debug Assistant
        debug_messages = await load_mcp_prompt(
            session,
            "debug_error",
            arguments={
                "error": "TypeError: unsupported operand type(s) for +: 'int' and 'str'",
                "context": "Trying to add user input to a counter"
            }
        )
        
        response2 = await model.ainvoke(debug_messages)
        print("\n[Debug Assistant]")
        print(response2.content)
        
        # 프롬프트 3: Explain Concept
        explain_messages = await load_mcp_prompt(
            session,
            "explain_concept",
            arguments={"concept": "async/await", "level": "intermediate"}
        )
        
        response3 = await model.ainvoke(explain_messages)
        print("\n[Concept Explanation]")
        print(response3.content)

asyncio.run(ollama_mcp_prompts())
```

***

## 7. LangGraph + MCP 통합 <a href="#id-7-langgraph--mcp" id="id-7-langgraph--mcp"></a>

### 7.1 Custom Workflow with MCP Tools (OpenAI)

```python
import asyncio
from langchain_mcp_adapters.client import MultiServerMCPClient
from langchain.chat_models import init_chat_model
from langgraph.graph import StateGraph, MessagesState, START, END
from langgraph.prebuilt import ToolNode

async def langgraph_mcp_openai():
    # init_chat_model로 모델 초기화
    model = init_chat_model("openai:gpt-4o-mini")
    
    # MCP 클라이언트 설정
    client = MultiServerMCPClient({
        "math": {
            "transport": "stdio",
            "command": "python",
            "args": ["/absolute/path/to/math_server.py"]
        },
        "weather": {
            "transport": "streamable_http",
            "url": "http://localhost:8000/mcp"
        }
    })
    
    # MCP 도구 가져오기
    tools = await client.get_tools()
    
    # 모델에 도구 바인딩
    model_with_tools = model.bind_tools(tools)
    
    # ToolNode 생성
    tool_node = ToolNode(tools)
    
    # 조건 분기 함수
    def should_continue(state: MessagesState):
        messages = state["messages"]
        last_message = messages[-1]
        if last_message.tool_calls:
            return "tools"
        return END
    
    # 모델 호출 함수
    async def call_model(state: MessagesState):
        messages = state["messages"]
        response = await model_with_tools.ainvoke(messages)
        return {"messages": [response]}
    
    # 그래프 구축
    builder = StateGraph(MessagesState)
    builder.add_node("call_model", call_model)
    builder.add_node("tools", tool_node)
    builder.add_edge(START, "call_model")
    builder.add_conditional_edges(
        "call_model",
        should_continue,
        {"tools": "tools", END: END}
    )
    builder.add_edge("tools", "call_model")
    
    # 그래프 컴파일
    graph = builder.compile()
    
    # 실행 1: Math
    math_response = await graph.ainvoke({
        "messages": [{"role": "user", "content": "What's (3 + 5) x 12?"}]
    })
    print("\n[Math Result]")
    print(math_response["messages"][-1].content)
    
    # 실행 2: Weather
    weather_response = await graph.ainvoke({
        "messages": [{"role": "user", "content": "What is the weather in Seoul?"}]
    })
    print("\n[Weather Result]")
    print(weather_response["messages"][-1].content)
    
    # 실행 3: 복합 작업
    combined_response = await graph.ainvoke({
        "messages": [{"role": "user", "content": "Calculate 25 * 8 and check weather in Tokyo"}]
    })
    print("\n[Combined Result]")
    print(combined_response["messages"][-1].content)

asyncio.run(langgraph_mcp_openai())
```

### 7.2 Streaming with MCP (Ollama)

```python
import asyncio
from langchain_mcp_adapters.client import MultiServerMCPClient
from langchain.chat_models import init_chat_model
from langgraph.prebuilt import create_react_agent

async def streaming_mcp_ollama():
    model = init_chat_model("ollama:llama3.1")
    
    client = MultiServerMCPClient({
        "math": {
            "transport": "stdio",
            "command": "python",
            "args": ["/absolute/path/to/math_server.py"]
        }
    })
    
    tools = await client.get_tools()
    
    # create_react_agent 사용
    agent = create_react_agent(model, tools)
    
    # 스트리밍 실행
    print("\n[Streaming Output]")
    async for event in agent.astream({
        "messages": [{"role": "user", "content": "Calculate (15 + 25) * 3 - 10"}]
    }):
        # 이벤트 타입에 따라 처리
        for key, value in event.items():
            if key == "agent":
                print(f"\n[Agent]: {value['messages'][-1].content}")
            elif key == "tools":
                print(f"\n[Tool Call]: {value['messages'][-1].content}")

asyncio.run(streaming_mcp_ollama())
```

***

## 8. 고급 패턴: Context 사용 <a href="#id-8---context" id="id-8---context"></a>

### 8.1 Context-aware MCP Server

```python
# context_server.py
from mcp.server.fastmcp import FastMCP, Context
from mcp.server.session import ServerSession
import asyncio

mcp = FastMCP("ContextServer")

@mcp.tool()
async def analyze_data(
    data: str,
    ctx: Context[ServerSession, None]
) -> str:
    """Analyze data with progress reporting."""
    # 로깅
    await ctx.info(f"Starting data analysis for: {data}")
    
    # 진행률 보고
    await ctx.report_progress(
        progress=0.0,
        total=1.0,
        message="Initializing..."
    )
    
    await asyncio.sleep(1)
    
    await ctx.report_progress(
        progress=0.5,
        total=1.0,
        message="Processing data..."
    )
    
    await asyncio.sleep(1)
    
    # 디버그 메시지
    await ctx.debug("Analysis computation complete")
    
    await ctx.report_progress(
        progress=1.0,
        total=1.0,
        message="Analysis complete"
    )
    
    result = f"Analysis of '{data}': 95% confidence, 3 patterns detected"
    await ctx.info("Analysis finished successfully")
    
    return result

@mcp.tool()
async def fetch_and_summarize(
    url: str,
    ctx: Context[ServerSession, None]
) -> str:
    """Fetch content and ask LLM to summarize."""
    await ctx.info(f"Fetching content from: {url}")
    
    # HTTP 요청 (Context 사용)
    # content = await ctx.http_request(url)
    
    # 시뮬레이션
    content = f"Content from {url}: This is a long article about AI..."
    
    # LLM 샘플링 요청 (클라이언트의 LLM 사용)
    summary = await ctx.sample(
        f"Summarize this content in one sentence:\n\n{content[:200]}..."
    )
    
    return summary.text

@mcp.tool()
async def read_document(
    doc_uri: str,
    ctx: Context[ServerSession, None]
) -> str:
    """Read a document resource."""
    await ctx.info(f"Reading document: {doc_uri}")
    
    # 리소스 읽기
    resource = await ctx.read_resource(doc_uri)
    
    return resource.contents[0].text if resource.contents else "Document not found"

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

### 8.2 Context Server 사용 (OpenAI)

```python
import asyncio
from langchain_mcp_adapters.client import MultiServerMCPClient
from langchain_mcp_adapters.tools import load_mcp_tools
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent

async def context_aware_openai_agent():
    model = init_chat_model("openai:gpt-4o")
    
    client = MultiServerMCPClient({
        "context": {
            "transport": "stdio",
            "command": "python",
            "args": ["/absolute/path/to/context_server.py"]
        }
    })
    
    async with client.session("context") as session:
        tools = await load_mcp_tools(session)
        
        agent = create_agent(
            model=model,
            tools=tools,
            system_prompt="You are an assistant that can analyze data with progress tracking."
        )
        
        # 진행률 보고와 함께 실행
        result = await agent.ainvoke({
            "messages": [{"role": "user", "content": "Analyze the sales data from Q4 2024"}]
        })
        print(result["messages"][-1].content)

asyncio.run(context_aware_openai_agent())
```

***

## 9. 실전 예제: 종합 MCP Agent <a href="#id-9----mcp-agent" id="id-9----mcp-agent"></a>

### 9.1 완전한 MCP 서버 (Tools + Resources + Prompts)

```python
# complete_server.py
from mcp.server.fastmcp import FastMCP, Context
from mcp.server.fastmcp.prompts import base
from mcp.server.session import ServerSession

mcp = FastMCP("CompleteServer")

# ===== Tools =====
@mcp.tool()
def search_database(query: str) -> str:
    """Search the database."""
    results = {
        "customers": ["Alice", "Bob", "Charlie"],
        "orders": ["Order-001", "Order-002"],
        "products": ["Product-A", "Product-B", "Product-C"]
    }
    return f"Search results for '{query}': {results.get(query, 'No results')}"

@mcp.tool()
async def send_notification(
    recipient: str,
    message: str,
    ctx: Context[ServerSession, None]
) -> str:
    """Send a notification."""
    await ctx.info(f"Sending notification to {recipient}")
    return f"Notification sent to {recipient}: {message}"

# ===== Resources =====
@mcp.resource("data://company/employees")
def get_employees() -> str:
    """Get employee directory."""
    return """
    Employee Directory:
    - Alice (Engineering)
    - Bob (Sales)
    - Charlie (Marketing)
    """

@mcp.resource("data://company/policies/{policy_name}")
def get_policy(policy_name: str) -> str:
    """Get company policy."""
    policies = {
        "vacation": "Employees get 15 days of vacation per year.",
        "remote": "Remote work is allowed 3 days per week.",
        "expenses": "Submit expenses within 30 days of purchase."
    }
    return policies.get(policy_name, "Policy not found")

# ===== Prompts =====
@mcp.prompt(title="Employee Onboarding")
def onboarding_prompt(employee_name: str, department: str) -> str:
    """Generate onboarding instructions."""
    return f"""Welcome {employee_name} to the {department} department!

Please complete these onboarding steps:
1. Review company policies
2. Set up your workstation
3. Meet your team members
4. Complete required training

Let me know if you need help with anything!
"""

@mcp.prompt(title="Data Analysis Request")
def analysis_prompt(dataset: str) -> list[base.Message]:
    """Generate data analysis conversation."""
    return [
        base.UserMessage(f"I need to analyze the {dataset} dataset"),
        base.AssistantMessage("I'll help you with that. What specific insights are you looking for?"),
        base.UserMessage("I want to understand trends and patterns")
    ]

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

### 9.2 완전한 MCP Agent (OpenAI + Ollama)

```python
import asyncio
from langchain_mcp_adapters.client import MultiServerMCPClient
from langchain_mcp_adapters.tools import load_mcp_tools
from langchain_mcp_adapters.resources import load_mcp_resources
from langchain_mcp_adapters.prompts import load_mcp_prompt
from langchain.chat_models import init_chat_model
from langchain.agents import create_agent

async def complete_mcp_agent():
    # 두 모델 모두 사용
    openai_model = init_chat_model("openai:gpt-4o", temperature=0.7)
    ollama_model = init_chat_model("ollama:llama3.1", temperature=0.7)
    
    # MCP 클라이언트
    client = MultiServerMCPClient({
        "complete": {
            "transport": "stdio",
            "command": "python",
            "args": ["/absolute/path/to/complete_server.py"]
        },
        "math": {
            "transport": "stdio",
            "command": "python",
            "args": ["/absolute/path/to/math_server.py"]
        }
    })
    
    async with client.session("complete") as session:
        # ===== Tools 로드 =====
        tools = await load_mcp_tools(session)
        print(f"\nAvailable tools: {[t.name for t in tools]}")
        
        # ===== Resources 로드 =====
        resources = await session.list_resources()
        print(f"\nAvailable resources:")
        for res in resources.resources:
            print(f"  - {res.uri}")
        
        # 특정 리소스 읽기
        employees_data = await session.read_resource("data://company/employees")
        vacation_policy = await session.read_resource("data://company/policies/vacation")
        
        print(f"\n[Employees Data]")
        print(employees_data.contents[0].text)
        
        # ===== Prompts 로드 =====
        prompts = await session.list_prompts()
        print(f"\nAvailable prompts: {[p.name for p in prompts]}")
        
        onboarding_messages = await load_mcp_prompt(
            session,
            "onboarding_prompt",
            arguments={"employee_name": "David", "department": "Engineering"}
        )
        
        # ===== OpenAI Agent =====
        print("\n" + "="*70)
        print("OpenAI Agent")
        print("="*70)
        
        openai_agent = create_agent(
            model=openai_model,
            tools=tools,
            system_prompt=f"""You are a helpful company assistant with access to:
- Database search tools
- Notification tools
- Company resources: {employees_data.contents[0].text}
- Vacation policy: {vacation_policy.contents[0].text}
"""
        )
        
        result1 = await openai_agent.ainvoke({
            "messages": [{"role": "user", "content": "Search for customers in the database"}]
        })
        print("\n[OpenAI - Database Search]")
        print(result1["messages"][-1].content)
        
        result2 = await openai_agent.ainvoke({
            "messages": [{"role": "user", "content": "How many vacation days do employees get?"}]
        })
        print("\n[OpenAI - Policy Question]")
        print(result2["messages"][-1].content)
        
        # ===== Ollama Agent =====
        print("\n" + "="*70)
        print("Ollama Agent")
        print("="*70)
        
        ollama_agent = create_agent(
            model=ollama_model,
            tools=tools,
            system_prompt="You are a company assistant with database and notification tools."
        )
        
        result3 = await ollama_agent.ainvoke({
            "messages": [{"role": "user", "content": "Send a welcome notification to the new employee David"}]
        })
        print("\n[Ollama - Notification]")
        print(result3["messages"][-1].content)
        
        # ===== Prompt 사용 =====
        print("\n" + "="*70)
        print("Using Prompts")
        print("="*70)
        
        # OpenAI로 프롬프트 실행
        onboarding_response = await openai_model.ainvoke(onboarding_messages)
        print("\n[Onboarding with Prompt]")
        print(onboarding_response.content)

asyncio.run(complete_mcp_agent())
```

***

## 10. Best Practices <a href="#id-10-best-practices" id="id-10-best-practices"></a>

### MCP 서버 개발

1. **명확한 도구 설명**: `@mcp.tool()` 데코레이터와 docstring으로 도구 목적 명확히 설명​
2. **타입 힌트 사용**: 모든 매개변수와 반환 타입에 타입 힌트 추가​
3. **적절한 Transport 선택**: 로컬은 stdio, 원격은 streamable\_http 사용​
4. **Stateful vs Stateless**: 순차적 작업이 필요한 경우만 stateful 사용​

### LangChain 통합

1. **절대 경로 사용**: stdio transport에서 서버 파일 경로는 절대 경로로 지정​
2. **세션 관리**: Stateful 서버는 `async with client.session()` 사용​
3. **에러 처리**: MCP 도구 호출 실패 시 적절한 에러 핸들링​
4. **리소스 효율**: Stateless가 기본값이므로 필요한 경우만 세션 유지​

### 성능 최적화

1. **도구 선택 최소화**: LLM이 필요한 도구만 노출​
2. **리소스 지연 로딩**: 모든 리소스를 미리 로드하지 말고 필요시 로드​
3. **적절한 모델 선택**: 간단한 작업은 Ollama, 복잡한 작업은 OpenAI 사용​
4. **병렬 처리**: 독립적인 MCP 서버는 병렬로 호출​

***

### 관련 리소스 <a href="#undefined" id="undefined"></a>

* **LangChain MCP 문서**: [docs.langchain.com/mcp](https://docs.langchain.com/oss/python/langchain/mcp)​
* **LangGraph MCP 가이드**: [langchain-ai.github.io/langgraph/agents/mcp](https://langchain-ai.github.io/langgraph/agents/mcp/)​
* **FastMCP 문서**: [gofastmcp.com](https://gofastmcp.com/)​
* **MCP Python SDK**: [github.com/modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk)​
* **LangChain MCP Adapters**: [github.com/langchain-ai/langchain-mcp-adapters](https://github.com/langchain-ai/langchain-mcp-adapters)​
