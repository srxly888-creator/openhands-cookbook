# OpenHands 数据库优化完全指南

> 数据库性能调优实战

---

## 📋 目录

- [索引优化](#索引优化)
- [查询优化](#查询优化)
- [连接池管理](#连接池管理)
- [分库分表](#分库分表)

---

## 📊 索引优化

### 索引设计原则

```python
# database/indexing/principles.py
from typing import List, Dict

class IndexingPrinciples:
    """索引设计原则"""
    
    @staticmethod
    def design_rules() -> List[str]:
        """设计规则"""
        return [
            "选择性高的列优先 - 唯一值多的列",
            "组合索引遵循最左前缀 - 查询条件顺序",
            "避免过度索引 - 索引占用空间",
            "考虑查询模式 - WHERE/ORDER BY/GROUP BY",
            "定期维护索引 - 重建和统计"
        ]
    
    @staticmethod
    def anti_patterns() -> List[str]:
        """反模式"""
        return [
            "在低选择性列建索引 - 如性别",
            "频繁更新的列建索引 - 影响写入性能",
            "小表建索引 - 全表扫描更快",
            "过度使用组合索引 - 索引爆炸",
            "忽略索引维护 - 索引碎片化"
        ]

# 使用
principles = IndexingPrinciples()

print("索引设计规则:")
for rule in principles.design_rules():
    print(f"  - {rule}")

print("\n索引反模式:")
for pattern in principles.anti_patterns():
    print(f"  - {pattern}")
```

### 索引创建策略

```python
# database/indexing/creation.py
import sqlalchemy
from sqlalchemy import Index

class IndexManager:
    """索引管理器"""
    
    def __init__(self, engine):
        self.engine = engine
        self.metadata = sqlalchemy.MetaData()
    
    def create_single_index(
        self,
        table: str,
        column: str,
        unique: bool = False
    ):
        """创建单列索引"""
        index_name = f"idx_{table}_{column}"
        
        sql = f"""
        CREATE {'UNIQUE' if unique else ''} INDEX {index_name}
        ON {table} ({column});
        """
        
        with self.engine.connect() as conn:
            conn.execute(sqlalchemy.text(sql))
            conn.commit()
    
    def create_composite_index(
        self,
        table: str,
        columns: List[str],
        name: str = None
    ):
        """创建组合索引"""
        if not name:
            name = f"idx_{table}_{'_'.join(columns)}"
        
        columns_str = ', '.join(columns)
        
        sql = f"""
        CREATE INDEX {name}
        ON {table} ({columns_str});
        """
        
        with self.engine.connect() as conn:
            conn.execute(sqlalchemy.text(sql))
            conn.commit()
    
    def create_partial_index(
        self,
        table: str,
        column: str,
        condition: str,
        name: str = None
    ):
        """创建部分索引"""
        if not name:
            name = f"idx_{table}_{column}_partial"
        
        sql = f"""
        CREATE INDEX {name}
        ON {table} ({column})
        WHERE {condition};
        """
        
        with self.engine.connect() as conn:
            conn.execute(sqlalchemy.text(sql))
            conn.commit()
    
    def drop_index(self, index_name: str):
        """删除索引"""
        sql = f"DROP INDEX IF EXISTS {index_name};"
        
        with self.engine.connect() as conn:
            conn.execute(sqlalchemy.text(sql))
            conn.commit()
    
    def analyze_table(self, table: str):
        """分析表统计信息"""
        sql = f"ANALYZE {table};"
        
        with self.engine.connect() as conn:
            conn.execute(sqlalchemy.text(sql))
            conn.commit()
    
    def get_index_usage(self, table: str) -> List[Dict]:
        """获取索引使用情况"""
        sql = f"""
        SELECT
            indexname,
            idx_scan,
            idx_tup_read,
            idx_tup_fetch
        FROM pg_stat_user_indexes
        WHERE relname = '{table}';
        """
        
        with self.engine.connect() as conn:
            result = conn.execute(sqlalchemy.text(sql))
            return [dict(row._mapping) for row in result]

# 使用
index_manager = IndexManager(engine)

# 创建单列索引
index_manager.create_single_index("users", "email", unique=True)

# 创建组合索引
index_manager.create_composite_index("orders", ["user_id", "created_at"])

# 创建部分索引
index_manager.create_partial_index(
    "orders",
    "status",
    condition="status = 'pending'"
)

# 分析表
index_manager.analyze_table("users")

# 查看索引使用情况
usage = index_manager.get_index_usage("users")
```

---

## 🔍 查询优化

### 查询分析

```python
# database/query/analysis.py
import sqlalchemy
from typing import List, Dict

class QueryAnalyzer:
    """查询分析器"""
    
    def __init__(self, engine):
        self.engine = engine
    
    def explain_query(self, query: str) -> Dict:
        """解释查询计划"""
        sql = f"EXPLAIN (FORMAT JSON) {query}"
        
        with self.engine.connect() as conn:
            result = conn.execute(sqlalchemy.text(sql))
            plan = result.fetchone()[0]
        
        return plan
    
    def analyze_query(self, query: str) -> Dict:
        """分析查询性能"""
        sql = f"EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) {query}"
        
        with self.engine.connect() as conn:
            result = conn.execute(sqlalchemy.text(sql))
            analysis = result.fetchone()[0]
        
        return analysis
    
    def get_slow_queries(self, threshold_ms: int = 1000) -> List[Dict]:
        """获取慢查询"""
        sql = f"""
        SELECT
            query,
            calls,
            total_time,
            mean_time,
            max_time
        FROM pg_stat_statements
        WHERE mean_time > {threshold_ms}
        ORDER BY total_time DESC
        LIMIT 10;
        """
        
        with self.engine.connect() as conn:
            result = conn.execute(sqlalchemy.text(sql))
            return [dict(row._mapping) for row in result]
    
    def suggest_index(self, query: str) -> List[str]:
        """建议索引"""
        plan = self.explain_query(query)
        
        suggestions = []
        
        # 分析计划，提取建议
        def analyze_plan_node(node):
            if 'Seq Scan' in node.get('Node Type', ''):
                table = node.get('Relation Name')
                suggestions.append(f"Consider adding index on table: {table}")
            
            for key, value in node.items():
                if isinstance(value, dict):
                    analyze_plan_node(value)
                elif isinstance(value, list):
                    for item in value:
                        if isinstance(item, dict):
                            analyze_plan_node(item)
        
        analyze_plan_node(plan[0]['Plan'])
        
        return suggestions

# 使用
analyzer = QueryAnalyzer(engine)

# 解释查询计划
plan = analyzer.explain_query("SELECT * FROM users WHERE email = 'test@example.com'")
print(json.dumps(plan, indent=2))

# 分析查询性能
analysis = analyzer.analyze_query("SELECT * FROM users WHERE email = 'test@example.com'")
print(f"Execution time: {analysis[0]['Execution Time']}ms")

# 获取慢查询
slow_queries = analyzer.get_slow_queries(threshold_ms=1000)
for query in slow_queries:
    print(f"Query: {query['query']}")
    print(f"Mean time: {query['mean_time']}ms")

# 建议索引
suggestions = analyzer.suggest_index("SELECT * FROM orders WHERE user_id = 123")
for suggestion in suggestions:
    print(suggestion)
```

### 查询优化技巧

```python
# database/query/optimization.py
from typing import List, Dict

class QueryOptimizer:
    """查询优化器"""
    
    @staticmethod
    def optimize_select_star(query: str) -> str:
        """优化 SELECT *"""
        # 避免 SELECT *，只选择需要的列
        return query.replace("SELECT *", "SELECT id, name, email")
    
    @staticmethod
    def optimize_or_conditions(query: str) -> str:
        """优化 OR 条件"""
        # 将 OR 转换为 UNION
        if " OR " in query:
            # 解析并转换
            pass
        return query
    
    @staticmethod
    def optimize_like_query(query: str) -> str:
        """优化 LIKE 查询"""
        # 避免前导通配符
        return query.replace("LIKE '%", "LIKE '")
    
    @staticmethod
    def optimize_subquery(query: str) -> str:
        """优化子查询"""
        # 将子查询转换为 JOIN
        return query
    
    @staticmethod
    def get_optimization_rules() -> List[str]:
        """获取优化规则"""
        return [
            "避免 SELECT * - 只选择需要的列",
            "避免在 WHERE 中使用函数 - 影响索引使用",
            "避免前导通配符 - LIKE '%xxx' 无法使用索引",
            "避免 OR 条件 - 使用 UNION 或 IN",
            "避免子查询 - 使用 JOIN",
            "使用 LIMIT - 限制结果集大小",
            "使用 EXISTS 替代 IN - 性能更好",
            "避免 DISTINCT - 考虑 GROUP BY",
            "使用覆盖索引 - 避免回表",
            "批量操作 - 减少数据库交互"
        ]

# 使用
optimizer = QueryOptimizer()

# 优化查询
query = "SELECT * FROM users WHERE email LIKE '%@example.com'"
optimized = optimizer.optimize_like_query(query)
print(f"Optimized: {optimized}")

# 获取优化规则
for rule in optimizer.get_optimization_rules():
    print(f"  - {rule}")
```

---

## 🔌 连接池管理

### 连接池配置

```python
# database/pool/config.py
import sqlalchemy
from sqlalchemy.pool import QueuePool

class ConnectionPoolConfig:
    """连接池配置"""
    
    @staticmethod
    def get_config() -> Dict:
        """获取配置"""
        return {
            'pool_size': 10,           # 连接池大小
            'max_overflow': 5,          # 最大溢出连接数
            'pool_timeout': 30,         # 获取连接超时时间
            'pool_recycle': 3600,       # 连接回收时间
            'pool_pre_ping': True,      # 连接健康检查
            'echo': False               # 是否打印 SQL
        }
    
    @staticmethod
    def create_engine(connection_string: str):
        """创建引擎"""
        config = ConnectionPoolConfig.get_config()
        
        return sqlalchemy.create_engine(
            connection_string,
            poolclass=QueuePool,
            **config
        )

# 使用
engine = ConnectionPoolConfig.create_engine(
    "postgresql://user:pass@localhost/db"
)
```

### 连接池监控

```python
# database/pool/monitoring.py
import time
from typing import Dict

class ConnectionPoolMonitor:
    """连接池监控"""
    
    def __init__(self, engine):
        self.engine = engine
        self.pool = engine.pool
    
    def get_status(self) -> Dict:
        """获取连接池状态"""
        return {
            'pool_size': self.pool.size(),
            'checked_in': self.pool.checkedin(),
            'checked_out': self.pool.checkedout(),
            'overflow': self.pool.overflow(),
            'invalid': self.pool.invalidatedcount()
        }
    
    def get_metrics(self) -> Dict:
        """获取指标"""
        status = self.get_status()
        
        return {
            'utilization': status['checked_out'] / status['pool_size'],
            'available': status['checked_in'],
            'total': status['pool_size'] + status['overflow']
        }
    
    def check_health(self) -> bool:
        """检查健康"""
        metrics = self.get_metrics()
        
        # 如果利用率超过 80%，认为不健康
        if metrics['utilization'] > 0.8:
            return False
        
        return True
    
    def log_status(self):
        """记录状态"""
        status = self.get_status()
        metrics = self.get_metrics()
        
        print(f"Pool Status:")
        print(f"  Size: {status['pool_size']}")
        print(f"  Checked In: {status['checked_in']}")
        print(f"  Checked Out: {status['checked_out']}")
        print(f"  Overflow: {status['overflow']}")
        print(f"  Utilization: {metrics['utilization']:.2%}")

# 使用
monitor = ConnectionPoolMonitor(engine)

# 获取状态
status = monitor.get_status()
print(status)

# 获取指标
metrics = monitor.get_metrics()
print(f"Utilization: {metrics['utilization']:.2%}")

# 检查健康
is_healthy = monitor.check_health()
print(f"Healthy: {is_healthy}")

# 记录状态
monitor.log_status()
```

---

## 🔀 分库分表

### 垂直分表

```python
# database/sharding/vertical.py
from typing import Dict

class VerticalSharding:
    """垂直分表"""
    
    def __init__(self):
        self.tables = {}
    
    def split_table(
        self,
        table: str,
        hot_columns: List[str],
        cold_columns: List[str]
    ):
        """分表"""
        # 热数据表
        hot_table = f"{table}_hot"
        self.tables[hot_table] = hot_columns
        
        # 冷数据表
        cold_table = f"{table}_cold"
        self.tables[cold_table] = cold_columns
    
    def get_query_strategy(self, columns: List[str]) -> str:
        """获取查询策略"""
        hot_table = None
        cold_table = None
        
        for table, cols in self.tables.items():
            if all(col in cols for col in columns):
                if 'hot' in table:
                    hot_table = table
                elif 'cold' in table:
                    cold_table = table
        
        if hot_table:
            return hot_table
        elif cold_table:
            return cold_table
        else:
            return "JOIN"

# 使用
sharding = VerticalSharding()

# 分表
sharding.split_table(
    table="users",
    hot_columns=["id", "email", "name", "created_at"],
    cold_columns=["bio", "preferences", "settings"]
)

# 查询策略
strategy = sharding.get_query_strategy(["email", "name"])
print(f"Query strategy: {strategy}")
```

### 水平分表

```python
# database/sharding/horizontal.py
from typing import Dict, Any
import hashlib

class HorizontalSharding:
    """水平分表"""
    
    def __init__(self, shard_count: int = 10):
        self.shard_count = shard_count
    
    def get_shard_key(self, key: Any) -> int:
        """获取分片键"""
        key_str = str(key)
        hash_value = int(hashlib.md5(key_str.encode()).hexdigest(), 16)
        return hash_value % self.shard_count
    
    def get_shard_table(self, base_table: str, key: Any) -> str:
        """获取分片表"""
        shard_key = self.get_shard_key(key)
        return f"{base_table}_{shard_key}"
    
    def insert(self, base_table: str, key: Any, data: Dict):
        """插入数据"""
        shard_table = self.get_shard_table(base_table, key)
        
        # 插入到对应的分片表
        sql = f"INSERT INTO {shard_table} VALUES (:data)"
        # 执行插入
    
    def query(self, base_table: str, key: Any) -> Dict:
        """查询数据"""
        shard_table = self.get_shard_table(base_table, key)
        
        # 从对应的分片表查询
        sql = f"SELECT * FROM {shard_table} WHERE id = :key"
        # 执行查询

# 使用
sharding = HorizontalSharding(shard_count=10)

# 获取分片表
shard_table = sharding.get_shard_table("orders", user_id=12345)
print(f"Shard table: {shard_table}")

# 插入数据
sharding.insert("orders", 12345, {"user_id": 12345, "amount": 99.99})

# 查询数据
data = sharding.query("orders", 12345)
```

---

## 🎯 数据库优化最佳实践

### 1. 索引最佳实践

- **选择性优先** - 高选择性列优先建索引
- **组合索引** - 遵循最左前缀原则
- **避免过度** - 不要创建太多索引
- **定期维护** - 定期重建和分析

### 2. 查询最佳实践

- **避免 SELECT *** - 只选择需要的列
- **使用索引** - 确保查询使用索引
- **批量操作** - 减少数据库交互
- **合理分页** - 避免大结果集

### 3. 连接池最佳实践

- **合理配置** - 根据负载调整大小
- **监控指标** - 监控利用率和溢出
- **健康检查** - 定期检查连接健康
- **及时回收** - 避免连接泄漏

### 4. 分库分表最佳实践

- **合理分片** - 根据业务选择分片键
- **避免跨片** - 尽量避免跨分片查询
- **数据均衡** - 确保数据分布均匀
- **扩展性** - 预留扩展空间

---

<div align="center">
  <p>📊 数据库优化，性能倍增！</p>
</div>
