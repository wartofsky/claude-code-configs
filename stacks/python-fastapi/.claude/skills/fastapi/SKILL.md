---
name: fastapi
description: FastAPI patterns with Pydantic v2, async/await, dependency injection, and modern Python. Use when creating API endpoints, schemas, dependencies, or middleware. Triggers on FastAPI, API, endpoint, Pydantic, or route keywords.
---

# FastAPI Modern Patterns

## Quick Reference

```python
from fastapi import FastAPI, APIRouter, Depends, HTTPException, status, Query, Path, Body
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field, ConfigDict, EmailStr
from typing import Annotated
```

## App Setup

```python
from fastapi import FastAPI
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    await init_db()
    yield
    # Shutdown
    await close_db()

app = FastAPI(
    title="My API",
    version="1.0.0",
    lifespan=lifespan
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routers
app.include_router(users.router, prefix="/api/v1")
```

## Pydantic v2 Schemas

```python
from pydantic import BaseModel, Field, ConfigDict, field_validator
from datetime import datetime

class UserBase(BaseModel):
    email: EmailStr
    name: str = Field(min_length=1, max_length=100)

class UserCreate(UserBase):
    password: str = Field(min_length=8)
    
    @field_validator('password')
    @classmethod
    def validate_password(cls, v: str) -> str:
        if not any(c.isupper() for c in v):
            raise ValueError('must contain uppercase')
        return v

class UserResponse(UserBase):
    id: int
    created_at: datetime
    
    model_config = ConfigDict(from_attributes=True)

# Partial update (all optional)
class UserUpdate(BaseModel):
    email: EmailStr | None = None
    name: str | None = Field(default=None, min_length=1, max_length=100)
```

## Routes

```python
from fastapi import APIRouter, Depends, HTTPException, status, Query
from typing import Annotated

router = APIRouter(prefix="/users", tags=["users"])

# List with pagination
@router.get("", response_model=list[UserResponse])
async def list_users(
    skip: Annotated[int, Query(ge=0, description="Items to skip")] = 0,
    limit: Annotated[int, Query(ge=1, le=100, description="Items per page")] = 20,
    service: UserService = Depends(get_user_service)
) -> list[User]:
    return await service.list(skip=skip, limit=limit)

# Get by ID
@router.get("/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: Annotated[int, Path(gt=0)],
    service: UserService = Depends(get_user_service)
) -> User:
    user = await service.get(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

# Create
@router.post("", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def create_user(
    data: UserCreate,
    service: UserService = Depends(get_user_service)
) -> User:
    return await service.create(data)

# Update
@router.patch("/{user_id}", response_model=UserResponse)
async def update_user(
    user_id: int,
    data: UserUpdate,
    service: UserService = Depends(get_user_service)
) -> User:
    user = await service.update(user_id, data)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

# Delete
@router.delete("/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_user(
    user_id: int,
    service: UserService = Depends(get_user_service)
) -> None:
    deleted = await service.delete(user_id)
    if not deleted:
        raise HTTPException(status_code=404, detail="User not found")
```

## Dependency Injection

```python
from typing import Annotated, AsyncGenerator
from fastapi import Depends

# Database session
async def get_session() -> AsyncGenerator[AsyncSession, None]:
    async with async_session_maker() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise

SessionDep = Annotated[AsyncSession, Depends(get_session)]

# Auth
async def get_current_user(
    token: Annotated[str, Depends(oauth2_scheme)],
    session: SessionDep
) -> User:
    payload = decode_token(token)
    user = await get_user(session, payload["sub"])
    if not user:
        raise HTTPException(status_code=401, detail="Invalid token")
    return user

CurrentUser = Annotated[User, Depends(get_current_user)]

# Service with dependencies
def get_user_service(session: SessionDep) -> UserService:
    repo = UserRepository(session)
    return UserService(repo)
```

## Error Handling

```python
from fastapi import Request
from fastapi.responses import JSONResponse

class AppError(Exception):
    def __init__(self, code: str, message: str, status_code: int = 400):
        self.code = code
        self.message = message
        self.status_code = status_code

@app.exception_handler(AppError)
async def app_error_handler(request: Request, exc: AppError):
    return JSONResponse(
        status_code=exc.status_code,
        content={"error": {"code": exc.code, "message": exc.message}}
    )

# Validation errors are automatic with Pydantic
# Returns 422 with field details
```

## Background Tasks

```python
from fastapi import BackgroundTasks

@router.post("/users")
async def create_user(
    data: UserCreate,
    background_tasks: BackgroundTasks
):
    user = await service.create(data)
    background_tasks.add_task(send_welcome_email, user.email)
    return user

async def send_welcome_email(email: str):
    # Send email async
    ...
```

## WebSocket

```python
from fastapi import WebSocket, WebSocketDisconnect

@app.websocket("/ws/{client_id}")
async def websocket_endpoint(websocket: WebSocket, client_id: str):
    await websocket.accept()
    try:
        while True:
            data = await websocket.receive_text()
            await websocket.send_text(f"Echo: {data}")
    except WebSocketDisconnect:
        print(f"Client {client_id} disconnected")
```

## File Upload

```python
from fastapi import UploadFile, File

@router.post("/upload")
async def upload_file(
    file: Annotated[UploadFile, File(description="File to upload")]
):
    contents = await file.read()
    return {"filename": file.filename, "size": len(contents)}
```

## Response Models

```python
# Exclude fields
class UserResponse(BaseModel):
    id: int
    email: str
    # password excluded automatically - not in schema

# Paginated response
class PaginatedResponse[T](BaseModel):
    data: list[T]
    meta: PaginationMeta

class PaginationMeta(BaseModel):
    total: int
    page: int
    per_page: int
    total_pages: int
```
