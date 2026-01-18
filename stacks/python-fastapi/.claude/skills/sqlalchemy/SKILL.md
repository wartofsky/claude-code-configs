---
name: sqlalchemy-async
description: SQLAlchemy 2.0 async patterns for FastAPI. Use when creating models, writing database queries, handling relationships, or managing sessions. Triggers on SQLAlchemy, database, model, query, ORM, async session keywords.
---

# SQLAlchemy 2.0 Async Patterns

## Setup

### Engine & Session Factory

```python
# db/session.py
from sqlalchemy.ext.asyncio import (
    AsyncSession,
    async_sessionmaker,
    create_async_engine,
)
from sqlalchemy.orm import DeclarativeBase

DATABASE_URL = "postgresql+asyncpg://user:pass@localhost/db"

engine = create_async_engine(
    DATABASE_URL,
    echo=False,  # Set True for SQL logging
    pool_size=5,
    max_overflow=10,
)

async_session_maker = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,
)


class Base(DeclarativeBase):
    pass


async def get_session() -> AsyncGenerator[AsyncSession, None]:
    async with async_session_maker() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
```

### FastAPI Dependency

```python
from typing import Annotated
from fastapi import Depends

SessionDep = Annotated[AsyncSession, Depends(get_session)]
```

## Model Definition

### Basic Model

```python
from datetime import datetime
from sqlalchemy import String, Text, Boolean, ForeignKey, Index
from sqlalchemy.orm import Mapped, mapped_column, relationship

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(255), unique=True, index=True)
    name: Mapped[str] = mapped_column(String(100))
    hashed_password: Mapped[str] = mapped_column(String(255))
    is_active: Mapped[bool] = mapped_column(Boolean, default=True)
    created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow)
    updated_at: Mapped[datetime] = mapped_column(
        default=datetime.utcnow,
        onupdate=datetime.utcnow
    )

    # Relationships
    posts: Mapped[list["Post"]] = relationship(back_populates="author")
    profile: Mapped["Profile"] = relationship(back_populates="user", uselist=False)

    def __repr__(self) -> str:
        return f"<User {self.email}>"


class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(200))
    content: Mapped[str | None] = mapped_column(Text, nullable=True)
    published: Mapped[bool] = mapped_column(Boolean, default=False)
    created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow)

    author_id: Mapped[int] = mapped_column(ForeignKey("users.id", ondelete="CASCADE"))
    author: Mapped["User"] = relationship(back_populates="posts")

    # Many-to-many
    categories: Mapped[list["Category"]] = relationship(
        secondary="post_categories",
        back_populates="posts"
    )

    __table_args__ = (
        Index("idx_posts_author_published", "author_id", "published"),
    )


class Profile(Base):
    __tablename__ = "profiles"

    id: Mapped[int] = mapped_column(primary_key=True)
    bio: Mapped[str | None] = mapped_column(Text, nullable=True)
    avatar_url: Mapped[str | None] = mapped_column(String(500), nullable=True)

    user_id: Mapped[int] = mapped_column(
        ForeignKey("users.id", ondelete="CASCADE"),
        unique=True
    )
    user: Mapped["User"] = relationship(back_populates="profile")


class Category(Base):
    __tablename__ = "categories"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(50), unique=True)

    posts: Mapped[list["Post"]] = relationship(
        secondary="post_categories",
        back_populates="categories"
    )


# Association table for many-to-many
from sqlalchemy import Table, Column, Integer

post_categories = Table(
    "post_categories",
    Base.metadata,
    Column("post_id", Integer, ForeignKey("posts.id", ondelete="CASCADE"), primary_key=True),
    Column("category_id", Integer, ForeignKey("categories.id", ondelete="CASCADE"), primary_key=True),
)
```

## CRUD Operations

### Create

```python
from sqlalchemy import select

async def create_user(session: AsyncSession, email: str, name: str) -> User:
    user = User(email=email, name=name, hashed_password="...")
    session.add(user)
    await session.flush()  # Get ID without committing
    return user


async def create_post_with_categories(
    session: AsyncSession,
    title: str,
    author_id: int,
    category_ids: list[int]
) -> Post:
    # Fetch categories
    result = await session.execute(
        select(Category).where(Category.id.in_(category_ids))
    )
    categories = result.scalars().all()

    post = Post(
        title=title,
        author_id=author_id,
        categories=list(categories)
    )
    session.add(post)
    await session.flush()
    return post


async def bulk_create_users(
    session: AsyncSession,
    users_data: list[dict]
) -> list[User]:
    users = [User(**data) for data in users_data]
    session.add_all(users)
    await session.flush()
    return users
```

