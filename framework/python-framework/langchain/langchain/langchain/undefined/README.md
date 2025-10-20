# 간단한 예제

### LangChain v1.0 주요 변경사항 요약 <a href="#langchain-v10" id="langchain-v10"></a>

<table><thead><tr><th width="192.49609375">구성요소</th><th>v0.3 이하</th><th>v1.0</th></tr></thead><tbody><tr><td><strong>Models</strong></td><td><code>from langchain_openai import ChatOpenAI</code></td><td><code>from langchain.chat_models import init_chat_model</code></td></tr><tr><td><strong>Prompts</strong></td><td>동일</td><td>동일 (변경 없음)</td></tr><tr><td><strong>Output Parsers</strong></td><td>동일</td><td>동일 (변경 없음)</td></tr><tr><td><strong>Document Loaders</strong></td><td><code>from langchain.document_loaders</code></td><td><code>from langchain_community.document_loaders</code></td></tr><tr><td><strong>Text Splitters</strong></td><td><code>from langchain.text_splitters</code></td><td><code>from langchain_text_splitters</code></td></tr><tr><td><strong>Vector Stores</strong></td><td>동일</td><td>동일 (변경 없음)</td></tr><tr><td><strong>Tools</strong></td><td><code>from langchain.tools import tool</code></td><td><code>from langchain.tools import tool</code> (동일)</td></tr><tr><td><strong>Agent</strong></td><td><code>from langgraph.prebuilt import create_react_agent</code></td><td><code>from langchain.agents import create_agent</code></td></tr><tr><td><strong>Example Selectors</strong></td><td>동일</td><td>동일 (변경 없음)</td></tr></tbody></table>

### 성능 비교 및 선택 가이드 <a href="#undefined" id="undefined"></a>

| 특징         | OpenSearch            | PGVector               |
| ---------- | --------------------- | ---------------------- |
| **설치 복잡도** | 중간 (별도 서비스)           | 낮음 (PostgreSQL 확장)     |
| **검색 속도**  | 매우 빠름 (대규모)           | 빠름 (중소규모)              |
| **메모리 사용** | 높음                    | 중간                     |
| **확장성**    | 수평 확장 용이              | 수직 확장 위주               |
| **필터링**    | 강력한 DSL               | SQL 기반 (직관적)           |
| **유사도 점수** | 미지원                   | 지원                     |
| **검색 엔진**  | nmslib, faiss, lucene | HNSW, IVFFlat          |
| **추천 용도**  | 대규모, 복잡한 검색           | 중소규모, 기존 PostgreSQL 활용 |
