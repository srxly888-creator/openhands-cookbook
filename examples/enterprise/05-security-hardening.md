# OpenHands 安全加固指南

> 从零到一的安全防护体系

---

## 📋 目录

- [安全架构](#安全架构)
- [认证授权](#认证授权)
- [数据加密](#数据加密)
- [网络安全](#网络安全)

---

## 🔒 安全架构

### 多层防护体系

```
┌─────────────────────────────────────┐
│         用户层（Users）              │
└────────────────┬────────────────────┘
                 │
┌────────────────▼────────────────────┐
│      WAF（Web Application Firewall）│
└────────────────┬────────────────────┘
                 │
┌────────────────▼────────────────────┐
│         API Gateway（认证/限流）     │
└────────────────┬────────────────────┘
                 │
┌────────────────▼────────────────────┐
│      Service Mesh（mTLS）            │
└────────────────┬────────────────────┘
                 │
┌────────────────▼────────────────────┐
│      Application（RBAC）             │
└────────────────┬────────────────────┘
                 │
┌────────────────▼────────────────────┐
│      Data Layer（加密）              │
└─────────────────────────────────────┘
```

---

## 🔐 认证授权

### 1. JWT 认证

```python
# auth/jwt_handler.py
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import HTTPException, status

SECRET_KEY = "your-secret-key-keep-it-secret"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

class AuthHandler:
    def create_password_hash(self, password: str) -> str:
        """创建密码哈希"""
        return pwd_context.hash(password)
    
    def verify_password(self, plain_password: str, hashed_password: str) -> bool:
        """验证密码"""
        return pwd_context.verify(plain_password, hashed_password)
    
    def create_access_token(self, data: dict, expires_delta: Optional[timedelta] = None):
        """创建访问令牌"""
        to_encode = data.copy()
        
        if expires_delta:
            expire = datetime.utcnow() + expires_delta
        else:
            expire = datetime.utcnow() + timedelta(minutes=15)
        
        to_encode.update({"exp": expire})
        encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
        
        return encoded_jwt
    
    def decode_access_token(self, token: str):
        """解码访问令牌"""
        try:
            payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
            return payload
        except JWTError:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Could not validate credentials"
            )

# 使用示例
auth_handler = AuthHandler()

# 注册
hashed_password = auth_handler.create_password_hash("user_password")

# 登录
if auth_handler.verify_password("user_password", hashed_password):
    token = auth_handler.create_access_token(
        data={"sub": "user_id"},
        expires_delta=timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    )
```

### 2. OAuth2 集成

```python
# auth/oauth2.py
from fastapi import FastAPI, Depends, HTTPException
from fastapi.security import OAuth2PasswordBearer
from authlib.integrations.starlette_client import OAuth

app = FastAPI()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

# OAuth 配置
oauth = OAuth()
oauth.register(
    name='github',
    client_id='your-github-client-id',
    client_secret='your-github-client-secret',
    access_token_url='https://github.com/login/oauth/access_token',
    access_token_params=None,
    authorize_url='https://github.com/login/oauth/authorize',
    authorize_params=None,
    api_base_url='https://api.github.com/',
    client_kwargs={'scope': 'user:email'},
)

@app.get("/auth/github")
async def auth_github():
    """GitHub OAuth 认证"""
    redirect_uri = "http://localhost:8000/auth/github/callback"
    return await oauth.github.authorize_redirect(request, redirect_uri)

@app.get("/auth/github/callback")
async def auth_github_callback():
    """GitHub OAuth 回调"""
    token = await oauth.github.authorize_access_token(request)
    user = await oauth.github.parse_id_token(request, token)
    return {"user": user, "token": token}

# 依赖注入
async def get_current_user(token: str = Depends(oauth2_scheme)):
    """获取当前用户"""
    auth_handler = AuthHandler()
    payload = auth_handler.decode_access_token(token)
    user_id = payload.get("sub")
    
    if user_id is None:
        raise HTTPException(status_code=401, detail="Invalid token")
    
    return user_id
```

### 3. RBAC 权限控制

```python
# auth/rbac.py
from enum import Enum
from typing import List
from fastapi import Depends, HTTPException

class Role(str, Enum):
    ADMIN = "admin"
    MANAGER = "manager"
    USER = "user"
    GUEST = "guest"

class Permission(str, Enum):
    READ = "read"
    WRITE = "write"
    DELETE = "delete"
    ADMIN = "admin"

# 角色权限映射
ROLE_PERMISSIONS = {
    Role.ADMIN: [Permission.READ, Permission.WRITE, Permission.DELETE, Permission.ADMIN],
    Role.MANAGER: [Permission.READ, Permission.WRITE, Permission.DELETE],
    Role.USER: [Permission.READ, Permission.WRITE],
    Role.GUEST: [Permission.READ],
}

class RBAC:
    def __init__(self, user_roles: List[Role]):
        self.user_roles = user_roles
    
    def has_permission(self, required_permission: Permission) -> bool:
        """检查权限"""
        for role in self.user_roles:
            if required_permission in ROLE_PERMISSIONS[role]:
                return True
        return False

# 依赖注入
def require_permission(permission: Permission):
    """权限检查装饰器"""
    async def permission_checker(current_user: str = Depends(get_current_user)):
        # 获取用户角色
        user_roles = await get_user_roles(current_user)
        
        rbac = RBAC(user_roles)
        
        if not rbac.has_permission(permission):
            raise HTTPException(
                status_code=403,
                detail=f"Permission denied: {permission}"
            )
        
        return current_user
    
    return permission_checker

# 使用示例
@app.delete("/api/v1/tasks/{task_id}")
async def delete_task(
    task_id: str,
    current_user: str = Depends(require_permission(Permission.DELETE))
):
    """删除任务（需要 DELETE 权限）"""
    # 执行删除
    pass
```

---

## 🛡️ 数据加密

### 1. 数据库加密

```python
# encryption/database.py
from cryptography.fernet import Fernet
from sqlalchemy import Column, String
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class EncryptionHandler:
    def __init__(self, key: bytes):
        self.cipher = Fernet(key)
    
    def encrypt(self, data: str) -> str:
        """加密数据"""
        return self.cipher.encrypt(data.encode()).decode()
    
    def decrypt(self, encrypted_data: str) -> str:
        """解密数据"""
        return self.cipher.decrypt(encrypted_data.encode()).decode()

# 生成密钥
key = Fernet.generate_key()
encryption_handler = EncryptionHandler(key)

class EncryptedColumn(Column):
    """加密列"""
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
    
    def process_bind_param(self, value, dialect):
        """加密"""
        if value is not None:
            return encryption_handler.encrypt(value)
        return value
    
    def process_result_value(self, value, dialect):
        """解密"""
        if value is not None:
            return encryption_handler.decrypt(value)
        return value

# 使用示例
class User(Base):
    __tablename__ = "users"
    
    id = Column(String, primary_key=True)
    email = Column(String, unique=True)
    # 敏感字段加密
    api_key = EncryptedColumn(String)
    password = EncryptedColumn(String)
```

### 2. 传输加密（TLS）

```python
# security/tls.py
from fastapi import FastAPI
import ssl

app = FastAPI()

# SSL 上下文
ssl_context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
ssl_context.load_cert_chain(
    certfile="/path/to/cert.pem",
    keyfile="/path/to/key.pem"
)

# 启用 HTTPS
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        app,
        host="0.0.0.0",
        port=443,
        ssl=ssl_context
    )
```

### 3. 敏感数据脱敏

```python
# security/masking.py
import re

def mask_email(email: str) -> str:
    """邮箱脱敏"""
    if not email or "@" not in email:
        return email
    
    username, domain = email.split("@")
    masked_username = username[:2] + "*" * (len(username) - 2)
    
    return f"{masked_username}@{domain}"

def mask_phone(phone: str) -> str:
    """手机号脱敏"""
    if not phone or len(phone) < 7:
        return phone
    
    return phone[:3] + "*" * 4 + phone[-4:]

def mask_api_key(api_key: str) -> str:
    """API Key 脱敏"""
    if not api_key or len(api_key) < 8:
        return api_key
    
    return api_key[:4] + "*" * (len(api_key) - 8) + api_key[-4:]

# 使用示例
email = "alice@example.com"
print(mask_email(email))  # al***@example.com

phone = "13812345678"
print(mask_phone(phone))  # 138****5678

api_key = "sk-ant-1234567890abcdef"
print(mask_api_key(api_key))  # sk-a************cdef
```

---

## 🌐 网络安全

### 1. Rate Limiting

```python
# security/rate_limit.py
from fastapi import FastAPI, Request, HTTPException
from fastapi_limiter import FastAPILimiter
from fastapi_limiter.depends import RateLimiter
import redis.asyncio as redis

app = FastAPI()

# 初始化 Redis
@app.on_event("startup")
async def startup():
    redis_client = redis.from_url("redis://localhost:6379")
    await FastAPILimiter.init(redis_client)

# 限流装饰器
@app.get("/api/v1/tasks", dependencies=[Depends(RateLimiter(times=100, seconds=60))])
async def list_tasks():
    """列出任务（100 次/分钟）"""
    return {"tasks": []}

@app.post("/api/v1/tasks", dependencies=[Depends(RateLimiter(times=10, seconds=60))])
async def create_task():
    """创建任务（10 次/分钟）"""
    return {"id": "task-123"}
```

### 2. IP 白名单

```python
# security/ip_whitelist.py
from fastapi import FastAPI, Request, HTTPException
from typing import List

app = FastAPI()

# IP 白名单
ALLOWED_IPS = [
    "192.168.1.100",
    "10.0.0.50",
    "172.16.0.0/12",  # CIDR
]

@app.middleware("http")
async def ip_whitelist_middleware(request: Request, call_next):
    """IP 白名单中间件"""
    client_ip = request.client.host
    
    if not is_ip_allowed(client_ip):
        raise HTTPException(status_code=403, detail="IP not allowed")
    
    return await call_next(request)

def is_ip_allowed(ip: str) -> bool:
    """检查 IP 是否在白名单"""
    import ipaddress
    
    for allowed in ALLOWED_IPS:
        if "/" in allowed:
            # CIDR 格式
            if ipaddress.ip_address(ip) in ipaddress.ip_network(allowed):
                return True
        else:
            # 单个 IP
            if ip == allowed:
                return True
    
    return False
```

### 3. CORS 配置

```python
# security/cors.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# CORS 配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://example.com",
        "https://app.example.com",
    ],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["*"],
    max_age=3600,
)
```

---

## 🔍 安全审计

### 1. 日志记录

```python
# security/audit_log.py
import logging
from datetime import datetime
from typing import Dict
import json

class AuditLogger:
    def __init__(self):
        self.logger = logging.getLogger("audit")
        self.logger.setLevel(logging.INFO)
        
        # 文件处理器
        handler = logging.FileHandler("audit.log")
        handler.setFormatter(logging.Formatter('%(message)s'))
        self.logger.addHandler(handler)
    
    def log_event(self, event_type: str, user_id: str, details: Dict):
        """记录审计事件"""
        event = {
            "timestamp": datetime.utcnow().isoformat(),
            "event_type": event_type,
            "user_id": user_id,
            "details": details
        }
        
        self.logger.info(json.dumps(event))

# 使用示例
audit_logger = AuditLogger()

@app.post("/api/v1/tasks")
async def create_task(request: Request, current_user: str = Depends(get_current_user)):
    """创建任务（记录审计日志）"""
    # 记录操作
    audit_logger.log_event(
        event_type="task_created",
        user_id=current_user,
        details={
            "ip": request.client.host,
            "user_agent": request.headers.get("user-agent"),
            "task_data": await request.json()
        }
    )
    
    # 执行创建
    task = await create_task_in_db()
    
    return task
```

### 2. 安全扫描

```python
# security/scanner.py
import subprocess
import json

class SecurityScanner:
    def scan_dependencies(self):
        """扫描依赖漏洞"""
        result = subprocess.run(
            ["safety", "check", "--json"],
            capture_output=True,
            text=True
        )
        
        vulnerabilities = json.loads(result.stdout)
        return vulnerabilities
    
    def scan_code(self):
        """扫描代码漏洞"""
        result = subprocess.run(
            ["bandit", "-r", "src", "-f", "json"],
            capture_output=True,
            text=True
        )
        
        issues = json.loads(result.stdout)
        return issues
    
    def scan_container(self, image: str):
        """扫描容器漏洞"""
        result = subprocess.run(
            ["trivy", "image", "--format", "json", image],
            capture_output=True,
            text=True
        )
        
        vulnerabilities = json.loads(result.stdout)
        return vulnerabilities

# 使用示例
scanner = SecurityScanner()

# 扫描依赖
vulns = scanner.scan_dependencies()
if vulns:
    print(f"发现 {len(vulns)} 个依赖漏洞")

# 扫描代码
issues = scanner.scan_code()
if issues:
    print(f"发现 {len(issues)} 个代码问题")

# 扫描容器
container_vulns = scanner.scan_container("openhands:latest")
if container_vulns:
    print(f"发现 {len(container_vulns)} 个容器漏洞")
```

---

## 🎯 安全检查清单

### ✅ 认证授权
- [ ] JWT Token 有效期设置
- [ ] 密码复杂度要求
- [ ] 多因素认证（MFA）
- [ ] 会话超时控制
- [ ] 权限最小化原则

### ✅ 数据安全
- [ ] 敏感数据加密
- [ ] 传输加密（TLS）
- [ ] 数据脱敏
- [ ] 备份加密
- [ ] 密钥轮换

### ✅ 网络安全
- [ ] Rate Limiting
- [ ] IP 白名单
- [ ] WAF 配置
- [ ] DDoS 防护
- [ ] VPN 访问

### ✅ 应用安全
- [ ] SQL 注入防护
- [ ] XSS 防护
- [ ] CSRF 防护
- [ ] 输入验证
- [ ] 输出编码

### ✅ 监控审计
- [ ] 审计日志
- [ ] 异常告警
- [ ] 安全扫描
- [ ] 漏洞修复
- [ ] 合规检查

---

<div align="center">
  <p>🔒 安全第一，防护到位！</p>
</div>
