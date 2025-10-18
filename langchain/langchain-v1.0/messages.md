# Messages

LangChain v1.0에서 Messages는 채팅 모델의 입출력을 나타내는 핵심 단위로  모든 메시지 타입, 패턴, 실전 예제&#x20;

## 1. 환경 설정 <a href="#undefined" id="undefined"></a>

```python
# 필수 패키지 설치
pip install -qU langchain langchain-openai langchain-ollama langchain-core

# 환경 변수 설정
import os
os.environ["OPENAI_API_KEY"] = "your-openai-api-key"
# Ollama는 로컬에서 실행되므로 API 키 불필요
```

## 2. Messages의 기본 구조 <a href="#messages" id="messages"></a>

Messages는 세 가지 핵심 요소로 구성된다:​

* **Role**: 메시지 타입 식별 (system, user, assistant, tool)
* **Content**: 실제 메시지 내용 (텍스트, 이미지, 오디오 등)
* **Metadata**: 선택적 필드 (ID, 토큰 사용량, 응답 메타데이터 등)

LangChain v1.0는 5가지 주요 메시지 타입을 제공합니다.​

### 2-1. SystemMessage - 시스템 지침

모델의 행동과 역할을 정의하는 메시지.​

```python
from langchain.messages import SystemMessage, HumanMessage
from langchain.chat_models import init_chat_model

# OpenAI 예제
model = init_chat_model("openai:gpt-4o", temperature=0)

system_msg = SystemMessage(content="""
당신은 전문 Python 개발자입니다.
항상 코드 예제와 함께 설명하고, 간결하면서도 상세하게 답변하세요.
""")

human_msg = HumanMessage(content="REST API를 어떻게 만드나요?")

response = model.invoke([system_msg, human_msg])
print(response.content)
```

```python
# Ollama 예제
model = init_chat_model("ollama:llama3.1", temperature=0)

messages = [
    SystemMessage(content="You are a helpful assistant."),
    HumanMessage(content="Explain quantum computing in simple terms.")
]

response = model.invoke(messages)
print(response.content)
```

**SystemMessage 속성**:​

* `content` (필수): 시스템 지침 내용
* `name` (선택): 메시지 식별자
* `id` (선택): 고유 ID
* `response_metadata` (선택): 응답 메타데이터

### 2-2. HumanMessage - 사용자 입력

사용자로부터의 입력.​

```python
from langchain.messages import HumanMessage
from langchain.chat_models import init_chat_model

# OpenAI 기본 텍스트
model = init_chat_model("openai:gpt-4o")
response = model.invoke([HumanMessage(content="안녕하세요!")])
print(response.content)

# Ollama 기본 텍스트
model = init_chat_model("ollama:llama3.1")
response = model.invoke([HumanMessage(content="What is LangChain?")])
print(response.content)
```

**메타데이터 추가 예제**:​

```python
from langchain.messages import HumanMessage

human_msg = HumanMessage(
    content="안녕하세요!",
    name="alice",  # 사용자 식별
    id="msg_123"   # 추적용 고유 ID
)

model = init_chat_model("openai:gpt-4o")
response = model.invoke([human_msg])
```

### 2-3. AIMessage - AI 응답

모델의 응답을 나타냅니다. 텍스트, 도구 호출, 사용량 메타데이터를 포함할 수 있다.​

```python
from langchain.messages import AIMessage, SystemMessage, HumanMessage
from langchain.chat_models import init_chat_model

# OpenAI 예제 - 대화 히스토리에 AIMessage 추가
model = init_chat_model("openai:gpt-4o")

messages = [
    SystemMessage(content="You are a helpful assistant"),
    HumanMessage(content="Can you help me?"),
    AIMessage(content="I'd be happy to help you with that question!"),
    HumanMessage(content="What's 2+2?")
]

response = model.invoke(messages)
print(response.content)
print(f"토큰 사용량: {response.usage_metadata}")
```

**AIMessage 주요 속성**:​

* `content`: 텍스트 응답
* `tool_calls`: 도구 호출 정보
* `usage_metadata`: 토큰 사용량
* `response_metadata`: 제공자별 메타데이터
* `id`: 메시지 고유 ID

**토큰 사용량 확인**:​

```python
from langchain.chat_models import init_chat_model

model = init_chat_model("openai:gpt-4o")
response = model.invoke("Hello!")

# 토큰 사용량 메타데이터
print(response.usage_metadata)
# 출력: {'input_tokens': 8, 'output_tokens': 304, 'total_tokens': 312}
```

