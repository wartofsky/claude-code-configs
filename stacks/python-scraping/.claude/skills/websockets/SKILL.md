---
name: websockets
description: WebSocket patterns for real-time notifications in FastAPI. Use when implementing real-time updates, push notifications, live task status, or bidirectional communication. Triggers on WebSocket, real-time, notification, push keywords.
---

# WebSocket Patterns

## Connection Manager

```python
from fastapi import WebSocket
from typing import Dict, Set
import asyncio
import json

class WebSocketManager:
    """Manages WebSocket connections per user."""
    
    def __init__(self):
        self.connections: Dict[str, Set[WebSocket]] = {}
        self._lock = asyncio.Lock()
    
    async def connect(self, user_id: str, websocket: WebSocket):
        """Accept and register a connection."""
        await websocket.accept()
        
        async with self._lock:
            if user_id not in self.connections:
                self.connections[user_id] = set()
            self.connections[user_id].add(websocket)
    
    async def disconnect(self, user_id: str, websocket: WebSocket):
        """Remove a connection."""
        async with self._lock:
            if user_id in self.connections:
                self.connections[user_id].discard(websocket)
                if not self.connections[user_id]:
                    del self.connections[user_id]
    
    async def send_to_user(self, user_id: str, message: dict):
        """Send message to all connections of a user."""
        if user_id not in self.connections:
            return
        
        dead_connections = []
        
        for websocket in self.connections[user_id]:
            try:
                await websocket.send_json(message)
            except Exception:
                dead_connections.append(websocket)
        
        # Clean up dead connections
        for ws in dead_connections:
            await self.disconnect(user_id, ws)
    
    async def broadcast(self, message: dict):
        """Send to all connected users."""
        for user_id in list(self.connections.keys()):
            await self.send_to_user(user_id, message)
    
    def is_connected(self, user_id: str) -> bool:
        """Check if user has active connections."""
        return user_id in self.connections and len(self.connections[user_id]) > 0

# Singleton
ws_manager = WebSocketManager()
```

## FastAPI WebSocket Endpoint

```python
from fastapi import WebSocket, WebSocketDisconnect, Depends, Query

@app.websocket("/ws/{user_id}")
async def websocket_endpoint(
    websocket: WebSocket,
    user_id: str,
    token: str = Query(...)  # Auth via query param
):
    # Validate token
    try:
        payload = decode_token(token)
        if payload["sub"] != user_id:
            await websocket.close(code=4003, reason="Unauthorized")
            return
    except Exception:
        await websocket.close(code=4001, reason="Invalid token")
        return
    
    # Connect
    await ws_manager.connect(user_id, websocket)
    
    try:
        while True:
            # Keep connection alive, handle incoming messages
            data = await websocket.receive_text()
            
            # Handle ping/pong for keep-alive
            if data == "ping":
                await websocket.send_text("pong")
            
            # Handle other messages if needed
            # message = json.loads(data)
            # await handle_client_message(user_id, message)
            
    except WebSocketDisconnect:
        await ws_manager.disconnect(user_id, websocket)
    except Exception as e:
        logging.error(f"WebSocket error for {user_id}: {e}")
        await ws_manager.disconnect(user_id, websocket)
```

## Task Notifier

```python
class TaskNotifier:
    """Sends task updates via WebSocket."""
    
    def __init__(self, ws_manager: WebSocketManager):
        self.ws_manager = ws_manager
    
    async def notify(self, user_id: str, event: dict):
        """Send event to user."""
        await self.ws_manager.send_to_user(user_id, {
            "timestamp": datetime.utcnow().isoformat(),
            **event
        })
    
    async def task_queued(self, user_id: str, task_id: str):
        await self.notify(user_id, {
            "type": "task_queued",
            "task_id": task_id
        })
    
    async def task_started(self, user_id: str, task_id: str):
        await self.notify(user_id, {
            "type": "task_started",
            "task_id": task_id
        })
    
    async def task_progress(
        self,
        user_id: str,
        task_id: str,
        progress: int,
        message: str | None = None
    ):
        await self.notify(user_id, {
            "type": "task_progress",
            "task_id": task_id,
            "progress": progress,
            "message": message
        })
    
    async def task_completed(
        self,
        user_id: str,
        task_id: str,
        result: dict
    ):
        await self.notify(user_id, {
            "type": "task_completed",
            "task_id": task_id,
            "result": result
        })
    
    async def task_failed(
        self,
        user_id: str,
        task_id: str,
        error: str
    ):
        await self.notify(user_id, {
            "type": "task_failed",
            "task_id": task_id,
            "error": error
        })

notifier = TaskNotifier(ws_manager)
```

