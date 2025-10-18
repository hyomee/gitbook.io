# Ollama API 호출

Ollama는 로컬 또는 원격에 배포된 고성능 언어 모델을 간편하게 호출할 수 있는 HTTP 기반 API를 제공합니다. 아래 예제에서는 Python `requests` 라이브러리를 사용하여 Ollama 서버에 텍스트 완성 요청을 보내고 응답을 처리하는 방법을 보여줍니다.

***

### 1. 사전 준비 <a href="#id-1" id="id-1"></a>

1. Ollama 서버가 실행 중이어야 합니다.\
   기본 호스트: `http://localhost`\
   기본 포트: `11434`
2.  Python 환경에 `requests` 패키지를 설치합니다.

    ```bash
    pip install requests
    ```

***

### 2. 단순 텍스트 완성 예제 <a href="#id-2" id="id-2"></a>

```python
import requests

# Ollama 서버 엔드포인트
OLLAMA_HOST = "http://localhost"
OLLAMA_PORT = 11434
MODEL_NAME = "llama2"  # 예: llama2, mixtral-8x7b 등

def ollama_complete(prompt: str, max_tokens: int = 128, temperature: float = 0.7):
    """
    Ollama API에 텍스트 완성 요청을 보내고 결과를 반환하는 함수.
    """
    url = f"{OLLAMA_HOST}:{OLLAMA_PORT}/v1/models/{MODEL_NAME}/completions"
    payload = {
        "prompt": prompt,
        "max_tokens": max_tokens,
        "temperature": temperature
    }
    headers = {
        "Content-Type": "application/json"
    }
    
    response = requests.post(url, json=payload, headers=headers)
    response.raise_for_status()  # 에러 발생 시 예외 발생
    
    data = response.json()
    # Ollama 응답 구조에 따라 'completion' 필드를 추출
    return data.get("completion", "")

if __name__ == "__main__":
    user_prompt = "안녕하세요, 인공지능에 대해 간단히 설명해줘."
    completion = ollama_complete(user_prompt, max_tokens=100, temperature=0.5)
    print("모델 응답:")
    print(completion)
```

### 코드 설명

1. `OLLAMA_HOST`, `OLLAMA_PORT`, `MODEL_NAME`을 설정하여 호출할 서버와 모델을 지정합니다.
2. `ollama_complete()` 함수는:
   * HTTP POST로 `/v1/models/{MODEL_NAME}/completions` 엔드포인트에 JSON 페이로드(`prompt`, `max_tokens`, `temperature`)를 전송
   * 응답 JSON에서 `completion` 필드를 추출하여 반환
3. `response.raise_for_status()`로 HTTP 에러(4xx/5xx)를 예외 처리하여 호출 안정성을 높였습니다.
4. `__main__` 블록에서 실제 프롬프트를 보내고 결과를 출력합니다.

***

### 3. 스트리밍 응답 처리 예제 <a href="#id-3" id="id-3"></a>

Ollama는 스트리밍 모드를 지원하여 응답이 생성되는 대로 부분별로 전달받을 수 있습니다. 큰 문장을 점진적으로 처리해야 할 때 유용합니다.

```python
import requests

def ollama_stream(prompt: str, max_tokens: int = 256, temperature: float = 0.8):
    url = f"{OLLAMA_HOST}:{OLLAMA_PORT}/v1/models/{MODEL_NAME}/completions?stream=true"
    payload = {
        "prompt": prompt,
        "max_tokens": max_tokens,
        "temperature": temperature
    }
    headers = {
        "Content-Type": "application/json"
    }

    with requests.post(url, json=payload, headers=headers, stream=True) as resp:
        resp.raise_for_status()
        for line in resp.iter_lines(decode_unicode=True):
            if line:
                # 각 라인이 JSON 형식으로 전송됨
                chunk = line.lstrip("data: ")
                data = json.loads(chunk)
                print(data.get("completion_chunk", ""), end="", flush=True)

if __name__ == "__main__":
    print("스트리밍 응답:")
    ollama_stream("지구 온난화의 원인에 대해 설명해줘.")
```

***

### 4. 응용: 대화형 챗봇 예제 <a href="#id-4" id="id-4"></a>

```python
import requests

def chat_with_ollama(history: list[dict], max_tokens=128):
    """
    history: [{"role": "user"|"assistant", "content": "..."}, ...]
    """
    url = f"{OLLAMA_HOST}:{OLLAMA_PORT}/v1/models/{MODEL_NAME}/chat/completions"
    payload = {
        "messages": history,
        "max_tokens": max_tokens
    }
    headers = {"Content-Type": "application/json"}

    resp = requests.post(url, json=payload, headers=headers)
    resp.raise_for_status()
    return resp.json()["choices"][0]["message"]["content"]

if __name__ == "__main__":
    conversation = [
        {"role": "user", "content": "안녕하세요!"},
        {"role": "assistant", "content": "안녕하세요! 무엇을 도와드릴까요?"}
    ]
    reply = chat_with_ollama(conversation)
    print("Assistant:", reply)
```
