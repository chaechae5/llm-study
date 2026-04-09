# Perceptron to RAG 🤖

퍼셉트론부터 RAG, 그리고 AI Agent까지 LLM 핵심 개념을 직접 구현하며 학습한 내용을 정리하는 저장소입니다.

학습하면서 단순히 개념만 읽는 것이 아니라, 직접 실습하고 구현한 결과를 단계별로 쌓아가고 있습니다.

## 주요 프로젝트

### 노션 트러블슈팅 검색기

RAG를 활용해 노션 트러블슈팅 검색기를 구현했습니다.

- 프로젝트 문서: [03_RAG/notion_troubleshooting_search/notion_troubleshooting_search.md](03_RAG/notion_troubleshooting_search/notion_troubleshooting_search.md)
- 실행 노트북: [03_RAG/notion_troubleshooting_search/llm_노션_트러블슈팅_검색기.ipynb](03_RAG/notion_troubleshooting_search/llm_노션_트러블슈팅_검색기.ipynb)
- 샘플 데이터: [03_RAG/notion_troubleshooting_search/dummy_data.zip](03_RAG/notion_troubleshooting_search/dummy_data.zip)

**기술 스택**
- Python
- LangChain
- OpenAI Embeddings / ChatOpenAI
- Chroma
- Gradio

**구현한 핵심 흐름**
- Notion 문서를 Markdown으로 읽어 벡터 DB에 적재
- Chroma 기반 문서 검색과 Q&A 응답 연결
- Gradio 2단 레이아웃으로 문서 목록, 전체 문서, 요약, Q&A 제공
- 검색창과 Q&A 입력을 분리해 키워드 검색과 자연어 질문을 각각 다르게 처리
- 검색 품질 개선을 위해 키워드 매칭, 별칭 확장, 하이브리드 검색 방식 적용

**프로젝트 구조**
- `03_RAG/notion_troubleshooting_search/llm_노션_트러블슈팅_검색기.ipynb`: 검색기 구현 및 Gradio UI
- `03_RAG/notion_troubleshooting_search/notion_troubleshooting_search.md`: 구현 내용과 적용 개념 정리
- `03_RAG/notion_troubleshooting_search/dummy_data.zip`: 실습용 트러블슈팅 문서 데이터

**배운 점 / 어려웠던 점**
- 검색창과 Q&A 입력은 목적이 다르기 때문에, 같은 검색 로직으로 처리하면 품질이 쉽게 떨어진다는 점을 체감했습니다.
- 벡터 검색만으로는 `jenkins`처럼 명확한 키워드에서 관련 없는 문서가 섞여 들어올 수 있어, 키워드 매칭과 별칭 확장을 함께 쓰는 하이브리드 방식이 더 안정적이었습니다.
- Gradio 레이아웃을 2단 구조로 만들면서 스크롤 영역, 요약 박스 노출 조건, 문서 선택 시 우측 패널 갱신 같은 UI 상태 관리에서 시행착오가 있었습니다.

## Repository Structure

```text
llm-study/
├── 01_AI_기초/
│   ├── perceptron.md
│   ├── perceptron_core_concepts.md
│   ├── llm.md
│   ├── perceptron.ipynb
│   └── embedding_token.ipynb
├── 02_LangChain/
│   ├── langchain.md
│   ├── lcel_basic.ipynb
│   ├── tool_calling.ipynb
│   └── agent.ipynb
├── 03_RAG/
│   ├── notion_troubleshooting_search/
│   │   ├── notion_troubleshooting_search.md
│   │   ├── llm_노션_트러블슈팅_검색기.ipynb
│   │   └── dummy_data.zip
│   ├── vector_db_basic.ipynb
│   ├── notion_rag.ipynb
│   └── chunking_improved.ipynb
├── 04_AI_Agent/
│   ├── AI_Agent_개념.md
│   └── LangGraph_개념.md
└── README.md
```

## 학습 흐름

퍼셉트론  
→ 가중치 / 미분 / 경사하강법  
→ 임베딩 / 토크나이저  
→ LLM 동작 원리  
→ LangChain / LCEL  
→ Tool Calling / Agent  
→ 벡터 DB / RAG

## 구현한 내용

### 1. AI 기초
- 퍼셉트론 개념 정리 문서 작성
- 퍼셉트론 핵심 구성 요소 정리
- LLM 개념 정리 문서 작성
- 퍼셉트론으로 AND / OR 게이트 구현
- 임베딩 개념 이해 및 실습
- 토크나이저 동작 방식 학습

### 2. LangChain
- LangChain 개념 문서 작성
- LCEL 기반 체인 구성
- Tool Calling 실습
- Agent 자동화 흐름 구현

### 3. RAG
- 노션 트러블슈팅 검색기 문서 및 Gradio 앱 구현
- 벡터 DB 기초 실습
- 노션 트러블슈팅 문서 기반 RAG 구현
- 청킹 방식 개선 및 검색 품질 비교

### 4. AI Agent
- AI Agent 개념 문서 작성
- LangGraph 개념 문서 작성

## 사용 기술
- Python
- LangChain
- OpenAI API
- Chroma DB
- Gradio
- LangGraph

## 문서 사이트

레포의 `md` 파일들을 웹 문서처럼 탐색할 수 있도록 `MkDocs` 설정을 추가했습니다.

- 설정 파일: [mkdocs.yml](/Users/eunchae/repository/skax/mkdocs.yml)
- 홈 문서: `README.md`
- 문서 메뉴: `01_AI_기초`, `02_LangChain`, `03_RAG`, `04_AI_Agent`

로컬에서 실행:

```bash
pip install mkdocs
mkdocs serve
```

정적 사이트 빌드:

```bash
mkdocs build
```

## 배운 점
- 청킹 방식이 RAG 품질에 큰 영향을 준다.
- Tool Calling은 LLM의 판단과 코드 실행을 연결하는 구조다.
- 임베딩은 텍스트 의미를 숫자 벡터로 압축하는 핵심 개념이다.
