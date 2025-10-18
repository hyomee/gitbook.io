# Structured Output

Structured output은 LLM이 자연어 대신 JSON, Pydantic 객체, TypedDict 등 정형화된 형식으로 응답하도록 하는 기능으로  LangChain v1.0에서는 `init_chat_model`을 통해 모델을 초기화하고 `with_structured_output()` 메서드를 사용하여 구조화된 출력을 설정할 수 있다.​

## 1. 기본 설정 (init\_chat\_model 사용) <a href="#id-1---initchatmodel" id="id-1---initchatmodel"></a>

### 1-1. OpenAI 모델 초기화

```python
from langchain.chat_models import init_chat_model

# OpenAI 모델 초기화
llm = init_chat_model(
    "gpt-4o", 
    model_provider="openai",
    temperature=0
)

# 또는 단축 형식
llm = init_chat_model("openai:gpt-4o", temperature=0)
```

### 1-2. Ollama 모델 초기화

```python
from langchain.chat_models import init_chat_model

# Ollama 모델 초기화
llm = init_chat_model(
    "llama3.1",
    model_provider="ollama",
    temperature=0
)

# 또는 단축 형식
llm = init_chat_model("ollama:llama3.1", temperature=0)
```

## 2. Pydantic 스키마를 사용한 Structured Output <a href="#id-2-pydantic---structured-output" id="id-2-pydantic---structured-output"></a>

### 2-1. 기본 패턴

```python
from pydantic import BaseModel, Field
from typing import Optional
from langchain.chat_models import init_chat_model

# Pydantic 스키마 정의
class Joke(BaseModel):
    """Joke to tell user."""
    setup: str = Field(description="The setup of the joke")
    punchline: str = Field(description="The punchline to the joke")
    rating: Optional[int] = Field(
        default=None, 
        description="How funny the joke is, from 1 to 10"
    )

# OpenAI로 structured output 설정
llm = init_chat_model("openai:gpt-4o", temperature=0)
structured_llm = llm.with_structured_output(Joke)

# 실행
result = structured_llm.invoke("Tell me a joke about cats")
print(result)
# 출력: Joke(setup='Why was the cat sitting on the computer?', 
#             punchline='Because it wanted to keep an eye on the mouse!', 
#             rating=7)
```

### 2-2. Ollama with Pydantic&#x20;

```python
from pydantic import BaseModel, Field
from langchain.chat_models import init_chat_model

class Person(BaseModel):
    """Information about a person."""
    name: str = Field(description="The name of the person")
    age: int = Field(description="The age of the person")
    email: str = Field(description="The email address")

# Ollama 모델 초기화 (tool calling 지원 모델 필요)
llm = init_chat_model("ollama:llama3.1", temperature=0)
structured_llm = llm.with_structured_output(Person)

# 실행
result = structured_llm.invoke(
    "John is 30 years old and his email is john@example.com"
)
print(result)
# 출력: Person(name='John', age=30, email='john@example.com')
```

## 3. TypedDict를 사용한 Structured Output <a href="#id-3-typeddict--structured-output" id="id-3-typeddict--structured-output"></a>

### 3-1. 기본 패턴

```python
from typing import Optional
from typing_extensions import Annotated, TypedDict
from langchain.chat_models import init_chat_model

# TypedDict 스키마 정의
class Joke(TypedDict):
    """Joke to tell user."""
    setup: Annotated[str, ..., "The setup of the joke"]
    punchline: Annotated[str, ..., "The punchline of the joke"]
    rating: Annotated[Optional[int], None, "How funny the joke is, from 1 to 10"]

# OpenAI로 structured output 설정
llm = init_chat_model("openai:gpt-4o", temperature=0)
structured_llm = llm.with_structured_output(Joke)

# 실행
result = structured_llm.invoke("Tell me a joke about dogs")
print(result)
# 출력: {'setup': 'Why did the dog sit in the shade?', 
#        'punchline': 'Because it didn't want to be a hot dog!', 
#        'rating': 8}
```

### 3-2. TypedDict 스트리밍

```python
from typing_extensions import Annotated, TypedDict
from langchain.chat_models import init_chat_model

class Joke(TypedDict):
    """Joke to tell user."""
    setup: Annotated[str, ..., "The setup of the joke"]
    punchline: Annotated[str, ..., "The punchline of the joke"]

llm = init_chat_model("openai:gpt-4o", temperature=0)
structured_llm = llm.with_structured_output(Joke)

# 스트리밍 실행 (TypedDict만 가능)
for chunk in structured_llm.stream("Tell me a joke about cats"):
    print(chunk)
# 출력: 
# {}
# {'setup': ''}
# {'setup': 'Why'}
# {'setup': 'Why was'}
# ...
```

