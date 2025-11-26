# 프롬프트 엔지니어링

## 1. 프롬프트 작성 일반적인 문제에 대한 개선

GPT, Perplexity 등 일반적으로 프롬프트 작성시 다음과 같은 문제점이 있습니다.

### **1. 모호한 질문**

* 나쁜 예시\
  “AI가 뭐야?”
* 문제점
  * 범위가 지나치게 넓고, 깊이/난이도/관심 영역이 전혀 안 보임.
  * 사용 목적(개념 이해, 기술 비교, 도입 검토 등)이 없어서 답변이 뜬구름식이 되기 쉬움.
* 개선 예시\
  “머신러닝과 딥러닝의 차이를 핵심 개념 위주로 설명하고, 각 기술의 실제 활용 사례를 2가지씩 알려줘. 비전공자도 이해할 수 있는 수준으로 설명해줘.”

### **2. 출력 형식 없는 질문**

* 나쁜 예시\
  “Spring Boot 마이크로서비스 아키텍처 설계 방법 알려줘.”
* 문제점
  * 텍스트/리스트/표/다이어그램 등 어떤 형식을 원하는지 모름.
  * 단계별 설계, 샘플 코드, 비교표 중 무엇이 중요한지 알 수 없음.
* 개선 예시\
  “Spring Boot 기반 마이크로서비스 아키텍처 설계 원칙을 다음 형식으로 정리해줘.
  1. 핵심 설계 원칙 5가지를 불릿 리스트로,
  2. 서비스 간 통신 방식(REST, gRPC, 메시지 큐)을 비교하는 표,
  3. 간단한 예시 서비스 구조(서비스 A/B, API Gateway)를 글로 설명해줘.”

### **3. 예제 없는 질문**

* 나쁜 예시\
  “좋은 프롬프트 작성법을 알려줘.”
* 문제점
  * 이론만 나열되기 쉽고, 실제로 복붙해서 쓸 수 있는 형태로 안 나옴.
  * 사용자가 원하는 도메인(코딩, 문서 요약, 기획서 작성 등)을 알 수 없음.
* 개선 예시\
  “코드 리뷰용 프롬프트를 잘 쓰는 방법을 설명해주고,
  1. Python 코드 성능 개선용 프롬프트 예제 2개,
  2. 버그 원인 분석용 프롬프트 예제 2개를 실제로 써서 보여줘.”

1. **너무 많은 요구 사항**&#x20;
   * 나쁜 예시\
     “Kafka 기반 실시간 데이터 파이프라인 전체 아키텍처를 설명해줘. 환경은 AWS고, 가용성과 비용, 보안, 모니터링, 장애 대응 전략까지 모두 상세히, 코드 예제랑 Terraform 스크립트, Helm 차트, 운영 가이드, 교육 자료까지 한 번에 만들어줘.”
   * 문제점
     * 한 번에 커버하기 어려운 범위를 모두 요구해서 답이 피상적이거나 중간에 잘릴 수 있음.
     * 우선순위가 없어 모델이 어디에 집중해야 할지 모름.
   * 개선 예시\
     “Kafka 기반 실시간 데이터 파이프라인을 AWS에서 구축할 때,
     1. 전체 아키텍처 개요와 주요 컴포넌트 역할을 그림 대신 텍스트로 설명해주고,
     2. 최소 구성 예시로 사용할 수 있는 서비스 목록과 이유를 bullet로 정리해줘.\
        이후에 인프라 코드(Terraform, Helm)는 추가로 물어볼게.”

### **4. 일관성 없는 평가**&#x20;

* 나쁜 예시
  1. 첫 질문: “이 프롬프트 어떤 것 같아? 솔직하게 평가해줘.”
  2. 모델이 장단점을 말했더니: “너무 까다롭게 보지 말고, 그냥 괜찮다고 해줘.”
  3. 다시: “근데 또 냉정하게 평가해봐. 어디가 많이 부족해?”
