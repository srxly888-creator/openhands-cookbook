# OpenHands 微服务设计模式

> 微服务架构设计模式

---

## 📋 目录

- [服务拆分模式](#服务拆分模式)
- [通信模式](#通信模式)
- [数据管理模式](#数据管理模式)
- [部署模式](#部署模式)

---

## 🔧 服务拆分模式

### 按业务能力拆分

```python
# microservices/patterns/business_capability.py
from typing import List, Dict

class BusinessCapabilityPattern:
    """按业务能力拆分"""
    
    @staticmethod
    def design_principles() -> List[str]:
        """设计原则"""
        return [
            "单一职责 - 每个服务对应一个业务能力",
            "高内聚 - 服务内部功能高度相关",
            "低耦合 - 服务之间依赖最小化",
            "独立部署 - 服务可以独立部署",
            "团队 ownership - 一个团队负责一个服务"
        ]
    
    @staticmethod
    def service_examples() -> Dict[str, List[str]]:
        """服务示例"""
        return {
            "用户服务": [
                "用户注册",
                "用户认证",
                "用户资料管理"
            ],
            "订单服务": [
                "订单创建",
                "订单查询",
                "订单状态管理"
            ],
            "商品服务": [
                "商品管理",
                "库存管理",
                "价格管理"
            ],
            "支付服务": [
                "支付处理",
                "退款处理",
                "账单管理"
            ]
        }

# 使用
pattern = BusinessCapabilityPattern()

print("设计原则:")
for p in pattern.design_principles():
    print(f"  - {p}")

print("\n服务示例:")
for service, capabilities in pattern.service_examples().items():
    print(f"\n{service}:")
    for cap in capabilities:
        print(f"  - {cap}")
```

### 按子域拆分

```python
# microservices/patterns/subdomain.py
from typing import List, Dict

class SubdomainPattern:
    """按子域拆分"""
    
    @staticmethod
    def subdomain_types() -> Dict[str, str]:
        """子域类型"""
        return {
            "核心子域": "业务核心竞争力，需要重点投入",
            "支撑子域": "支持核心业务，可以外包或使用现成方案",
            "通用子域": "通用业务功能，使用标准解决方案"
        }
    
    @staticmethod
    def example_breakdown() -> Dict[str, Dict]:
        """电商系统子域拆分"""
        return {
            "核心子域": {
                "服务": ["商品服务", "订单服务"],
                "说明": "电商核心竞争力"
            },
            "支撑子域": {
                "服务": ["用户服务", "支付服务"],
                "说明": "支持核心业务"
            },
            "通用子域": {
                "服务": ["通知服务", "搜索服务"],
                "说明": "通用功能"
            }
        }

# 使用
pattern = SubdomainPattern()

print("子域类型:")
for name, desc in pattern.subdomain_types().items():
    print(f"  {name}: {desc}")

print("\n电商系统子域拆分:")
for domain, info in pattern.example_breakdown().items():
    print(f"\n{domain}:")
    print(f"  服务: {', '.join(info['服务'])}")
    print(f"  说明: {info['说明']}")
```

---

## 🔗 通信模式

### 同步通信

```python
# microservices/patterns/sync_communication.py
import aiohttp
from typing import Dict, Optional

class SyncCommunication:
    """同步通信模式"""
    
    def __init__(self, base_url: str):
        self.base_url = base_url
        self.session = None
    
    async def call_service(
        self,
        service_name: str,
        endpoint: str,
        method: str = "GET",
        data: Optional[Dict] = None
    ) -> Dict:
        """调用服务"""
        if not self.session:
            self.session = aiohttp.ClientSession()
        
        url = f"{self.base_url}/{service_name}/{endpoint}"
        
        async with self.session.request(
            method,
            url,
            json=data
        ) as response:
            return await response.json()
    
    async def chain_calls(self, calls: List[Dict]) -> List[Dict]:
        """链式调用"""
        results = []
        
        for call in calls:
            result = await self.call_service(
                service_name=call['service'],
                endpoint=call['endpoint'],
                method=call.get('method', 'GET'),
                data=call.get('data')
            )
            
            results.append(result)
        
        return results

# 使用
comm = SyncCommunication("http://api.example.com")

# 单次调用
user = await comm.call_service(
    service_name="users",
    endpoint="123",
    method="GET"
)

# 链式调用
results = await comm.chain_calls([
    {"service": "users", "endpoint": "123"},
    {"service": "orders", "endpoint": "user/123"},
    {"service": "products", "endpoint": "456"}
])
```

### 异步通信

```python
# microservices/patterns/async_communication.py
import asyncio
from typing import Dict, List
from dataclasses import dataclass
from datetime import datetime

@dataclass
class Message:
    """消息"""
    id: str
    topic: str
    payload: Dict
    timestamp: datetime

class MessageQueue:
    """消息队列"""
    
    def __init__(self):
        self.topics: Dict[str, List[Message]] = {}
        self.subscribers: Dict[str, List[callable]] = {}
    
    async def publish(self, topic: str, payload: Dict):
        """发布消息"""
        message = Message(
            id=str(uuid.uuid4()),
            topic=topic,
            payload=payload,
            timestamp=datetime.utcnow()
        )
        
        if topic not in self.topics:
            self.topics[topic] = []
        
        self.topics[topic].append(message)
        
        # 通知订阅者
        if topic in self.subscribers:
            for subscriber in self.subscribers[topic]:
                await subscriber(message)
    
    async def subscribe(self, topic: str, handler: callable):
        """订阅消息"""
        if topic not in self.subscribers:
            self.subscribers[topic] = []
        
        self.subscribers[topic].append(handler)
    
    async def get_messages(self, topic: str) -> List[Message]:
        """获取消息"""
        return self.topics.get(topic, [])

# 使用
mq = MessageQueue()

# 订阅
async def handle_order(message: Message):
    print(f"Processing order: {message.payload}")

await mq.subscribe("orders", handle_order)

# 发布
await mq.publish("orders", {
    "order_id": "123",
    "user_id": "456",
    "amount": 99.99
})
```

### 事件驱动

```python
# microservices/patterns/event_driven.py
from typing import Dict, List, Callable
from dataclasses import dataclass
from datetime import datetime
import asyncio

@dataclass
class Event:
    """事件"""
    type: str
    data: Dict
    timestamp: datetime
    metadata: Dict

class EventBus:
    """事件总线"""
    
    def __init__(self):
        self.handlers: Dict[str, List[Callable]] = {}
        self.event_store: List[Event] = []
    
    async def emit(self, event_type: str, data: Dict, metadata: Dict = None):
        """发射事件"""
        event = Event(
            type=event_type,
            data=data,
            timestamp=datetime.utcnow(),
            metadata=metadata or {}
        )
        
        # 保存事件
        self.event_store.append(event)
        
        # 调用处理器
        if event_type in self.handlers:
            for handler in self.handlers[event_type]:
                await handler(event)
    
    def on(self, event_type: str, handler: Callable):
        """监听事件"""
        if event_type not in self.handlers:
            self.handlers[event_type] = []
        
        self.handlers[event_type].append(handler)
    
    def get_events(self, event_type: str = None) -> List[Event]:
        """获取事件"""
        if event_type:
            return [e for e in self.event_store if e.type == event_type]
        return self.event_store

# 使用
event_bus = EventBus()

# 监听事件
async def handle_user_created(event: Event):
    print(f"User created: {event.data}")

async def send_welcome_email(event: Event):
    print(f"Sending welcome email to: {event.data['email']}")

event_bus.on("user_created", handle_user_created)
event_bus.on("user_created", send_welcome_email)

# 发射事件
await event_bus.emit("user_created", {
    "user_id": "123",
    "email": "user@example.com"
})
```

---

## 💾 数据管理模式

### 数据库 per 服务

```python
# microservices/patterns/database_per_service.py
from typing import Dict
import sqlalchemy
from sqlalchemy.orm import sessionmaker

class DatabasePerService:
    """每个服务一个数据库"""
    
    def __init__(self):
        self.databases: Dict[str, sqlalchemy.engine.Engine] = {}
        self.sessions: Dict[str, sessionmaker] = {}
    
    def create_database(self, service_name: str, connection_string: str):
        """创建数据库连接"""
        engine = sqlalchemy.create_engine(connection_string)
        self.databases[service_name] = engine
        self.sessions[service_name] = sessionmaker(bind=engine)
    
    def get_session(self, service_name: str):
        """获取会话"""
        if service_name not in self.sessions:
            raise ValueError(f"Database for {service_name} not found")
        
        return self.sessions[service_name]()
    
    def close_all(self):
        """关闭所有连接"""
        for engine in self.databases.values():
            engine.dispose()

# 使用
db_manager = DatabasePerService()

# 为每个服务创建数据库
db_manager.create_database(
    service_name="users",
    connection_string="postgresql://localhost/users"
)

db_manager.create_database(
    service_name="orders",
    connection_string="postgresql://localhost/orders"
)

# 使用
user_session = db_manager.get_session("users")
order_session = db_manager.get_session("orders")
```

### 事件溯源

```python
# microservices/patterns/event_sourcing.py
from typing import Dict, List
from dataclasses import dataclass
from datetime import datetime
import json

@dataclass
class DomainEvent:
    """领域事件"""
    id: str
    type: str
    aggregate_id: str
    data: Dict
    version: int
    timestamp: datetime

class EventStore:
    """事件存储"""
    
    def __init__(self):
        self.events: List[DomainEvent] = []
    
    def append(self, event: DomainEvent):
        """追加事件"""
        self.events.append(event)
    
    def get_events(self, aggregate_id: str) -> List[DomainEvent]:
        """获取聚合的事件"""
        return [
            e for e in self.events
            if e.aggregate_id == aggregate_id
        ]
    
    def get_all_events(self) -> List[DomainEvent]:
        """获取所有事件"""
        return self.events

class Aggregate:
    """聚合根"""
    
    def __init__(self, aggregate_id: str):
        self.id = aggregate_id
        self.version = 0
        self.uncommitted_events: List[DomainEvent] = []
    
    def apply_event(self, event: DomainEvent):
        """应用事件"""
        self.version = event.version
        self._handle_event(event)
    
    def emit_event(self, event_type: str, data: Dict):
        """发射事件"""
        event = DomainEvent(
            id=str(uuid.uuid4()),
            type=event_type,
            aggregate_id=self.id,
            data=data,
            version=self.version + 1,
            timestamp=datetime.utcnow()
        )
        
        self.uncommitted_events.append(event)
        self.apply_event(event)
    
    def _handle_event(self, event: DomainEvent):
        """处理事件（子类实现）"""
        pass

# 示例：订单聚合
class Order(Aggregate):
    """订单聚合"""
    
    def __init__(self, order_id: str):
        super().__init__(order_id)
        self.status = None
        self.items = []
        self.total = 0
    
    def create(self, items: List[Dict]):
        """创建订单"""
        self.emit_event("order_created", {
            "items": items,
            "total": sum(item['price'] for item in items)
        })
    
    def add_item(self, item: Dict):
        """添加商品"""
        self.emit_event("item_added", {
            "item": item
        })
    
    def complete(self):
        """完成订单"""
        self.emit_event("order_completed", {})
    
    def _handle_event(self, event: DomainEvent):
        """处理事件"""
        if event.type == "order_created":
            self.status = "created"
            self.items = event.data["items"]
            self.total = event.data["total"]
        
        elif event.type == "item_added":
            self.items.append(event.data["item"])
            self.total += event.data["item"]["price"]
        
        elif event.type == "order_completed":
            self.status = "completed"

# 使用
event_store = EventStore()

# 创建订单
order = Order("order-123")
order.create([
    {"product_id": "prod-1", "price": 10},
    {"product_id": "prod-2", "price": 20}
])

# 添加商品
order.add_item({"product_id": "prod-3", "price": 15})

# 完成订单
order.complete()

# 保存事件
for event in order.uncommitted_events:
    event_store.append(event)

# 重建订单
def rebuild_order(order_id: str) -> Order:
    """重建订单"""
    order = Order(order_id)
    events = event_store.get_events(order_id)
    
    for event in sorted(events, key=lambda e: e.version):
        order.apply_event(event)
    
    return order

# 重建
rebuilt_order = rebuild_order("order-123")
print(f"Order status: {rebuilt_order.status}")
print(f"Total: {rebuilt_order.total}")
```

---

## 🚀 部署模式

### 蓝绿部署

```python
# microservices/patterns/blue_green.py
from typing import Dict

class BlueGreenDeployment:
    """蓝绿部署"""
    
    def __init__(self):
        self.blue = {"version": "v1", "active": True}
        self.green = {"version": "v2", "active": False}
    
    def deploy(self, new_version: str):
        """部署新版本"""
        # 部署到非活跃环境
        if not self.blue["active"]:
            self.blue["version"] = new_version
        else:
            self.green["version"] = new_version
    
    def switch(self):
        """切换流量"""
        if self.blue["active"]:
            self.blue["active"] = False
            self.green["active"] = True
        else:
            self.green["active"] = False
            self.blue["active"] = True
    
    def rollback(self):
        """回滚"""
        self.switch()
    
    def get_status(self) -> Dict:
        """获取状态"""
        return {
            "blue": self.blue,
            "green": self.green
        }

# 使用
deployment = BlueGreenDeployment()

# 部署新版本
deployment.deploy("v2.0.0")

# 切换流量
deployment.switch()

# 查看状态
status = deployment.get_status()
print(status)

# 回滚
deployment.rollback()
```

### 金丝雀发布

```python
# microservices/patterns/canary.py
from typing import Dict

class CanaryDeployment:
    """金丝雀发布"""
    
    def __init__(self):
        self.stable_version = "v1"
        self.canary_version = None
        self.canary_weight = 0
    
    def deploy_canary(self, version: str):
        """部署金丝雀版本"""
        self.canary_version = version
        self.canary_weight = 10  # 10% 流量
    
    def increase_weight(self, increment: int = 10):
        """增加金丝雀流量"""
        self.canary_weight = min(100, self.canary_weight + increment)
        
        if self.canary_weight >= 100:
            # 完全切换
            self.stable_version = self.canary_version
            self.canary_version = None
            self.canary_weight = 0
    
    def rollback(self):
        """回滚"""
        self.canary_version = None
        self.canary_weight = 0
    
    def get_status(self) -> Dict:
        """获取状态"""
        return {
            "stable_version": self.stable_version,
            "canary_version": self.canary_version,
            "canary_weight": self.canary_weight
        }

# 使用
canary = CanaryDeployment()

# 部署金丝雀
canary.deploy_canary("v2.0.0")

# 逐步增加流量
canary.increase_weight(10)  # 10% -> 20%
canary.increase_weight(10)  # 20% -> 30%
canary.increase_weight(70)  # 30% -> 100%

# 查看状态
status = canary.get_status()
print(status)
```

---

## 🎯 微服务最佳实践

### 1. 服务拆分原则

- **单一职责** - 每个服务只负责一个功能
- **高内聚** - 服务内部功能高度相关
- **低耦合** - 服务之间依赖最小化
- **独立部署** - 服务可以独立部署

### 2. 通信原则

- **异步优先** - 优先使用异步通信
- **幂等性** - 操作支持幂等
- **超时控制** - 设置合理的超时
- **重试机制** - 实现自动重试

### 3. 数据管理原则

- **数据库隔离** - 每个服务独立数据库
- **事件驱动** - 使用事件同步数据
- **最终一致性** - 接受最终一致性
- **数据冗余** - 允许数据冗余

### 4. 部署原则

- **容器化** - 所有服务容器化
- **自动化** - 自动化部署流程
- **可回滚** - 支持快速回滚
- **监控** - 实时监控服务

---

<div align="center">
  <p>🏗️ 微服务架构，灵活可扩展！</p>
</div>
