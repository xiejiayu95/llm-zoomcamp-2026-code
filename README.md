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
├── Module 2 — Vector Search
│   ├── 02-vector-search.ipynb                                      # Core vector search concepts
│   ├── 02-vector-search-reopening-the-index.ipynb                  # Persisting and reloading a vector index
│   ├── 02-vector-search-using-sqlitsearch-vector-search-in-RAG.ipynb  # sqlite-vec integration in RAG
│   ├── 02-08-pgvector.ipynb                                        # pgvector (PostgreSQL) for vector search
│   ├── sqlite-ingest.ipynb                                         # Ingesting embeddings into SQLite
│   └── faq_vectors2.db                                             # SQLite DB storing document embeddings
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

---

## Course Reference

[DataTalksClub/llm-zoomcamp](https://github.com/DataTalksClub/llm-zoomcamp)