* 문제점
  * “솔직하게 / 까다롭지 않게 / 냉정하게”처럼 기준이 계속 바뀌어 일관된 피드백 기준을 만들기 어려움.
  * 사용자가 원하는 평가 척도(점수, 등급, 사용성, 명확성 등)가 없음.
* 개선 예시\
  “이 프롬프트를 다음 기준으로 평가해줘.
  1. 명확성, 2) 구체성, 3) 출력 형식 정의, 4) 길이 적절성.\
     각 항목별로 1\~5점으로 점수를 주고, 한 줄짜리 코멘트를 붙여줘.\
     마지막에 ‘가장 먼저 수정하면 좋은 부분’ 3가지만 bullet로 알려줘.”

## 2. 프롬프트 원칙

### 1. 목표·출력을 명확히 하기 <a href="#id-1" id="id-1"></a>

* 원칙\
  “무엇을, 누구를 위해, 어떤 형태로” 만들지를 처음에 명시한다.​
* 프롬프트 예시\
  “너는 시니어 백엔드 개발자다.\
  지금부터 Kafka 기반 이벤트 처리 구조를 신입 개발자 교육용 문서로 설명해줘.
  * 대상: 백엔드 1\~3년차 개발자
  * 길이: A4 2장 분량 정도
  * 형식:
    1. 개념 설명
    2. 기본 아키텍처 다이어그램을 글로 묘사
    3. 간단한 예시 시나리오 2개 (주문 생성, 주문 취소)”

### 2. 역할(Persona)를 부여하기 <a href="#id-2-persona" id="id-2-persona"></a>

* 원칙\
  모델에 “어떤 전문가인지” 역할을 부여하면, 용어 선택·깊이·관점이 일관된다.​
* 프롬프트 예시\
  “너는 대규모 트래픽을 처리하는 이커머스 플랫폼을 5년 이상 운영한 SRE 팀 리드다.\
  우리 서비스에 API Gateway + Spring Boot + Keycloak을 도입하려고 한다.\
  SRE 관점에서
  * 가용성
  * 관측성(로그/메트릭/트레이싱)
  * 장애 대응 프로세스\
    측면에서 고려해야 할 체크리스트를 항목별 bullet로 정리해줘.”

### 3. 맥락·제약조건 충분히 주기 <a href="#id-3" id="id-3"></a>

* 원칙\
  도메인, 현재 상황, 제약(예산, 기술 스택, 조직 상황)을 알려주면 현실적인 답이 나온다.​
* 프롬프트 예시\
  “현재 상황:
  * 회사 규모: 개발자 10명, 인프라 담당 1명
  * 클라우드: AWS 만 사용 가능
  * 사용 중: ALB, ECS Fargate, RDS(PostgreSQL), Cache(Redis)
  * 요구사항: 월 1억 요청, RTO 1시간 이내, RPO 15분\
    위 조건을 전제로, FastAPI 기반 멀티테넌트 검색/에이전트 백엔드 아키텍처를 제안해줘.
  * 구성 요소 목록
  * 각 컴포넌트 역할
  * 비용을 크게 올리지 않는 선에서의 가용성 전략만 설명해줘.”

### 4. 예시(Few-shot)를 함께 제공하기 <a href="#id-4-few-shot" id="id-4-few-shot"></a>

* 원칙\
  “이런 느낌으로 써줘”를 말로 설명하기보다, 짧은 샘플을 1\~2개 주는 것이 가장 효과적이다.​
* 프롬프트 예시\
  “아래 형식의 아키텍처 설명을 그대로 따라해줘. 도메인만 다르게 작성하면 된다.\
  \[예시]
  * 시스템 개요: 모바일 주문을 처리하는 REST API 서버
  * 주요 컴포넌트:
    1. API Gateway: 인증/라우팅
    2. 주문 서비스: 주문 생성/조회
    3. 결제 서비스: 외부 PG 연동
  * 데이터 흐름: 사용자가 주문을 생성하면 … (요약)\
    이제 이 형식을 그대로 사용해서 ‘멀티테넌트 검색/QA 에이전트 시스템’을 설명해줘.”