## 4. JSON Schema를 사용한 Structured Output <a href="#id-4-json-schema--structured-output" id="id-4-json-schema--structured-output"></a>

```python
from langchain.chat_models import init_chat_model

# JSON Schema 정의
json_schema = {
    "title": "joke",
    "description": "Joke to tell user.",
    "type": "object",
    "properties": {
        "setup": {
            "type": "string",
            "description": "The setup of the joke",
        },
        "punchline": {
            "type": "string",
            "description": "The punchline to the joke",
        },
        "rating": {
            "type": "integer",
            "description": "How funny the joke is, from 1 to 10",
            "default": None,
        },
    },
    "required": ["setup", "punchline"],
}

# OpenAI로 structured output 설정
llm = init_chat_model("openai:gpt-4o", temperature=0)
structured_llm = llm.with_structured_output(json_schema)

# 실행
result = structured_llm.invoke("Tell me a joke about cats")
print(result)
# 출력: {'setup': 'Why was the cat sitting on the computer?', 
#        'punchline': 'Because it wanted to keep an eye on the mouse!', 
#        'rating': 7}
```

### 4-1. Ollama JSON Mode 사용 <a href="#id-5-ollama-json-mode" id="id-5-ollama-json-mode"></a>

Ollama는 tool calling과 별도로 JSON mode를 지원합니다. JSON mode는 모델이 JSON 형식으로만 응답하도록 강제하지만, 스키마 검증은 하지 않는다.​

```python
from pydantic import BaseModel, Field
from langchain.chat_models import init_chat_model

class AnswerWithJustification(BaseModel):
    answer: str = Field(description="The answer to the question")
    justification: str = Field(description="The justification for the answer")

# Ollama 모델 초기화
llm = init_chat_model("ollama:llama3.1", temperature=0)

# JSON mode로 structured output 설정
structured_llm = llm.with_structured_output(
    AnswerWithJustification,
    method="json_mode",
    include_raw=True
)

# 실행 (프롬프트에 JSON 형식 명시 필요)
result = structured_llm.invoke(
    "Answer the following question. "
    "Make sure to return a JSON blob with keys 'answer' and 'justification'.\n\n"
    "What's heavier a pound of bricks or a pound of feathers?"
)

print(result)
# 출력:
# {
#     'raw': AIMessage(content='{"answer": "They weigh the same", ...}'),
#     'parsed': AnswerWithJustification(
#         answer='They weigh the same', 
#         justification='Both weigh one pound.'
#     ),
#     'parsing_error': None
# }
```

**JSON mode 중요 사항:**

* 프롬프트에 JSON 형식으로 응답하라는 명시적 지시 필요​
* 스키마는 출력 파싱에만 사용되며 모델에 전달되지 않음​
* `include_raw=True`를 사용하면 원본 응답, 파싱된 결과, 오류를 함께 반환​

## 5. 복수 스키마 선택 (Union 타입) <a href="#id-6----union" id="id-6----union"></a>

### 5-1. Pydantic Union 패턴

```python
from pydantic import BaseModel, Field
from typing import Union, Optional
from langchain.chat_models import init_chat_model

class Joke(BaseModel):
    """Joke to tell user."""
    setup: str = Field(description="The setup of the joke")
    punchline: str = Field(description="The punchline to the joke")
    rating: Optional[int] = Field(
        default=None, 
        description="How funny the joke is, from 1 to 10"
    )

class ConversationalResponse(BaseModel):
    """Respond in a conversational manner. Be kind and helpful."""
    response: str = Field(description="A conversational response to the user's query")

class FinalResponse(BaseModel):
    final_output: Union[Joke, ConversationalResponse]

# OpenAI로 설정
llm = init_chat_model("openai:gpt-4o", temperature=0)
structured_llm = llm.with_structured_output(FinalResponse)

# 농담 요청
result1 = structured_llm.invoke("Tell me a joke about cats")
print(result1)
# 출력: FinalResponse(final_output=Joke(setup='Why was the cat...', ...))

# 일반 대화 요청
result2 = structured_llm.invoke("How are you today?")
print(result2)
# 출력: FinalResponse(final_output=ConversationalResponse(response="I'm here to help!"))
```

