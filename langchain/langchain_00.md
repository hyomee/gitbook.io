---
description: Langchain 맛보기
---

# 맛보기

Langchain 학습을 하기 전에 어떻게 동작하는지 살펴 보기로 합니다,

## 1. 라이브러리 설치

vscode를 실행하고 주피터 노트북 파일을 생성한 후 다음과 같이 입력하여 라이브러리를 설치 합니다.

* 파일명 : langchain.ipynb

```python
#LangChain사용을 위한 라이브러리 설치
%pip install langchain
%pip install langchain_community
```

## 2. 환경 설정

Open API를 사용하는 예제는 다른 곳에 많이 나오는 것을 인터넷을 통해서 확인 하고 여기서는 로컬에 설치 되어 있는 Ollama를 사용하는 방법으로 합니다.

```python
import os

# 로컬에 설치 되어 있는 아이피 주소
os.environ["OLLAMA_BASE_URI"] = "http://localhost:11434" 
os.environ["OLLAMA_BASE_URI_PORT"] = "11434"
os.environ["MODEL"] = "exaone3.5"  # qwen2.5:3b Replace with your actual API key
```

만약 Python을 사용하면 .env 파일에 작성을 하고 구글에서 제공하는 주피터 노트북을 사용하면 설정에 등록 하면 됩니다.

* 라이브러리 설치&#x20;

```python
#LangChain사용을 위한 라이브러리 설치
%pip install langchain
%pip install langchain_community
```

## 3.  모델 초기화

Ollama를 사용해서 모델 초기화 하는 방법은 다음과 같은 방법이 있습니다.

### 3-1. Ollama (from langchain.llms)

* 모델 유형: 일반적인 언어 모델(LLM).
* 특징:
  * 단순한 입력 텍스트에 대해 응답을 생성합니다.
  * 대화 컨텍스트를 유지하지 않습니다.
  * 주로 단일 질문-응답 작업에 적합합니다.

```python
import os
import json
from langchain.llms import Ollama

# Initialize the Ollama LLM
llm = Ollama(base_url=os.environ["OLLAMA_BASE_URI"], 
             model=os.environ["MODEL"], 
             temperature=0.5)

# Pretty print the llm object as JSON
print(json.dumps(llm.dict(), indent=4))

# Ollama LLM을 사용하여 질문에 대한 응답을 예측
response_predict = llm.predict("대한 민국의 수도는 어디인가요?")
print(response_predict)
```

### 3-2. ChatOllama (from langchain.chat\_models)

* 모델 유형: 대화형 언어 모델(Chat Model).
* 특징:
  * 대화 컨텍스트를 유지하며, 이전 메시지와의 연속성을 고려합니다.
  * 대화형 애플리케이션에 적합합니다.
  * 응답은 invoke 메서드를 통해 생성됩니다.

```python
from langchain.chat_models import ChatOllama

# Initialize the ChatOllama LLM
chat_llm = ChatOllama(base_url=os.environ["OLLAMA_BASE_URI"], 
                      model=os.environ["MODEL"], 
                      temperature=0.5)
 
# invoke 메서드는 ChatOllama 모델을 사용하여 입력된 질문에 대한 응답을 생성합니다.
response = chat_llm.invoke("하늘의 별에 대한 시를 만들어줘")
print(response.content)
```

## 5. 메세지

LangChain에서 제공하는 메시지 유형인 HumanMessage, AIMessage, SystemMessage, **FunctionMessage**는 언어 모델과의 상호작용에서 메시지의 출처와 역할을 명확히 구분하기 위해 사용됩니다. 각각의 메시지 유형은 대화의 맥락을 유지하거나 특정 작업을 수행하는 데 중요한 역할을 합니다.

### 5-1. HumanMessage

* 설명: 사람이 언어 모델에 전달하는 메시지를 나타냅니다.
* 용도: 사용자가 입력한 질문, 요청, 또는 명령을 모델에 전달할 때 사용됩니다.
*   예시:

    ```python
    from langchain.schema import HumanMessage

    message = HumanMessage(content="서울의 날씨는 어떤가요?")
    print(message.content)  # 출력: 서울의 날씨는 어떤가요?
    ```