### 5. 복잡한 요청은 단계로 나누기 <a href="#id-5" id="id-5"></a>

* 원칙\
  “한 번에 설계·코드·문서·테스트까지” 요구하지 말고, 단계를 쪼개서 순차적으로 진행한다.​
* 프롬프트 예시\
  “Kafka 기반 실시간 이벤트 처리 시스템 설계가 필요하다.\
  1단계: 비즈니스 요구사항을 정리하고, 핵심 이벤트 타입 목록만 정의해줘.\
  2단계: 그 다음 요청에서 토픽 설계와 파티션 전략을 논의할 거야.\
  지금은 1단계만 수행해. bullet 리스트로 작성해줘.”

### 6. 출력 형식·구조를 미리 정의하기 <a href="#id-6" id="id-6"></a>

* 원칙\
  글/리스트/표/코드블록 등 원하는 구조를 먼저 지정하면 후처리·자동화가 쉬워진다.​
* 프롬프트 예시\
  “아래 형식을 ‘그대로’ 지켜서 답변해줘.
  1. 요약 (3줄 이내)
  2. 장점 리스트 (bullet)
  3. 단점 리스트 (bullet)
  4. 적용 시 체크리스트 (번호 매긴 리스트)\
     주제: ‘Keycloak + Spring Security + API Gateway 조합으로 B2B SaaS 인증/인가 구성할 때 고려사항’”

### 7. 평가 기준·제약을 명시하기 <a href="#id-7" id="id-7"></a>

* 원칙\
  “좋은지 나쁜지 봐줘”가 아니라, 어떤 기준으로 평가할지 숫자·척도를 정한다.​
* 프롬프트 예시\
  “아래 프롬프트를 다음 4가지 기준으로 평가해줘.
  * 명확성
  * 구체성
  * 기술적 정확성
  * 재사용 가능성\
    각 항목별로 1\~5점 점수를 주고, 한 줄 코멘트를 붙여줘.\
    마지막에 ‘가장 먼저 개선해야 할 점’ 3가지만 bullet로 정리해줘.\
    \[프롬프트 본문] ……”

### 8. 모델에게 ‘생각 과정’을 요구하기 <a href="#id-8" id="id-8"></a>

* 원칙\
  정답만 달라고 하기보다, 가설·근거·대안을 같이 말하게 하면 품질과 신뢰도가 올라간다.​
* 프롬프트 예시\
  “아래 두 가지 아키텍처 대안 중에서 선택해줘.
  * A안: 단일 PostgreSQL + 스키마 분리 멀티테넌시
  * B안: 테넌트별 별도 DB 인스턴스
  1. 전제 조건과 가정
  2. 각 안의 장단점
  3. 트래픽이 월 10억 요청 수준으로 커질 때의 리스크\
     를 단계적으로 설명하고, 마지막에 ‘현재 상황에서 추천하는 안’을 한 줄로 정리해줘.”





## 3. 프로프트 엔지니어링 가이드  <a href="#id-1" id="id-1"></a>

#### 1. 공식 프롬프트 엔지니어링 가이드 <a href="#id-1" id="id-1"></a>

