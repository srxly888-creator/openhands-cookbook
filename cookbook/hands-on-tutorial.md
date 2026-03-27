# OpenHands 实战教程 - 从零开始

> 2026 年最新实战教程，手把手教你使用 OpenHands

---

## 📋 教程目录

- [第 1 课：环境准备](#第-1-课环境准备)
- [第 2 课：第一个任务](#第-2-课第一个任务)
- [第 3 课：进阶技巧](#第-3-课进阶技巧)
- [第 4 课：实战项目](#第-4-课实战项目)
- [第 5 课：最佳实践](#第-5-课最佳实践)

---

## 🎓 第 1 课：环境准备

### 1.1 安装 OpenHands

**CLI 模式**（推荐新手）:
```bash
# 安装
pip install openhands

# 验证
openhands --version

# 配置 API Key
openhands config set api-key sk-ant-xxx
```

**GUI 模式**:
```bash
# 克隆仓库
git clone https://github.com/OpenHands/OpenHands.git
cd OpenHands

# 启动
make build
make run

# 访问
open http://localhost:3000
```

**Cloud 模式**（最简单）:
```
1. 访问 https://app.all-hands.dev
2. 使用 GitHub 登录
3. 开始使用
```

---

## 🎯 第 2 课：第一个任务

### 2.1 创建 Hello World

**任务描述**:
```
创建一个 Python 程序，打印 "Hello, World!"
```

**执行**:
```bash
openhands run "创建一个 Python 程序，打印 'Hello, World!'"
```

**预期结果**:
```python
# hello.py
def main():
    print("Hello, World!")

if __name__ == "__main__":
    main()
```

---

### 2.2 创建 FastAPI 项目

**任务描述**:
```
创建一个 FastAPI 项目，包含：
1. GET / 返回 {"message": "Hello"}
2. GET /health 返回 {"status": "ok"}
3. requirements.txt
4. README.md
```

**执行**:
```bash
openhands run "创建 FastAPI 项目，包含基础路由和文档"
```

**预期结果**:
```
my-api/
├── main.py
├── requirements.txt
└── README.md
```

**运行**:
```bash
pip install -r requirements.txt
uvicorn main:app --reload
```

---

## 🚀 第 3 课：进阶技巧

### 3.1 任务分解

**大任务** → **小任务**

**原始任务**:
```
创建一个完整的电商系统
```

**分解后**:
```bash
# 1. 创建项目结构
openhands run "创建电商项目目录结构"

# 2. 实现用户认证
openhands run "实现用户注册和登录"

# 3. 实现商品管理
openhands run "实现商品的 CRUD 操作"

# 4. 实现订单系统
openhands run "实现订单创建和查询"

# 5. 集成支付
openhands run "集成支付宝支付"
```

---

### 3.2 使用模板

**创建模板**:
```yaml
# .openhands/templates/api.yaml
name: API 项目模板
description: 创建标准 API 项目
prompt: |
  创建 {framework} API 项目：
  
  1. 框架：{framework}
  2. 数据库：{database}
  3. 认证：{auth}
  4. 文档：{docs}
  
  包含：
  - 项目结构
  - 基础路由
  - 数据库连接
  - API 文档
  - 单元测试
```

**使用模板**:
```bash
openhands run --template api.yaml \
  --param framework=FastAPI \
  --param database=PostgreSQL \
  --param auth=JWT \
  --param docs=OpenAPI
```

---

### 3.3 错误处理

**自动重试**:
```python
from openhands import Agent

agent = Agent(
    model="claude-3.5-sonnet",
    max_retries=3,
    retry_delay=5
)

try:
    result = agent.run("创建项目")
except Exception as e:
    print(f"任务失败: {e}")
    # 降级到简单任务
    result = agent.run("创建简单项目")
```

---

## 💼 第 4 课：实战项目

### 4.1 项目：待办事项 API

**需求**:
```
创建待办事项 API：
1. 用户注册/登录
2. 创建待办
3. 查询待办
4. 更新待办
5. 删除待办
```

**执行**:
```bash
openhands run "创建待办事项 API，包含用户认证和 CRUD 操作"
```

**预期结果**:
```
todo-api/
├── main.py              # FastAPI 应用
├── models.py            # 数据模型
├── auth.py              # 认证模块
├── requirements.txt     # 依赖
├── tests/               # 测试
│   └── test_api.py
└── README.md            # 文档
```

**测试**:
```bash
# 运行测试
pytest tests/

# 测试覆盖率 > 90%
pytest --cov=.
```

---

### 4.2 项目：数据分析脚本

**需求**:
```
分析销售数据：
1. 读取 CSV 文件
2. 数据清洗
3. 统计分析
4. 可视化
5. 生成报告
```

**执行**:
```bash
openhands run "创建销售数据分析脚本，包含数据清洗和可视化"
```

**预期结果**:
```python
# analyze_sales.py
import pandas as pd
import matplotlib.pyplot as plt

def analyze_sales(csv_file):
    """分析销售数据"""
    # 1. 读取数据
    df = pd.read_csv(csv_file)
    
    # 2. 数据清洗
    df = df.dropna()
    
    # 3. 统计分析
    total_sales = df['sales'].sum()
    avg_order = df['sales'].mean()
    
    # 4. 可视化
    df.groupby('date')['sales'].sum().plot()
    plt.savefig('sales_trend.png')
    
    # 5. 生成报告
    report = f"""
    销售分析报告
    ━━━━━━━━━━━━
    总销售额：${total_sales:,.2f}
    平均订单：${avg_order:,.2f}
    """
    
    return report
```

---

### 4.3 项目：自动化测试

**需求**:
```
为现有代码生成单元测试：
1. 测试覆盖率 > 90%
2. 包含边界测试
3. 包含异常测试
```

**执行**:
```bash
openhands run "为 calculator.py 生成完整的单元测试"
```

**预期结果**:
```python
# test_calculator.py
import pytest
from calculator import Calculator

class TestCalculator:
    def setup_method(self):
        self.calc = Calculator()
    
    def test_add_positive(self):
        assert self.calc.add(2, 3) == 5
    
    def test_add_negative(self):
        assert self.calc.add(-1, -2) == -3
    
    def test_add_zero(self):
        assert self.calc.add(5, 0) == 5
    
    def test_divide_by_zero(self):
        with pytest.raises(ValueError):
            self.calc.divide(10, 0)
    
    def test_divide_normal(self):
        assert self.calc.divide(10, 2) == 5
```

---

## 🏆 第 5 课：最佳实践

### 5.1 任务描述技巧

**❌ 不好的描述**:
```
优化代码
```

**✅ 好的描述**:
```
优化 main.py 中的数据处理函数：

1. 减少内存占用（目标：<100MB）
2. 提高处理速度（目标：<1秒）
3. 保持代码可读性
4. 添加单元测试
5. 更新文档
```

---

### 5.2 代码质量保证

**配置质量检查**:
```yaml
# .openhands/quality.yaml
checks:
  - name: lint
    command: npm run lint
  
  - name: test
    command: npm test
  
  - name: coverage
    command: npm run test:coverage
    threshold: 80
  
  - name: security
    command: npm audit
```

**执行质量检查**:
```bash
openhands run --quality-check "创建项目"
```

---

### 5.3 性能优化

**监控性能**:
```python
from openhands import Agent, Monitor

monitor = Monitor()
agent = Agent(model="claude-3.5-sonnet", monitor=monitor)

result = agent.run("创建项目")

# 查看性能报告
print(monitor.get_report())
```

**输出**:
```
性能报告
━━━━━━━━━━━━━━━━━━━━━━
执行时间：2.5 秒
Token 使用：1,234
成本：$0.015
━━━━━━━━━━━━━━━━━━━━━━
```

---

### 5.4 团队协作

**Git 集成**:
```bash
# 自动提交
openhands run --git-commit "实现用户登录功能"

# 自动创建 PR
openhands run --git-pr "添加用户认证"
```

**代码审查**:
```bash
# 自动审查 PR
openhands run --review-pr 123
```

---

## 📊 学习检查清单

### ✅ 基础（第 1-2 课）

- [ ] 安装 OpenHands
- [ ] 配置 API Key
- [ ] 完成第一个任务
- [ ] 创建 FastAPI 项目

### ✅ 进阶（第 3 课）

- [ ] 学会任务分解
- [ ] 使用模板
- [ ] 处理错误

### ✅ 实战（第 4 课）

- [ ] 完成 3 个实战项目
- [ ] 理解测试覆盖率
- [ ] 学会性能监控

### ✅ 最佳实践（第 5 课）

- [ ] 掌握任务描述技巧
- [ ] 配置质量检查
- [ ] 集成 Git 工作流

---

## 🎓 结业证书

完成以上所有课程后，你将获得：

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    OpenHands 实战教程
       结业证书
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

兹证明：

  [你的名字]

已成功完成 OpenHands 实战教程全部课程

课程内容：
✅ 环境准备
✅ 基础任务
✅ 进阶技巧
✅ 实战项目
✅ 最佳实践

颁发日期：2026-03-27

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 📚 扩展学习

### 进阶课程

1. **OpenHands SDK 编程** - Python API 使用
2. **多 Agent 协作** - 复杂任务处理
3. **企业部署** - 生产环境部署
4. **性能优化** - 提升效率技巧

### 社区资源

- **GitHub**: https://github.com/OpenHands/OpenHands
- **Discord**: https://discord.gg/openhands
- **文档**: https://docs.openhands.dev

---

<div align="center">
  <p>🎓 恭喜你完成 OpenHands 实战教程！</p>
  <p>🚀 开始你的 AI 驱动开发之旅！</p>
</div>
