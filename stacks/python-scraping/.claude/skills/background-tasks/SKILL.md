---
name: background-tasks
description: Background task patterns with asyncio, queues, and concurrency control. Use when implementing job queues, background processing, task tracking, checkpointing, or worker patterns. Triggers on task queue, background job, worker, async processing keywords.
---

# Background Task Patterns

## Task Model

```python
from enum import Enum
from datetime import datetime
from sqlalchemy import String, JSON, Text
from sqlalchemy.orm import Mapped, mapped_column

class TaskStatus(str, Enum):
    PENDING = "pending"
    PROCESSING = "processing"
    COMPLETED = "completed"
    FAILED = "failed"
    CANCELLED = "cancelled"

class Task(Base):
    __tablename__ = "tasks"
    
    id: Mapped[str] = mapped_column(String(36), primary_key=True)
    type: Mapped[str] = mapped_column(String(50), index=True)
    status: Mapped[TaskStatus] = mapped_column(default=TaskStatus.PENDING, index=True)
    user_id: Mapped[str] = mapped_column(String(36), index=True)
    
    payload: Mapped[dict] = mapped_column(JSON, default=dict)
    result: Mapped[dict | None] = mapped_column(JSON, nullable=True)
    error: Mapped[str | None] = mapped_column(Text, nullable=True)
    
    checkpoint: Mapped[dict | None] = mapped_column(JSON, nullable=True)
    progress: Mapped[int] = mapped_column(default=0)  # 0-100
    
    created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow)
    started_at: Mapped[datetime | None] = mapped_column(nullable=True)
    completed_at: Mapped[datetime | None] = mapped_column(nullable=True)
```

## Task Queue with Concurrency Control

```python
import asyncio
from typing import Callable, Awaitable
import uuid

class TaskQueue:
    MAX_CONCURRENT = 7       # Total concurrent tasks
    MAX_PER_USER = 5         # Per-user limit
    
    def __init__(
        self,
        repo: TaskRepository,
        notifier: TaskNotifier
    ):
        self.repo = repo
        self.notifier = notifier
        self.semaphore = asyncio.Semaphore(self.MAX_CONCURRENT)
        self.handlers: dict[str, Callable] = {}
    
    def register_handler(self, task_type: str, handler: Callable):
        """Register a handler for a task type."""
        self.handlers[task_type] = handler
    
    async def enqueue(
        self,
        task_type: str,
        payload: dict,
        user_id: str
    ) -> Task:
        """Add task to queue."""
        # Check user limit
        user_count = await self.repo.count_active(user_id=user_id)
        if user_count >= self.MAX_PER_USER:
            raise TaskLimitError(
                f"Maximum {self.MAX_PER_USER} concurrent tasks per user"
            )
        
        # Create task
        task = Task(
            id=str(uuid.uuid4()),
            type=task_type,
            payload=payload,
            user_id=user_id,
            status=TaskStatus.PENDING
        )
        await self.repo.create(task)
        
        # Notify
        await self.notifier.notify(user_id, {
            "type": "task_queued",
            "task_id": task.id
        })
        
        # Start processing
        asyncio.create_task(self._process(task))
        
        return task
    
    async def _process(self, task: Task):
        """Process task with concurrency control."""
        async with self.semaphore:
            handler = self.handlers.get(task.type)
            if not handler:
                await self._fail(task, f"Unknown task type: {task.type}")
                return
            
            try:
                # Update status
                task.status = TaskStatus.PROCESSING
                task.started_at = datetime.utcnow()
                await self.repo.update(task)
                
                await self.notifier.notify(task.user_id, {
                    "type": "task_started",
                    "task_id": task.id
                })
                
                # Execute handler
                result = await handler(task, self)
                
                # Complete
                await self._complete(task, result)
                
            except asyncio.CancelledError:
                await self._cancel(task)
            except Exception as e:
                await self._fail(task, str(e))
    
    async def _complete(self, task: Task, result: dict):
        task.status = TaskStatus.COMPLETED
        task.result = result
        task.progress = 100
        task.completed_at = datetime.utcnow()
        await self.repo.update(task)
        
        await self.notifier.notify(task.user_id, {
            "type": "task_completed",
            "task_id": task.id,
            "result": result
        })
    
    async def _fail(self, task: Task, error: str):
        task.status = TaskStatus.FAILED
        task.error = error
        task.completed_at = datetime.utcnow()
        await self.repo.update(task)
        
        await self.notifier.notify(task.user_id, {
            "type": "task_failed",
            "task_id": task.id,
            "error": error
        })
    
    async def update_progress(self, task: Task, progress: int):
        """Update task progress (0-100)."""
        task.progress = min(100, max(0, progress))
        await self.repo.update(task)
        
        await self.notifier.notify(task.user_id, {
            "type": "task_progress",
            "task_id": task.id,
            "progress": task.progress
        })
```

