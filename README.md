# NovelCopyKiller
소설 카피 킬러입니다.

## 저작권 사항
[[붙임1] AI 학습과 공정이용 안내서(안).pdf](https://github.com/user-attachments/files/24403603/1.AI.pdf)
에 따르면 위배되지 않는다고 판단하였습니다.

## 개요
https://docs.google.com/document/d/1-Tb6Lt4PlRidGceB7p8iRJkXxS8DEfenQHbFTBu4LlY/edit?usp=sharing

## 구현 파이프라인
### Phase 1: 데이터셋 구축 및 서사 온톨로지 정의 (Foundation)
서비스의 성능을 좌우할 'Ground Truth'와 'Taxonomy'를 정의하는 단계입니다.

장르 클리셰 및 서사 온톨로지(Taxonomy) 정의

작업 내용: 웹소설의 필수 구성 요소(회귀, 빙의, 상태창, 아카데미 등)를 정의하고, 이를 TVTropes의 데이터와 매핑하여 한국형 트로프(Trope) 사전을 구축합니다.

핵심 기술: Knowledge Graph Schema Design.

참조: TVTropes 데이터셋 활용, 나무위키 덤프 분석.

'Namuwiki-Novel' 실버 데이터셋 구축

작업 내용: 나무위키의 '줄거리', '등장인물', '설정' 파트를 파싱하여, (요약 텍스트, 구조화된 메타데이터) 쌍으로 구성된 대규모 데이터셋을 구축합니다. 이는 소설 원문 없이도 구조적 요약을 학습시키는 데 사용됩니다.

핵심 기술: namu-wiki-extractor, Python Parsing Pipeline.

합성 표절 데이터(Synthetic Plagiarism Dataset) 생성

작업 내용: 기존 소설 A의 구조(Plot Skeleton)는 유지하되, 고유명사, 문체, 배경 설정을 LLM(GPT-4 또는 Claude 3.5)으로 '세탁(Rewriting)'하여 인공적인 표절 쌍(Positive Pair)을 생성합니다.

핵심 기술: LLM Data Augmentation, Prompt Engineering (Style Transfer).

참조: PlagBench 방법론, LLM을 이용한 데이터 증강.

### Phase 2: 핵심 모듈 개발 - NLP & Extraction (Core Engine)
비정형 텍스트를 정형 데이터(Graph/Sequence)로 변환하는 단계입니다.

사건 추출기(Event Extractor) 미세 조정 (Fine-tuning)

작업 내용: 입력된 텍스트에서 5W1H 기반의 사건 튜플 $(Subject, Action, Object, Effect)$을 추출하는 SLM(Small Language Model)을 학습시킵니다. 비용 효율성을 위해 Llama-3-8B 또는 Solar-10.7B(한국어 특화)를 사용합니다.

핵심 기술: PEFT (LoRA), Instruction Tuning.

참조: 비용 효율적인 SLM 활용, 사건 추출 기법.

캐릭터 관계 추출 및 상호참조 해결 (Coreference Resolution)

작업 내용: "그", "길드장", "주인공" 등이 누구를 지칭하는지 식별하고, 인물 간의 관계(우호/적대/지배)를 추출하여 그래프 엣지로 변환합니다.

핵심 기술: BERT 기반 Coreference Resolution, Relation Extraction.

참조: 한국어 특화 상호참조 데이터셋 활용.

서사 임베딩 모델(Narrative Embedding) 학습

작업 내용: 추출된 사건 시퀀스와 텍스트를 벡터화합니다. 이때 '문체' 정보는 배제하고 '내용' 정보만 남기도록 Contrastive Learning을 적용합니다.

핵심 기술: SimCSE, Sentence-BERT.

### Phase 3: 구조적 분석 및 비교 알고리즘 구현 (Advanced Analysis)
추출된 데이터를 바탕으로 실제 유사도를 계산하는 로직을 구현합니다.

캐릭터 네트워크 임베딩 (GNN Implementation)

작업 내용: Neo4j에 저장된 캐릭터 그래프를 GNN(Graph Neural Networks)으로 학습시켜, 인물의 이름이 바뀌어도 '역할(Role)'이 유사하면 비슷한 벡터값을 갖도록 Node2Vec 혹은 GraphSAGE를 구현합니다.

핵심 기술: PyTorch Geometric, Graph Isomorphism Network (GIN).

참조: 서브그래프 임베딩(Sub2Vec), 그래프 신경망 활용.

시퀀스 정렬 알고리즘(Sequence Alignment) 최적화

작업 내용: 두 소설의 사건 벡터 시퀀스를 비교하기 위해 Smith-Waterman 알고리즘을 벡터 유사도 기반으로 변형(Soft-Alignment)하여 구현합니다. 중간에 삽입된 '필러(Filler) 에피소드'를 무시하고 핵심 사건의 흐름을 탐지합니다.

핵심 기술: Dynamic Programming, Vector Similarity Matrix.

참조: 시퀀스 정렬 알고리즘.

Narrative TF-IDF (클리셰 필터링) 로직 구현

작업 내용: 전체 데이터셋에서 너무 자주 등장하는 사건(예: 트럭에 치임, 상태창 각성)의 가중치를 낮추는 IDF(Inverse Document Frequency) 로직을 그래프/이벤트 레벨에 적용합니다.

핵심 기술: Weighted Jaccard Similarity, Statistical NLP.

참조: 그래프 구조에 대한 TF-IDF 적용.

### Phase 4: 인프라 구축 및 파이프라인 통합 (System Integration)
대규모 데이터를 처리하기 위한 시스템 아키텍처를 구축합니다.

Vector DB & Graph DB 하이브리드 구축

작업 내용: 사건 벡터 검색을 위한 Milvus와 캐릭터 관계 저장을 위한 Neo4j를 구축하고 연동합니다.

핵심 기술: Milvus (ANN Search), Neo4j (Cypher Query).

참조: 벡터 데이터베이스 활용, 그래프 검색 연동.

추론 파이프라인(Inference Pipeline) 최적화

작업 내용: 300화 이상의 장편 소설을 처리하기 위해 텍스트를 청크(Chunk) 단위로 병렬 처리하고, 결과를 취합(Reduce)하는 Map-Reduce 형태의 파이프라인을 구성합니다.

핵심 기술: Apache Kafka, Ray or Celery (Distributed Task Queue).

### Phase 5: 평가 및 시각화 (Validation & UX)
사용자에게 결과를 설득력 있게 보여주기 위한 단계입니다.

유사도 리포트 생성 및 시각화 (Explainability)

작업 내용: 단순 표절률(%) 뿐만 아니라, "A소설의 15화 사건 시퀀스가 B소설의 20화와 구조적으로 매핑됨"을 보여주는 시각화 대시보드를 개발합니다.

핵심 기술: D3.js or Recharts (Sequence/Graph Visualization).

참조: XAI(설명 가능한 AI) 기법 적용.

법적 타당성 검토를 위한 벤치마크 테스트

작업 내용: 실제 판례(저작권 침해 인정/불인정 사례) 데이터를 시스템에 입력하여, 법원의 판단과 시스템의 유사도 점수가 상관관계를 가지는지 검증합니다.

참조: 실질적 유사성 법적 기준.
