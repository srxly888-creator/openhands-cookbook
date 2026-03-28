# OpenHands 消息队列完全指南

> 消息队列架构和实现

---

## 📋 目录

- [消息队列架构](#消息队列架构)
- [RabbitMQ 集成](#rabbitmq-集成)
- [消息模式](#消息模式)
- [最佳实践](#最佳实践)

---

## 🏗️ 消息队列架构

### 消息队列架构图

```
┌─────────────────────────────────────────┐
│           生产者（Producer）              │
└────────────────┬────────────────────────┘
                 │
                 │ 发布消息
                 │
┌────────────────▼────────────────────────┐
│         消息队列（RabbitMQ）           │
└────────────────┬────────────────────────┘
                 │
                 │ 订阅消息
                 │
┌─────────────────────────────────────────┐
│           消费者（Consumer）              │
└─────────────────────────────────────────┘
```

### 消息队列核心组件

```python
# messaging/queue.py
from typing import Dict, List, Callable
import asyncio
from dataclasses import dataclass
from datetime import datetime

@dataclass
class Message:
    """消息"""
    id: str
    topic: str
    payload: Dict
    timestamp: datetime
    retry_count: int = 0

class MessageQueue:
    """消息队列"""
    
    def __init__(self):
        self.topics: Dict[str, List[Message]] = {}
        self.subscriptions: Dict[str, List[Callable]] = {}
    
    async def publish(self, topic: str, payload: Dict):
        """发布消息"""
        message = Message(
            id=str(uuid.uuid4()),
            topic=topic,
            payload=payload,
            timestamp=datetime.utcnow(),
            retry_count=0
        )
        
        if topic not in self.topics:
            self.topics[topic] = []
        
        self.topics[topic].append(message)
        
        # 通知订阅者
        if topic in self.subscriptions:
            for handler in self.subscriptions[topic]:
                await handler(message)
    
    def subscribe(self, topic: str, handler: Callable):
        """订阅消息"""
        if topic not in self.subscriptions:
            self.subscriptions[topic] = []
        
        self.subscriptions[topic].append(handler)
    
    def unsubscribe(self, topic: str, handler: Callable):
        """取消订阅"""
        if topic in self.subscriptions:
            self.subscriptions[topic] = [
                h for h in self.subscriptions[topic]
                if h != handler
            ]
    
    def get_messages(self, topic: str) -> List[Message]:
        """获取消息"""
        return self.topics.get(topic, [])
    
    def ack(self, message_id: str):
        """确认消息"""
        # 在实际实现中，会从队列中删除消息
        pass

# 使用
mq = MessageQueue()

# 订阅
async def handle_order(message: Message):
    print(f"Processing order: {message.payload}")

mq.subscribe("orders", handle_order)

# 发布
await mq.publish("orders", {
    "order_id": "123",
    "user_id": "456",
    "amount": 99.99
})
```

---

## 🐰 RabbitMQ 集成

### RabbitMQ 生产者

```python
# messaging/rabbitmq_producer.py
import aio_pika
from typing import Dict
import json

class RabbitMQProducer:
    """RabbitMQ 生产者"""
    
    def __init__(self, url: str = "amqp://guest:guest@localhost/"):
        self.url = url
        self.connection = None
        self.channel = None
    
    async def connect(self):
        """连接"""
        self.connection = await aio_pika.connect_robust(self.url)
        self.channel = await self.connection.channel()
    
    async def publish(self, topic: str, message: Dict):
        """发布消息"""
        await self.channel.default_exchange.publish(
            aio_pika.Message(
                body=json.dumps(message).encode()
            ),
            routing_key=topic
        )
    
    async def close(self):
        """关闭连接"""
        if self.connection:
            await self.connection.close()

# 使用
producer = RabbitMQProducer()
await producer.connect()

# 发布消息
await producer.publish("orders", {
    "order_id": "123",
    "user_id": "456"
})

await producer.close()
```

### RabbitMQ 消费者

```python
# messaging/rabbitmq_consumer.py
import aio_pika
from typing import Callable
import json

class RabbitMQConsumer:
    """RabbitMQ 消费者"""
    
    def __init__(self, url: str = "amqp://guest:guest@localhost/"):
        self.url = url
        self.connection = None
        self.channel = None
    
    async def connect(self):
        """连接"""
        self.connection = await aio_pika.connect_robust(self.url)
        self.channel = await self.connection.channel()
    
    async def subscribe(self, topic: str, handler: Callable):
        """订阅消息"""
        # 声明队列
        queue = await self.channel.declare_queue(
            topic,
            durable=True
        )
        
        # 绑定到交换器
        await queue.bind(
            self.channel.default_exchange,
            routing_key=topic
        )
        
        # 消费消息
        async with queue.iterator() as queue_iter:
            async for message in queue_iter:
                async with message.process():
                    body = json.loads(message.body)
                    await handler(body)
    
    async def close(self):
        """关闭连接"""
        if self.connection:
            await self.connection.close()

# 使用
consumer = RabbitMQConsumer()
await consumer.connect()

# 订阅消息
async def handle_order(message: Dict):
    print(f"Processing order: {message}")

await consumer.subscribe("orders", handle_order)

# 保持运行
await asyncio.Future()

await consumer.close()
```

---

## 📨 消息模式

### 巌息发布-订阅模式

```python
# messaging/patterns/pubsub.py
from typing import Dict, List, Callable

class PubSubPattern:
    """发布-订阅模式"""
    
    def __init__(self):
        self.topics: Dict[str, List[Callable]] = {}
    
    def publish(self, topic: str, message: Dict):
        """发布消息"""
        if topic in self.topics:
            for handler in self.topics[topic]:
                handler(message)
    
    def subscribe(self, topic: str, handler: Callable):
        """订阅消息"""
        if topic not in self.topics:
            self.topics[topic] = []
        
        self.topics[topic].append(handler)
    
    def unsubscribe(self, topic: str, handler: Callable):
        """取消订阅"""
        if topic in self.topics:
            self.topics[topic] = [
                h for h in self.topics[topic]
                if h != handler
            ]

# 使用
pubsub = PubSubPattern()

# 订阅
def handle_user_created(message):
    print(f"User created: {message}")

pubsub.subscribe("user_created", handle_user_created)

# 发布
pubsub.publish("user_created", {
    "user_id": "123",
    "email": "user@example.com"
})
```

### 消息队列模式

```python
# messaging/patterns/queue.py
from typing import Dict, List
from dataclasses import dataclass
from datetime import datetime

@dataclass
class QueueMessage:
    """队列消息"""
    id: str
    payload: Dict
    timestamp: datetime
    status: str = "pending"

class QueuePattern:
    """消息队列模式"""
    
    def __init__(self):
        self.queue: List[QueueMessage] = []
    
    def enqueue(self, payload: Dict) -> str:
        """入队"""
        message = QueueMessage(
            id=str(uuid.uuid4()),
            payload=payload,
            timestamp=datetime.utcnow()
        )
        
        self.queue.append(message)
        return message.id
    
    def dequeue(self) -> QueueMessage:
        """出队"""
        if self.queue:
            message = self.queue.pop(0)
            message.status = "processing"
            return message
        return None
    
    def complete(self, message_id: str):
        """完成"""
        for message in self.queue:
            if message.id == message_id:
                message.status = "completed"
                break
    
    def get_queue_length(self) -> int:
        """获取队列长度"""
        return len(self.queue)

# 使用
queue = QueuePattern()

# 入队
message_id = queue.enqueue({
    "order_id": "123",
    "amount": 99.99
})

# 出队
message = queue.dequeue()
print(f"Processing: {message.payload}")

# 完成
queue.complete(message_id)
```

---

## 🎯 最佳实践

### 1. 消息设计原则

- **幂等性** - 消息处理支持重复
- **顺序性** - 保证消息顺序
- **可靠性** - 消息不丢失
- **可扩展** - 支持水平扩展

### 2. 队列管理原则

- **监控** - 宨时监控队列状态
- **限流** - 掰制消息堆积
- **重试** - 失败自动重试
- **死信队列** - 处理失败消息

### 3. 性能优化原则

- **批量处理** - 批量处理消息
- **预取** - 预取消息
- **异步** - 异步处理消息
- **分区** - 队列分区

---

<div align="center">
  <p>📨 消息队列，异步解耦！</p>
</div>
