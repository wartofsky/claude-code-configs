# Python Scraping API

FastAPI backend for web scraping and data extraction.

## Tech Stack

| Component | Technology |
|-----------|------------|
| Language | Python 3.12+ |
| Framework | FastAPI, Pydantic v2 |
| Database | PostgreSQL 16, SQLAlchemy 2.0 (async), Alembic |
| AI | OpenAI gpt-4o-mini |
| Scraping | BeautifulSoup, httpx |
| Real-time | WebSockets |
| Infrastructure | Docker |
| Monitoring | Prometheus metrics |

## Project Structure

```
src/
├── api/
│   └── v1/
│       ├── routes/
│       │   ├── scrape.py          # Scraping endpoints
│       │   ├── extract.py         # AI extraction endpoints
│       │   ├── tasks.py           # Task queue management
│       │   └── websocket.py       # WS connections
│       └── router.py
├── core/
│   ├── config.py                  # Settings (Pydantic)
│   ├── openai.py                  # OpenAI client
│   ├── exceptions.py              # Custom exceptions
│   └── security.py                # Auth/JWT
├── models/
│   ├── source.py                  # Data sources
│   ├── extraction.py              # Extracted data
│   └── task.py
├── schemas/
│   ├── source.py
│   ├── extraction.py
│   └── task.py
├── services/
│   ├── scraper.py                 # Web scraping logic
│   ├── extractor.py               # AI data extraction
│   ├── parser.py                  # HTML/PDF parsing
│   └── task_queue.py              # Background tasks
├── repositories/
│   ├── source.py
│   ├── extraction.py
│   └── task.py
└── main.py
```

## Key Domains

### Web Scraping
1. User submits URL via `/api/v1/scrape`
2. Task queued → WebSocket notification
3. Worker fetches and parses content
4. Data normalized and saved
5. WebSocket notification on complete/fail

### AI Extraction
1. Trigger via `/api/v1/extract`
2. Send content to OpenAI for structured extraction
3. Parse and validate response
4. Save extracted data

### Task Queue
- Max concurrent tasks configurable
- Checkpointing for recovery
- WebSocket progress updates

## Environment Variables

```bash
# Database
DATABASE_URL=postgresql+asyncpg://user:pass@host/db

# OpenAI
OPENAI_API_KEY=sk-...

# Auth
JWT_SECRET=
JWT_ALGORITHM=HS256
```

## Commands

```bash
# Development
uvicorn src.main:app --reload

# Database
alembic upgrade head
alembic revision --autogenerate -m "description"

# Docker
docker build -t scraping-api .
docker run -p 8000:8000 scraping-api

# Tests
pytest
pytest --cov=src
```

## API Response Format

```json
// Success
{ "data": {...}, "meta": {...} }

// Error
{ "error": { "code": "ERROR_CODE", "message": "..." } }

// Task Created
{ "task_id": "uuid", "status": "pending" }
```

## WebSocket Events

```json
// Client → Server
"ping"

// Server → Client
{ "type": "task_queued", "task_id": "..." }
{ "type": "task_started", "task_id": "..." }
{ "type": "task_progress", "task_id": "...", "progress": 50 }
{ "type": "task_completed", "task_id": "...", "result": {...} }
{ "type": "task_failed", "task_id": "...", "error": "..." }
```

## Best Practices

### Rate Limiting
- Scraping: 2 requests/second per domain
- OpenAI: Exponential backoff on 429

### Deduplication
- Use content fingerprints to avoid duplicates

### Error Handling
- Retry transient failures with backoff
- Log and report permanent failures

## Notes

<!-- Project-specific notes, decisions, TODOs -->
