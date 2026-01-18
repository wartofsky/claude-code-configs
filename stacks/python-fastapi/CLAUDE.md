# Project Context

## Tech Stack

- **Language**: Python 3.12+
- **Framework**: FastAPI
- **ORM**: SQLAlchemy 2.0 (async)
- **Validation**: Pydantic v2
- **Database**: PostgreSQL
- **Migrations**: Alembic
- **Testing**: pytest, pytest-asyncio

## Project Structure

```
src/
├── api/
│   ├── v1/
│   │   ├── routes/          # Route handlers
│   │   └── router.py        # Version router
│   └── deps.py              # Dependencies
├── core/
│   ├── config.py            # Settings
│   ├── security.py          # Auth utilities
│   └── exceptions.py        # Custom exceptions
├── models/                   # SQLAlchemy models
├── schemas/                  # Pydantic schemas
├── services/                 # Business logic
├── repositories/             # Data access
└── main.py
tests/
├── conftest.py
├── unit/
├── integration/
└── api/
```

## Conventions

### Naming
- Files: `snake_case.py`
- Classes: `PascalCase`
- Functions: `snake_case`
- Constants: `UPPER_SNAKE_CASE`

### Layers
1. **Routes**: HTTP handling, validation
2. **Services**: Business logic
3. **Repositories**: Database access
4. **Models**: SQLAlchemy entities
5. **Schemas**: Pydantic DTOs

### Async
- All database operations are async
- Use `httpx` for HTTP calls (not requests)
- Use `asyncio.gather` for parallel operations

## Commands

```bash
# Development
uvicorn src.main:app --reload

# Database
alembic upgrade head
alembic revision --autogenerate -m "message"

# Tests
pytest
pytest --cov=src

# Lint
ruff check .
ruff format .

# Type check
mypy src
```

## Environment

```bash
# .env
DATABASE_URL=postgresql+asyncpg://user:pass@localhost/db
SECRET_KEY=
DEBUG=true
```

## API Standards

### Response Format
```json
// Success
{ "data": { ... }, "meta": { ... } }

// Error
{ "error": { "code": "ERROR_CODE", "message": "..." } }
```

### Status Codes
- 200: OK
- 201: Created
- 204: No Content
- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 422: Validation Error
- 500: Internal Error

## Notes

<!-- Project-specific notes -->