## Client-Side Handler (JavaScript)

```javascript
class TaskWebSocket {
  constructor(userId, token) {
    this.userId = userId;
    this.token = token;
    this.ws = null;
    this.reconnectDelay = 1000;
    this.maxReconnectDelay = 30000;
    this.handlers = new Map();
  }

  connect() {
    const url = `wss://api.example.com/ws/${this.userId}?token=${this.token}`;
    this.ws = new WebSocket(url);

    this.ws.onopen = () => {
      console.log('WebSocket connected');
      this.reconnectDelay = 1000;
      this.startHeartbeat();
    };

    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleMessage(data);
    };

    this.ws.onclose = () => {
      this.stopHeartbeat();
      this.scheduleReconnect();
    };

    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  }

  handleMessage(data) {
    const handler = this.handlers.get(data.type);
    if (handler) {
      handler(data);
    }
  }

  on(eventType, handler) {
    this.handlers.set(eventType, handler);
  }

  startHeartbeat() {
    this.heartbeatInterval = setInterval(() => {
      if (this.ws.readyState === WebSocket.OPEN) {
        this.ws.send('ping');
      }
    }, 30000);
  }

  stopHeartbeat() {
    clearInterval(this.heartbeatInterval);
  }

  scheduleReconnect() {
    setTimeout(() => {
      this.connect();
      this.reconnectDelay = Math.min(
        this.reconnectDelay * 2,
        this.maxReconnectDelay
      );
    }, this.reconnectDelay);
  }
}

// Usage
const ws = new TaskWebSocket('user123', 'jwt-token');

ws.on('task_progress', (data) => {
  updateProgressBar(data.task_id, data.progress);
});

ws.on('task_completed', (data) => {
  showNotification('Task completed!', data.result);
});

ws.on('task_failed', (data) => {
  showError('Task failed', data.error);
});

ws.connect();
```

## Connection Health Monitoring

```python
from prometheus_client import Gauge

ws_connections = Gauge(
    'websocket_connections',
    'Active WebSocket connections',
    ['user_id']
)

class MonitoredWebSocketManager(WebSocketManager):
    async def connect(self, user_id: str, websocket: WebSocket):
        await super().connect(user_id, websocket)
        ws_connections.labels(user_id=user_id).inc()
    
    async def disconnect(self, user_id: str, websocket: WebSocket):
        await super().disconnect(user_id, websocket)
        ws_connections.labels(user_id=user_id).dec()

# Health endpoint
@app.get("/health/websockets")
async def websocket_health():
    return {
        "total_users": len(ws_manager.connections),
        "total_connections": sum(
            len(conns) for conns in ws_manager.connections.values()
        )
    }
```

## Testing WebSockets

```python
import pytest
from httpx import AsyncClient, ASGITransport
from starlette.testclient import TestClient

@pytest.mark.asyncio
async def test_websocket_connection():
    client = TestClient(app)
    
    with client.websocket_connect(f"/ws/user1?token={valid_token}") as ws:
        # Test ping/pong
        ws.send_text("ping")
        response = ws.receive_text()
        assert response == "pong"

@pytest.mark.asyncio
async def test_receives_task_notification():
    client = TestClient(app)
    
    with client.websocket_connect(f"/ws/user1?token={token}") as ws:
        # Trigger a task completion
        await notifier.task_completed("user1", "task-123", {"count": 5})
        
        # Receive notification
        data = ws.receive_json()
        assert data["type"] == "task_completed"
        assert data["task_id"] == "task-123"
```