### 2-4. AIMessageChunk - 스트리밍 응답

스트리밍 시 사용되는 메시지 청크.​

```python
from langchain.chat_models import init_chat_model
from langchain.messages import HumanMessage

# OpenAI 스트리밍
model = init_chat_model("openai:gpt-4o")

chunks = []
for chunk in model.stream([HumanMessage(content="Python이란?")]):
    chunks.append(chunk)
    print(chunk.content, end="", flush=True)

# 전체 메시지로 병합
full_message = chunks[0]
for chunk in chunks[1:]:
    full_message = full_message + chunk

print("\n\n전체 메시지:", full_message.content)
```

```python
# Ollama 스트리밍
model = init_chat_model("ollama:llama3.1")

for chunk in model.stream("Explain machine learning"):
    print(chunk.content, end="", flush=True)
```

**비동기 스트리밍**:​

```python
import asyncio
from langchain.chat_models import init_chat_model

async def async_stream():
    model = init_chat_model("openai:gpt-4o")
    
    async for chunk in model.astream("Tell me a joke"):
        print(chunk.content, end="", flush=True)

asyncio.run(async_stream())
```

### 2-5. ToolMessage - 도구 실행 결과

도구 호출의 결과를 모델에 반환.​

```python
from langchain.messages import HumanMessage, AIMessage, ToolMessage
from langchain.chat_models import init_chat_model

# OpenAI Tool Calling
model = init_chat_model("openai:gpt-4o")

def get_weather(location: str) -> str:
    """지정된 위치의 날씨를 가져옵니다."""
    return f"{location}의 날씨는 맑고 섭씨 22도입니다."

# 도구 바인딩
model_with_tools = model.bind_tools([get_weather])

# 1단계: 사용자 질문
messages = [HumanMessage(content="서울 날씨 어때?")]
ai_message = model_with_tools.invoke(messages)

# 2단계: 도구 호출 확인 및 실행
if ai_message.tool_calls:
    for tool_call in ai_message.tool_calls:
        print(f"Tool: {tool_call['name']}")
        print(f"Args: {tool_call['args']}")
        
        # 도구 실행
        result = get_weather(**tool_call['args'])
        
        # 3단계: ToolMessage로 결과 반환
        tool_message = ToolMessage(
            content=result,
            tool_call_id=tool_call['id']
        )
        
        # 4단계: 최종 응답 생성
        messages.extend([ai_message, tool_message])
        final_response = model.invoke(messages)
        print(f"\n최종 응답: {final_response.content}")
```

**ToolMessage 속성**:​

* `content`: 도구 실행 결과 (문자열)
* `tool_call_id`: AIMessage의 tool\_call ID와 일치해야 함
* `name`: 도구 이름
* `artifact`: 모델에 전달되지 않는 추가 데이터

**artifact 사용 예제**:​

```python
from langchain.messages import ToolMessage

# 검색 도구 결과 예제
tool_output = {
    "stdout": "상관관계는 0.85입니다.",
    "stderr": None,
    "artifacts": {"type": "image", "base64_data": "/9j/4gIc..."}
}

tool_message = ToolMessage(
    content=tool_output["stdout"],  # 모델에 전달
    artifact=tool_output,            # 프로그래밍 방식으로만 접근
    tool_call_id="call_123",
    name="analyze_data"
)
```

### 2-6. RemoveMessage - 메시지 삭제 (LangGraph)

LangGraph에서 대화 히스토리 관리에 사용.​

```python
from langchain_core.messages import HumanMessage, RemoveMessage

# 특정 메시지 ID로 삭제
messages = [
    HumanMessage(content="첫 번째 질문", id="msg_1"),
    HumanMessage(content="두 번째 질문", id="msg_2")
]

# msg_1 삭제
remove_msg = RemoveMessage(id="msg_1")
```

## 3. Content와 Content Blocks <a href="#id-3-content-content-blocks" id="id-3-content-content-blocks"></a>

### 3-1. 기본 Content 형식

메시지 content는 세 가지 형식을 지원​

```python
from langchain.messages import HumanMessage

# 1. 문자열 (가장 간단)
msg1 = HumanMessage(content="Hello, how are you?")

# 2. Provider-native 형식 (OpenAI)
msg2 = HumanMessage(content=[
    {"type": "text", "text": "이미지를 설명해주세요."},
    {"type": "image_url", "image_url": {"url": "https://example.com/image.jpg"}}
])

# 3. LangChain 표준 content_blocks
msg3 = HumanMessage(content_blocks=[
    {"type": "text", "text": "이미지를 설명해주세요."},
    {"type": "image", "url": "https://example.com/image.jpg"}
])
```

