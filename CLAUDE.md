# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI-powered semantic search system for UK Parliament petitions. Users enter natural language queries and the system finds relevant petitions using sentence embeddings and cosine similarity, rather than keyword matching.

## Architecture

**Backend** (`backend/app/`): FastAPI application with a single `PetitionSearchEngine` class that loads petition data from CSV, generates embeddings via Sentence Transformers (`paraphrase-multilingual-MiniLM-L12-v2`), and performs cosine similarity search. Embeddings are cached to disk as pickle files.

**Frontend** (`frontend/`): Vanilla HTML/CSS/JS app (no build step). Communicates with the backend via fetch to `http://localhost:8000`. Uses Chart.js (loaded via CDN) for analytics visualizations (coverage pie chart, signatures bar chart, status distribution).

**Data flow**: CSV → petition dicts held in memory → embeddings generated/cached as pickle → query encoded at search time → cosine similarity → ranked results returned as JSON.

## Commands

```bash
# Install backend dependencies
cd backend && pip install -r requirements.txt

# Start backend (runs on localhost:8000)
cd backend && python -m app.main

# Serve frontend (runs on localhost:3000)
cd frontend && python -m http.server 3000

# API docs available at http://localhost:8000/docs
```

No test suite exists yet. Testing would use `pytest` and `httpx` (commented out in requirements.txt).

## Key Files

- `backend/app/search_engine.py` — Core search logic: `PetitionSearchEngine` class with `search()` and `get_search_analytics()` methods
- `backend/app/main.py` — FastAPI routes: `/api/search` (POST), `/api/search/analytics` (POST), `/api/stats` (GET), `/api/health` (GET)
- `backend/app/models.py` — Pydantic models: `SearchRequest`, `PetitionResult`, `SearchResponse`
- `backend/app/config.py` — Config constants: paths, model name, host/port
- `backend/data/petitions.csv` — Source petition data (~800KB)
- `frontend/script.js` — All frontend logic including API calls and Chart.js chart rendering

## API Endpoints

| Endpoint | Method | Description |
|---|---|---|
| `/api/search` | POST | Semantic search. Body: `{"query": "...", "limit": 10}` |
| `/api/search/analytics` | POST | Search analytics with chart data. Body: `{"query": "..."}` |
| `/api/stats` | GET | Dataset statistics |
| `/api/health` | GET | Health check |

## Notable Design Decisions

- Embeddings are cached as pickle in `backend/data/embeddings_cache.pkl` (gitignored). First run generates them (~5s for ~1000 petitions).
- CORS is fully open (`allow_origins=["*"]`) for local development.
- The frontend hardcodes `API_BASE_URL = 'http://localhost:8000'` in `script.js`.
- The `SearchRequest` model only has `query` and `limit` fields; the frontend sends `filters` in the request body but the backend ignores them (filtering is not implemented server-side).