### 5-2. TypedDict Union 패턴

```python
from typing import Optional, Union
from typing_extensions import Annotated, TypedDict
from langchain.chat_models import init_chat_model

class Joke(TypedDict):
    """Joke to tell user."""
    setup: Annotated[str, ..., "The setup of the joke"]
    punchline: Annotated[str, ..., "The punchline of the joke"]
    rating: Annotated[Optional[int], None, "How funny the joke is, from 1 to 10"]

class ConversationalResponse(TypedDict):
    """Respond in a conversational manner."""
    response: Annotated[str, ..., "A conversational response to the user's query"]

class FinalResponse(TypedDict):
    final_output: Union[Joke, ConversationalResponse]

llm = init_chat_model("openai:gpt-4o", temperature=0)
structured_llm = llm.with_structured_output(FinalResponse)

result = structured_llm.invoke("Tell me a joke about cats")
print(result)
# 출력: {'final_output': {'setup': 'Why was the cat...', ...}}
```

## 6. Few-shot 프롬프팅 <a href="#id-7-few-shot" id="id-7-few-shot"></a>

### 6-1. 시스템 메시지 방식

```python
from langchain_core.prompts import ChatPromptTemplate
from pydantic import BaseModel, Field
from langchain.chat_models import init_chat_model

class Joke(BaseModel):
    """Joke to tell user."""
    setup: str = Field(description="The setup of the joke")
    punchline: str = Field(description="The punchline to the joke")
    rating: int = Field(description="How funny the joke is, from 1 to 10")

llm = init_chat_model("openai:gpt-4o", temperature=0)
structured_llm = llm.with_structured_output(Joke)

system = """You are a hilarious comedian. Your specialty is knock-knock jokes.
Return a joke which has the setup and the final punchline.

Here are some examples of jokes:

example_user: Tell me a joke about planes
example_assistant: {"setup": "Why don't planes ever get tired?", "punchline": "Because they have rest wings!", "rating": 2}

example_user: Tell me another joke about planes
example_assistant: {"setup": "Cargo", "punchline": "Cargo 'vroom vroom', but planes go 'zoom zoom'!", "rating": 10}

example_user: Now about caterpillars
example_assistant: {"setup": "Caterpillar", "punchline": "Caterpillar really slow, but watch me turn into a butterfly!", "rating": 5}"""

prompt = ChatPromptTemplate.from_messages([
    ("system", system), 
    ("human", "{input}")
])

few_shot_structured_llm = prompt | structured_llm
result = few_shot_structured_llm.invoke("what's something funny about woodpeckers")
print(result)
```

### 6-2. Tool Calling 방식

```python
from langchain_core.messages import AIMessage, HumanMessage, ToolMessage
from langchain_core.prompts import ChatPromptTemplate
from langchain.chat_models import init_chat_model

examples = [
    HumanMessage("Tell me a joke about planes", name="example_user"),
    AIMessage(
        "",
        name="example_assistant",
        tool_calls=[{
            "name": "joke",
            "args": {
                "setup": "Why don't planes ever get tired?",
                "punchline": "Because they have rest wings!",
                "rating": 2,
            },
            "id": "1",
        }],
    ),
    ToolMessage("", tool_call_id="1"),
    HumanMessage("Tell me another joke about planes", name="example_user"),
    AIMessage(
        "",
        name="example_assistant",
        tool_calls=[{
            "name": "joke",
            "args": {
                "setup": "Cargo",
                "punchline": "Cargo 'vroom vroom', but planes go 'zoom zoom'!",
                "rating": 10,
            },
            "id": "2",
        }],
    ),
    ToolMessage("", tool_call_id="2"),
]

system = """You are a hilarious comedian. Your specialty is knock-knock jokes.
Return a joke which has the setup and the punchline."""

prompt = ChatPromptTemplate.from_messages([
    ("system", system), 
    ("placeholder", "{examples}"), 
    ("human", "{input}")
])

llm = init_chat_model("openai:gpt-4o", temperature=0)
structured_llm = llm.with_structured_output(Joke)
few_shot_structured_llm = prompt | structured_llm

result = few_shot_structured_llm.invoke({
    "input": "crocodiles", 
    "examples": examples
})
print(result)
```

