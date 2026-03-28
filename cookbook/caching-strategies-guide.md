# OpenHands 缓存策略完全指南

> 高性能缓存架构设计

---

## 📋 目录

- [缓存架构](#缓存架构)
- [Redis 集成](#redis-集成)
- [缓存策略](#缓存策略)
- [最佳实践](#最佳实践)

---

## 🏗️ 缓存架构

### 多层缓存架构

```
┌─────────────────────────────────────────┐
│           客户端缓存                      │
│       (Browser/CDN)                     │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│         应用层缓存                        │
│       (In-Memory Cache)                 │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│         分布式缓存                        │
│          (Redis)                        │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│           数据库                          │
│       (PostgreSQL)                      │
└─────────────────────────────────────────┘
```

### 缓存核心组件

```python
# cache/core.py
from typing import Dict, Optional, Any
from datetime import timedelta
import asyncio

class CacheManager:
    """缓存管理器"""
    
    def __init__(self):
        self.local_cache: Dict[str, Any] = {}
        self.ttl: Dict[str, float] = {}
        self.distributed_cache = None
    
    async def get(self, key: str) -> Optional[Any]:
        """获取缓存"""
        # 1. 检查本地缓存
        if key in self.local_cache:
            if self._is_valid(key):
                return self.local_cache[key]
            else:
                del self.local_cache[key]
        
        # 2. 检查分布式缓存
        if self.distributed_cache:
            value = await self.distributed_cache.get(key)
            if value:
                # 回填本地缓存
                self.local_cache[key] = value
                self.ttl[key] = asyncio.get_event_loop().time() + 60
                return value
        
        return None
    
    async def set(
        self,
        key: str,
        value: Any,
        ttl: int = 300
    ):
        """设置缓存"""
        # 设置本地缓存
        self.local_cache[key] = value
        self.ttl[key] = asyncio.get_event_loop().time() + min(ttl, 60)
        
        # 设置分布式缓存
        if self.distributed_cache:
            await self.distributed_cache.set(key, value, ttl)
    
    async def delete(self, key: str):
        """删除缓存"""
        if key in self.local_cache:
            del self.local_cache[key]
        
        if key in self.ttl:
            del self.ttl[key]
        
        if self.distributed_cache:
            await self.distributed_cache.delete(key)
    
    def _is_valid(self, key: str) -> bool:
        """检查缓存是否有效"""
        if key not in self.ttl:
            return False
        
        return asyncio.get_event_loop().time() < self.ttl[key]
    
    async def clear(self):
        """清空缓存"""
        self.local_cache.clear()
        self.ttl.clear()
        
        if self.distributed_cache:
            await self.distributed_cache.clear()

# 使用
cache = CacheManager()

# 获取缓存
value = await cache.get("user:123")

# 设置缓存
await cache.set("user:123", {"name": "John"}, ttl=300)

# 删除缓存
await cache.delete("user:123")
```

---

## 🔴 Redis 集成

### Redis 客户端

```python
# cache/redis_client.py
import aioredis
from typing import Optional, Any
import json

class RedisCache:
    """Redis 缓存"""
    
    def __init__(self, url: str = "redis://localhost"):
        self.url = url
        self.redis = None
    
    async def connect(self):
        """连接"""
        self.redis = await aioredis.from_url(
            self.url,
            encoding="utf-8",
            decode_responses=True
        )
    
    async def get(self, key: str) -> Optional[Any]:
        """获取"""
        value = await self.redis.get(key)
        
        if value:
            try:
                return json.loads(value)
            except:
                return value
        
        return None
    
    async def set(
        self,
        key: str,
        value: Any,
        ttl: int = 300
    ):
        """设置"""
        if isinstance(value, (dict, list)):
            value = json.dumps(value)
        
        await self.redis.setex(key, ttl, value)
    
    async def delete(self, key: str):
        """删除"""
        await self.redis.delete(key)
    
    async def exists(self, key: str) -> bool:
        """存在"""
        return await self.redis.exists(key) > 0
    
    async def expire(self, key: str, ttl: int):
        """设置过期时间"""
        await self.redis.expire(key, ttl)
    
    async def ttl(self, key: str) -> int:
        """获取剩余时间"""
        return await self.redis.ttl(key)
    
    async def incr(self, key: str) -> int:
        """递增"""
        return await self.redis.incr(key)
    
    async def decr(self, key: str) -> int:
        """递减"""
        return await self.redis.decr(key)
    
    async def hset(self, name: str, key: str, value: Any):
        """哈希设置"""
        if isinstance(value, (dict, list)):
            value = json.dumps(value)
        
        await self.redis.hset(name, key, value)
    
    async def hget(self, name: str, key: str) -> Optional[Any]:
        """哈希获取"""
        value = await self.redis.hget(name, key)
        
        if value:
            try:
                return json.loads(value)
            except:
                return value
        
        return None
    
    async def hgetall(self, name: str) -> dict:
        """哈希获取所有"""
        return await self.redis.hgetall(name)
    
    async def lpush(self, key: str, value: Any):
        """列表左推"""
        if isinstance(value, (dict, list)):
            value = json.dumps(value)
        
        await self.redis.lpush(key, value)
    
    async def rpop(self, key: str) -> Optional[Any]:
        """列表右弹"""
        value = await self.redis.rpop(key)
        
        if value:
            try:
                return json.loads(value)
            except:
                return value
        
        return None
    
    async def close(self):
        """关闭"""
        if self.redis:
            await self.redis.close()

# 使用
redis_cache = RedisCache()
await redis_cache.connect()

# 基本操作
await redis_cache.set("user:123", {"name": "John"}, ttl=300)
user = await redis_cache.get("user:123")

# 哈希操作
await redis_cache.hset("users", "123", {"name": "John"})
user = await redis_cache.hget("users", "123")

# 列表操作
await redis_cache.lpush("queue", {"task": "process"})
task = await redis_cache.rpop("queue")

await redis_cache.close()
```

---

## 📊 缓存策略

### Cache-Aside 模式

```python
# cache/patterns/cache_aside.py
from typing import Optional, Callable

class CacheAside:
    """Cache-Aside 模式"""
    
    def __init__(self, cache, db):
        self.cache = cache
        self.db = db
    
    async def get(
        self,
        key: str,
        db_loader: Callable,
        ttl: int = 300
    ):
        """获取数据"""
        # 1. 尝试从缓存获取
        value = await self.cache.get(key)
        
        if value:
            return value
        
        # 2. 从数据库加载
        value = await db_loader()
        
        # 3. 写入缓存
        if value:
            await self.cache.set(key, value, ttl)
        
        return value
    
    async def set(self, key: str, value: Any, ttl: int = 300):
        """设置数据"""
        # 1. 更新数据库
        await self.db.save(value)
        
        # 2. 更新缓存
        await self.cache.set(key, value, ttl)
    
    async def delete(self, key: str):
        """删除数据"""
        # 1. 删除缓存
        await self.cache.delete(key)
        
        # 2. 删除数据库
        await self.db.delete(key)

# 使用
cache_aside = CacheAside(redis_cache, db)

# 获取数据
user = await cache_aside.get(
    "user:123",
    db_loader=lambda: db.get_user(123),
    ttl=300
)

# 更新数据
await cache_aside.set("user:123", {"name": "John"}, ttl=300)

# 删除数据
await cache_aside.delete("user:123")
```

### Write-Through 模式

```python
# cache/patterns/write_through.py
class WriteThrough:
    """Write-Through 模式"""
    
    def __init__(self, cache, db):
        self.cache = cache
        self.db = db
    
    async def get(self, key: str):
        """获取数据"""
        return await self.cache.get(key)
    
    async def set(self, key: str, value: Any, ttl: int = 300):
        """设置数据"""
        # 1. 同时写入缓存和数据库
        await asyncio.gather(
            self.cache.set(key, value, ttl),
            self.db.save(value)
        )
    
    async def delete(self, key: str):
        """删除数据"""
        # 1. 同时删除缓存和数据库
        await asyncio.gather(
            self.cache.delete(key),
            self.db.delete(key)
        )

# 使用
write_through = WriteThrough(redis_cache, db)

# 设置数据（同步写入缓存和数据库）
await write_through.set("user:123", {"name": "John"}, ttl=300)

# 获取数据（只从缓存读取）
user = await write_through.get("user:123")
```

### Write-Behind 模式

```python
# cache/patterns/write_behind.py
import asyncio
from collections import defaultdict

class WriteBehind:
    """Write-Behind 模式"""
    
    def __init__(self, cache, db, flush_interval: int = 5):
        self.cache = cache
        self.db = db
        self.flush_interval = flush_interval
        self.write_buffer = defaultdict(dict)
        self.running = False
    
    async def start(self):
        """启动"""
        self.running = True
        asyncio.create_task(self._flush_loop())
    
    async def stop(self):
        """停止"""
        self.running = False
        await self._flush()
    
    async def get(self, key: str):
        """获取数据"""
        return await self.cache.get(key)
    
    async def set(self, key: str, value: Any, ttl: int = 300):
        """设置数据"""
        # 1. 写入缓存
        await self.cache.set(key, value, ttl)
        
        # 2. 写入缓冲区
        self.write_buffer[key] = value
    
    async def _flush_loop(self):
        """刷新循环"""
        while self.running:
            await asyncio.sleep(self.flush_interval)
            await self._flush()
    
    async def _flush(self):
        """刷新到数据库"""
        if not self.write_buffer:
            return
        
        # 批量写入数据库
        tasks = []
        for key, value in self.write_buffer.items():
            tasks.append(self.db.save(key, value))
        
        await asyncio.gather(*tasks)
        
        # 清空缓冲区
        self.write_buffer.clear()

# 使用
write_behind = WriteBehind(redis_cache, db, flush_interval=5)
await write_behind.start()

# 设置数据（先写缓存，延迟写数据库）
await write_behind.set("user:123", {"name": "John"}, ttl=300)

# 停止时刷新
await write_behind.stop()
```

---

## 🎯 最佳实践

### 1. 缓存键设计

```python
# cache/key_design.py
class CacheKey:
    """缓存键设计"""
    
    @staticmethod
    def user(user_id: str) -> str:
        """用户缓存键"""
        return f"user:{user_id}"
    
    @staticmethod
    def user_posts(user_id: str, page: int) -> str:
        """用户文章缓存键"""
        return f"user:{user_id}:posts:page:{page}"
    
    @staticmethod
    def product(product_id: str) -> str:
        """商品缓存键"""
        return f"product:{product_id}"
    
    @staticmethod
    def category_products(category_id: str, page: int) -> str:
        """分类商品缓存键"""
        return f"category:{category_id}:products:page:{page}"
    
    @staticmethod
    def search_results(query: str, page: int) -> str:
        """搜索结果缓存键"""
        return f"search:{hashlib.md5(query.encode()).hexdigest()}:page:{page}"

# 使用
user_key = CacheKey.user("123")
posts_key = CacheKey.user_posts("123", page=1)
product_key = CacheKey.product("456")
```

### 2. 缓存穿透防护

```python
# cache/penetration.py
class CachePenetration:
    """缓存穿透防护"""
    
    def __init__(self, cache, db):
        self.cache = cache
        self.db = db
        self.null_cache_ttl = 60  # 空值缓存时间
    
    async def get(self, key: str, db_loader: Callable):
        """获取数据"""
        # 1. 尝试从缓存获取
        value = await self.cache.get(key)
        
        # 2. 如果是空值标记，返回 None
        if value == "NULL":
            return None
        
        if value:
            return value
        
        # 3. 从数据库加载
        value = await db_loader()
        
        # 4. 写入缓存
        if value:
            await self.cache.set(key, value, ttl=300)
        else:
            # 缓存空值，防止穿透
            await self.cache.set(key, "NULL", ttl=self.null_cache_ttl)
        
        return value

# 使用
penetration = CachePenetration(redis_cache, db)

# 获取数据（自动防护穿透）
user = await penetration.get(
    "user:999",  # 不存在的用户
    db_loader=lambda: db.get_user(999)
)
```

### 3. 缓存雪崩防护

```python
# cache/avalanche.py
import random

class CacheAvalanche:
    """缓存雪崩防护"""
    
    def __init__(self, cache):
        self.cache = cache
        self.base_ttl = 300
        self.random_range = 60  # 随机范围 ±60 秒
    
    async def set(self, key: str, value: Any):
        """设置缓存（随机 TTL）"""
        # 添加随机偏移，避免同时过期
        ttl = self.base_ttl + random.randint(
            -self.random_range,
            self.random_range
        )
        
        await self.cache.set(key, value, ttl=ttl)
    
    async def warm_up(self, keys: List[str], loader: Callable):
        """缓存预热"""
        tasks = []
        
        for key in keys:
            # 随机延迟，避免同时加载
            delay = random.uniform(0, 10)
            
            async def load():
                await asyncio.sleep(delay)
                value = await loader(key)
                await self.set(key, value)
            
            tasks.append(load())
        
        await asyncio.gather(*tasks)

# 使用
avalanche = CacheAvalanche(redis_cache)

# 设置缓存（自动添加随机 TTL）
await avalanche.set("user:123", {"name": "John"})

# 缓存预热
await avalanche.warm_up(
    keys=["user:1", "user:2", "user:3"],
    loader=lambda k: db.get_user(k)
)
```

### 4. 缓存击穿防护

```python
# cache/breakdown.py
import asyncio
from threading import Lock

class CacheBreakdown:
    """缓存击穿防护"""
    
    def __init__(self, cache, db):
        self.cache = cache
        self.db = db
        self.locks = defaultdict(Lock)
    
    async def get(self, key: str, db_loader: Callable):
        """获取数据（互斥锁）"""
        # 1. 尝试从缓存获取
        value = await self.cache.get(key)
        
        if value:
            return value
        
        # 2. 获取互斥锁
        with self.locks[key]:
            # 3. 再次检查缓存（双重检查）
            value = await self.cache.get(key)
            
            if value:
                return value
            
            # 4. 从数据库加载
            value = await db_loader()
            
            # 5. 写入缓存
            if value:
                await self.cache.set(key, value, ttl=300)
            
            return value

# 使用
breakdown = CacheBreakdown(redis_cache, db)

# 并发获取（自动防护击穿）
users = await asyncio.gather(*[
    breakdown.get("user:123", lambda: db.get_user(123))
    for _ in range(100)
])
```

---

## 🎯 性能优化

### 1. 批量操作

```python
# cache/batch.py
class BatchCache:
    """批量缓存"""
    
    def __init__(self, cache):
        self.cache = cache
    
    async def mget(self, keys: List[str]) -> Dict[str, Any]:
        """批量获取"""
        values = await asyncio.gather(*[
            self.cache.get(key)
            for key in keys
        ])
        
        return dict(zip(keys, values))
    
    async def mset(self, mapping: Dict[str, Any], ttl: int = 300):
        """批量设置"""
        await asyncio.gather(*[
            self.cache.set(key, value, ttl)
            for key, value in mapping.items()
        ])
    
    async def mdelete(self, keys: List[str]):
        """批量删除"""
        await asyncio.gather(*[
            self.cache.delete(key)
            for key in keys
        ])

# 使用
batch_cache = BatchCache(redis_cache)

# 批量获取
users = await batch_cache.mget(["user:1", "user:2", "user:3"])

# 批量设置
await batch_cache.mset({
    "user:1": {"name": "John"},
    "user:2": {"name": "Jane"},
    "user:3": {"name": "Bob"}
}, ttl=300)
```

### 2. 缓存压缩

```python
# cache/compression.py
import gzip
import json

class CompressedCache:
    """压缩缓存"""
    
    def __init__(self, cache, threshold: int = 1024):
        self.cache = cache
        self.threshold = threshold
    
    async def get(self, key: str) -> Optional[Any]:
        """获取（自动解压）"""
        value = await self.cache.get(key)
        
        if not value:
            return None
        
        # 如果是字节，尝试解压
        if isinstance(value, bytes):
            try:
                decompressed = gzip.decompress(value)
                return json.loads(decompressed)
            except:
                pass
        
        return value
    
    async def set(self, key: str, value: Any, ttl: int = 300):
        """设置（自动压缩）"""
        # 序列化
        serialized = json.dumps(value).encode()
        
        # 如果超过阈值，压缩
        if len(serialized) > self.threshold:
            serialized = gzip.compress(serialized)
        
        await self.cache.set(key, serialized, ttl)

# 使用
compressed_cache = CompressedCache(redis_cache, threshold=1024)

# 设置大数据（自动压缩）
large_data = {"data": list(range(10000))}
await compressed_cache.set("large:key", large_data, ttl=300)

# 获取数据（自动解压）
data = await compressed_cache.get("large:key")
```

---

<div align="center">
  <p>⚡ 缓存策略，性能倍增！</p>
</div>
