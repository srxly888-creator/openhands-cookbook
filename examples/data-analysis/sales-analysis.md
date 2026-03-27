# 数据分析案例集

> 使用 OpenHands 进行数据分析和可视化

---

## 📋 案例列表

- [案例 1：销售数据分析](#案例-1销售数据分析)
- [案例 2：用户行为分析](#案例-2用户行为分析)
- [案例 3：财务数据可视化](#案例-3财务数据可视化)
- [案例 4：A/B 测试分析](#案例-4ab-测试分析)
- [案例 5：时间序列预测](#案例-5时间序列预测)

---

## 案例 1：销售数据分析

### 任务描述

分析销售数据，生成报告和可视化图表。

### 实现方案

**Python 脚本**：
```python
from openhands import Agent
import pandas as pd
import matplotlib.pyplot as plt

def analyze_sales(data_file):
    """销售数据分析"""
    
    agent = Agent(model="claude-3.5-sonnet")
    
    # 1. 数据加载和清洗
    result = agent.run(f"""
    分析销售数据：

    数据文件：{data_file}

    任务：
    1. 加载数据
    2. 数据清洗（处理缺失值、异常值）
    3. 计算统计指标：
       - 总销售额
       - 平均订单金额
       - 销售趋势
       - 最畅销产品
    4. 生成可视化图表
    5. 生成分析报告
    """)
    
    # 2. 执行分析代码
    exec_code = result.output
    
    # 保存分析脚本
    with open("sales_analysis.py", 'w') as f:
        f.write(exec_code)
    
    # 运行分析
    agent.execute_command("python sales_analysis.py")
    
    print("✅ 销售数据分析完成")
    print("📊 报告：sales_report.txt")
    print("📈 图表：sales_charts/")

# 使用
analyze_sales("sales_data.csv")
```

**生成的分析代码**：
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# 加载数据
df = pd.read_csv('sales_data.csv')

# 数据清洗
df['date'] = pd.to_datetime(df['date'])
df = df.dropna()

# 统计分析
total_sales = df['sales'].sum()
avg_order = df['sales'].mean()
top_products = df.groupby('product')['sales'].sum().sort_values(ascending=False).head(10)

# 可视化
plt.figure(figsize=(12, 6))
plt.subplot(1, 2, 1)
df.groupby('date')['sales'].sum().plot()
plt.title('销售趋势')
plt.xlabel('日期')
plt.ylabel('销售额')

plt.subplot(1, 2, 2)
top_products.plot(kind='bar')
plt.title('最畅销产品 Top 10')
plt.xlabel('产品')
plt.ylabel('销售额')

plt.tight_layout()
plt.savefig('sales_analysis.png')
plt.close()

# 生成报告
with open('sales_report.txt', 'w') as f:
    f.write(f"销售分析报告\n")
    f.write(f"总销售额: ${total_sales:,.2f}\n")
    f.write(f"平均订单金额: ${avg_order:,.2f}\n")
    f.write(f"\n最畅销产品:\n")
    for product, sales in top_products.items():
        f.write(f"  {product}: ${sales:,.2f}\n")
```

### 学习要点

- ✅ Pandas 数据分析
- ✅ 数据清洗
- ✅ 统计分析
- ✅ 数据可视化

---

## 案例 2：用户行为分析

### 任务描述

分析用户行为数据，生成用户画像。

### 实现方案

**Python 脚本**：
```python
from openhands import Agent

def analyze_user_behavior(log_file):
    """用户行为分析"""
    
    agent = Agent(model="claude-3.5-sonnet")
    
    result = agent.run(f"""
    分析用户行为数据：

    日志文件：{log_file}

    分析内容：
    1. 用户访问模式
    2. 页面停留时间
    3. 转化漏斗分析
    4. 用户分群
    5. 生成用户画像
    """)
    
    print("✅ 用户行为分析完成")

# 使用
analyze_user_behavior("user_logs.csv")
```

### 学习要点

- ✅ 用户行为分析
- ✅ 漏斗分析
- ✅ 用户分群
- ✅ 用户画像

---

## 案例 3：财务数据可视化

### 任务描述

可视化财务数据，生成财务报表。

### 实现方案

**Python 脚本**：
```python
from openhands import Agent

def visualize_financial_data(data_file):
    """财务数据可视化"""
    
    agent = Agent(model="claude-3.5-sonnet")
    
    result = agent.run(f"""
    可视化财务数据：

    数据文件：{data_file}

    包含：
    1. 收入支出趋势图
    2. 利润分析图
    3. 成本结构饼图
    4. 现金流图
    5. 财务报表
    """)
    
    print("✅ 财务数据可视化完成")

# 使用
visualize_financial_data("financial_data.xlsx")
```

### 学习要点

- ✅ 财务数据分析
- ✅ 数据可视化
- ✅ 报表生成
- ✅ Excel 处理

---

## 案例 4：A/B 测试分析

### 任务描述

分析 A/B 测试结果，确定最佳方案。

### 实现方案

**Python 脚本**：
```python
from openhands import Agent

def ab_test_analysis(test_data):
    """A/B 测试分析"""
    
    agent = Agent(model="claude-3.5-sonnet")
    
    result = agent.run(f"""
    分析 A/B 测试数据：

    数据：{test_data}

    分析：
    1. 统计显著性检验
    2. 效果大小计算
    3. 置信区间
    4. 推荐方案
    5. 生成报告
    """)
    
    print("✅ A/B 测试分析完成")

# 使用
ab_test_analysis("ab_test_results.csv")
```

### 学习要点

- ✅ 统计检验
- ✅ A/B 测试
- ✅ 数据分析
- ✅ 决策支持

---

## 案例 5：时间序列预测

### 任务描述

预测未来销售趋势。

### 实现方案

**Python 脚本**：
```python
from openhands import Agent

def time_series_forecast(data_file):
    """时间序列预测"""
    
    agent = Agent(model="claude-3.5-sonnet")
    
    result = agent.run(f"""
    时间序列预测：

    数据文件：{data_file}

    预测：
    1. 未来 30 天销售预测
    2. 趋势分析
    3. 季节性分析
    4. 置信区间
    5. 可视化
    """) 
    
    print("✅ 时间序列预测完成")

# 使用
time_series_forecast("sales_history.csv")
```

### 学习要点

- ✅ 时间序列分析
- ✅ 预测建模
- ✅ 趋势分析
- ✅ 季节性分析

---

## 📚 相关资源

- [Pandas 官方文档](https://pandas.pydata.org)
- [Matplotlib 官方文档](https://matplotlib.org)
- [Seaborn 官方文档](https://seaborn.pydata.org)
- [数据分析最佳实践](https://docs.openhands.dev/best-practices/data-analysis)

---

<div align="center">
  <p>🎉 用 AI 助力数据分析！</p>
</div>
