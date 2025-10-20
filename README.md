---
description: 본LLM RAG 학습 공간
---

# LLM\_RAG 기본

## 1. 파이선 환경 구성

* 환경 :
  * tool : vscode
  * language : python 3.11.4
* 환경 구성
  1. github 복제 : https://github.com/hyomee/llm\_rag.git
     1. 읽기 권한 신청 필요
  2. vscode 코드 실행
  3. shift + cmd + p :
  4. 가상 환경 선택
     *   Python 가상 환경이 없는 경우 다음을 참조 해서 환경 구성을 합니다.

         파이썬 가상환경이 없는 경우 참고 문서

## 2. Ollama 설치

1. 올라마 웹사이트(https://ollama.com/)에 접속합니다.
2. 다운로드(https://ollama.com/download) 페이지에서 운영체제(Linux, Mac OS, Windows)에 맞는 설치파일을 다운로드합니다.
3. 설치파일을 실행하여 설치를 완료합니다.
4.  설치가 잘 되었는지 확인하기 위해서, 터미널 또는 프롬프트에서 아래와 같이 입력합니다.

    ```shell
    ollama
    ```

## 3. Model 다운로드

1. ollama model(https://www.ollama.com/search)애서 모델을 선택 한다.
2.  올라마(Ollama) 모델을 다운로드하는 명령은 아래와 같이 모델 이름을 입력하면 설치 됩니다.\


    ```shell
    # ollama run <모델 이름>
    ollama run llama3
    ```



    | Model     | Parameters | Size  | Download Command        |
    | --------- | ---------- | ----- | ----------------------- |
    | Llama 3.2 | 8B         | 4.7GB | `ollama run llama3.2`   |
    | Llama 3   | 70B        | 40GB  | `ollama run llama3:70b` |
    | Phi-3     | 3.8B       | 2.3GB | `ollama run phi3`       |
    | Mistral   | 7B         | 4.1GB | `ollama run mistral`    |
    | LLaVA     | 7B         | 4.5GB | `ollama run llava`      |
    | Gemma3    | 2B         | 1.4GB | `ollama run gemma3:1b`  |
    | Gemma3    | 7B         | 4.8GB | `ollama run gemma3:4b`  |
    | Solar     | 10.7B      | 6.1GB | `ollama run solar`      |


3.  올라마 설치 확인\


    ```sh
    ollama list  
    ```



    | NAME                     | ID           | SIZE   | MODIFIED     |
    | ------------------------ | ------------ | ------ | ------------ |
    | qwq:latest               | 009cb3f08d74 | 19 GB  | 9 days ago   |
    | bge-m3:latest            | 790764642607 | 1.2 GB | 3 weeks ago  |
    | gemma3:latest            | c0494fe00251 | 3.3 GB | 3 weeks ago  |
    | exaone3.5:2.4b           | 13644fc3d28e | 1.6 GB | 4 weeks ago  |
    | mxbai-embed-large:latest | 468836162de7 | 669 MB | 4 weeks ago  |
    | nomic-embed-text:v1.5    | 0a109f422b47 | 274 MB | 4 weeks ago  |
    | sqlcoder:latest          | 77ac14348387 | 4.1 GB | 6 weeks ago  |
    | Qwen2.5-Coder:1.5b       | 6d3abb8d2d53 | 986 MB | 6 weeks ago  |
    | qwen2.5:3b               | 357c53fb659c | 1.9 GB | 6 weeks ago  |
    | llama3.1:latest          | 46e0c10c039e | 4.9 GB | 6 weeks ago  |
    | llama3.2:latest          | a80c4f17acd5 | 2.0 GB | 6 weeks ago  |
    | exaone3.5:latest         | c7c4e3d1ca22 | 4.8 GB | 6 weeks ago  |
    | nomic-embed-text:latest  | 0a109f422b47 | 274 MB | 6 weeks ago  |
    | deepseek-r1:7b           | 0a8c26691023 | 4.7 GB | 6 weeks ago  |
    | deepseek-r1:1.5b         | a42b25d8c10a | 1.1 GB | 6 weeks ago  |
    | codellama:latest         | 8fdf8f752f6e | 3.8 GB | 6 months ago |

    \