## 7. 고급 패턴 <a href="#id-8" id="id-8"></a>

### 7.1 메서드 지정 (method 파라미터)

```python
from langchain.chat_models import init_chat_model

# JSON Schema 메서드 사용
llm = init_chat_model("openai:gpt-4o", temperature=0)
structured_llm = llm.with_structured_output(None, method="json_schema")

result = structured_llm.invoke(
    "Tell me a joke about cats, respond in JSON with `setup` and `punchline` keys"
)
print(result)
# 출력: {'setup': 'Why was the cat sitting on the computer?', 
#        'punchline': 'Because it wanted to keep an eye on the mouse!'}
```

### 7.2 Raw Output 포함

```python
from pydantic import BaseModel, Field
from langchain.chat_models import init_chat_model

class Joke(BaseModel):
    setup: str = Field(description="The setup of the joke")
    punchline: str = Field(description="The punchline to the joke")
    rating: int = Field(description="How funny the joke is, from 1 to 10")

llm = init_chat_model("openai:gpt-4o", temperature=0)
structured_llm = llm.with_structured_output(Joke, include_raw=True)

result = structured_llm.invoke("Tell me a joke about cats")
print(result)
# 출력:
# {
#     'raw': AIMessage(content='', tool_calls=[...]),
#     'parsed': Joke(setup='Why was the cat...', punchline='...', rating=7),
#     'parsing_error': None
# }
```

### 7.3 도구와 함께 사용

**중요:** 도구를 먼저 바인딩한 후 `with_structured_output`을 적용해야 한다.​

```python
from pydantic import BaseModel, Field
from langchain.chat_models import init_chat_model

class SearchResult(BaseModel):
    """Structured search result."""
    query: str = Field(description="The search query")
    findings: str = Field(description="Summary of findings")

# 도구 정의
search_tool = {
    "type": "function",
    "function": {
        "name": "web_search",
        "description": "Search the web for information",
        "parameters": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Search query"}
            },
            "required": ["query"],
        },
    },
}

# 올바른 순서: 1. 도구 바인딩 → 2. Structured output 적용
llm = init_chat_model("openai:gpt-4o", temperature=0)
llm_with_search = llm.bind_tools([search_tool])
structured_search_llm = llm_with_search.with_structured_output(SearchResult)

# 잘못된 순서 (오류 발생)
# structured_llm = llm.with_structured_output(SearchResult)
# broken_llm = structured_llm.bind_tools([search_tool])  # 오류!

result = structured_search_llm.invoke("Search for latest AI research and summarize")
```

### 7-4. Ollama Tool Calling <a href="#id-9-ollama-tool-calling" id="id-9-ollama-tool-calling"></a>

Ollama는 `llama3.1`, `mistral`, `command-r` 등 일부 모델에서 tool calling을 지원한다.​

```python
from pydantic import BaseModel, Field
from langchain.chat_models import init_chat_model

class GetWeather(BaseModel):
    """Get the current weather in a given location"""
    location: str = Field(..., description="The city and state, e.g. San Francisco, CA")

class GetPopulation(BaseModel):
    """Get the current population in a given location"""
    location: str = Field(..., description="The city and state, e.g. San Francisco, CA")

# Ollama 모델 초기화 및 도구 바인딩
llm = init_chat_model("ollama:llama3.1", temperature=0)
llm_with_tools = llm.bind_tools([GetWeather, GetPopulation])

# 실행
result = llm_with_tools.invoke("What's the weather like in San Francisco?")
print(result.tool_calls)
# 출력: [{'name': 'GetWeather', 'args': {'location': 'San Francisco, CA'}, ...}]
```

**Ollama Tool Calling 지원 모델:**

* Llama 3.1​
* Mistral Nemo​
* Firefunction v2​
* Command-R / Command-R+​
* Gemma 2 (일부 버전)
* Qwen 2.5

## 8. 전체 통합 예제 <a href="#id-10" id="id-10"></a>

### 8-1. OpenAI 완전 예제

