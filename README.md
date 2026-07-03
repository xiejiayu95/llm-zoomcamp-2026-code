# LLM Zoomcamp 2026 — Personal Notes & Code

Personal working repository for the [LLM Zoomcamp](https://github.com/DataTalksClub/llm-zoomcamp) course by DataTalksClub. Contains notebooks, helper scripts, and SQLite databases built while following the course modules.

---

## Project Structure

```
llm-zoomcamp-code/
│
├── Module 1 — Agentic RAG
│   ├── 01-agentic-rag-environment.ipynb   # RAG pipeline walkthrough (OpenAI + minsearch)
│   ├── HW1-agentic-RAG.ipynb              # Homework 1 submission
│   ├── ingest.py                          # Loads FAQ data and builds a minsearch index
│   ├── rag_helper.py                      # RAGBase class: search → prompt → LLM
│   ├── main.py                            # Entry point (stub)
│   └── faq.db                             # SQLite DB for BM25/text search
│
├── Module 3 — Orchestration (Kestra)
│   ├── docker-compose.yml          # Spins up Kestra + Postgres backend
│   ├── .env                        # API keys for Kestra (not committed)
│   └── flows/
│       ├── 1_chat_without_rag.yaml         # Baseline: LLM query with no context
│       ├── 2_chat_with_rag.yaml            # RAG via Kestra KV Store + Gemini embeddings
│       ├── 3_rag_with_websearch.yaml       # RAG using live Tavily web search
│       ├── 4_simple_agent.yaml             # Single agent: summarisation with token tracking
│       ├── 5_web_research_agent.yaml       # Autonomous agent with web search tool
│       └── 6_multi_agent_research.yaml     # Multi-agent: analyst + research sub-agent
│
├── Module 2 — Vector Search
│   ├── 02-vector-search.ipynb                                         # Core vector search concepts
│   ├── 02-vector-search-reopening-the-index.ipynb                     # Persisting and reloading a vector index
│   ├── 02-vector-search-using-sqlitsearch-vector-search-in-RAG.ipynb  # sqlite-vec integration in RAG
│   ├── 02-08-pgvector.ipynb                                           # pgvector (PostgreSQL) for vector search
│   ├── sqlite-ingest.ipynb                                            # Ingesting embeddings into SQLite
│   ├── HW02-vector-search.ipynb                                       # Homework 2 submission
│   ├── download.py                                                    # Downloads ONNX model + tokenizer from HuggingFace
│   ├── embedder.py                                                    # Local ONNX inference embedder (all-MiniLM-L6-v2)
│   ├── faq_vectors2.db                                                # SQLite DB storing document embeddings
│   └── models/Xenova/all-MiniLM-L6-v2/                               # Downloaded ONNX model files
│
├── Config & Environment
│   ├── .env                   # API keys (not committed)
│   ├── .gitignore
│   ├── .python-version        # Python 3.12
│   ├── pyproject.toml         # Project metadata and dependencies
│   └── uv.lock                # Locked dependency versions
│
└── not-to-commit/             # Local secrets — never push this folder
```

---

## Setup

This project uses [uv](https://github.com/astral-sh/uv) for dependency management.

```bash
# Install dependencies
uv sync

# Or activate the virtual environment directly
source .venv/bin/activate   # macOS/Linux
.venv\Scripts\activate      # Windows
```

Create a `.env` file at the root with your OpenAI API key:

```
OPENAI_API_KEY=sk-...
```

Then launch Jupyter:

```bash
jupyter notebook
```

---

## Dependencies

Key packages (see `pyproject.toml` for full list):

| Package | Purpose |
|---|---|
| `openai` | LLM calls (GPT models) |
| `minsearch` | Lightweight in-memory BM25/text search index |
| `sqlitesearch` | SQLite-backed text + vector search |
| `sentence-transformers` | Local embedding models |
| `psycopg` | PostgreSQL client (used for pgvector in Module 2) |
| `python-dotenv` | Load `.env` variables |
| `requests` | Fetch FAQ data from DataTalksClub |

---

## Core Modules

### `ingest.py`
Fetches the DataTalksClub FAQ JSON, normalises document IDs, and builds a `minsearch.Index` over `question`, `section`, and `answer` fields filtered by `course`.

### `rag_helper.py`
Defines `RAGBase` — a reusable class that wires together search, prompt construction, and LLM calls:
- `search(query)` — retrieves top-k FAQ docs with field boosting
- `build_prompt(query, results)` — formats context into a prompt
- `llm(prompt)` — sends the prompt to the OpenAI Responses API
- `rag(query)` — end-to-end: search → prompt → answer

### `03-orchestration/`
Runs Kestra (v1.3.21) via Docker Compose with a Postgres backend. The six flows in `flows/` progress from a no-RAG baseline through RAG with static ingestion, RAG with live web search (Tavily), a single summarisation agent, an autonomous web research agent, and finally a multi-agent system where a main analyst delegates search work to a sub-agent. Uses Gemini (embedding + chat) and OpenAI interchangeably across flows. API keys (Gemini, OpenAI, Tavily) are injected as Kestra secrets via environment variables — store them in `03-orchestration/.env`.

```bash
cd 03-orchestration
docker compose up -d
# Kestra UI → http://localhost:8080
```

### `download.py`
Downloads the `Xenova/all-MiniLM-L6-v2` ONNX model and tokenizer from HuggingFace Hub into `models/`. Handles multiple ONNX candidate filenames and skips files that already exist locally. Run once before using `embedder.py`.

### `embedder.py`
Local embedding inference using ONNX Runtime — no OpenAI API needed. The `Embedder` class loads the tokenizer and ONNX model from `models/`, tokenizes input text, runs a mean-pooled forward pass, and returns L2-normalized embeddings compatible with cosine similarity search.

---

## Course Reference

[DataTalksClub/llm-zoomcamp](https://github.com/DataTalksClub/llm-zoomcamp)
