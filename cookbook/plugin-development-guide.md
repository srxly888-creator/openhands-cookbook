# OpenHands 插件开发指南

> 开发自定义插件

---

## 📋 目录

- [插件架构](#插件架构)
- [开发入门](#开发入门)
- [插件示例](#插件示例)
- [发布指南](#发布指南)

---

## 🏗️ 插件架构

### 插件系统架构

```
┌─────────────────────────────────────┐
│         OpenHands Core              │
└────────────────┬────────────────────┘
                 │
    ┌────────────┼────────────┐
    │            │            │
┌───▼────┐  ┌───▼────┐  ┌───▼────┐
│Plugin 1│  │Plugin 2│  │Plugin 3│
└────────┘  └────────┘  └────────┘
```

### 插件生命周期

```python
# plugins/base.py
from abc import ABC, abstractmethod
from typing import Dict, Any

class Plugin(ABC):
    """插件基类"""
    
    def __init__(self, config: Dict[str, Any] = None):
        self.config = config or {}
    
    @abstractmethod
    def on_load(self):
        """插件加载时调用"""
        pass
    
    @abstractmethod
    def on_enable(self):
        """插件启用时调用"""
        pass
    
    @abstractmethod
    def on_disable(self):
        """插件禁用时调用"""
        pass
    
    @abstractmethod
    def on_unload(self):
        """插件卸载时调用"""
        pass
    
    def get_info(self) -> Dict[str, Any]:
        """获取插件信息"""
        return {
            "name": self.__class__.__name__,
            "version": "1.0.0",
            "description": "Plugin description"
        }
```

---

## 🚀 开发入门

### 创建第一个插件

```python
# plugins/hello_plugin.py
from plugins.base import Plugin
from typing import Dict, Any

class HelloPlugin(Plugin):
    """Hello 插件"""
    
    def on_load(self):
        """加载插件"""
        print("HelloPlugin loaded")
    
    def on_enable(self):
        """启用插件"""
        print("HelloPlugin enabled")
    
    def on_disable(self):
        """禁用插件"""
        print("HelloPlugin disabled")
    
    def on_unload(self):
        """卸载插件"""
        print("HelloPlugin unloaded")
    
    def say_hello(self, name: str) -> str:
        """说 Hello"""
        return f"Hello, {name}!"
    
    def get_info(self) -> Dict[str, Any]:
        """获取插件信息"""
        return {
            "name": "HelloPlugin",
            "version": "1.0.0",
            "description": "A simple hello plugin",
            "author": "Your Name",
            "methods": ["say_hello"]
        }

# 使用
plugin = HelloPlugin()
plugin.on_load()
plugin.on_enable()

result = plugin.say_hello("World")
print(result)  # Hello, World!
```

### 插件配置

```yaml
# plugins/hello_plugin.yaml
name: HelloPlugin
version: 1.0.0
description: A simple hello plugin
author: Your Name

config:
  default_name: "World"
  greeting_style: "formal"

methods:
  - name: say_hello
    description: Say hello to someone
    parameters:
      - name: name
        type: string
        required: true
        description: The name to greet
    returns:
      type: string
      description: The greeting message
```

---

## 💡 插件示例

### 示例 1：数据库插件

```python
# plugins/database_plugin.py
from plugins.base import Plugin
from typing import Dict, Any, List, Optional
import sqlite3

class DatabasePlugin(Plugin):
    """数据库插件"""
    
    def __init__(self, config: Dict[str, Any] = None):
        super().__init__(config)
        self.connection = None
    
    def on_load(self):
        """加载插件"""
        print("DatabasePlugin loaded")
    
    def on_enable(self):
        """启用插件"""
        # 连接数据库
        db_path = self.config.get("db_path", "data.db")
        self.connection = sqlite3.connect(db_path)
        print(f"Connected to database: {db_path}")
    
    def on_disable(self):
        """禁用插件"""
        if self.connection:
            self.connection.close()
            print("Database connection closed")
    
    def on_unload(self):
        """卸载插件"""
        print("DatabasePlugin unloaded")
    
    def execute_query(self, query: str, params: tuple = None) -> List:
        """执行查询"""
        cursor = self.connection.cursor()
        
        if params:
            cursor.execute(query, params)
        else:
            cursor.execute(query)
        
        results = cursor.fetchall()
        cursor.close()
        
        return results
    
    def execute_update(self, query: str, params: tuple = None) -> int:
        """执行更新"""
        cursor = self.connection.cursor()
        
        if params:
            cursor.execute(query, params)
        else:
            cursor.execute(query)
        
        self.connection.commit()
        row_count = cursor.rowcount
        cursor.close()
        
        return row_count
    
    def create_table(self, table_name: str, columns: Dict[str, str]):
        """创建表"""
        columns_def = ", ".join([
            f"{name} {dtype}" for name, dtype in columns.items()
        ])
        
        query = f"CREATE TABLE IF NOT EXISTS {table_name} ({columns_def})"
        self.execute_update(query)
    
    def insert(self, table_name: str, data: Dict[str, Any]) -> int:
        """插入数据"""
        columns = ", ".join(data.keys())
        placeholders = ", ".join(["?" for _ in data])
        values = tuple(data.values())
        
        query = f"INSERT INTO {table_name} ({columns}) VALUES ({placeholders})"
        return self.execute_update(query, values)
    
    def select(self, table_name: str, where: str = None) -> List:
        """查询数据"""
        query = f"SELECT * FROM {table_name}"
        
        if where:
            query += f" WHERE {where}"
        
        return self.execute_query(query)
    
    def get_info(self) -> Dict[str, Any]:
        """获取插件信息"""
        return {
            "name": "DatabasePlugin",
            "version": "1.0.0",
            "description": "SQLite database plugin",
            "methods": [
                "execute_query",
                "execute_update",
                "create_table",
                "insert",
                "select"
            ]
        }

# 使用
config = {"db_path": "test.db"}
db_plugin = DatabasePlugin(config)
db_plugin.on_load()
db_plugin.on_enable()

# 创建表
db_plugin.create_table("users", {
    "id": "INTEGER PRIMARY KEY",
    "name": "TEXT",
    "email": "TEXT"
})

# 插入数据
db_plugin.insert("users", {
    "name": "Alice",
    "email": "alice@example.com"
})

# 查询数据
users = db_plugin.select("users")
print(users)

db_plugin.on_disable()
```

---

### 示例 2：HTTP 插件

```python
# plugins/http_plugin.py
from plugins.base import Plugin
from typing import Dict, Any, Optional
import aiohttp
import asyncio

class HTTPPlugin(Plugin):
    """HTTP 插件"""
    
    def __init__(self, config: Dict[str, Any] = None):
        super().__init__(config)
        self.session = None
    
    def on_load(self):
        """加载插件"""
        print("HTTPPlugin loaded")
    
    async def on_enable(self):
        """启用插件"""
        timeout = self.config.get("timeout", 30)
        self.session = aiohttp.ClientSession(
            timeout=aiohttp.ClientTimeout(total=timeout)
        )
        print("HTTP session created")
    
    async def on_disable(self):
        """禁用插件"""
        if self.session:
            await self.session.close()
            print("HTTP session closed")
    
    def on_unload(self):
        """卸载插件"""
        print("HTTPPlugin unloaded")
    
    async def get(self, url: str, headers: Dict = None) -> Dict:
        """GET 请求"""
        async with self.session.get(url, headers=headers) as response:
            return await response.json()
    
    async def post(self, url: str, data: Dict = None, headers: Dict = None) -> Dict:
        """POST 请求"""
        async with self.session.post(url, json=data, headers=headers) as response:
            return await response.json()
    
    async def put(self, url: str, data: Dict = None, headers: Dict = None) -> Dict:
        """PUT 请求"""
        async with self.session.put(url, json=data, headers=headers) as response:
            return await response.json()
    
    async def delete(self, url: str, headers: Dict = None) -> Dict:
        """DELETE 请求"""
        async with self.session.delete(url, headers=headers) as response:
            return await response.json()
    
    def get_info(self) -> Dict[str, Any]:
        """获取插件信息"""
        return {
            "name": "HTTPPlugin",
            "version": "1.0.0",
            "description": "HTTP client plugin",
            "methods": ["get", "post", "put", "delete"]
        }

# 使用
async def main():
    config = {"timeout": 30}
    http_plugin = HTTPPlugin(config)
    http_plugin.on_load()
    await http_plugin.on_enable()
    
    # GET 请求
    data = await http_plugin.get("https://api.example.com/data")
    print(data)
    
    # POST 请求
    result = await http_plugin.post(
        "https://api.example.com/create",
        data={"name": "Alice"}
    )
    print(result)
    
    await http_plugin.on_disable()

asyncio.run(main())
```

---

### 示例 3：缓存插件

```python
# plugins/cache_plugin.py
from plugins.base import Plugin
from typing import Dict, Any, Optional
import time

class CachePlugin(Plugin):
    """缓存插件"""
    
    def __init__(self, config: Dict[str, Any] = None):
        super().__init__(config)
        self.cache: Dict[str, Any] = {}
        self.ttls: Dict[str, float] = {}
        self.default_ttl = config.get("default_ttl", 3600)
    
    def on_load(self):
        """加载插件"""
        print("CachePlugin loaded")
    
    def on_enable(self):
        """启用插件"""
        print("CachePlugin enabled")
    
    def on_disable(self):
        """禁用插件"""
        self.cache.clear()
        self.ttls.clear()
        print("Cache cleared")
    
    def on_unload(self):
        """卸载插件"""
        print("CachePlugin unloaded")
    
    def get(self, key: str) -> Optional[Any]:
        """获取缓存"""
        # 检查过期
        if key in self.ttls and time.time() > self.ttls[key]:
            self.delete(key)
            return None
        
        return self.cache.get(key)
    
    def set(self, key: str, value: Any, ttl: int = None):
        """设置缓存"""
        self.cache[key] = value
        
        # 设置过期时间
        ttl = ttl or self.default_ttl
        self.ttls[key] = time.time() + ttl
    
    def delete(self, key: str):
        """删除缓存"""
        if key in self.cache:
            del self.cache[key]
        
        if key in self.ttls:
            del self.ttls[key]
    
    def clear(self):
        """清空缓存"""
        self.cache.clear()
        self.ttls.clear()
    
    def get_stats(self) -> Dict[str, Any]:
        """获取统计"""
        return {
            "total_keys": len(self.cache),
            "total_size": sum(len(str(v)) for v in self.cache.values()),
            "expired_keys": sum(
                1 for k, v in self.ttls.items() if time.time() > v
            )
        }
    
    def get_info(self) -> Dict[str, Any]:
        """获取插件信息"""
        return {
            "name": "CachePlugin",
            "version": "1.0.0",
            "description": "In-memory cache plugin",
            "methods": ["get", "set", "delete", "clear", "get_stats"]
        }

# 使用
config = {"default_ttl": 3600}
cache_plugin = CachePlugin(config)
cache_plugin.on_load()
cache_plugin.on_enable()

# 设置缓存
cache_plugin.set("user:1", {"name": "Alice", "email": "alice@example.com"})

# 获取缓存
user = cache_plugin.get("user:1")
print(user)

# 获取统计
stats = cache_plugin.get_stats()
print(stats)

cache_plugin.on_disable()
```

---

## 📦 发布指南

### 插件目录结构

```
my_plugin/
├── __init__.py
├── plugin.py          # 插件主文件
├── config.yaml        # 插件配置
├── README.md          # 说明文档
├── setup.py           # 安装脚本
└── tests/
    └── test_plugin.py # 单元测试
```

### setup.py

```python
from setuptools import setup, find_packages

setup(
    name="openhands-my-plugin",
    version="1.0.0",
    description="My custom OpenHands plugin",
    author="Your Name",
    packages=find_packages(),
    install_requires=[
        "openhands>=1.0.0"
    ],
    entry_points={
        "openhands.plugins": [
            "my_plugin = my_plugin.plugin:MyPlugin"
        ]
    }
)
```

### 发布到 PyPI

```bash
# 构建
python setup.py sdist bdist_wheel

# 上传
twine upload dist/*
```

---

<div align="center">
  <p>🔌 开发插件，扩展功能！</p>
</div>
