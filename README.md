# RAG Contract — Local Runbook

This repository contains two main parts:

- Backend RAG API (FastAPI + Python) — located at the repo root under `src/`
- Frontend UI (Vite + React + TypeScript) — located in `clear-clause-ai-main/clear-clause-ai-main`
- Small News proxy service (Flask) used by the frontend — located at `clear-clause-ai-main/clear-clause-ai-main/src/components/news`

This README explains how to run the full stack locally using three terminals (backend, news proxy, frontend) concurrently.

Prerequisites
-------------

- Python 3.10+ (ensure `python3` is available)
- Node.js 18+ and npm
- Optional: `virtualenv` (we use `python3 -m venv` below)

Ports used
----------

- Backend FastAPI: `http://localhost:8000`
- News Flask proxy: `http://localhost:3000`
- Frontend Vite dev server: typically `http://localhost:5173` or `http://localhost:8080`

Quick start (three terminals)
-----------------------------

Terminal 1 — Backend (Python / FastAPI)

```zsh
cd /Users/tanishkasingh/Desktop/projects/contract/rag-contract

# Create virtual environment
python3 -m venv .venv

# Activate it
source .venv/bin/activate

# Install dependencies
pip install --upgrade pip
pip install -r requirements.txt
# Make sure you're in the venv (again, if needed)
source .venv/bin/activate

# Set the API key (replace with your Gemini API key)
export GEMINI_API_KEY="api key"

# Start the backend
python3 src/main.py
```

Terminal 2 — News proxy (Flask)

```zsh
cd /Users/tanishkasingh/Desktop/projects/contract/rag-contract/clear-clause-ai-main/clear-clause-ai-main/src/components/news

# Install dependencies if needed (use the same Python venv or system Python)
pip install flask flask-cors requests feedparser

# Start the news server
python3 app.py
```

Terminal 3 — Frontend (Vite + React)

```zsh
cd /Users/tanishkasingh/Desktop/projects/contract/rag-contract/clear-clause-ai-main/clear-clause-ai-main

# Install node dependencies (only required once or when package.json changes)
npm install

# Start the dev server (run concurrently with the backend)
npm run dev
```

Notes & tips
------------

- Environment variables: Frontend (Vite) environment variables must start with `VITE_` to be embedded at build time. For backend secrets like `GEMINI_API_KEY` keep them in shell env or a `.env` file read by the backend (`python-dotenv` is used).
- Add your `.env` or `.env.local` to `.gitignore` to avoid committing secrets.
- If you prefer a single terminal, consider using a process manager (tmux, GNU parallel, or a simple shell script) but running in separate terminals makes logs easier to inspect.
- If the frontend cannot reach `http://localhost:3000` due to mixed content (HTTPS) or CORS, run both frontend and news proxy over HTTP for local development.
- If any Python packages fail to install (native wheels like `faiss-cpu`, `opencv-python`), follow pip error guidance — on macOS Homebrew may be required for system libraries.

Troubleshooting
---------------

- `ModuleNotFoundError: No module named 'pydantic_settings'` — ensure you installed `requirements.txt` inside the activated virtualenv. If missing, run `pip install pydantic-settings`.
- News not appearing in UI — make sure the news Flask server is running on `http://localhost:3000/news/<topic>` and check the browser DevTools Network tab. The frontend will try the backend proxy first, then fall back to a public CORS proxy if the backend is unavailable.
- If you see CORS errors, confirm `flask_cors` is installed and the Flask app shows CORS enabled (it is enabled in `app.py`).

Project layout (top-level)
--------------------------

```
rag-contract/
├─ src/                       # Backend (FastAPI) + RAG pipeline
├─ requirements.txt           # Python dependencies
├─ clear-clause-ai-main/      # Frontend app (Vite + React + TS)
│  └─ clear-clause-ai-main/
│     └─ src/
│        └─ components/news/  # News Flask proxy lives under src/components/news for convenience
```

If you'd like, I can also:

