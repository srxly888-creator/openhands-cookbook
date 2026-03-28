# OpenHands 安全最佳实践

> 企业级安全防护

---

## 📋 目录

- [安全架构](#安全架构)
- [认证授权](#认证授权)
- [数据安全](#数据安全)
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

### 安全检查清单

```python
# security/checklist.py
from typing import List, Dict

class SecurityChecklist:
    """安全检查清单"""
    
    def __init__(self):
        self.checks = []
    
    def add_check(self, category: str, item: str, status: bool):
        """添加检查项"""
        self.checks.append({
            'category': category,
            'item': item,
            'status': status
        })
    
    def run_checks(self) -> Dict:
        """运行检查"""
        results = {
            'total': len(self.checks),
            'passed': 0,
            'failed': 0,
            'categories': {}
        }
        
        for check in self.checks:
            category = check['category']
            
            if category not in results['categories']:
                results['categories'][category] = {
                    'passed': 0,
                    'failed': 0,
                    'items': []
                }
            
            if check['status']:
                results['passed'] += 1
                results['categories'][category]['passed'] += 1
            else:
                results['failed'] += 1
                results['categories'][category]['failed'] += 1
            
            results['categories'][category]['items'].append(check)
        
        return results

# 使用
checklist = SecurityChecklist()

# 添加检查项
checklist.add_check('认证', '启用多因素认证', True)
checklist.add_check('认证', '密码复杂度要求', True)
checklist.add_check('认证', '会话超时控制', False)

checklist.add_check('数据', '敏感数据加密', True)
checklist.add_check('数据', '数据脱敏', False)
checklist.add_check('数据', '备份加密', True)

# 运行检查
results = checklist.run_checks()
print(results)
```

---

## 🔐 认证授权

### JWT 认证

```python
# security/jwt_auth.py
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import HTTPException, status

SECRET_KEY = "your-secret-key-keep-it-secret"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

class JWTAuth:
    """JWT 认证"""
    
    @staticmethod
    def hash_password(password: str) -> str:
        """哈希密码"""
        return pwd_context.hash(password)
    
    @staticmethod
    def verify_password(plain_password: str, hashed_password: str) -> bool:
        """验证密码"""
        return pwd_context.verify(plain_password, hashed_password)
    
    @staticmethod
    def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
        """创建访问令牌"""
        to_encode = data.copy()
        
        if expires_delta:
            expire = datetime.utcnow() + expires_delta
        else:
            expire = datetime.utcnow() + timedelta(minutes=15)
        
        to_encode.update({"exp": expire})
        encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
        
        return encoded_jwt
    
    @staticmethod
    def decode_access_token(token: str):
        """解码访问令牌"""
        try:
            payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
            return payload
        except JWTError:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Could not validate credentials"
            )

# 使用
jwt_auth = JWTAuth()

# 注册
hashed_password = jwt_auth.hash_password("user_password")

# 登录
if jwt_auth.verify_password("user_password", hashed_password):
    token = jwt_auth.create_access_token(
        data={"sub": "user_id"},
        expires_delta=timedelta(minutes=30)
    )
```

### OAuth2 集成

```python
# security/oauth2.py
from fastapi import FastAPI, Depends
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
    authorize_url='https://github.com/login/oauth/authorize',
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
    jwt_auth = JWTAuth()
    payload = jwt_auth.decode_access_token(token)
    user_id = payload.get("sub")
    
    if user_id is None:
        raise HTTPException(status_code=401, detail="Invalid token")
    
    return user_id
```

### RBAC 权限控制

```python
# security/rbac.py
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
    """基于角色的访问控制"""
    
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

# 使用
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

## 🔒 数据安全

### 数据加密

```python
# security/encryption.py
from cryptography.fernet import Fernet
from typing import Optional

class EncryptionHandler:
    """加密处理器"""
    
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

# 加密
encrypted = encryption_handler.encrypt("sensitive data")

# 解密
decrypted = encryption_handler.decrypt(encrypted)
```

### 数据脱敏

```python
# security/masking.py
import re

class DataMasker:
    """数据脱敏"""
    
    @staticmethod
    def mask_email(email: str) -> str:
        """邮箱脱敏"""
        if not email or "@" not in email:
            return email
        
        username, domain = email.split("@")
        masked_username = username[:2] + "*" * (len(username) - 2)
        
        return f"{masked_username}@{domain}"
    
    @staticmethod
    def mask_phone(phone: str) -> str:
        """手机号脱敏"""
        if not phone or len(phone) < 7:
            return phone
        
        return phone[:3] + "*" * 4 + phone[-4:]
    
    @staticmethod
    def mask_api_key(api_key: str) -> str:
        """API Key 脱敏"""
        if not api_key or len(api_key) < 8:
            return api_key
        
        return api_key[:4] + "*" * (len(api_key) - 8) + api_key[-4:]

# 使用
masker = DataMasker()

email = "alice@example.com"
print(masker.mask_email(email))  # al***@example.com

phone = "13812345678"
print(masker.mask_phone(phone))  # 138****5678

api_key = "sk-ant-1234567890abcdef"
print(masker.mask_api_key(api_key))  # sk-a************cdef
```

---

## 🌐 网络安全

### Rate Limiting

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

### IP 白名单

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

### CORS 配置

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

### 审计日志

```python
# security/audit_log.py
import logging
from datetime import datetime
from typing import Dict
import json

class AuditLogger:
    """审计日志"""
    
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

# 使用
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

### 安全扫描

```python
# security/scanner.py
import subprocess
import json

class SecurityScanner:
    """安全扫描器"""
    
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

# 使用
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

## 🎯 安全最佳实践

### ✅ 认证授权

- [ ] 启用多因素认证
- [ ] 设置密码复杂度
- [ ] 实现会话超时
- [ ] 使用 HTTPS

### ✅ 数据安全

- [ ] 敏感数据加密
- [ ] 实现数据脱敏
- [ ] 定期备份数据
- [ ] 使用安全存储

### ✅ 网络安全

- [ ] 配置防火墙
- [ ] 实现限流
- [ ] 使用 IP 白名单
- [ ] 配置 CORS

### ✅ 监控审计

- [ ] 记录审计日志
- [ ] 实现实时监控
- [ ] 定期安全扫描
- [ ] 建立响应机制

---

<div align="center">
  <p>🔒 安全第一，防护到位！</p>
</div>