* 특징:
  * 대화의 시작점이 되는 메시지.
  * 주로 사용자 입력을 표현.

### 5-2.  AIMessage

* 설명: AI(언어 모델)가 생성한 응답 메시지를 나타냅니다.
* 용도: 모델이 사용자 질문에 대해 생성한 답변을 표현할 때 사용됩니다.
*   예시:

    ```python
    from langchain.schema import AIMessage

    message = AIMessage(content="서울의 날씨는 맑습니다.")
    print(message.content)  # 출력: 서울의 날씨는 맑습니다.
    ```
* 특징:
  * 모델이 생성한 응답을 저장.
  * 대화의 흐름에서 AI의 역할을 명확히 구분.

### 5-3.  SystemMessage

* 설명: 시스템이 모델에 전달하는 지침이나 설정 정보를 나타냅니다.
* 용도: 모델의 행동을 제어하거나 특정 역할을 부여할 때 사용됩니다.
*   예시:

    ```python
    from langchain.schema import SystemMessage

    message = SystemMessage(content="당신은 친절한 어시스턴트입니다.")
    print(message.content)  # 출력: 당신은 친절한 어시스턴트입니다.
    ```
* 특징:
  * 모델의 초기 설정이나 역할 지정을 위해 사용.
  * 예를 들어, "당신은 번역가입니다." 또는 "당신은 데이터 분석 전문가입니다."와 같은 지침 제공.

### 5-4. FunctionMessage

* 설명: 함수 호출 또는 함수 실행 결과를 나타내는 메시지입니다.
* 용도: 모델이 특정 함수 호출을 요청하거나, 함수 실행 결과를 전달할 때 사용됩니다.
*   예시:

    ```python
    from langchain.schema import FunctionMessage

    message = FunctionMessage(content="{'result': '서울의 날씨는 맑습니다.'}")
    print(message.content)  # 출력: {'result': '서울의 날씨는 맑습니다.'}
    ```
* 특징:
  * 함수 호출과 관련된 대화에서 사용.
  * 주로 모델이 외부 API 호출이나 계산 작업을 요청하거나, 그 결과를 반환할 때 사용.

### 5-5. 요약

| 메시지 유형          | 설명                       | 주요 용도                |
| --------------- | ------------------------ | -------------------- |
| HumanMessage    | 사람이 모델에 전달하는 메시지         | 사용자 입력 전달            |
| AIMessage       | 모델이 생성한 응답 메시지           | 모델의 답변 표현            |
| SystemMessage   | 모델의 행동을 제어하기 위한 시스템 지침   | 모델의 역할 설정 및 초기화      |
| FunctionMessage | 함수 호출 또는 실행 결과를 나타내는 메시지 | 외부 함수 호출 요청 또는 결과 전달 |

### 5-6. Ollama (from langchain.llms) 예시

```python
from langchain.schema import HumanMessage
from langchain.llms import Ollama

# Initialize the Ollama LLM
llm = Ollama(base_url=os.environ["OLLAMA_BASE_URI"], 
             model=os.environ["MODEL"], 
             temperature=0.5)
             
text = "2025년 12월 31일 13시에 태어난 아이의 이름으로 좋은 것은?"
messages = [HumanMessage(content=text)]

llm.invoke(messages)
```

### 5-7. ChatOllama (from langchain.chat\_models) 예시

```python
from langchain.schema import HumanMessage
from langchain.chat_models import ChatOllama

# Initialize the ChatOllama LLM
chat_llm = ChatOllama(base_url=os.environ["OLLAMA_BASE_URI"], 
                      model=os.environ["MODEL"], 
                      temperature=0.5)
             
text = "2025년 12월 31일 13시에 태어난 아이의 이름으로 좋은 것은?"
messages = [HumanMessage(content=text)]

llm.invoke(messages)
```

## 6. 프롬프트 템플릿

LangChain의 \*\*프롬프트 템플릿(Prompt Template)\*\*은 언어 모델(LLM)과 상호작용하기 위해 입력 텍스트를 동적으로 생성하는 도구입니다. 이를 통해 사용자 입력, 변수, 고정 텍스트 등을 조합하여 모델에 전달할 최적화된 프롬프트를 생성할 수 있습니다.

