# OpenHands 基础案例集

> 通过实际案例快速上手 OpenHands

---

## 📋 案例列表

| 案例 | 难度 | 时间 | 学习目标 |
|------|------|------|---------|
| [案例 1：创建 Hello World API](#案例-1创建-hello-world-api) | ⭐ | 5 分钟 | 基础 API 创建 |
| [案例 2：文件处理工具](#案例-2文件处理工具) | ⭐⭐ | 10 分钟 | 文件操作 |
| [案例 3：数据处理脚本](#案例-3数据处理脚本) | ⭐⭐ | 15 分钟 | 数据处理 |
| [案例 4：单元测试生成](#案例-4单元测试生成) | ⭐⭐⭐ | 20 分钟 | 测试驱动 |
| [案例 5：CLI 工具开发](#案例-5cli-工具开发) | ⭐⭐⭐ | 25 分钟 | 命令行工具 |

---

## 案例 1：创建 Hello World API

### 任务描述

创建一个简单的 FastAPI 应用，返回 "Hello, World!"。

### CLI 命令

```bash
openhands run "创建一个 FastAPI 应用：
1. 路由 GET / 返回 {'message': 'Hello, World!'}
2. 路由 GET /health 返回 {'status': 'ok'}
3. 添加 requirements.txt
4. 添加 README.md，包含运行说明"
```

### 预期结果

**main.py**：
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello, World!"}

@app.get("/health")
async def health():
    return {"status": "ok"}
```

**requirements.txt**：
```
fastapi==0.104.1
uvicorn[standard]==0.24.0
```

**运行**：
```bash
pip install -r requirements.txt
uvicorn main:app --reload
```

**测试**：
```bash
curl http://localhost:8000/
# {"message":"Hello, World!"}

curl http://localhost:8000/health
# {"status":"ok"}
```

### 学习要点

- ✅ FastAPI 基础路由
- ✅ JSON 响应
- ✅ 依赖管理
- ✅ 文档编写

---

## 案例 2：文件处理工具

### 任务描述

创建一个文件处理工具，支持：
- 读取 JSON 文件
- 过滤数据
- 导出为 CSV

### CLI 命令

```bash
openhands run "创建一个 Python 脚本 file_processor.py：
1. 读取 data.json（包含用户列表）
2. 过滤年龄 > 18 的用户
3. 导出到 filtered_users.csv
4. 添加日志记录
5. 添加错误处理
6. 编写单元测试 test_file_processor.py"
```

### 预期结果

**file_processor.py**：
```python
import json
import csv
import logging
from typing import List, Dict

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def read_json(file_path: str) -> List[Dict]:
    """读取 JSON 文件"""
    try:
        with open(file_path, 'r') as f:
            data = json.load(f)
        logger.info(f"成功读取 {file_path}")
        return data
    except Exception as e:
        logger.error(f"读取失败: {e}")
        raise

def filter_users(users: List[Dict], min_age: int = 18) -> List[Dict]:
    """过滤用户"""
    filtered = [u for u in users if u.get('age', 0) > min_age]
    logger.info(f"过滤完成，剩余 {len(filtered)} 个用户")
    return filtered

def export_to_csv(users: List[Dict], output_path: str):
    """导出为 CSV"""
    if not users:
        logger.warning("没有数据可导出")
        return
    
    with open(output_path, 'w', newline='') as f:
        writer = csv.DictWriter(f, fieldnames=users[0].keys())
        writer.writeheader()
        writer.writerows(users)
    
    logger.info(f"成功导出到 {output_path}")

if __name__ == "__main__":
    # 示例用法
    users = read_json("data.json")
    filtered = filter_users(users, min_age=18)
    export_to_csv(filtered, "filtered_users.csv")
```

**test_file_processor.py**：
```python
import pytest
from file_processor import read_json, filter_users, export_to_csv
import tempfile
import os

def test_filter_users():
    users = [
        {"name": "Alice", "age": 20},
        {"name": "Bob", "age": 15},
        {"name": "Charlie", "age": 25}
    ]
    result = filter_users(users, min_age=18)
    assert len(result) == 2
    assert result[0]["name"] == "Alice"

def test_export_to_csv():
    users = [{"name": "Alice", "age": 20}]
    with tempfile.NamedTemporaryFile(mode='w', delete=False, suffix='.csv') as f:
        export_to_csv(users, f.name)
        assert os.path.exists(f.name)
        os.unlink(f.name)
```

### 学习要点

- ✅ 文件读写操作
- ✅ 数据过滤
- ✅ 日志记录
- ✅ 错误处理
- ✅ 单元测试

---

## 案例 3：数据处理脚本

### 任务描述

创建一个数据分析脚本，计算销售数据的统计信息。

### CLI 命令

```bash
openhands run "创建一个数据分析脚本 sales_analyzer.py：
1. 读取 sales.csv（日期、产品、销售额）
2. 计算总销售额
3. 计算每日平均销售额
4. 找出最畅销产品
5. 生成可视化图表（使用 matplotlib）
6. 导出报告到 report.txt"
```

### 预期结果

**sales_analyzer.py**：
```python
import pandas as pd
import matplotlib.pyplot as plt
from typing import Dict

def load_data(file_path: str) -> pd.DataFrame:
    """加载销售数据"""
    df = pd.read_csv(file_path)
    df['date'] = pd.to_datetime(df['date'])
    return df

def calculate_stats(df: pd.DataFrame) -> Dict:
    """计算统计信息"""
    stats = {
        'total_sales': df['sales'].sum(),
        'avg_daily_sales': df.groupby('date')['sales'].sum().mean(),
        'top_product': df.groupby('product')['sales'].sum().idxmax(),
        'total_records': len(df)
    }
    return stats

def visualize_data(df: pd.DataFrame, output_path: str = 'sales_chart.png'):
    """可视化数据"""
    daily_sales = df.groupby('date')['sales'].sum()
    
    plt.figure(figsize=(10, 6))
    daily_sales.plot(kind='bar')
    plt.title('Daily Sales')
    plt.xlabel('Date')
    plt.ylabel('Sales ($)')
    plt.tight_layout()
    plt.savefig(output_path)
    plt.close()

def generate_report(stats: Dict, output_path: str = 'report.txt'):
    """生成报告"""
    with open(output_path, 'w') as f:
        f.write("=== Sales Report ===\n\n")
        f.write(f"Total Sales: ${stats['total_sales']:,.2f}\n")
        f.write(f"Average Daily Sales: ${stats['avg_daily_sales']:,.2f}\n")
        f.write(f"Top Product: {stats['top_product']}\n")
        f.write(f"Total Records: {stats['total_records']}\n")

if __name__ == "__main__":
    df = load_data("sales.csv")
    stats = calculate_stats(df)
    visualize_data(df)
    generate_report(stats)
    print("报告已生成！")
```

### 学习要点

- ✅ Pandas 数据分析
- ✅ 数据可视化
- ✅ 报告生成
- ✅ 类型注解

---

## 案例 4：单元测试生成

### 任务描述

为现有代码生成完整的单元测试。

### CLI 命令

```bash
openhands run "为 calculator.py 生成单元测试：
1. 测试所有公共方法
2. 包含边界条件测试
3. 包含异常测试
4. 测试覆盖率 > 90%"
```

### 预期结果

**calculator.py**：
```python
class Calculator:
    def add(self, a: float, b: float) -> float:
        return a + b
    
    def divide(self, a: float, b: float) -> float:
        if b == 0:
            raise ValueError("Cannot divide by zero")
        return a / b
```

**test_calculator.py**：
```python
import pytest
from calculator import Calculator

class TestCalculator:
    def setup_method(self):
        self.calc = Calculator()
    
    def test_add_positive_numbers(self):
        assert self.calc.add(2, 3) == 5
    
    def test_add_negative_numbers(self):
        assert self.calc.add(-1, -1) == -2
    
    def test_add_zero(self):
        assert self.calc.add(5, 0) == 5
    
    def test_divide_normal(self):
        assert self.calc.divide(10, 2) == 5
    
    def test_divide_by_zero(self):
        with pytest.raises(ValueError):
            self.calc.divide(10, 0)
    
    def test_divide_negative(self):
        assert self.calc.divide(-10, 2) == -5
```

### 学习要点

- ✅ pytest 使用
- ✅ 边界条件测试
- ✅ 异常测试
- ✅ 测试覆盖率

---

## 案例 5：CLI 工具开发

### 任务描述

创建一个命令行工具，管理待办事项。

### CLI 命令

```bash
openhands run "创建一个 CLI 待办事项工具 todo.py：
1. 支持命令：add、list、complete、delete
2. 数据存储在 JSON 文件
3. 使用 argparse 解析参数
4. 添加颜色输出
5. 支持优先级（高/中/低）"
```

### 预期结果

**todo.py**：
```python
import argparse
import json
from datetime import datetime
from typing import List, Dict
import os

TODO_FILE = "todos.json"

def load_todos() -> List[Dict]:
    """加载待办事项"""
    if not os.path.exists(TODO_FILE):
        return []
    with open(TODO_FILE, 'r') as f:
        return json.load(f)

def save_todos(todos: List[Dict]):
    """保存待办事项"""
    with open(TODO_FILE, 'w') as f:
        json.dump(todos, f, indent=2)

def add_todo(task: str, priority: str = "medium"):
    """添加待办事项"""
    todos = load_todos()
    todo = {
        "id": len(todos) + 1,
        "task": task,
        "priority": priority,
        "completed": False,
        "created_at": datetime.now().isoformat()
    }
    todos.append(todo)
    save_todos(todos)
    print(f"✅ Added: {task}")

def list_todos():
    """列出所有待办事项"""
    todos = load_todos()
    if not todos:
        print("No todos found.")
        return
    
    for todo in todos:
        status = "✓" if todo["completed"] else " "
        priority_color = {"high": "🔴", "medium": "🟡", "low": "🟢"}
        color = priority_color.get(todo["priority"], "⚪")
        print(f"[{status}] {todo['id']}. {color} {todo['task']}")

def complete_todo(todo_id: int):
    """完成待办事项"""
    todos = load_todos()
    for todo in todos:
        if todo["id"] == todo_id:
            todo["completed"] = True
            save_todos(todos)
            print(f"✅ Completed: {todo['task']}")
            return
    print(f"❌ Todo {todo_id} not found")

def delete_todo(todo_id: int):
    """删除待办事项"""
    todos = load_todos()
    todos = [t for t in todos if t["id"] != todo_id]
    save_todos(todos)
    print(f"🗑️  Deleted todo {todo_id}")

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Todo CLI Tool")
    subparsers = parser.add_subparsers(dest="command", help="Commands")
    
    # Add command
    add_parser = subparsers.add_parser("add", help="Add a todo")
    add_parser.add_argument("task", help="Task description")
    add_parser.add_argument("--priority", "-p", default="medium", 
                           choices=["high", "medium", "low"])
    
    # List command
    subparsers.add_parser("list", help="List all todos")
    
    # Complete command
    complete_parser = subparsers.add_parser("complete", help="Complete a todo")
    complete_parser.add_argument("id", type=int, help="Todo ID")
    
    # Delete command
    delete_parser = subparsers.add_parser("delete", help="Delete a todo")
    delete_parser.add_argument("id", type=int, help="Todo ID")
    
    args = parser.parse_args()
    
    if args.command == "add":
        add_todo(args.task, args.priority)
    elif args.command == "list":
        list_todos()
    elif args.command == "complete":
        complete_todo(args.id)
    elif args.command == "delete":
        delete_todo(args.id)
    else:
        parser.print_help()
```

### 使用示例

```bash
# 添加任务
python todo.py add "学习 OpenHands" --priority high

# 列出所有任务
python todo.py list

# 完成任务
python todo.py complete 1

# 删除任务
python todo.py delete 2
```

### 学习要点

- ✅ argparse 使用
- ✅ JSON 数据存储
- ✅ CLI 交互设计
- ✅ 类型注解

---

## 🎓 总结

通过这 5 个基础案例，你已经掌握了：

- ✅ FastAPI 基础开发
- ✅ 文件处理
- ✅ 数据分析
- ✅ 单元测试
- ✅ CLI 工具开发

### 下一步

- 🎯 [Web 开发案例](../web-development/)
- 🎯 [数据分析案例](../data-analysis/)
- 🎯 [自动化案例](../automation/)

---

<div align="center">
  <p>🎉 继续探索更多案例！</p>
</div>
