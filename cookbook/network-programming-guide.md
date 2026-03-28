# OpenHands 网络编程完全指南

> 网络通信与协议

---

## 📋 目录

- [TCP/UDP 通信](#tcpudp-通信)
- [HTTP 客户端](#http-客户端)
- [WebSocket](#websocket)
- [网络最佳实践](#网络最佳实践)

---

## 🔌 TCP/UDP 通信

### TCP 客户端

```python
# network/tcp/client.py
import socket
from typing import Optional

class TCPClient:
    """TCP 客户端"""
    
    def __init__(self, host: str, port: int):
        self.host = host
        self.port = port
        self.socket = None
    
    def connect(self):
        """连接"""
        self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.socket.connect((self.host, self.port))
    
    def send(self, data: bytes) -> int:
        """发送数据"""
        return self.socket.send(data)
    
    def receive(self, buffer_size: int = 4096) -> bytes:
        """接收数据"""
        return self.socket.recv(buffer_size)
    
    def close(self):
        """关闭连接"""
        if self.socket:
            self.socket.close()

# 使用
client = TCPClient("example.com", 8080)
client.connect()

# 发送数据
client.send(b"Hello, Server!")

# 接收数据
data = client.receive()
print(f"Received: {data.decode()}")

client.close()
```

### TCP 服务器

```python
# network/tcp/server.py
import socket
import threading
from typing import Callable

class TCPServer:
    """TCP 服务器"""
    
    def __init__(self, host: str = "0.0.0.0", port: int = 8080):
        self.host = host
        self.port = port
        self.server_socket = None
        self.running = False
        self.handler = None
    
    def start(self):
        """启动服务器"""
        self.server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.server_socket.bind((self.host, self.port))
        self.server_socket.listen(5)
        
        self.running = True
        print(f"Server listening on {self.host}:{self.port}")
        
        while self.running:
            client_socket, address = self.server_socket.accept()
            print(f"Connection from {address}")
            
            # 创建线程处理客户端
            thread = threading.Thread(
                target=self._handle_client,
                args=(client_socket, address)
            )
            thread.start()
    
    def set_handler(self, handler: Callable):
        """设置处理器"""
        self.handler = handler
    
    def _handle_client(self, client_socket: socket.socket, address: tuple):
        """处理客户端"""
        try:
            while True:
                data = client_socket.recv(4096)
                if not data:
                    break
                
                if self.handler:
                    response = self.handler(data)
                    client_socket.send(response)
        
        finally:
            client_socket.close()
    
    def stop(self):
        """停止服务器"""
        self.running = False
        if self.server_socket:
            self.server_socket.close()

# 使用
server = TCPServer(port=8080)

# 设置处理器
def handle_request(data: bytes) -> bytes:
    print(f"Received: {data.decode()}")
    return b"ACK"

server.set_handler(handle_request)

# 启动服务器
server.start()
```

### UDP 客户端

```python
# network/udp/client.py
import socket

class UDPClient:
    """UDP 客户端"""
    
    def __init__(self, host: str, port: int):
        self.host = host
        self.port = port
        self.socket = None
    
    def connect(self):
        """创建套接字"""
        self.socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    
    def send(self, data: bytes) -> int:
        """发送数据"""
        return self.socket.sendto(data, (self.host, self.port))
    
    def receive(self, buffer_size: int = 4096) -> tuple:
        """接收数据"""
        return self.socket.recvfrom(buffer_size)
    
    def close(self):
        """关闭连接"""
        if self.socket:
            self.socket.close()

# 使用
client = UDPClient("example.com", 8080)
client.connect()

# 发送数据
client.send(b"Hello, UDP Server!")

# 接收数据
data, address = client.receive()
print(f"Received: {data.decode()} from {address}")

client.close()
```

---

## 🌐 HTTP 客户端

### aiohttp 客户端

```python
# network/http/client.py
import aiohttp
from typing import Dict, Optional

class AsyncHTTPClient:
    """异步 HTTP 客户端"""
    
    def __init__(self, base_url: str = ""):
        self.base_url = base_url
        self.session = None
    
    async def __aenter__(self):
        self.session = aiohttp.ClientSession()
        return self
    
    async def __aexit__(self, *args):
        if self.session:
            await self.session.close()
    
    async def get(
        self,
        endpoint: str,
        params: Optional[Dict] = None,
        headers: Optional[Dict] = None
    ) -> Dict:
        """GET 请求"""
        url = f"{self.base_url}{endpoint}"
        
        async with self.session.get(
            url,
            params=params,
            headers=headers
        ) as response:
            return await response.json()
    
    async def post(
        self,
        endpoint: str,
        data: Optional[Dict] = None,
        json: Optional[Dict] = None,
        headers: Optional[Dict] = None
    ) -> Dict:
        """POST 请求"""
        url = f"{self.base_url}{endpoint}"
        
        async with self.session.post(
            url,
            data=data,
            json=json,
            headers=headers
        ) as response:
            return await response.json()
    
    async def put(
        self,
        endpoint: str,
        data: Optional[Dict] = None,
        headers: Optional[Dict] = None
    ) -> Dict:
        """PUT 请求"""
        url = f"{self.base_url}{endpoint}"
        
        async with self.session.put(
            url,
            data=data,
            headers=headers
        ) as response:
            return await response.json()
    
    async def delete(
        self,
        endpoint: str,
        headers: Optional[Dict] = None
    ) -> Dict:
        """DELETE 请求"""
        url = f"{self.base_url}{endpoint}"
        
        async with self.session.delete(
            url,
            headers=headers
        ) as response:
            return await response.json()

# 使用
async def main():
    async with AsyncHTTPClient("https://api.example.com") as client:
        # GET 请求
        data = await client.get("/users", params={"page": 1})
        
        # POST 请求
        result = await client.post("/users", json={"name": "John"})
        
        # PUT 请求
        updated = await client.put("/users/123", json={"name": "Jane"})
        
        # DELETE 请求
        deleted = await client.delete("/users/123")

import asyncio
asyncio.run(main())
```

### 重试机制

```python
# network/http/retry.py
import aiohttp
import asyncio
from typing import Optional, Callable

class RetryHTTPClient:
    """带重试的 HTTP 客户端"""
    
    def __init__(
        self,
        max_retries: int = 3,
        backoff_factor: float = 1.0
    ):
        self.max_retries = max_retries
        self.backoff_factor = backoff_factor
        self.session = None
    
    async def request_with_retry(
        self,
        method: str,
        url: str,
        **kwargs
    ) -> Optional[Dict]:
        """带重试的请求"""
        for attempt in range(self.max_retries):
            try:
                async with self.session.request(method, url, **kwargs) as response:
                    if response.status == 200:
                        return await response.json()
                    elif response.status >= 500:
                        # 服务器错误，重试
                        await self._wait_backoff(attempt)
                        continue
                    else:
                        # 客户端错误，不重试
                        return None
            
            except (aiohttp.ClientError, asyncio.TimeoutError) as e:
                if attempt < self.max_retries - 1:
                    await self._wait_backoff(attempt)
                    continue
                else:
                    raise
        
        return None
    
    async def _wait_backoff(self, attempt: int):
        """等待退避时间"""
        wait_time = self.backoff_factor * (2 ** attempt)
        await asyncio.sleep(wait_time)

# 使用
async def main():
    client = RetryHTTPClient(max_retries=3, backoff_factor=1.0)
    client.session = aiohttp.ClientSession()
    
    try:
        result = await client.request_with_retry(
            "GET",
            "https://api.example.com/data"
        )
        print(result)
    
    finally:
        await client.session.close()

asyncio.run(main())
```

---

## 🔗 WebSocket

### WebSocket 客户端

```python
# network/websocket/client.py
import websockets
import asyncio
from typing import Callable

class WebSocketClient:
    """WebSocket 客户端"""
    
    def __init__(self, uri: str):
        self.uri = uri
        self.websocket = None
        self.on_message = None
    
    async def connect(self):
        """连接"""
        self.websocket = await websockets.connect(self.uri)
    
    async def send(self, message: str):
        """发送消息"""
        if self.websocket:
            await self.websocket.send(message)
    
    async def receive(self) -> str:
        """接收消息"""
        if self.websocket:
            return await self.websocket.recv()
    
    async def listen(self, handler: Callable):
        """监听消息"""
        self.on_message = handler
        
        async for message in self.websocket:
            await self.on_message(message)
    
    async def close(self):
        """关闭连接"""
        if self.websocket:
            await self.websocket.close()

# 使用
async def main():
    client = WebSocketClient("wss://echo.websocket.org")
    await client.connect()
    
    # 发送消息
    await client.send("Hello, WebSocket!")
    
    # 接收消息
    response = await client.receive()
    print(f"Received: {response}")
    
    await client.close()

asyncio.run(main())
```

### WebSocket 服务器

```python
# network/websocket/server.py
import websockets
import asyncio
from typing import Set

class WebSocketServer:
    """WebSocket 服务器"""
    
    def __init__(self, host: str = "0.0.0.0", port: int = 8765):
        self.host = host
        self.port = port
        self.clients: Set[websockets.WebSocketServerProtocol] = set()
    
    async def handler(self, websocket, path):
        """处理连接"""
        # 添加客户端
        self.clients.add(websocket)
        
        try:
            async for message in websocket:
                # 处理消息
                await self.on_message(websocket, message)
        
        finally:
            # 移除客户端
            self.clients.remove(websocket)
    
    async def on_message(self, websocket, message: str):
        """处理消息"""
        # 广播给所有客户端
        await self.broadcast(f"Echo: {message}")
    
    async def broadcast(self, message: str):
        """广播消息"""
        if self.clients:
            await asyncio.gather(*[
                client.send(message)
                for client in self.clients
            ])
    
    async def start(self):
        """启动服务器"""
        async with websockets.serve(self.handler, self.host, self.port):
            print(f"WebSocket server started on {self.host}:{self.port}")
            await asyncio.Future()  # 永久运行

# 使用
server = WebSocketServer(port=8765)
asyncio.run(server.start())
```

---

## 🎯 网络最佳实践

### 1. 连接管理

- **连接池** - 复用连接，避免频繁创建
- **超时设置** - 设置合理的超时时间
- **重试机制** - 实现自动重试
- **优雅关闭** - 正确关闭连接

### 2. 错误处理

- **异常捕获** - 捕获网络异常
- **重连策略** - 实现自动重连
- **日志记录** - 记录错误日志
- **降级处理** - 失败时的降级方案

### 3. 性能优化

- **异步 I/O** - 使用异步网络库
- **批量请求** - 合并多个请求
- **压缩传输** - 启用数据压缩
- **缓存** - 缓存频繁请求的数据

### 4. 安全考虑

- **HTTPS** - 使用加密传输
- **认证授权** - 实现身份验证
- **输入验证** - 验证所有输入
- **限流** - 防止滥用

---

<div align="center">
  <p>🌐 网络编程，连接世界！</p>
</div>