### Read

```python
from sqlalchemy import select, func
from sqlalchemy.orm import selectinload, joinedload

async def get_user_by_id(session: AsyncSession, user_id: int) -> User | None:
    result = await session.execute(
        select(User).where(User.id == user_id)
    )
    return result.scalar_one_or_none()


async def get_user_by_email(session: AsyncSession, email: str) -> User | None:
    result = await session.execute(
        select(User).where(User.email == email)
    )
    return result.scalar_one_or_none()


async def get_users(
    session: AsyncSession,
    skip: int = 0,
    limit: int = 20,
    is_active: bool | None = None
) -> list[User]:
    query = select(User)

    if is_active is not None:
        query = query.where(User.is_active == is_active)

    query = query.offset(skip).limit(limit).order_by(User.created_at.desc())

    result = await session.execute(query)
    return list(result.scalars().all())


# With eager loading (avoid N+1)
async def get_user_with_posts(session: AsyncSession, user_id: int) -> User | None:
    result = await session.execute(
        select(User)
        .where(User.id == user_id)
        .options(
            selectinload(User.posts),  # Load posts in separate query
            joinedload(User.profile),  # Load profile with JOIN
        )
    )
    return result.scalar_one_or_none()


async def get_posts_with_author(
    session: AsyncSession,
    published_only: bool = True
) -> list[Post]:
    query = select(Post).options(joinedload(Post.author))

    if published_only:
        query = query.where(Post.published == True)

    result = await session.execute(query.order_by(Post.created_at.desc()))
    return list(result.scalars().unique().all())
```

### Update

```python
from sqlalchemy import update

async def update_user(
    session: AsyncSession,
    user_id: int,
    **kwargs
) -> User | None:
    # Fetch first to return updated object
    user = await get_user_by_id(session, user_id)
    if not user:
        return None

    for key, value in kwargs.items():
        if hasattr(user, key) and value is not None:
            setattr(user, key, value)

    await session.flush()
    return user


# Bulk update without loading objects
async def deactivate_users(
    session: AsyncSession,
    user_ids: list[int]
) -> int:
    result = await session.execute(
        update(User)
        .where(User.id.in_(user_ids))
        .values(is_active=False)
    )
    return result.rowcount
```

### Delete

```python
from sqlalchemy import delete

async def delete_user(session: AsyncSession, user_id: int) -> bool:
    result = await session.execute(
        delete(User).where(User.id == user_id)
    )
    return result.rowcount > 0


async def delete_old_posts(session: AsyncSession, before: datetime) -> int:
    result = await session.execute(
        delete(Post).where(Post.created_at < before)
    )
    return result.rowcount
```

## Advanced Queries

### Filtering

```python
from sqlalchemy import and_, or_, not_

async def search_posts(
    session: AsyncSession,
    query: str | None = None,
    author_id: int | None = None,
    published: bool | None = None,
) -> list[Post]:
    stmt = select(Post)
    conditions = []

    if query:
        conditions.append(
            or_(
                Post.title.ilike(f"%{query}%"),
                Post.content.ilike(f"%{query}%"),
            )
        )

    if author_id:
        conditions.append(Post.author_id == author_id)

    if published is not None:
        conditions.append(Post.published == published)

    if conditions:
        stmt = stmt.where(and_(*conditions))

    result = await session.execute(stmt.order_by(Post.created_at.desc()))
    return list(result.scalars().all())
```

### Aggregations

```python
from sqlalchemy import func, distinct

async def count_users(session: AsyncSession, is_active: bool = True) -> int:
    result = await session.execute(
        select(func.count(User.id)).where(User.is_active == is_active)
    )
    return result.scalar_one()


async def get_post_stats(session: AsyncSession, author_id: int) -> dict:
    result = await session.execute(
        select(
            func.count(Post.id).label("total"),
            func.count(Post.id).filter(Post.published == True).label("published"),
        ).where(Post.author_id == author_id)
    )
    row = result.one()
    return {"total": row.total, "published": row.published}


async def get_posts_per_author(session: AsyncSession) -> list[dict]:
    result = await session.execute(
        select(
            User.id,
            User.name,
            func.count(Post.id).label("post_count")
        )
        .join(Post, Post.author_id == User.id)
        .group_by(User.id)
        .order_by(func.count(Post.id).desc())
    )
    return [
        {"id": row.id, "name": row.name, "post_count": row.post_count}
        for row in result.all()
    ]
```