### 6-1. 주요 특징

* 동적 프롬프트 생성:
  * 변수와 고정 텍스트를 조합하여 유연한 프롬프트를 생성합니다.
  * 예를 들어, 사용자 입력에 따라 질문을 다르게 구성할 수 있습니다.
* 재사용 가능성:
  * 동일한 템플릿을 다양한 입력 데이터에 재사용할 수 있습니다.
* 구조화된 입력:
  * 프롬프트를 체계적으로 관리하고, 코드의 가독성을 높입니다

### 6-2. Ollama (from langchain.llms)

```python
from langchain.prompts import PromptTemplate
from langchain.llms import Ollama

# Initialize the Ollama LLM
llm = Ollama(base_url=os.environ["OLLAMA_BASE_URI"], 
             model=os.environ["MODEL"], 
             temperature=0.5)
             
# 프롬프트 템플릿 정의
primpt_template = PromptTemplate(
  input_variables=["text"],
  template="Please write a poem about {text} in Korean."
)

# 입력 텍스트 정의
input_text = "하늘의 별"

# 프롬프트 생성
prompt = primpt_template.format(text=input_text)

# 생성된 프롬프트 출력
print(prompt)

# LLM을 사용하여 프롬프트에 대한 응답 생성
response = llm.invoke(prompt)

# Ollama LLM 응답 출력
print("Ollama LLM Response:")
print(response)
```

### 6-3. ChatOllama (from langchain.chat\_models)

```python
from pprint import pprint
import json
from langchain.prompts.chat import ChatPromptTemplate
from langchain.chat_models import ChatOllama

# Initialize the ChatOllama LLM
chat_llm = ChatOllama(base_url=os.environ["OLLAMA_BASE_URI"], 
                      model=os.environ["MODEL"], 
                      temperature=0.5)

# 채팅 프롬프트 템플릿 정의
chat_prompt_template = ChatPromptTemplate.from_messages([
    ("system", "당신은 도움이 되는 어시스턴트입니다."),  # 시스템 메시지
    ("human", "한국어로 {text}에 대한 시를 작성해주세요.")  # 사용자 메시지
])

# 입력 텍스트 정의
input_text = "하늘의 별"

# 채팅 프롬프트 생성
chat_prompt = chat_prompt_template.format_prompt(text=input_text)

# 생성된 채팅 프롬프트 출력
pprint(chat_prompt.model_dump())

# ChatOllama LLM을 사용하여 채팅 프롬프트에 대한 응답 생성
response = chat_llm.invoke(chat_prompt.to_messages())

# ChatOllama LLM의 응답 출력
print("ChatOllama LLM Response:")
print(response.content)
```

## 7. OutputParser

LangChain의 **OutputParser**는 언어 모델(LLM)의 응답을 특정 형식으로 변환하거나 처리하는 데 사용되는 도구입니다. 모델의 출력은 일반적으로 텍스트 형식으로 제공되지만, 이를 구조화된 데이터(예: JSON, Python 객체 등)로 변환하거나 특정 요구사항에 맞게 처리해야 할 때 유용합니다.

### 7-1. 주요 역할

* 모델 출력 변환: 모델의 텍스트 응답을 특정 데이터 구조(예: 딕셔너리, 리스트 등)로 변환합니다.
* 출력 검증: 모델의 응답이 예상된 형식이나 조건을 충족하는지 확인하고, 잘못된 형식의 응답이 반환되었을 경우, 에러를 처리하거나 기본값을 반환할 수 있습니다.
* 후처리: 모델 출력에 추가적인 처리를 적용하여 애플리케이션에서 바로 사용할 수 있도록 준비합니다.

### 7-2.  OutputParser의 주요 메서드

* parse:
  * 모델의 텍스트 응답을 입력으로 받아, 이를 원하는 형식으로 변환합니다.
  * 예를 들어, JSON 응답을 파싱하거나 특정 키워드를 추출할 수 있습니다.
* parse\_with\_prompt:
  * 모델의 출력과 함께 사용된 프롬프트를 입력으로 받아, 이를 처리합니다.
  * 프롬프트와 출력 간의 관계를 고려하여 응답을 처리할 때 유용합니다.

### 7-3. 예시