## Checkpointing for Recovery

```python
class CheckpointedTask:
    """Mixin for tasks with checkpoint support."""
    
    async def save_checkpoint(self, task: Task, queue: TaskQueue, state: dict):
        """Save checkpoint for recovery."""
        task.checkpoint = state
        await queue.repo.update(task)
    
    async def load_checkpoint(self, task: Task) -> dict | None:
        """Load checkpoint if exists."""
        return task.checkpoint

# Example handler with checkpoints
async def pdf_extraction_handler(task: Task, queue: TaskQueue) -> dict:
    state = task.checkpoint or {}
    
    # Step 1: Extract text
    if "text" not in state:
        state["text"] = await extract_pdf_text(task.payload["file_id"])
        await queue.save_checkpoint(task, state)
        await queue.update_progress(task, 25)
    
    # Step 2: AI extraction
    if "schools" not in state:
        state["schools"] = await ai_extract_schools(state["text"])
        await queue.save_checkpoint(task, state)
        await queue.update_progress(task, 50)
    
    # Step 3: Normalize
    if "normalized" not in state:
        state["normalized"] = normalize_schools(state["schools"])
        await queue.save_checkpoint(task, state)
        await queue.update_progress(task, 75)
    
    # Step 4: Save to database
    saved = await save_schools(state["normalized"])
    await queue.update_progress(task, 100)
    
    return {"saved_count": len(saved)}
```

## Worker for Startup Recovery

```python
class TaskWorker:
    """Recovers and processes abandoned tasks on startup."""
    
    def __init__(self, queue: TaskQueue, repo: TaskRepository):
        self.queue = queue
        self.repo = repo
    
    async def recover_tasks(self):
        """Recover tasks that were processing when server stopped."""
        abandoned = await self.repo.get_by_status(TaskStatus.PROCESSING)
        
        for task in abandoned:
            # Reset to pending and re-queue
            task.status = TaskStatus.PENDING
            task.started_at = None
            await self.repo.update(task)
            
            # Re-enqueue (uses checkpoint)
            asyncio.create_task(self.queue._process(task))
        
        logging.info(f"Recovered {len(abandoned)} tasks")

# In FastAPI lifespan
@asynccontextmanager
async def lifespan(app: FastAPI):
    worker = TaskWorker(queue, repo)
    await worker.recover_tasks()
    yield
```

## Batch Processing

```python
async def process_batch(
    items: list,
    processor: Callable,
    task: Task,
    queue: TaskQueue,
    batch_size: int = 10
) -> list:
    """Process items in batches with progress updates."""
    results = []
    total = len(items)
    
    for i in range(0, total, batch_size):
        batch = items[i:i + batch_size]
        batch_results = await asyncio.gather(
            *[processor(item) for item in batch],
            return_exceptions=True
        )
        
        for result in batch_results:
            if not isinstance(result, Exception):
                results.append(result)
        
        # Update progress
        progress = int((i + len(batch)) / total * 100)
        await queue.update_progress(task, progress)
    
    return results
```

## Cancellation Support

```python
class CancellableTaskQueue(TaskQueue):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.running_tasks: dict[str, asyncio.Task] = {}
    
    async def _process(self, task: Task):
        async_task = asyncio.current_task()
        self.running_tasks[task.id] = async_task
        
        try:
            await super()._process(task)
        finally:
            self.running_tasks.pop(task.id, None)
    
    async def cancel(self, task_id: str) -> bool:
        """Cancel a running task."""
        if task_id in self.running_tasks:
            self.running_tasks[task_id].cancel()
            return True
        return False
```

## Retry Logic

```python
from functools import wraps

def with_retry(max_retries: int = 3, delay: float = 1.0):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            last_error = None
            for attempt in range(max_retries):
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    last_error = e
                    if attempt < max_retries - 1:
                        await asyncio.sleep(delay * (2 ** attempt))
            raise last_error
        return wrapper
    return decorator

@with_retry(max_retries=3)
async def fetch_with_retry(url: str):
    ...
```
