# OpenHands API 网关设计

> API 网关架构和实现

---

## 📋 目录

- [网关架构](#网关架构)
- [路由设计](#路由设计)
- [限流熔断](#限流熔断)
- [认证授权](#认证授权)

---

## 🏗️ 网关架构

### 网关架构图

```
┌─────────────────────────────────────────┐
│           客户端请求                     │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│      API Gateway                        │
│  ┌─────────────────────────────────┐   │
│  │   认证授权                       │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │   限流熔断                       │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │   路由转发                       │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │   日志监控                       │   │
│  └─────────────────────────────────┘   │
└────────────────┬────────────────────────┘
                 │
    ┌────────────┼────────────┐
    │            │            │
┌───▼────┐  ┌───▼────┐  ┌───▼────┐
│Service1│  │Service2│  │Service3│
└────────┘  └────────┘  └────────┘
```

### 网关核心功能

```python
# gateway/core.py
from typing import Dict, List, Optional
from fastapi import FastAPI, Request, Response
from fastapi.middleware.cors import CORSMiddleware
import httpx

class APIGateway:
    """API 网关"""
    
    def __init__(self):
        self.app = FastAPI()
        self.routes: Dict[str, str] = {}
        self.middlewares: List = []
        
        # CORS
        self.app.add_middleware(
            CORSMiddleware,
            allow_origins=["*"],
            allow_credentials=True,
            allow_methods=["*"],
            allow_headers=["*"],
        )
    
    def register_route(self, path: str, service_url: str):
        """注册路由"""
        self.routes[path] = service_url
    
    def add_middleware(self, middleware):
        """添加中间件"""
        self.middlewares.append(middleware)
    
    async def forward_request(
        self,
        request: Request,
        service_url: str
    ) -> Response:
        """转发请求"""
        # 构建目标 URL
        target_url = f"{service_url}{request.url.path}"
        
        # 转发请求
        async with httpx.AsyncClient() as client:
            response = await client.request(
                method=request.method,
                url=target_url,
                headers=dict(request.headers),
                content=await request.body(),
                params=dict(request.query_params)
            )
        
        return Response(
            content=response.content,
            status_code=response.status_code,
            headers=dict(response.headers)
        )
    
    def setup_routes(self):
        """设置路由"""
        @self.app.api_route("/{path:path}", methods=["GET", "POST", "PUT", "DELETE"])
        async def proxy(request: Request, path: str):
            # 执行中间件
            for middleware in self.middlewares:
                await middleware(request)
            
            # 查找服务
            for route_path, service_url in self.routes.items():
                if path.startswith(route_path):
                    return await self.forward_request(request, service_url)
            
            return Response(status_code=404)

# 使用
gateway = APIGateway()

# 注册路由
gateway.register_route("users", "http://user-service:8000")
gateway.register_route("orders", "http://order-service:8000")
gateway.register_route("products", "http://product-service:8000")

# 设置路由
gateway.setup_routes()

# 运行
import uvicorn
uvicorn.run(gateway.app, host="0.0.0.0", port=8000)
```

---

## 🔀 路由设计

### 动态路由

```python
# gateway/routing.py
from typing import Dict, List, Optional
import re

class DynamicRouter:
    """动态路由器"""
    
    def __init__(self):
        self.routes: List[Dict] = []
    
    def add_route(
        self,
        pattern: str,
        service_url: str,
        methods: List[str] = None
    ):
        """添加路由"""
        self.routes.append({
            "pattern": re.compile(pattern),
            "service_url": service_url,
            "methods": methods or ["GET", "POST", "PUT", "DELETE"]
        })
    
    def match(self, path: str, method: str) -> Optional[Dict]:
        """匹配路由"""
        for route in self.routes:
            if route["pattern"].match(path) and method in route["methods"]:
                return {
                    "service_url": route["service_url"],
                    "params": route["pattern"].match(path).groupdict()
                }
        
        return None
    
    def remove_route(self, pattern: str):
        """删除路由"""
        self.routes = [
            r for r in self.routes
            if r["pattern"].pattern != pattern
        ]
    
    def list_routes(self) -> List[Dict]:
        """列出路由"""
        return [
            {
                "pattern": r["pattern"].pattern,
                "service_url": r["service_url"],
                "methods": r["methods"]
            }
            for r in self.routes
        ]

# 使用
router = DynamicRouter()

# 添加路由
router.add_route(
    pattern=r"^/api/v1/users/(?P<user_id>\d+)$",
    service_url="http://user-service:8000",
    methods=["GET", "PUT", "DELETE"]
)

router.add_route(
    pattern=r"^/api/v1/orders$",
    service_url="http://order-service:8000",
    methods=["GET", "POST"]
)

# 匹配路由
match = router.match("/api/v1/users/123", "GET")
print(match)
# {
#   "service_url": "http://user-service:8000",
#   "params": {"user_id": "123"}
# }
```

### 负载均衡

```python
# gateway/load_balancer.py
from typing import List, Dict
import random

class LoadBalancer:
    """负载均衡器"""
    
    def __init__(self, strategy: str = "round_robin"):
        self.strategy = strategy
        self.services: Dict[str, List[str]] = {}
        self.current_index: Dict[str, int] = {}
    
    def register_service(self, service_name: str, instances: List[str]):
        """注册服务实例"""
        self.services[service_name] = instances
        self.current_index[service_name] = 0
    
    def get_instance(self, service_name: str) -> str:
        """获取实例"""
        instances = self.services.get(service_name, [])
        
        if not instances:
            raise ValueError(f"No instances for {service_name}")
        
        if self.strategy == "round_robin":
            return self._round_robin(service_name, instances)
        elif self.strategy == "random":
            return self._random(instances)
        elif self.strategy == "weighted":
            return self._weighted(service_name)
    
    def _round_robin(self, service_name: str, instances: List[str]) -> str:
        """轮询"""
        index = self.current_index[service_name]
        instance = instances[index]
        
        self.current_index[service_name] = (index + 1) % len(instances)
        
        return instance
    
    def _random(self, instances: List[str]) -> str:
        """随机"""
        return random.choice(instances)
    
    def _weighted(self, service_name: str) -> str:
        """加权"""
        # 实现加权负载均衡
        pass
    
    def add_instance(self, service_name: str, instance: str):
        """添加实例"""
        if service_name not in self.services:
            self.services[service_name] = []
            self.current_index[service_name] = 0
        
        self.services[service_name].append(instance)
    
    def remove_instance(self, service_name: str, instance: str):
        """删除实例"""
        if service_name in self.services:
            self.services[service_name] = [
                i for i in self.services[service_name]
                if i != instance
            ]

# 使用
lb = LoadBalancer(strategy="round_robin")

# 注册服务
lb.register_service("user-service", [
    "http://user-service-1:8000",
    "http://user-service-2:8000",
    "http://user-service-3:8000"
])

# 获取实例
for _ in range(5):
    instance = lb.get_instance("user-service")
    print(instance)
```

---

## ⚡ 限流熔断

### 限流器

```python
# gateway/rate_limiter.py
from typing import Dict
import time
from collections import defaultdict

class RateLimiter:
    """限流器"""
    
    def __init__(self):
        self.requests: Dict[str, list] = defaultdict(list)
        self.limits: Dict[str, Dict] = {}
    
    def set_limit(self, key: str, max_requests: int, window_seconds: int):
        """设置限制"""
        self.limits[key] = {
            "max_requests": max_requests,
            "window_seconds": window_seconds
        }
    
    def is_allowed(self, key: str) -> bool:
        """检查是否允许"""
        if key not in self.limits:
            return True
        
        limit = self.limits[key]
        now = time.time()
        
        # 清理过期请求
        self.requests[key] = [
            t for t in self.requests[key]
            if now - t < limit["window_seconds"]
        ]
        
        # 检查是否超过限制
        if len(self.requests[key]) >= limit["max_requests"]:
            return False
        
        # 记录请求
        self.requests[key].append(now)
        
        return True
    
    def get_remaining(self, key: str) -> int:
        """获取剩余配额"""
        if key not in self.limits:
            return -1
        
        limit = self.limits[key]
        now = time.time()
        
        # 清理过期请求
        self.requests[key] = [
            t for t in self.requests[key]
            if now - t < limit["window_seconds"]
        ]
        
        return max(0, limit["max_requests"] - len(self.requests[key]))

# FastAPI 集成
from fastapi import FastAPI, Request, HTTPException

app = FastAPI()
rate_limiter = RateLimiter()

# 设置限制
rate_limiter.set_limit("global", max_requests=1000, window_seconds=60)
rate_limiter.set_limit("user:123", max_requests=100, window_seconds=60)

@app.middleware("http")
async def rate_limit_middleware(request: Request, call_next):
    # 检查限流
    if not rate_limiter.is_allowed("global"):
        raise HTTPException(status_code=429, detail="Too many requests")
    
    # 用户级别限流
    user_id = request.headers.get("X-User-ID")
    if user_id:
        if not rate_limiter.is_allowed(f"user:{user_id}"):
            raise HTTPException(status_code=429, detail="Too many requests")
    
    return await call_next(request)
```

### 熔断器

```python
# gateway/circuit_breaker.py
from enum import Enum
from typing import Dict
import time

class CircuitState(str, Enum):
    CLOSED = "closed"      # 正常
    OPEN = "open"          # 熔断
    HALF_OPEN = "half_open"  # 半开

class CircuitBreaker:
    """熔断器"""
    
    def __init__(
        self,
        failure_threshold: int = 5,
        recovery_timeout: int = 60
    ):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        
        self.state: Dict[str, CircuitState] = {}
        self.failures: Dict[str, int] = {}
        self.last_failure_time: Dict[str, float] = {}
    
    def is_available(self, service: str) -> bool:
        """检查服务是否可用"""
        state = self.state.get(service, CircuitState.CLOSED)
        
        if state == CircuitState.CLOSED:
            return True
        
        elif state == CircuitState.OPEN:
            # 检查是否可以尝试恢复
            if time.time() - self.last_failure_time.get(service, 0) > self.recovery_timeout:
                self.state[service] = CircuitState.HALF_OPEN
                return True
            return False
        
        elif state == CircuitState.HALF_OPEN:
            return True
        
        return False
    
    def record_success(self, service: str):
        """记录成功"""
        self.failures[service] = 0
        self.state[service] = CircuitState.CLOSED
    
    def record_failure(self, service: str):
        """记录失败"""
        self.failures[service] = self.failures.get(service, 0) + 1
        self.last_failure_time[service] = time.time()
        
        if self.failures[service] >= self.failure_threshold:
            self.state[service] = CircuitState.OPEN
    
    def get_state(self, service: str) -> CircuitState:
        """获取状态"""
        return self.state.get(service, CircuitState.CLOSED)

# 使用
circuit_breaker = CircuitBreaker(
    failure_threshold=5,
    recovery_timeout=60
)

async def call_service(service: str, request):
    """调用服务"""
    # 检查熔断器
    if not circuit_breaker.is_available(service):
        raise Exception(f"Service {service} is circuit broken")
    
    try:
        # 调用服务
        response = await make_request(service, request)
        
        # 记录成功
        circuit_breaker.record_success(service)
        
        return response
    
    except Exception as e:
        # 记录失败
        circuit_breaker.record_failure(service)
        
        raise
```

---

## 🔐 认证授权

### JWT 认证

```python
# gateway/auth.py
from fastapi import FastAPI, Request, HTTPException
from jose import JWTError, jwt
from typing import Optional

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"

class AuthMiddleware:
    """认证中间件"""
    
    def __init__(self):
        self.public_paths = [
            "/health",
            "/login",
            "/register"
        ]
    
    async def __call__(self, request: Request):
        # 跳过公开路径
        if request.url.path in self.public_paths:
            return
        
        # 获取 token
        auth_header = request.headers.get("Authorization")
        
        if not auth_header or not auth_header.startswith("Bearer "):
            raise HTTPException(
                status_code=401,
                detail="Invalid authentication credentials"
            )
        
        token = auth_header.replace("Bearer ", "")
        
        # 验证 token
        try:
            payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
            user_id = payload.get("sub")
            
            if not user_id:
                raise HTTPException(
                    status_code=401,
                    detail="Invalid token"
                )
            
            # 添加用户信息到请求
            request.state.user_id = user_id
        
        except JWTError:
            raise HTTPException(
                status_code=401,
                detail="Invalid token"
            )

# FastAPI 集成
app = FastAPI()
auth_middleware = AuthMiddleware()

@app.middleware("http")
async def auth(request: Request, call_next):
    await auth_middleware(request)
    return await call_next(request)
```

### RBAC 授权

```python
# gateway/authorization.py
from typing import Dict, List
from fastapi import Request, HTTPException

class AuthorizationMiddleware:
    """授权中间件"""
    
    def __init__(self):
        self.role_permissions: Dict[str, List[str]] = {
            "admin": ["read", "write", "delete", "admin"],
            "user": ["read", "write"],
            "guest": ["read"]
        }
        
        self.path_permissions: Dict[str, str] = {
            "/api/v1/admin": "admin",
            "/api/v1/users": "write",
            "/api/v1/products": "read"
        }
    
    async def __call__(self, request: Request):
        # 获取用户角色
        user_role = request.state.user_role
        
        # 获取路径所需权限
        required_permission = None
        
        for path, permission in self.path_permissions.items():
            if request.url.path.startswith(path):
                required_permission = permission
                break
        
        if not required_permission:
            return
        
        # 检查权限
        user_permissions = self.role_permissions.get(user_role, [])
        
        if required_permission not in user_permissions:
            raise HTTPException(
                status_code=403,
                detail="Permission denied"
            )

# 使用
authz_middleware = AuthorizationMiddleware()

@app.middleware("http")
async def authorize(request: Request, call_next):
    await authz_middleware(request)
    return await call_next(request)
```

---

## 🎯 网关最佳实践

### 1. 性能优化

- **缓存** - 缓存频繁访问的数据
- **压缩** - 启用 Gzip 压缩
- **连接池** - 复用 HTTP 连接
- **异步** - 使用异步 I/O

### 2. 安全最佳实践

- **HTTPS** - 强制使用 HTTPS
- **认证** - 实现统一认证
- **授权** - 实现细粒度授权
- **限流** - 防止滥用

### 3. 可靠性

- **熔断** - 实现熔断机制
- **重试** - 实现自动重试
- **降级** - 实现服务降级
- **监控** - 实时监控网关

---

<div align="center">
  <p>🚪 API 网关，统一入口！</p>
</div>
