# Web 开发案例

> 使用 OpenHands 进行 Web 开发的实战案例

---

## 📋 案例列表

- [案例 1：FastAPI 用户认证系统](#案例-1fastapi-用户认证系统)
- [案例 2：Django 博客系统](#案例-2django-博客系统)
- [案例 3：Flask RESTful API](#案例-3flask-restful-api)

---

## 案例 1：FastAPI 用户认证系统

### 任务描述

创建一个完整的用户认证系统，包括注册、登录、JWT 认证。

### CLI 命令

```bash
openhands run "创建一个 FastAPI 用户认证系统：
1. 用户注册（用户名、邮箱、密码）
2. 用户登录（JWT 认证）
3. 密码加密（bcrypt）
4. 用户信息查询
5. 单元测试
6. API 文档"
```

### 预期结果

**main.py**：
```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel, EmailStr
from passlib.context import CryptContext
import jwt
from datetime import datetime, timedelta
from typing import Optional

app = FastAPI(title="User Authentication API")
security = HTTPBearer()
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

class UserCreate(BaseModel):
    username: str
    email: EmailStr
    password: str

class UserLogin(BaseModel):
    username: str
    password: str

class Token(BaseModel):
    access_token: str
    token_type: str

# 模拟数据库
fake_db = {}

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

def create_access_token(data: dict) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

@app.post("/register", response_model=Token)
async def register(user: UserCreate):
    if user.username in fake_db:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Username already registered"
        )
    
    hashed_password = get_password_hash(user.password)
    fake_db[user.username] = {
        "username": user.username,
        "email": user.email,
        "hashed_password": hashed_password
    }
    
    access_token = create_access_token(data={"sub": user.username})
    return {"access_token": access_token, "token_type": "bearer"}

@app.post("/login", response_model=Token)
async def login(user: UserLogin):
    db_user = fake_db.get(user.username)
    if not db_user or not verify_password(user.password, db_user["hashed_password"]):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password"
        )
    
    access_token = create_access_token(data={"sub": user.username})
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/users/me")
async def read_users_me(credentials: HTTPAuthorizationCredentials = Depends(security)):
    token = credentials.credentials
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise HTTPException(status_code=401, detail="Invalid token")
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token")
    
    user = fake_db.get(username)
    if user is None:
        raise HTTPException(status_code=404, detail="User not found")
    
    return {"username": user["username"], "email": user["email"]}
```

### 运行说明

```bash
# 安装依赖
pip install fastapi uvicorn python-jose[cryptography] passlib[bcrypt] python-multipart

# 启动服务
uvicorn main:app --reload

# 访问 API 文档
open http://localhost:8000/docs
```

### 学习要点

- ✅ FastAPI 基础
- ✅ JWT 认证
- ✅ 密码加密
- ✅ Pydantic 数据验证

---

## 案例 2：Django 博客系统

### 任务描述

创建一个简单的博客系统，支持文章的增删改查。

### CLI 命令

```bash
openhands run "创建一个 Django 博客系统：
1. 文章模型（标题、内容、作者、时间）
2. 用户认证
3. 文章 CRUD API
4. 评论功能
5. 单元测试"
```

### 学习要点

- ✅ Django 模型
- ✅ REST API
- ✅ 用户认证
- ✅ 测试驱动

---

## 案例 3：Flask RESTful API

### 任务描述

创建一个 RESTful API，管理任务列表。

### CLI 命令

```bash
openhands run "创建一个 Flask RESTful API：
1. 任务模型（标题、描述、状态）
2. CRUD 接口
3. 分页查询
4. 搜索功能
5. API 文档"
```

### 学习要点

- ✅ Flask 基础
- ✅ RESTful 设计
- ✅ 数据库操作
- ✅ API 文档

---

## 📚 相关资源

- [FastAPI 官方文档](https://fastapi.tiangolo.com/)
- [Django 官方文档](https://docs.djangoproject.com/)
- [Flask 官方文档](https://flask.palletsprojects.com/)

---

<div align="center">
  <p>🎯 继续探索更多 Web 开发案例！</p>
</div>