* OpenAI Docs
  * Prompt engineering 가이드:\
    [https://platform.openai.com/docs/guides/prompt-engineering](https://platform.openai.com/docs/guides/prompt-engineering)​
  * Prompting(프롬프트 작성·관리) 개요:\
    [https://platform.openai.com/docs/guides/prompting](https://platform.openai.com/docs/guides/prompting)​
* OpenAI Cookbook (예제 중심)
  * 메인 페이지: [https://cookbook.openai.com](https://cookbook.openai.com/)​
  * GPT-5 Prompting Guide 예제: [https://cookbook.openai.com/examples/gpt-5/gpt-5\_prompting\_guide](https://cookbook.openai.com/examples/gpt-5/gpt-5_prompting_guide)​
  * GPT-4.1 Prompting Guide 예제: [https://cookbook.openai.com/examples/gpt4-1\_prompting\_guide](https://cookbook.openai.com/examples/gpt4-1_prompting_guide)​

#### 2. OpenAI Help Center 베스트 프랙티스 <a href="#id-2-openai-help-center" id="id-2-openai-help-center"></a>

* OpenAI Help Center
  * OpenAI API용 프롬프트 엔지니어링 베스트 프랙티스:\
    [https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-the-openai-api](https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-the-openai-api)​
  * ChatGPT용 프롬프트 베스트 프랙티스:\
    [https://help.openai.com/en/articles/10032626-prompt-engineering-best-practices-for-chatgpt](https://help.openai.com/en/articles/10032626-prompt-engineering-best-practices-for-chatgpt)​

#### 3. 참고하면 좋은 주변 리소스 <a href="#id-3" id="id-3"></a>

* Prompt Engineering Guide (서드파티지만 OpenAI 스타일 정리)
  * [https://www.promptingguide.ai](https://www.promptingguide.ai/)
* Prompt Engineering Guide (웹 서비스형 가이드)
  * 메인: [https://www.promptingguide.ai](https://www.promptingguide.ai/)​
  * General Tips for Designing Prompts: [https://www.promptingguide.ai/introduction/tips](https://www.promptingguide.ai/introduction/tips)​
* Learn Prompting (코스 형식, 초급\~고급)
  * The Ultimate Guide to Generative AI: [https://learnprompting.org/docs/introduction](https://learnprompting.org/docs/introduction)​
* FlowGPT Prompt Guide
  * General Tips for Designing Prompts: [https://guide.flowgpt.com/engineering/1basics/2general-tips](https://guide.flowgpt.com/engineering/1basics/2general-tips)​
* dair-ai Prompt Engineering Guide (논문·자료·예제 모음)
  * [https://github.com/dair-ai/Prompt-Engineering-Guide](https://github.com/dair-ai/Prompt-Engineering-Guide)​
* ChatGPT: Learning prompt engineering with 100+ examples (PDF 책)
  * [https://oa.upm.es/84328/1/book\_english\_version.pdf](https://oa.upm.es/84328/1/book_english_version.pdf)​
* Prompt Engineering Guide (National Bank of Greece, PDF)
  * [https://developer.nbg.gr/sites/default/files/PromptEngineeringF.pdf](https://developer.nbg.gr/sites/default/files/PromptEngineeringF.pdf)​
* Google Cloud Vertex AI – Prompt design strategies
  * [https://cloud.google.com/vertex-ai/generative-ai/docs/learn/prompts/prompt-design-strategies](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/prompts/prompt-design-strategies)​
  * 개념 설명용: Prompt Engineering for AI Guide: [https://cloud.google.com/discover/what-is-prompt-engineering](https://cloud.google.com/discover/what-is-prompt-engineering)​
* Google Gemini API – Prompting strategies
  * [https://ai.google.dev/gemini-api/docs/prompting-strategies](https://ai.google.dev/gemini-api/docs/prompting-strategies)​
* Microsoft Azure OpenAI – Prompt engineering techniques
  * [https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/prompt-engineering](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/prompt-engineering)​
* IBM – The 2025 Guide to Prompt Engineering
  * [https://www.ibm.com/think/prompt-engineering](https://www.ibm.com/think/prompt-engineering)​
* Reddit 커뮤니티 요약: Google 68-page prompt engineering guide 언급 글
  * [https://www.reddit.com/r/PromptEngineering/comments/1jws1ag/google\_just\_dropped\_a\_68page\_ultimate\_prompt/](https://www.reddit.com/r/PromptEngineering/comments/1jws1ag/google_just_dropped_a_68page_ultimate_prompt/)​
* Humanity’s Last Prompt Engineering Guide (실전 팁 모음 PDF)
  * [https://www.scribd.com/document/885491106/Humanity-s-Last-Prompt-Engineering-Guide](https://www.scribd.com/document/885491106/Humanity-s-Last-Prompt-Engineering-Guide)​
