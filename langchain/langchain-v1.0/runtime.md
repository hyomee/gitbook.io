# Runtime

## LangChain v1.0의 Runtime 개념

* LangChain v1.0에서 Runtime은 체인을 실행할 때 동적으로 전달되는 state(예: 대화 이력)와, 정적으로 전달되는 context(예: 유저 메타데이터 등)를 통합 관리한다.
* 모델의 invoke 단계에서 context 파라미터를 통해 손쉽게 상태 및 정보를 전달할 수 있다.​

***

## 1. OpenAI, Ollama 모델 모든 활용 패턴(init\_chat\_model 적용)

```python
from langchain.chat_models import init_chat_model

# OpenAI: 환경 변수 OPENAI_API_KEY를 반드시 설정
openai_llm = init_chat_model(
  "gpt-4o",           # 또는 "gpt-3.5-turbo" 등
  model_provider="openai",
  temperature=0.3,
  max_tokens=1024,
)

output = openai_llm.invoke("한글로 LangChain v1.0의 특징을 말해줘.")
print(output.content)
```

**구조 요약:**

* model: 모델명 지정 (gpt-4o 등)
* model\_provider: "openai" 명시
* 각종 파라미터(temperature, max\_tokens 등)
* invoke로 채팅 호출.​

***

```python
from langchain.chat_models import init_chat_model

# Ollama: 로컬에서 ollama 실행 필요, 별도 API Key 불필요, 서버는 http://localhost:11434/v1
ollama_llm = init_chat_model(
  "llama3",                # 설치한 ollama 모델명
  model_provider="ollama",
  temperature=0.7,
  base_url="http://localhost:11434/v1",  # Ollama REST API endpoint
  # 기타 파라미터(예: max_tokens) 추가 가능
)

output = ollama_llm.invoke("대한민국의 수도는 어디인가요?")
print(output.content)
```

* model\_provider를 **ollama**로 지정해주면 로컬 ollama가 활성화된 상태에서 LLM 호출이 가능.​
* Ollama로 툴 사용 등 advanced 패턴도 가능하며, stream/context 전달, 도구 바인딩 지원 등 OpenAI와 동일한 인터페이스 활용.​

***

## 2. Configurable 모델과 Runtime/Context

```
custom_llm = init_chat_model(
    model="gpt-4o",
    model_provider="openai",
    temperature=0,
    configurable_fields=("model", "model_provider", "temperature", "max_tokens"),
    config_prefix="first"
)

# Runtime context 전달
response = custom_llm.invoke(
    "이름이 뭐야?",
    context={"user_id": "123", "session_id": "abc"}
)
print(response.content)
```

* configurable\_fields와 context 인자를 통해 복수 모델 및 context 패턴 설계 가능.​

***

## 3. 참고 및 Best Practice

* 사용하려는 provider의 패키지(like `langchain-openai`, `langchain-ollama`)가 pip로 사전 설치되어 있어야 동작함.​
* 환경변수(API Key) 및 ollama base\_url 설정을 반드시 체크.​
* stream, structured output, 도구 자동 바인딩 등 최신 v1.0 기능도 완벽 지원.