### 3-2. Multimodal Content - 이미지

OpenAI는 멀티모달 입력을 지원.​

```python
from langchain.messages import HumanMessage
from langchain.chat_models import init_chat_model

# OpenAI 이미지 입력
model = init_chat_model("openai:gpt-4o")

# URL에서 이미지
message = HumanMessage(content=[
    {"type": "text", "text": "이 이미지에 무엇이 보이나요?"},
    {"type": "image_url", "image_url": {"url": "https://example.com/cat.jpg"}}
])

response = model.invoke([message])
print(response.content)
```

**Base64 인코딩 이미지**:​

```python
import base64
from langchain.messages import HumanMessage

# 이미지 파일을 base64로 인코딩
with open("image.jpg", "rb") as image_file:
    image_data = base64.b64encode(image_file.read()).decode()

message = HumanMessage(content=[
    {"type": "text", "text": "이 이미지를 분석해주세요."},
    {
        "type": "image_url",
        "image_url": {
            "url": f"data:image/jpeg;base64,{image_data}"
        }
    }
])

model = init_chat_model("openai:gpt-4o")
response = model.invoke([message])
```

### 3-3. Standard Content Blocks (v1.0)

LangChain v1.0는 제공자 간 표준화된 content blocks를 도입.​

```python
from langchain.chat_models import init_chat_model

model = init_chat_model("openai:gpt-4o")
response = model.invoke("Explain AI")

# 표준화된 content_blocks 접근
for block in response.content_blocks:
    if block["type"] == "reasoning":
        print(f"추론: {block.get('reasoning')}")
    elif block["type"] == "text":
        print(f"텍스트: {block.get('text')}")
```

## 4. 메시지 입력 형식 <a href="#id-4" id="id-4"></a>

### 4-1. LangChain Messages 형식

```python
from langchain.messages import SystemMessage, HumanMessage, AIMessage
from langchain.chat_models import init_chat_model

model = init_chat_model("openai:gpt-4o")

messages = [
    SystemMessage("You are a helpful assistant."),
    HumanMessage("What is Python?"),
    AIMessage("Python is a programming language."),
    HumanMessage("Tell me more.")
]

response = model.invoke(messages)
```

### 4-2. Dictionary (OpenAI) 형식

OpenAI 형식의 딕셔너리도 지원합니다.​

```python
from langchain.chat_models import init_chat_model

model = init_chat_model("openai:gpt-4o")

messages = [
    {"role": "system", "content": "You are a poetry expert"},
    {"role": "user", "content": "Write a haiku about spring"},
    {"role": "assistant", "content": "Cherry blossoms bloom..."},
    {"role": "user", "content": "Write another one"}
]

response = model.invoke(messages)
print(response.content)
```

### 4-3. 단일 문자열 (간단한 질문)

```python
from langchain.chat_models import init_chat_model

# 자동으로 HumanMessage로 변환됨
model = init_chat_model("openai:gpt-4o")
response = model.invoke("What is 2+2?")
print(response.content)

# Ollama도 동일
model = init_chat_model("ollama:llama3.1")
response = model.invoke("Tell me a joke")
print(response.content)
```

## 5. 실전 대화 패턴 <a href="#id-5" id="id-5"></a>

### 5-1. 기본 Q\&A 패턴

```python
from langchain.messages import HumanMessage
from langchain.chat_models import init_chat_model

# OpenAI
model = init_chat_model("openai:gpt-4o", temperature=0)
response = model.invoke([
    HumanMessage(content="LangChain이란 무엇인가요?")
])
print(response.content)
```

### 5-2. Multi-turn 대화 패턴

```python
from langchain.messages import SystemMessage, HumanMessage, AIMessage
from langchain.chat_models import init_chat_model

model = init_chat_model("ollama:llama3.1", temperature=0.7)

# 대화 히스토리 관리
conversation = [
    SystemMessage(content="You are a Python tutor."),
    HumanMessage(content="What are decorators?"),
]

# 첫 번째 응답
response1 = model.invoke(conversation)
conversation.append(AIMessage(content=response1.content))

# 후속 질문
conversation.append(HumanMessage(content="Can you show me an example?"))
response2 = model.invoke(conversation)
conversation.append(AIMessage(content=response2.content))

# 전체 대화 출력
for msg in conversation:
    print(f"{msg.__class__.__name__}: {msg.content}\n")
```