#### 7-3-1. 기본 OutputParser 사용

```python
from langchain.schema import BaseOutputParser

class CustomOutputParser(BaseOutputParser):
    def parse(self, text: str):
        # 모델의 텍스트 응답을 파싱하여 딕셔너리로 변환
        try:
            return {"response": text.strip()}
        except Exception as e:
            raise ValueError(f"Parsing failed: {e}")

# OutputParser 초기화
parser = CustomOutputParser()

# 모델 응답 파싱
response = parser.parse("  안녕하세요!  ")
print(response)  # 출력: {'response': '안녕하세요!'}
```

#### 7-3-2. JSON 응답 파싱

```python
import json
from langchain.schema import BaseOutputParser

class JSONOutputParser(BaseOutputParser):
    def parse(self, text: str):
        try:
            return json.loads(text)
        except json.JSONDecodeError:
            raise ValueError("Invalid JSON response")

# OutputParser 초기화
parser = JSONOutputParser()

# 모델 응답 파싱
response = parser.parse('{"name": "홍길동", "age": 30}')
print(response)  # 출력: {'name': '홍길동', 'age': 30}
```

### 7-4. OutputParser의 활용 사례

* 구조화된 데이터 추출: 모델이 생성한 텍스트에서 특정 정보를 추출하여 딕셔너리, 리스트 등으로 변환.
* 에러 처리: 모델 출력이 예상된 형식이 아닐 경우, 기본값을 반환하거나 에러를 발생시킴.
* 다양한 출력 형식 지원: JSON, XML, CSV 등 다양한 형식의 데이터를 처리하도록 확장 가능.
* 프롬프트와 출력의 연계 처리: 프롬프트와 모델 응답을 함께 분석하여 더 정교한 처리를 수행.

### 7-5. 장점

* 유연성: 다양한 출력 형식에 맞게 커스터마이징 가능.
* 확장성: 애플리케이션 요구사항에 따라 새로운 파서 클래스를 쉽게 정의 가능.
* 안정성: 모델 출력의 형식을 검증하고, 예상치 못한 오류를 방지.



## 8. chain

LangChain의 chain 은 여러 구성 요소(예: 프롬프트 템플릿, LLM, OutputParser 등)를 연결하여 작업을 순차적으로 처리하는 구조 이를 통해 복잡한 작업을 간단하고 재사용 가능한 방식으로 구성할 수 있습니다.

### 8-1. **chain 의 주요 개념**

1. **구성 요소 연결**:
   * `chain`은 여러 단계를 연결하여 입력 데이터를 처리하고 최종 결과를 생성합니다.
   * 각 단계는 이전 단계의 출력을 다음 단계의 입력으로 사용합니다.
2. **유연성**:
   * 다양한 구성 요소(예: 프롬프트 템플릿, LLM, OutputParser 등)를 조합하여 원하는 작업을 수행할 수 있습니다.
3. **재사용성**:
   * 체인을 정의하면 동일한 작업을 여러 번 재사용할 수 있습니다.
4. **가독성**:
   * 복잡한 작업을 체인으로 구성하면 코드가 더 간결하고 읽기 쉬워집니다.

***

### 8-2. **chain의 구성 요소**

1. **`PromptTemplate`**:
   * 입력 데이터를 기반으로 LLM에 전달할 프롬프트를 생성합니다.
2. **`LLM`**:
   * 프롬프트를 입력으로 받아 응답을 생성합니다.
   * 예: `ChatOllama`, `OpenAI`, `HuggingFaceHub` 등.
3. **`OutputParser`**:
   * LLM의 응답을 원하는 형식으로 변환하거나 처리합니다.
   * 예: JSON 파싱, 리스트 변환 등.

***

### 8-3. **체인 정의 방식**

LangChain에서는 체인을 정의하는 두 가지 주요 방식이 있습니다.

#### 8-3-1.  **파이프라인 연산자(`|`) 사용**

파이프라인 연산자를 사용하여 구성 요소를 간결하게 연결할 수 있습니다.