- Add a `start-dev.sh` script to run all three processes locally (using tmux or backgrounded processes).
- Add a small README section describing how to add the Gemini API key to a `.env` file and to `.gitignore` it.

Happy to update the repo with the README file now.
# AI-Powered Contract & Policy Assistant

## 📄 Project Overview

The AI Contract & Policy Assistant is a **Retrieval-Augmented Generation (RAG) system** that allows users to upload contracts or policy documents and interact with them intelligently.  
It can **extract, classify, and explain contract clauses, summarize documents, answer user queries, and compare clauses to standard templates** using a large language model (Gemini LLM).  

This project is designed for legal professionals, HR teams, or anyone needing fast insights from contracts without reading the full document manually.  

---

## 🚀 Features

- Upload and parse **PDF, DOCX, or TXT** contracts.  
- Automatic **text chunking** and **FAISS-based retrieval** for efficient search.  
- Ask **natural language questions** about any contract clause.  
- **View retrieved chunks** for transparency.  
- Generate a **concise summary** of the document.  
- **Compare specific clauses** against a standard template and highlight differences.  
- **Interactive Streamlit UI** for ease of use.  
- Support for **multiple queries** on the same document without re-uploading.  

---

## 🛠️ Tech Stack

- **Python 3.10+**  
- **Streamlit** – Web UI  
- **FAISS** – Vector search for document chunks  
- **Sentence-Transformers** – Embedding model (`all-MiniLM-L6-v2`)  
- **Gemini LLM** – Free-tier large language model for natural language understanding  
- **PDFPlumber, python-docx** – Document parsing  
- **NumPy** – Numerical computations  

---

## ⚙️ Setup & Installation

Follow these steps to run the project locally:

### 1. Clone the repository
```bash
git clone https://github.com/<your-username>/rag-contract-assistant.git
cd rag-contract-assistant

# 2. Create Virtual Env

# Windows
python -m venv venv
venv\Scripts\activate

# Mac/Linux
python3 -m venv venv
source venv/bin/activate

# 3.Install requirements
pip install -r requirements.txt

# Windows
set GEMINI_API_KEY="YOUR_API_KEY_HERE"

# Mac/Linux
export GEMINI_API_KEY="YOUR_API_KEY_HERE"

## FastAPI Backend Setup

This repository now includes a **production-ready FastAPI backend** that wraps the existing RAG modules and exposes REST endpoints for document processing, querying, summarization, clause comparison, and risk analysis.

### Installation

1. Install dependencies:
```bash
pip install -r requirements.txt
```

2. Copy `.env.example` to `.env` and configure:
```bash
cp .env.example .env
# Edit .env and add your GEMINI_API_KEY and other settings
```

### Running the API

**Development** (with auto-reload):
```bash
python src/main.py
```

**Production** (with uvicorn):
```bash
uvicorn src.api:app --host 0.0.0.0 --port 8000 --workers 4
```

### API Endpoints

**Base URL:** `http://localhost:8000` (development)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/upload` | Upload and process a document |
| POST | `/api/query` | Query an indexed document |
| POST | `/api/summarize` | Generate document summary |
| POST | `/api/compare` | Compare clause to templates |
| POST | `/api/analyze` | Analyze for risks/compliance |
| GET | `/api/health` | Health check with metrics |
| GET | `/api/health/detailed` | Detailed system metrics |

**Interactive Documentation:**
- Swagger UI: `http://localhost:8000/api/docs`
- ReDoc: `http://localhost:8000/api/redoc`

### Configuration

All settings are controlled via environment variables in `.env` file or system environment:

**Required:**
- `GEMINI_API_KEY` - Google Gemini API key

**Core Features:**
- `MAX_FILE_SIZE_MB` - Max upload size (default: 10)
- `CHUNK_SIZE` - Text chunk size for embeddings (default: 500)
- `TOP_K_RESULTS` - Retrieved chunks per query (default: 3)

**Session Management:**
- `SESSION_MAX_AGE_HOURS` - Session expiration (default: 24)
- `SESSION_CLEANUP_INTERVAL_MINUTES` - Cleanup frequency (default: 60)