### 5-3. System Prompt 활용 패턴

```python
from langchain.messages import SystemMessage, HumanMessage
from langchain.chat_models import init_chat_model

# OpenAI - 전문가 페르소나
model = init_chat_model("openai:gpt-4o")

system_prompt = SystemMessage(content="""
당신은 20년 경력의 시니어 백엔드 개발자입니다.
- 항상 보안을 최우선으로 고려합니다
- 코드 예제는 Python으로 작성합니다
- 성능과 확장성을 고려한 답변을 제공합니다
""")

response = model.invoke([
    system_prompt,
    HumanMessage(content="사용자 인증 시스템을 어떻게 설계해야 하나요?")
])

print(response.content)
```

### 5-4. Few-shot Learning 패턴

```python
from langchain.messages import SystemMessage, HumanMessage, AIMessage
from langchain.chat_models import init_chat_model

model = init_chat_model("openai:gpt-4o", temperature=0)

messages = [
    SystemMessage(content="You are a sentiment classifier. Classify text as positive, negative, or neutral."),
    HumanMessage(content="I love this product!"),
    AIMessage(content="Positive"),
    HumanMessage(content="This is terrible."),
    AIMessage(content="Negative"),
    HumanMessage(content="It's okay."),
    AIMessage(content="Neutral"),
    HumanMessage(content="Best purchase ever!")
]

response = model.invoke(messages)
print(response.content)  # 출력: Positive
```

### 5-5. Streaming 대화 패턴

```python
from langchain.messages import HumanMessage
from langchain.chat_models import init_chat_model

# OpenAI Streaming
model = init_chat_model("openai:gpt-4o")

print("AI: ", end="", flush=True)
for chunk in model.stream([HumanMessage(content="Explain recursion")]):
    print(chunk.content, end="", flush=True)
print("\n")

# Ollama Streaming
model = init_chat_model("ollama:llama3.1")

print("AI: ", end="", flush=True)
for chunk in model.stream([HumanMessage(content="What is LangChain?")]):
    print(chunk.content, end="", flush=True)
print("\n")
```

### 5-6. 대화 히스토리 관리 패턴

```python
from langchain.messages import SystemMessage, HumanMessage, AIMessage
from langchain.chat_models import init_chat_model

class ConversationManager:
    def __init__(self, model_name="openai:gpt-4o", system_prompt=None):
        self.model = init_chat_model(model_name, temperature=0.7)
        self.messages = []
        
        if system_prompt:
            self.messages.append(SystemMessage(content=system_prompt))
    
    def chat(self, user_input: str) -> str:
        # 사용자 메시지 추가
        self.messages.append(HumanMessage(content=user_input))
        
        # 모델 호출
        response = self.model.invoke(self.messages)
        
        # AI 응답 추가
        self.messages.append(AIMessage(content=response.content))
        
        return response.content
    
    def get_history(self):
        return self.messages

# 사용 예제
chat = ConversationManager(
    model_name="ollama:llama3.1",
    system_prompt="You are a helpful coding assistant."
)

print(chat.chat("What is a list comprehension in Python?"))
print(chat.chat("Show me an example"))
print(chat.chat("Can it be nested?"))

# 대화 히스토리 확인
for msg in chat.get_history():
    print(f"{msg.__class__.__name__}: {msg.content[:50]}...")
```

## 6. invoke vs stream 차이점 <a href="#id-6-invoke-vs-stream" id="id-6-invoke-vs-stream"></a>

### 6-1. invoke - 전체 응답 반환

```python
from langchain.chat_models import init_chat_model

model = init_chat_model("openai:gpt-4o")

# 전체 응답을 한 번에 받음
response = model.invoke("Explain quantum computing")
print(response.content)
print(f"Type: {type(response)}")  # AIMessage
```

### 6-2. stream - 실시간 스트리밍

```python
from langchain.chat_models import init_chat_model

model = init_chat_model("openai:gpt-4o")

# 토큰 단위로 실시간 출력
for chunk in model.stream("Explain quantum computing"):
    print(chunk.content, end="", flush=True)
    # Type: AIMessageChunk
```

### 6.3 비동기 처리

```python
import asyncio
from langchain.chat_models import init_chat_model

async def process_multiple_queries():
    model = init_chat_model("openai:gpt-4o")
    
    # 비동기 invoke
    response = await model.ainvoke("What is AI?")
    print(response.content)
    
    # 비동기 stream
    async for chunk in model.astream("What is ML?"):
        print(chunk.content, end="", flush=True)

asyncio.run(process_multiple_queries())
```