```python
from langchain.chat_models import ChatOllama
from langchain.prompts.chat import ChatPromptTemplate
from langchain.schema import BaseOutputParser
from pprint import pprint
import os

# OutputParser 정의
class CommaSeparatedListOutputParser(BaseOutputParser):
    def parse(self, text: str):
        return text.strip().split(", ")

# ChatOllama LLM 초기화
chat_llm = ChatOllama(base_url=os.environ["OLLAMA_BASE_URI"], model=os.environ["MODEL"], temperature=0.5)

# 시스템 메시지 템플릿
template = """당신을 쉼표로 구분된 단어를 사용하여 응답하는 어시스턴트로 설정합니다.
사용자의 질문에 대한 응답을 쉼표로 구분된 단어로 5개 작성하고 응답으로 오직 쉼표로 구분된 단어만 포함하세요.
질문: {question}
응답: {response}
"""

# 사용자 메시지 템플릿
human_template = "{input_text}"

# 채팅 프롬프트 템플릿 정의
chat_prompt = ChatPromptTemplate.from_messages([
    ("system", template),  # 시스템 메시지
    ("human", human_template)  # 사용자 메시지
])

# CommaSeparatedListOutputParser 초기화
output_parser = CommaSeparatedListOutputParser()

# 체인 정의 (chat_prompt | chat_llm | output_parser 형태)
chain = chat_prompt | chat_llm | output_parser

# 입력 텍스트 정의
input_text = "하늘의 별"

# 체인을 사용하여 응답 생성 및 파싱
response = chain.invoke({"input_text": input_text, "question": input_text, "response": ""})

# 결과 출력
print("Parsed Response:")
print(response)
```

#### 8-3-2.  LLMChain **클래스 사용**

* 파이프라인 연산자를 사용하여 구성 요소를 간결하게 연결할 수 있습니다.

```python
from langchain.chat_models import ChatOllama
from langchain.prompts.chat import ChatPromptTemplate
from langchain.schema import BaseOutputParser
from langchain.chains import LLMChain
from pprint import pprint
import os

# OutputParser 정의
class CommaSeparatedListOutputParser(BaseOutputParser):
    def parse(self, text: str):
        return text.strip().split(", ")

# ChatOllama LLM 초기화
chat_llm = ChatOllama(base_url=os.environ["OLLAMA_BASE_URI"], model=os.environ["MODEL"], temperature=0.5)

# 시스템 메시지 템플릿
template = """당신을 쉼표로 구분된 단어를 사용하여 응답하는 어시스턴트로 설정합니다.
사용자의 질문에 대한 응답을 쉼표로 구분된 단어로 5개 작성하고 응답으로 오직 쉼표로 구분된 단어만 포함하고 한국어로 답변해줘.
질문: {question}
응답: {response}
"""

# 사용자 메시지 템플릿
human_template = "{input_text}"

# 채팅 프롬프트 템플릿 정의
chat_prompt = ChatPromptTemplate.from_messages([
    ("system", template),  # 시스템 메시지
    ("human", human_template)  # 사용자 메시지
])

# CommaSeparatedListOutputParser 초기화
parser = CommaSeparatedListOutputParser()

# LLMChain 정의
chain = LLMChain(
    llm=chat_llm,
    prompt=chat_prompt,
    output_parser=parser
)


# 입력 텍스트 정의
input_text = "과자이름"

# 체인을 사용하여 응답 생성 및 파싱
response = chain.run({"input_text": input_text, "question": input_text, "response": ""})

# 결과 출력
print("Parsed Response:")
print(response)
```

### 8-4. **체인의 실행 흐름**

1. **프롬프트 생성**:
   * `PromptTemplate`이 입력 데이터를 기반으로 프롬프트를 생성합니다.
2. **LLM 호출**:
   * 생성된 프롬프트를 LLM에 전달하여 응답을 생성합니다.
3. **응답 파싱**:
   * `OutputParser`가 LLM의 응답을 원하는 형식으로 변환합니다.
4. **결과 반환**:
   * 최종 결과를 반환합니다.

### 8-5. 장점

1. **구조화**:
   * 작업을 단계별로 나누어 체계적으로 구성할 수 있습니다.
2. **확장성**:
   * 새로운 구성 요소를 쉽게 추가하거나 교체할 수 있습니다.
3. **재사용성**:
   * 동일한 체인을 다양한 입력 데이터에 재사용할 수 있습니다.
