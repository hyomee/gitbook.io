# RAG 관련 활용

## 1. hwp 파일에 대한 문서 분석&#x20;

* 한글과 컴퓨터(한컴)에서 만든 워드프로세서 "한글"의 파일에 대한 라이브러리: [https://github.com/neolord0/hwplib](https://github.com/neolord0/hwplib)
* 파이썬 라이브러리: pyhwp/hwp5/hancom
  * HWP 포맷의 서식·레이아웃까지 합치려면 한글 프로그램 자동화(win32com) 방식이 필요
  * `pyhwp`/`hwp5` 라이브러리는 리눅스 등 서버 환경에서 HWP 파일의 텍스트 추출
  * Windows + 한글 프로그램이 있으면 `win32com.client`로 한글을 직접 제어해 텍스트와 표 데이터를 파싱
*   ### 파이프라인 구ㅅ성

    1. pyhwp/hwp5/hancom 자동화로 텍스트 추출
    2. 파일별 텍스트 문자열 합치기
    3. LangChain/LlamaIndex 등의 텍스트 Splitter로 의미 단위 분할
    4. KoSBERT 등 한국어 임베딩 모델로 벡터화
    5. Chroma/FAISS 등 벡터DB에 인덱스 등록

    \