## 7. 고급 패턴 <a href="#id-7" id="id-7"></a>

### 7-1. Tool Calling 전체 워크플로우

```python
from langchain.chat_models import init_chat_model
from langchain.messages import HumanMessage, AIMessage, ToolMessage

def search_web(query: str) -> str:
    """웹을 검색합니다."""
    return f"'{query}'에 대한 검색 결과: 관련 정보..."

def calculate(expression: str) -> str:
    """수학 계산을 수행합니다."""
    return str(eval(expression))

# OpenAI 모델에 도구 바인딩
model = init_chat_model("openai:gpt-4o")
model_with_tools = model.bind_tools([search_web, calculate])

# 대화 시작
messages = [HumanMessage(content="2024년 파리 올림픽에서 한국은 금메달 몇 개를 땄나요?")]

# 1단계: 모델이 도구 사용 결정
ai_response = model_with_tools.invoke(messages)
messages.append(ai_response)

# 2단계: 도구 실행
tool_calls = ai_response.tool_calls
for tool_call in tool_calls:
    tool_name = tool_call['name']
    tool_args = tool_call['args']
    tool_id = tool_call['id']
    
    # 도구 실행
    if tool_name == "search_web":
        result = search_web(**tool_args)
    elif tool_name == "calculate":
        result = calculate(**tool_args)
    
    # ToolMessage 추가
    messages.append(ToolMessage(
        content=result,
        tool_call_id=tool_id
    ))

# 3단계: 최종 응답 생성
final_response = model.invoke(messages)
print(final_response.content)
```

### 7-2. Prompt Template과 Messages

```python
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import init_chat_model
from langchain_core.output_parsers import StrOutputParser

# OpenAI 체인
model = init_chat_model("openai:gpt-4o", temperature=0)

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a {role}. Always answer in {language}."),
    ("human", "{input}")
])

chain = prompt | model | StrOutputParser()

response = chain.invoke({
    "role": "travel expert",
    "language": "Korean",
    "input": "파리의 명소를 추천해주세요."
})

print(response)
```

### 7-3. LangGraph MessagesState 패턴

```python
from typing import Annotated
from langchain_core.messages import AnyMessage
from langgraph.graph.message import add_messages
from typing_extensions import TypedDict

# add_messages reducer 사용
class GraphState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]

# 또는 MessagesState 상속
from langgraph.graph import MessagesState

class State(MessagesState):
    documents: list[str]
```

## 8. 실전 통합 예제 <a href="#id-8" id="id-8"></a>

### 8-1 OpenAI + Ollama 통합 챗봇

```python
from langchain.messages import SystemMessage, HumanMessage, AIMessage
from langchain.chat_models import init_chat_model

class MultiModelChatbot:
    def __init__(self):
        self.openai_model = init_chat_model("openai:gpt-4o", temperature=0.7)
        self.ollama_model = init_chat_model("ollama:llama3.1", temperature=0.7)
        self.conversation = [
            SystemMessage(content="You are a helpful assistant.")
        ]
    
    def chat_with_openai(self, message: str) -> str:
        """OpenAI 모델로 대화"""
        self.conversation.append(HumanMessage(content=message))
        response = self.openai_model.invoke(self.conversation)
        self.conversation.append(AIMessage(content=response.content))
        return response.content
    
    def chat_with_ollama(self, message: str) -> str:
        """Ollama 모델로 대화"""
        self.conversation.append(HumanMessage(content=message))
        response = self.ollama_model.invoke(self.conversation)
        self.conversation.append(AIMessage(content=response.content))
        return response.content
    
    def stream_response(self, message: str, use_openai=True):
        """스트리밍 응답"""
        self.conversation.append(HumanMessage(content=message))
        
        model = self.openai_model if use_openai else self.ollama_model
        
        full_response = ""
        for chunk in model.stream(self.conversation):
            print(chunk.content, end="", flush=True)
            full_response += chunk.content
        
        self.conversation.append(AIMessage(content=full_response))
        print("\n")
        return full_response

# 사용 예제
bot = MultiModelChatbot()

print("=== OpenAI 응답 ===")
print(bot.chat_with_openai("Python의 장점은?"))

print("\n=== Ollama 응답 ===")
print(bot.chat_with_ollama("Java의 장점은?"))

print("\n=== 스트리밍 응답 ===")
bot.stream_response("두 언어를 비교해주세요.", use_openai=True)
```