### Pagination

```python
from typing import TypeVar, Generic
from pydantic import BaseModel

T = TypeVar("T")

class PaginatedResult(BaseModel, Generic[T]):
    items: list[T]
    total: int
    page: int
    per_page: int
    total_pages: int


async def get_users_paginated(
    session: AsyncSession,
    page: int = 1,
    per_page: int = 20
) -> tuple[list[User], int]:
    # Count total
    count_result = await session.execute(select(func.count(User.id)))
    total = count_result.scalar_one()

    # Get page
    offset = (page - 1) * per_page
    result = await session.execute(
        select(User)
        .order_by(User.created_at.desc())
        .offset(offset)
        .limit(per_page)
    )

    return list(result.scalars().all()), total
```

## Repository Pattern

```python
from abc import ABC, abstractmethod
from typing import Generic, TypeVar

ModelType = TypeVar("ModelType", bound=Base)

class BaseRepository(ABC, Generic[ModelType]):
    def __init__(self, session: AsyncSession):
        self.session = session

    @property
    @abstractmethod
    def model(self) -> type[ModelType]:
        ...

    async def get(self, id: int) -> ModelType | None:
        result = await self.session.execute(
            select(self.model).where(self.model.id == id)
        )
        return result.scalar_one_or_none()

    async def get_all(
        self,
        skip: int = 0,
        limit: int = 100
    ) -> list[ModelType]:
        result = await self.session.execute(
            select(self.model).offset(skip).limit(limit)
        )
        return list(result.scalars().all())

    async def create(self, **kwargs) -> ModelType:
        instance = self.model(**kwargs)
        self.session.add(instance)
        await self.session.flush()
        return instance

    async def delete(self, id: int) -> bool:
        result = await self.session.execute(
            delete(self.model).where(self.model.id == id)
        )
        return result.rowcount > 0


class UserRepository(BaseRepository[User]):
    @property
    def model(self) -> type[User]:
        return User

    async def get_by_email(self, email: str) -> User | None:
        result = await self.session.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()

    async def get_with_posts(self, user_id: int) -> User | None:
        result = await self.session.execute(
            select(User)
            .where(User.id == user_id)
            .options(selectinload(User.posts))
        )
        return result.scalar_one_or_none()
```

## Transactions

```python
# Automatic with get_session dependency
async def create_user_with_profile(
    session: AsyncSession,
    user_data: dict,
    profile_data: dict
) -> User:
    user = User(**user_data)
    session.add(user)
    await session.flush()

    profile = Profile(user_id=user.id, **profile_data)
    session.add(profile)
    await session.flush()

    # Both committed at end of request (or rolled back on error)
    return user


# Manual transaction control
async def transfer_posts(
    session: AsyncSession,
    from_user_id: int,
    to_user_id: int
) -> int:
    async with session.begin_nested():  # Savepoint
        result = await session.execute(
            update(Post)
            .where(Post.author_id == from_user_id)
            .values(author_id=to_user_id)
        )
        return result.rowcount
```

## Raw SQL

```python
from sqlalchemy import text

async def execute_raw_query(session: AsyncSession, email_domain: str) -> list:
    result = await session.execute(
        text("SELECT * FROM users WHERE email LIKE :pattern"),
        {"pattern": f"%@{email_domain}"}
    )
    return result.fetchall()
```

## Alembic Migrations

```bash
# Initialize
alembic init alembic

# Create migration
alembic revision --autogenerate -m "add users table"

# Apply migrations
alembic upgrade head

# Rollback
alembic downgrade -1
```

```python
# alembic/env.py
from db.session import Base
from models import *  # Import all models

target_metadata = Base.metadata
```

## Best Practices

1. **Use async session** - Always use `AsyncSession` for async code
2. **Eager loading** - Use `selectinload`/`joinedload` to avoid N+1
3. **Flush vs Commit** - Use `flush()` to get IDs, let dependency commit
4. **Indexes** - Add indexes for frequently queried columns
5. **Type hints** - Use `Mapped[]` for all columns
6. **Relationships** - Always define both sides with `back_populates`
7. **Cascade** - Be explicit about `ondelete` behavior
8. **Connection pooling** - Configure pool size based on load
