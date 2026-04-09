<section class="hero">
  <div class="hero-kicker">LLM Study Archive</div>
  <h1>Perceptron to RAG</h1>
  <p>퍼셉트론부터 RAG, 그리고 AI Agent까지 학습한 내용을 문서 중심으로 정리한 아카이브입니다. 개념 정리에서 끝나지 않고, 직접 구현한 결과와 시행착오까지 함께 남기는 것을 목표로 했습니다.</p>
  <div class="hero-links">
    <a class="primary" href="03_RAG/notion_troubleshooting_search/notion_troubleshooting_search/">대표 프로젝트 보기</a>
    <a class="secondary" href="04_AI_Agent/AI_Agent_개념/">AI Agent 문서 보기</a>
  </div>
</section>

<div class="stats">
  <div class="stat-card">
    <strong>4</strong>
    <span>학습 트랙</span>
  </div>
  <div class="stat-card">
    <strong>7+</strong>
    <span>핵심 문서</span>
  </div>
  <div class="stat-card">
    <strong>1</strong>
    <span>대표 실전 프로젝트</span>
  </div>
</div>

<div class="section-title">대표 프로젝트</div>
<p class="lead">이 저장소에서 가장 구현 비중이 큰 프로젝트는 <strong>노션 트러블슈팅 검색기</strong>입니다. 문서 검색, Q&amp;A, 요약, UI 상태 관리까지 한 흐름으로 엮었습니다.</p>

<div class="card-grid">
  <div class="link-card">
    <h3>노션 트러블슈팅 검색기</h3>
    <p>RAG 기반 문서 검색기 구현 배경, 검색 전략, UI 흐름, 시행착오를 정리한 문서입니다.</p>
    <a href="03_RAG/notion_troubleshooting_search/notion_troubleshooting_search/">프로젝트 문서 열기</a>
  </div>
  <div class="link-card">
    <h3>핵심 포인트</h3>
    <ul class="bullet-list">
      <li>키워드 검색과 Q&amp;A 입력 분리</li>
      <li>Chroma 기반 검색과 하이브리드 보정</li>
      <li>Gradio 2단 UI와 요약/문서 상태 관리</li>
    </ul>
  </div>
</div>

<div class="section-title">문서 둘러보기</div>
<p class="lead">각 학습 단계는 독립적인 문서로 정리되어 있어, 왼쪽 네비게이션만으로 흐름을 따라가며 볼 수 있습니다.</p>

<div class="card-grid">
  <div class="link-card">
    <h3>01. AI 기초</h3>
    <p>퍼셉트론, LLM 개념, 토큰과 임베딩까지 기초 흐름을 다룹니다.</p>
    <a href="01_AI_기초/llm/">AI 기초 문서 보기</a>
  </div>
  <div class="link-card">
    <h3>02. LangChain</h3>
    <p>LCEL, Tool Calling, Agent로 이어지는 실습 중심 내용을 정리했습니다.</p>
    <a href="02_LangChain/langchain/">LangChain 문서 보기</a>
  </div>
  <div class="link-card">
    <h3>03. RAG</h3>
    <p>문서 검색, 벡터 DB, 검색 품질 개선 같은 실전형 주제를 담았습니다.</p>
    <a href="03_RAG/notion_troubleshooting_search/notion_troubleshooting_search/">RAG 문서 보기</a>
  </div>
  <div class="link-card">
    <h3>04. AI Agent</h3>
    <p>AI Agent 개념과 LangGraph 구조를 별도 문서로 정리했습니다.</p>
    <a href="04_AI_Agent/LangGraph_개념/">AI Agent 문서 보기</a>
  </div>
</div>

<div class="section-title">기술 스택</div>
<div class="pill-list">
  <span>Python</span>
  <span>LangChain</span>
  <span>OpenAI API</span>
  <span>Chroma</span>
  <span>Gradio</span>
  <span>LangGraph</span>
  <span>MkDocs</span>
</div>

<div class="section-title">배운 점</div>
<div class="card-grid">
  <div class="timeline-card">
    <h3>검색 로직은 목적별로 달라야 했다</h3>
    <p>검색창과 Q&amp;A는 사용 목적이 달라 같은 로직으로 묶으면 품질이 쉽게 흔들렸습니다.</p>
  </div>
  <div class="timeline-card">
    <h3>벡터 검색만으로는 부족했다</h3>
    <p><code>jenkins</code> 같은 명확한 키워드에서는 키워드 매칭과 별칭 확장을 결합한 하이브리드 보정이 더 안정적이었습니다.</p>
  </div>
  <div class="timeline-card">
    <h3>문서보다 UI 상태 관리가 더 까다로웠다</h3>
    <p>Gradio에서 스크롤 영역, 요약 박스 노출 조건, 우측 패널 갱신 타이밍을 맞추는 과정에서 시행착오가 많았습니다.</p>
  </div>
  <div class="timeline-card">
    <h3>실행 환경도 구현의 일부였다</h3>
    <p>문서 인코딩, Chroma 저장 경로, 노트북 실행 위치에 따라 생기는 문제를 정리하는 과정도 중요한 학습 포인트였습니다.</p>
  </div>
</div>
