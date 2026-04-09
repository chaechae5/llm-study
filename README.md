# Perceptron to RAG 🤖

퍼셉트론부터 RAG, 그리고 AI Agent까지 LLM 핵심 개념을 직접 구현하며 학습한 내용을 정리하는 저장소입니다.

학습하면서 단순히 개념만 읽는 것이 아니라, 직접 실습하고 구현한 결과를 단계별로 쌓아가고 있습니다.

## 주요 프로젝트

### 노션 트러블슈팅 검색기

이 저장소에서 가장 구현 비중이 큰 프로젝트는 `03_RAG`의 노션 트러블슈팅 검색기입니다.

- 프로젝트 문서: [03_RAG/notion_troubleshooting_search/notion_troubleshooting_search.md](/Users/eunchae/repository/skax/03_RAG/notion_troubleshooting_search/notion_troubleshooting_search.md)
- 실행 노트북: [03_RAG/notion_troubleshooting_search/llm_노션_트러블슈팅_검색기.ipynb](/Users/eunchae/repository/skax/03_RAG/notion_troubleshooting_search/llm_노션_트러블슈팅_검색기.ipynb)
- 샘플 데이터: [03_RAG/notion_troubleshooting_search/dummy_data.zip](/Users/eunchae/repository/skax/03_RAG/notion_troubleshooting_search/dummy_data.zip)

구현한 핵심 흐름:
- Notion 문서를 Markdown으로 읽어 벡터 DB에 적재
- Chroma 기반 문서 검색과 Q&A 응답 연결
- Gradio 2단 레이아웃으로 문서 목록, 전체 문서, 요약, Q&A 제공
- 검색창과 Q&A 입력을 분리해 키워드 검색과 자연어 질문을 각각 다르게 처리
- 검색 품질 개선을 위해 키워드 매칭, 별칭 확장, 하이브리드 검색 방식 적용

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

## 배운 점
- 청킹 방식이 RAG 품질에 큰 영향을 준다.
- Tool Calling은 LLM의 판단과 코드 실행을 연결하는 구조다.
- 임베딩은 텍스트 의미를 숫자 벡터로 압축하는 핵심 개념이다.

## 커밋 예시
- `feat: 퍼셉트론 AND 게이트 구현`
- `feat: LCEL 기본 체인 구성`
- `feat: 주식 조회 Tool Calling 구현`
- `feat: 노션 RAG 기본 버전`
- `fix: 청킹 문제 발견 및 개선`
- `docs: README 업데이트`

## 진행 방식
- 각 주제별로 `ipynb` 파일을 하나씩 추가
- 실습 내용은 코드와 함께 핵심 개념을 바로 이해할 수 있도록 정리
- 필요하면 이후 `requirements.txt` 또는 실행 가이드도 추가 예정
