# FastAPI WebSocket Broadcasting for Real-Time Trading Data

Pattern for streaming live price updates to multiple frontend clients via WebSocket with asyncio.

## Architecture

```
Binance WS → Price Aggregator → FastAPI WS Broadcast → N Frontend Clients
```

## Implementation

```python
from fastapi import WebSocket
from asyncio import Queue
import asyncio

class ConnectionManager:
    def __init__(self):
        self.active: list[WebSocket] = []

    async def connect(self, ws: WebSocket):
        await ws.accept()
        self.active.append(ws)

    async def broadcast(self, data: dict):
        dead = []
        for ws in self.active:
            try:
                await ws.send_json(data)
            except Exception:
                dead.append(ws)
        for ws in dead:
            self.active.remove(ws)

manager = ConnectionManager()

# Background task pulls from Binance and broadcasts
async def price_feed():
    async with aiohttp.ClientSession() as session:
        async with session.ws_connect("wss://stream.binance.com:9443/ws/!ticker@arr") as ws:
            async for msg in ws:
                data = msg.json()
                prices = {t["s"]: float(t["c"]) for t in data}
                await manager.broadcast({"type": "prices", "data": prices})
```

## Key Considerations

- Use `orjson` for serialization (3-5x faster than stdlib json)
- Debounce broadcasts to `100ms` intervals to avoid flooding slow clients
- Track `last_seen` per connection and disconnect stale ones (>30s no pong)
- Binance rate limits: max 5 WS connections per IP, 1024 streams per connection