### 8-2 완전한 RAG 패턴

```python
from langchain.chat_models import init_chat_model
from langchain.messages import SystemMessage, HumanMessage, AIMessage, ToolMessage

def retrieve_documents(query: str) -> list[str]:
    """문서 검색 도구"""
    # 실제로는 벡터 DB에서 검색
    return [
        "LangChain은 LLM 애플리케이션을 구축하는 프레임워크입니다.",
        "Messages는 LangChain의 핵심 개념입니다."
    ]

# OpenAI 모델 초기화 및 도구 바인딩
model = init_chat_model("openai:gpt-4o")
model_with_tools = model.bind_tools([retrieve_documents])

# 사용자 질문
user_query = "LangChain이 뭐야?"
messages = [
    SystemMessage(content="You are a helpful AI assistant. Use the retrieve_documents tool to answer questions."),
    HumanMessage(content=user_query)
]

# 1단계: 모델이 문서 검색 결정
response = model_with_tools.invoke(messages)
messages.append(response)

# 2단계: 문서 검색 실행
if response.tool_calls:
    for tool_call in response.tool_calls:
        documents = retrieve_documents(**tool_call['args'])
        
        # 검색된 문서를 ToolMessage로 전달
        messages.append(ToolMessage(
            content="\n".join(documents),
            tool_call_id=tool_call['id']
        ))

# 3단계: 문서 기반 최종 답변 생성
final_response = model.invoke(messages)
print(final_response.content)
```

## 9. 모범 사례 및 팁 <a href="#id-9" id="id-9"></a>

### 9-1 Message 관리

```python
from langchain.messages import SystemMessage, HumanMessage, AIMessage

# ✅ 좋은 예: 명확한 역할 구분
messages = [
    SystemMessage(content="You are a coding assistant."),
    HumanMessage(content="Explain variables"),
    AIMessage(content="Variables store data..."),
    HumanMessage(content="Show an example")
]

# ❌ 나쁜 예: SystemMessage가 중간에 위치
messages = [
    HumanMessage(content="Hello"),
    SystemMessage(content="You are helpful"),  # 잘못된 위치
    AIMessage(content="Hi there")
]
```

### 9-2 대화 히스토리 크기 관리

```python
from langchain.messages import SystemMessage, HumanMessage, AIMessage

def trim_messages(messages, max_messages=10):
    """대화 히스토리를 최근 N개로 제한"""
    # SystemMessage는 유지
    system_msgs = [m for m in messages if isinstance(m, SystemMessage)]
    other_msgs = [m for m in messages if not isinstance(m, SystemMessage)]
    
    # 최근 메시지만 유지
    trimmed = other_msgs[-max_messages:]
    
    return system_msgs + trimmed

# 사용
conversation = [
    SystemMessage(content="You are helpful"),
    HumanMessage(content="Q1"), AIMessage(content="A1"),
    HumanMessage(content="Q2"), AIMessage(content="A2"),
    # ... 많은 메시지
]

conversation = trim_messages(conversation, max_messages=6)
```

### 9-3 에러 핸들링

```python
from langchain.chat_models import init_chat_model
from langchain.messages import HumanMessage

def safe_invoke(model, messages, max_retries=3):
    """재시도 로직이 있는 안전한 invoke"""
    for attempt in range(max_retries):
        try:
            response = model.invoke(messages)
            return response
        except Exception as e:
            print(f"Attempt {attempt + 1} failed: {e}")
            if attempt == max_retries - 1:
                raise
            continue

# 사용
model = init_chat_model("openai:gpt-4o")
response = safe_invoke(model, [HumanMessage(content="Hello")])
```

## 10. 요약 <a href="#id-10" id="id-10"></a>

LangChain v1.0의 Messages 시스템은 다음과 같은 특징을 제공한다:​

* **주요 메시지 타입**: SystemMessage, HumanMessage, AIMessage, AIMessageChunk, ToolMessage, RemoveMessage
* **통합 초기화**: `init_chat_model`로 OpenAI와 Ollama를 동일한 방식으로 사용
* **표준 Content Blocks**: 제공자 간 통일된 content 표현 (v1.0 신규)
* **Multimodal 지원**: 텍스트, 이미지, 오디오 등 다양한 입력 형식
* **스트리밍**: invoke와 stream 메서드로 동기/실시간 응답 선택
* **도구 통합**: ToolMessage를 통한 외부 도구 연계
