# RAG 구조

### Tokenizer

Hugging Face에 BPE가 많이 사용됨.
혹은 RecursiveTokenizer - 재귀적으로 문단 - 문장 - 단어 순으로 chunk 작업 수행

--------------------

임베딩 - 텍스트의 벡터 표현을 만드는 작업
벡터공간에서 가장 유사한 텍스트를 찾는 Semantic Search 작업을 수행
Langchain에서는 기본 임베딩 클래스는 두가지 메서드를 제공
1. Document를 임베딩
2. 사용자가 질문하는 것에 대한 임베딩

중요한 것은 두개에 대해서 임베딩 모델을 같은것을 사용해야한다.

------------------

### OpenEmbeddings

OpenAIEmbeddings를 default로 사용하는 경우에 업데이트를 통해서 버전이 달라지는것이 아닌지 확인이 필요함

+ 가장 범용적으로 사용. 유료임베딩 방법
+ 가장 무난하고 사용성이 좋다. 추가로 구축할 필요없음.
+ 토큰 당 과금이 되는 방식이며 성능이 보장. 
+ 로컬에 Embedding pipeline 생략 가능
- 하지만 문서의 양이 많다면 과금 부담이 클 수 있음.
가장 최근에 사용되는 OpenAIEMbedding 모델
  test-embedding-3-small(가성비) - 512dimension, 1536 dimensions
  text-embedding-3-large
  
--------------------

### CacheBackedEmbeddings

임베딩을 저장하거나 임시로 caching하여 다시 계산할 필요가 없도록 함.
OpenAIEmbedding를 통해서 한번 임베딩 수행한 이후에, 동일한 문서에 대해서는 OpenAIEmbedding를 사용하지 않고 cache에 저장된 데이터를 불러옴.
그렇지 않으면 동일한 내용에 대해서 OpenAIEmbedding 중복 과금 처리되어서 10명이 NLP 작업을 할 경우에는 계속 과금이 됨.
캐싱된 embedding vector를 가져오는 것을 매우 빠르게 조회 가능.

CacheBackedEmbeddings 사용법

'''
openai_embedding = OpenAIEmbeddings()
store = LocalFileStore("./cache/")
cached_embedder = CacheBackedEmbeddings.from_bytes_store(
  Openai_embedding, store, namespace=openai_embedding.model
)
'''

--------------------

Hugging Face에 있는 임베딩 방법들
리더보드 = https://huggingface.co/spaces/mteb/leaderboard
(MTEB) 벤치마크 상으로는 OpenAIEmbeddings 뛰어넘는 모델이 있음
벤치마크를 맹신하지말고 테스트가 필요함.

---------------------

###정리

OpenSource
1. 문서가 주로 영문으로 되어있다면 공개모델을 활용하는 것이 비용적인 측면과 향후 유지 보수에 유리
2. 임베딩 모델을 구동할 수 있는 서버가 뒷받침 되어야 함
3. HuggingFace InferenceAPI 활용 고려해볼 수 있으나 시간이 오래걸림. 반응속도

OpenAIEmbeddings
1.서버 인프라/RAG pipeline 인프라 구축이 힘든 상황이라면, OpenAIEmbedding 선택할 수 있음
2.다국어(한글포함)문서가 주를 이루는 경우라면 안정적인 선택지 일수있음
3.Cache를 반드시 사용하여 불필요한 과금을 줄일 필요가 있음

-----------------------

### VectorStore

랭체인은 수십개의 VectorStore 연동성을 제공 현재 80개
인기있는 벡터DB
클라우드 방법
Pinecone
Weaviate
ElasticSearch

로컬 방법
Chroma
*FAISS

FAISS 벡터 DB - 메타의 Fundamental AI Research 그룹에서 주도적으로 개발
Dense Vector의 유사성을 신속하게 검색하고 클러스터링하는 오픈소스 라이브러리

---------------------

### 전처리에 대한 팁(노하우)

페이지 단위 분할 - 전체 문서를 페이지 단위로 분할
중요한 metadata 정보를 태깅해서 벡터DB에 넣어주는것이 중요함
1.page 번호
2.파일명
3.MOD(modified date)
4.Author
5.키워드 추출을 할 수 있다면 키워드로도 태깅해줄수있으면 좋음.

필요한 영역 Crop
여백 등에 있는 불필요한 정보를 제거하고 중요한 내용만 추출하는 것 중요.
그리고 논문의 경우 한줄로 가져오게 되면 다른 문단 내용까지 가져오게 되는 오류가 발생.
Plumber를 사용해서 BoundingBox 처리해서 사용
혹은 PEFminer 쓰면 분할된 텍스트 구역을 잘 인식해서 가져오게 됨.

표나 그래프
1. 사전에 차트나 표를 제거 후 작업
2. 한 줄에 글자수<N개 이하는 제거

표 추출
표에는 중요한 정보가 많아서 정형화된 데이터로 잘 추출하는 것이 중요
관련 오픈소스 Library - Camelot, PaddleOCR


-----

### Retriever

1. Multi-Query Retrieve
거리 기반 벡터 DB 검색은 유사한 임베딩 문서를 찾음
그러나 Query 문구가 미묘하게 변경되거나 임베딩이 데이터 의미를 제대로 포착하지 못하는 경우 검색 결과가 달라질 수 있음

--> LLM을 사용해 주어진 사용자 입력 쿼리에 대해 서로 다른 관점에서 여러 쿼리를 생성함으로써 프롬프트 튜닝 프로세스를 자동화
각 쿼리에 대해 관련 문서 집합을 검색하고 모든 쿼리에서 고유한 유니온을 사용하여 잠재적으로 관련성이 높은 더 큰 문서 집합을 가져옴.

작동방식
우리가 입력한 질문에 대해서 여러개 잠재적인 쿼리를 만든다.
Page Rank에 대해 알려줘
+ Page Rank알고리즘의 작동원리는 무엇인가요
+ Page Rank 는 누가 개발했는가

2. Ensemble Retriever
일반적으로 의미 유사성은 Dense Retriever
키워드의 경우에는 Sparse Retriever
Ensemble Retreiver = Sparse Retriever + Dense Retriever

서로 다른 알고리즘의 강점을 활용함으로써 앙상블 리트리버는 단일 알고리즘보다 더 나은 성능을 얻을 수 있음
앙상블 간에 weight 조절 가능.

'''
#initialize the bm25 retreiver and faiss retriever
bm25_retriever = BM25Retriever.from_texts(doc_list)
bm25_retriever.k = 2

embedding = OpenAIEmbeddings()
faiss_vectorstore = FAISS.from_texts(doc_list, embedding)
faiss_retriever = faiss_vectorstore.as_retriever(search_kwargs={"k": 2})

#initialize the ensemble retriever
ensemble_retriever = EnsembleRetriever(
  retrievers=[bm25_retriever, faiss_retriever], weights=[0.5, 0.5]
  )
'''