```python
from pydantic import BaseModel, Field
from typing import Optional, List
from langchain.chat_models import init_chat_model
from langchain_core.prompts import ChatPromptTemplate

# 1. 복잡한 중첩 스키마 정의
class Address(BaseModel):
    street: str = Field(description="Street address")
    city: str = Field(description="City name")
    country: str = Field(description="Country name")

class Person(BaseModel):
    """Information about a person."""
    name: str = Field(description="Full name of the person")
    age: int = Field(description="Age in years")
    email: Optional[str] = Field(default=None, description="Email address")
    address: Address = Field(description="Residential address")
    hobbies: List[str] = Field(description="List of hobbies")

# 2. 모델 초기화 및 structured output 설정
llm = init_chat_model("openai:gpt-4o", temperature=0)
structured_llm = llm.with_structured_output(Person)

# 3. 프롬프트 템플릿 설정
prompt = ChatPromptTemplate.from_messages([
    ("system", "Extract person information from the text."),
    ("human", "{text}")
])

# 4. 체인 생성
chain = prompt | structured_llm

# 5. 실행
text = """
John Smith is 35 years old and lives at 123 Main Street in New York, USA.
His email is john.smith@email.com. He enjoys reading, hiking, and photography.
"""

result = chain.invoke({"text": text})
print(result)
# 출력: Person(
#     name='John Smith', 
#     age=35, 
#     email='john.smith@email.com',
#     address=Address(street='123 Main Street', city='New York', country='USA'),
#     hobbies=['reading', 'hiking', 'photography']
# )
```

### 8-2. Ollama 완전 예제

```python
from pydantic import BaseModel, Field
from typing import Optional
from langchain.chat_models import init_chat_model
from langchain_core.prompts import ChatPromptTemplate

# 1. 스키마 정의
class ProductReview(BaseModel):
    """Product review analysis"""
    product_name: str = Field(description="Name of the product")
    rating: int = Field(description="Rating from 1 to 5")
    sentiment: str = Field(description="Sentiment: positive, negative, or neutral")
    summary: str = Field(description="Brief summary of the review")

# 2. Ollama 모델 초기화 (JSON mode 사용)
llm = init_chat_model("ollama:llama3.1", temperature=0)
structured_llm = llm.with_structured_output(ProductReview, method="json_mode")

# 3. 프롬프트 템플릿 (JSON 형식 명시)
prompt = ChatPromptTemplate.from_messages([
    ("system", 
     "Analyze the product review and return a JSON with keys: "
     "'product_name', 'rating', 'sentiment', 'summary'."),
    ("human", "{review}")
])

# 4. 체인 생성
chain = prompt | structured_llm

# 5. 실행
review = """
I bought the SuperWidget 3000 last week and I'm absolutely thrilled!
The build quality is excellent and it does exactly what it promises.
Worth every penny. 5 stars from me!
"""

result = chain.invoke({"review": review})
print(result)
# 출력: ProductReview(
#     product_name='SuperWidget 3000',
#     rating=5,
#     sentiment='positive',
#     summary='Excellent build quality, performs as promised, worth the price'
# )
```

## 9. 주요 차이점 요약 <a href="#undefined" id="undefined"></a>

### OpenAI vs Ollama

| 특징               | OpenAI             | Ollama         |
| ---------------- | ------------------ | -------------- |
| **Tool Calling** | 네이티브 지원​           | 일부 모델만 지원​     |
| **JSON Mode**    | 지원​                | 지원 (v0.5+)​    |
| **기본 메서드**       | Tool calling (자동)​ | JSON mode 권장​  |
| **스키마 전달**       | 모델에 직접 전달​         | 파싱에만 사용​       |
| **프롬프트 명시**      | 불필요                | JSON 형식 명시 필요​ |
| **정확도**          | 매우 높음              | 모델에 따라 다름​     |

### Pydantic vs TypedDict

| 특징        | Pydantic     | TypedDict    |
| --------- | ------------ | ------------ |
| **검증**    | 자동 검증​       | 검증 없음​       |
| **반환 타입** | Pydantic 객체​ | 딕셔너리​        |
| **스트리밍**  | 불가능​         | 가능​          |
| **추천 사용** | 프로덕션 환경​     | 프로토타입, 스트리밍​ |

### 참고사항 <a href="#undefined" id="undefined"></a>

1. **모델 업데이트**: Ollama 모델은 `ollama pull <model>` 명령으로 최신 버전을 다운로드해야 한다​
2. **스키마 복잡도**: 복잡한 스키마일수록 few-shot 예제가 중요하다.​
3. **오류 처리**: `include_raw=True`를 사용하여 파싱 오류를 확인할 수 있다​
4. **Ollama 제한사항**: Ollama의 structured output은 토큰 샘플링을 제약하지만 모델이 스키마를 직접 보지는 못한다.