**API & Logging:**
- `DEBUG` - Enable debug mode (default: False)
- `LOG_LEVEL` - Logging verbosity: DEBUG, INFO, WARNING, ERROR (default: INFO)
- `LOG_FILE` - Log file path (default: logs/api.log)
- `ENABLE_API_DOCS` - Show API docs (default: True)
- `CORS_ORIGINS` - Allowed frontend URLs for CORS

See `.env.example` for complete settings and descriptions.

### Production Enhancements

The FastAPI backend includes production-ready features:

**Error Handling:**
- Custom exception handlers for validation, session, and server errors
- Structured error responses with HTTP status codes
- Detailed error logging with request tracing

**Logging & Monitoring:**
- Request/response logging with unique request IDs
- Configurable log levels (DEBUG to CRITICAL)
- File rotation to prevent disk space issues
- Health check endpoint with session metrics
- Detailed metrics endpoint for monitoring

**Session Management:**
- Thread-safe in-memory session storage
- Automatic session cleanup (background task)
- Session expiry validation
- Last-accessed tracking

**Security:**
- CORS configuration for frontend integration
- File type and size validation
- File content verification (PDF/DOCX/TXT)
- Path traversal prevention

**API Documentation:**
- OpenAPI/Swagger integration with examples
- Endpoint tagging and descriptions
- Pydantic model examples in API docs

### Architecture

The backend uses modular components shared with Streamlit:

- `document_loader.py` – Extract text from PDF/DOCX/TXT
- `chunking.py` – Split text into overlapping chunks
- `embedding_store.py` – Build FAISS index and search
- `rag_pipeline.py` – Retrieve chunks and query LLM
- `llm_interface.py` – Google Gemini LLM wrapper
- `api.py` – FastAPI app with endpoints and middleware
- `session_manager.py` – Thread-safe session storage
- `config.py` – Environment-based configuration
- `utils.py` – Validation, logging, file handling
- `models.py` – Pydantic request/response schemas

### Troubleshooting

**"GEMINI_API_KEY not set" error:**
- Ensure `GEMINI_API_KEY` is in `.env` or exported to environment
- Windows: `set GEMINI_API_KEY=your_key`
- Mac/Linux: `export GEMINI_API_KEY=your_key`

**"Session not found" (404):**
- Session has expired (default: 24 hours)
- Upload file again to create new session

**"File too large" (400):**
- Reduce file size or increase `MAX_FILE_SIZE_MB` in `.env`

**CORS errors from frontend:**
- Add frontend URL to `CORS_ORIGINS` in `.env`
- Example: `CORS_ORIGINS=http://localhost:5173,https://yourdomain.com`

**High memory usage:**
- Reduce `SESSION_MAX_AGE_HOURS`
- Increase `SESSION_CLEANUP_INTERVAL_MINUTES`
- Monitor with `/api/health/detailed` endpoint

### Frontend Integration

The React frontend includes an API service layer in `src/services/` that communicates with this backend:

```typescript
import { uploadDocument, queryDocument } from '@/services';

const response = await uploadDocument(file);
const answer = await queryDocument(response.session_id, "What are the terms?");
```

Configure frontend backend URL in `.env.local`:
```
VITE_API_BASE_URL=http://localhost:8000
```

### Architecture

The backend uses modular components that are shared with the Streamlit app:

- `document_loader.py` – Extracts text from PDF/DOCX/TXT
- `chunking.py` – Splits text into chunks
- `embedding_store.py` – Builds FAISS index and searches
- `rag_pipeline.py` – Orchestrates retrieval + LLM prompt
- `llm_interface.py` – Gemini LLM wrapper (now reads `GEMINI_API_KEY` from env)

The new FastAPI endpoints use a `SessionManager` to keep an in-memory mapping between `session_id` and the FAISS index + chunk map. This enables multiple queries against the same uploaded document without re-indexing.

Note: The Streamlit UI is still present for quick demos. The FastAPI backend is recommended for production or for integration with React frontends.
