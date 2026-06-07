# RAG Chatbot

A production-grade Retrieval-Augmented Generation (RAG) chatbot that answers questions grounded in your documents. Upload any PDF and get cited, accurate answers — not hallucinations.

Built as part of a structured AI/ML engineering learning plan.

---

## What it does

- Accepts PDF or text documents as input
- Chunks and embeds documents into a FAISS vector store
- Retrieves the most relevant chunks for any user question
- Re-ranks retrieved chunks using a cross-encoder for higher accuracy
- Generates grounded answers using GPT-4o-mini
- Returns answers with source citations and page numbers
- Evaluates answer quality using Ragas metrics (faithfulness, relevancy, precision)

---

## Tech stack

| Layer | Technology |
|---|---|
| LLM | OpenAI GPT-4o-mini |
| Embeddings | OpenAI text-embedding-3-small |
| Vector store | FAISS (local) |
| Orchestration | LangChain |
| Re-ranking | cross-encoder/ms-marco-MiniLM-L-6-v2 |
| Evaluation | Ragas |
| API | FastAPI |
| UI | Streamlit |
| Containerisation | Docker + Docker Compose |
| CI/CD | GitHub Actions |
| Deployment | AWS ECS |

---

## Project structure

```
rag-chatbot/
├── app/
│   ├── __init__.py
│   ├── rag_engine.py       # core RAG logic: load, chunk, embed, retrieve, generate
│   ├── reranker.py         # cross-encoder re-ranking
│   ├── evaluator.py        # Ragas evaluation runner
│   ├── api.py              # FastAPI routes
│   └── models.py           # Pydantic request/response schemas
├── ui/
│   └── streamlit_app.py    # Streamlit frontend
├── tests/
│   ├── __init__.py
│   ├── test_rag_engine.py
│   └── test_api.py
├── data/                   # place your PDFs here (gitignored)
├── .github/
│   └── workflows/
│       └── ci.yml          # lint + test + Docker build on every push
├── .env.example            # required environment variables
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
└── README.md
```

---

## Getting started

### Prerequisites

- Python 3.11+
- An OpenAI API key — get one at [platform.openai.com](https://platform.openai.com)

### Local setup

```bash
# 1. Clone the repo
git clone https://github.com/YOUR_USERNAME/rag-chatbot.git
cd rag-chatbot

# 2. Create and activate virtual environment
python -m venv venv
source venv/bin/activate        # Mac/Linux
venv\Scripts\activate           # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Set up environment variables
cp .env.example .env
# Open .env and add your OpenAI API key

# 5. Drop a PDF into the data/ folder, then run
python main.py
```

### Run with Docker

```bash
docker-compose up --build
```

The API will be available at `http://localhost:8000` and the UI at `http://localhost:8501`.

---

## How it works

### Ingestion pipeline (runs once per document)

```
PDF → load pages → chunk (500 chars, 200 overlap) → embed → store in FAISS
```

### Query pipeline (runs on every question)

```
Question → embed → vector search (top 10) → re-rank (top 3) → LLM → answer + citations
```

### Why re-ranking?

FAISS retrieves by embedding similarity, which is fast but coarse. A cross-encoder re-ranker then scores each retrieved chunk directly against the question, reordering them by true relevance. This significantly reduces irrelevant chunks reaching the LLM.

### Chunking strategy

Documents are split using `RecursiveCharacterTextSplitter` with `chunk_size=500` and `chunk_overlap=200`. The splitter tries paragraph boundaries first, then sentences, then words — never cutting mid-word. Overlap ensures context is not lost at chunk boundaries.

---

## Evaluation

Run the Ragas evaluation suite against a test question set:

```bash
python -m app.evaluator
```

Metrics reported:

| Metric | What it measures |
|---|---|
| Faithfulness | Does the answer stick to the retrieved chunks? |
| Answer relevancy | Does the answer address the question? |
| Context precision | Are the retrieved chunks actually relevant? |

---

## CI/CD

Every push to any branch triggers:
1. `flake8` linting
2. `pytest` unit tests
3. Docker image build check

Merges to `main` additionally trigger deployment to AWS ECS.

---

## Environment variables

| Variable | Required | Description |
|---|---|---|
| `OPENAI_API_KEY` | Yes | Your OpenAI API key |

---

## Design decisions

**Why FAISS over a hosted vector DB (Pinecone, Weaviate)?**
FAISS runs locally with zero cost and zero latency. For a portfolio project processing documents under 1GB, it is the right tool. A hosted DB would be appropriate if the index needed to persist across container restarts or be shared across multiple service instances.

**Why GPT-4o-mini over GPT-4o?**
Cost. GPT-4o-mini is 15× cheaper per token with comparable quality on document Q&A tasks. The system prompt constrains the model to answer only from retrieved context, so raw reasoning power matters less than instruction-following.

**Why FastAPI over Flask?**
Async support, automatic OpenAPI docs at `/docs`, and Pydantic validation built in. For an ML API that may need to handle concurrent embedding requests, async matters.

---

## Roadmap

- [ ] Persistent FAISS index (reload without re-embedding)
- [ ] Multi-document support with per-document filtering
- [ ] Streaming responses in the UI
- [ ] Authentication on the FastAPI endpoints
- [ ] AWS deployment with ECS + ECR

---

## Author

Vimodh — AI/ML engineering portfolio project, 2025.