# Perceptron to RAG 🤖

퍼셉트론부터 RAG까지, LLM 핵심 개념을 직접 구현하며 학습한 내용을 정리하는 저장소입니다.

학습하면서 단순히 개념만 읽는 것이 아니라, 직접 실습하고 구현한 결과를 단계별로 쌓아가려고 합니다.

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
│   ├── vector_db_basic.ipynb
│   ├── notion_rag.ipynb
│   └── chunking_improved.ipynb
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
- 벡터 DB 기초 실습
- 노션 트러블슈팅 문서 기반 RAG 구현
- 청킹 방식 개선 및 검색 품질 비교

## 사용 기술
- Python
- LangChain
- OpenAI API
- Chroma DB
- Gradio

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
