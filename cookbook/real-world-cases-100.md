# OpenHands 实战案例 100+

> 100+ 真实场景案例

---

## 📋 目录

- [Web 开发案例](#web-开发案例)
- [数据分析案例](#数据分析案例)
- [AI/ML 案例](#aiml-案例)
- [自动化案例](#自动化案例)

---

## 🌐 Web 开发案例

### 案例 1：电商网站

**场景**: 创建完整的电商网站

**任务描述**:
```
创建电商网站，包含：
1. 用户系统（注册/登录/个人中心）
2. 商品系统（分类/搜索/详情）
3. 购物车（添加/删除/修改）
4. 订单系统（下单/支付/物流）
5. 后台管理（商品/订单/用户）
```

**技术栈**:
- 后端：FastAPI + PostgreSQL
- 前端：React + Tailwind CSS
- 支付：支付宝/微信支付
- 部署：Docker + Kubernetes

**实现代码**:
```python
# models/user.py
from sqlalchemy import Column, String, Boolean, DateTime
from sqlalchemy.sql import func
from database import Base

class User(Base):
    __tablename__ = "users"
    
    id = Column(String, primary_key=True)
    email = Column(String, unique=True, index=True)
    username = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)
    is_admin = Column(Boolean, default=False)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())

# routers/auth.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from database import get_db
from models.user import User
from auth import verify_password, create_access_token

router = APIRouter()

@router.post("/register")
async def register(
    email: str,
    username: str,
    password: str,
    db: Session = Depends(get_db)
):
    """用户注册"""
    # 检查邮箱是否存在
    if db.query(User).filter(User.email == email).first():
        raise HTTPException(status_code=400, detail="Email already registered")
    
    # 检查用户名是否存在
    if db.query(User).filter(User.username == username).first():
        raise HTTPException(status_code=400, detail="Username already taken")
    
    # 创建用户
    user = User(
        id=str(uuid.uuid4()),
        email=email,
        username=username,
        hashed_password=hash_password(password)
    )
    
    db.add(user)
    db.commit()
    
    return {"message": "User created successfully"}

@router.post("/login")
async def login(
    username: str,
    password: str,
    db: Session = Depends(get_db)
):
    """用户登录"""
    user = db.query(User).filter(User.username == username).first()
    
    if not user or not verify_password(password, user.hashed_password):
        raise HTTPException(status_code=401, detail="Invalid credentials")
    
    # 创建访问令牌
    access_token = create_access_token(data={"sub": user.id})
    
    return {"access_token": access_token, "token_type": "bearer"}
```

---

### 案例 2：博客系统

**场景**: 创建多用户博客系统

**任务描述**:
```
创建博客系统，包含：
1. 用户系统
2. 文章管理（发布/编辑/删除）
3. 评论系统
4. 标签分类
5. 搜索功能
```

**技术栈**:
- 后端：Django + PostgreSQL
- 前端：Vue.js
- 搜索：Elasticsearch
- 缓存：Redis

---

### 案例 3：在线教育平台

**场景**: 创建在线教育平台

**任务描述**:
```
创建在线教育平台，包含：
1. 课程管理
2. 视频播放
3. 作业系统
4. 考试系统
5. 学习进度
```

**技术栈**:
- 后端：NestJS + MongoDB
- 前端：React + Redux
- 视频：阿里云 OSS
- 直播：声网

---

## 📊 数据分析案例

### 案例 4：销售数据分析

**场景**: 分析销售数据

**任务描述**:
```
分析销售数据，包含：
1. 数据清洗
2. 探索性分析
3. 可视化
4. 趋势预测
5. 报告生成
```

**实现代码**:
```python
# analysis/sales_analysis.py
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

class SalesAnalyzer:
    def __init__(self, data_path):
        self.df = pd.read_csv(data_path)
    
    def clean_data(self):
        """数据清洗"""
        # 删除缺失值
        self.df = self.df.dropna()
        
        # 转换日期
        self.df['date'] = pd.to_datetime(self.df['date'])
        
        # 删除异常值
        self.df = self.df[self.df['sales'] > 0]
        
        return self.df
    
    def exploratory_analysis(self):
        """探索性分析"""
        # 统计摘要
        print(self.df.describe())
        
        # 相关性分析
        print(self.df.corr())
        
        # 分组统计
        print(self.df.groupby('category')['sales'].sum())
    
    def visualize(self):
        """可视化"""
        # 销售趋势
        plt.figure(figsize=(12, 6))
        self.df.groupby('date')['sales'].sum().plot()
        plt.title('Sales Trend')
        plt.xlabel('Date')
        plt.ylabel('Sales')
        plt.savefig('sales_trend.png')
        
        # 分类销售
        plt.figure(figsize=(10, 6))
        self.df.groupby('category')['sales'].sum().plot(kind='bar')
        plt.title('Sales by Category')
        plt.xlabel('Category')
        plt.ylabel('Sales')
        plt.savefig('sales_by_category.png')
    
    def generate_report(self):
        """生成报告"""
        report = f"""
        销售数据分析报告
        ================
        
        总销售额: ${self.df['sales'].sum():,.2f}
        平均订单: ${self.df['sales'].mean():,.2f}
        最高订单: ${self.df['sales'].max():,.2f}
        最低订单: ${self.df['sales'].min():,.2f}
        
        销售趋势:
        - 整体呈上升趋势
        - 周末销售较高
        - 节假日有显著提升
        
        建议:
        - 增加周末促销活动
        - 优化节假日营销策略
        - 关注低销售分类
        """
        
        with open('sales_report.md', 'w') as f:
            f.write(report)

# 使用
analyzer = SalesAnalyzer('sales.csv')
analyzer.clean_data()
analyzer.exploratory_analysis()
analyzer.visualize()
analyzer.generate_report()
```

---

### 案例 5：用户行为分析

**场景**: 分析用户行为

**任务描述**:
```
分析用户行为，包含：
1. 用户分群
2. 行为路径
3. 留存分析
4. 转化漏斗
```

**技术栈**:
- Python + Pandas
- Scikit-learn
- Plotly

---

## 🤖 AI/ML 案例

### 案例 6：推荐系统

**场景**: 商品推荐系统

**任务描述**:
```
创建推荐系统，包含：
1. 协同过滤
2. 内容过滤
3. 混合推荐
4. 实时推荐
```

**实现代码**:
```python
# ml/recommender.py
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

class CollaborativeFiltering:
    def __init__(self, ratings_matrix):
        self.ratings = ratings_matrix
        self.similarity = cosine_similarity(ratings_matrix)
    
    def recommend(self, user_id, k=10):
        """推荐商品"""
        # 计算用户相似度
        user_sim = self.similarity[user_id]
        
        # 找到相似用户
        similar_users = np.argsort(user_sim)[::-1][1:k+1]
        
        # 获取推荐商品
        recommendations = []
        for similar_user in similar_users:
            # 找到该用户喜欢但目标用户未购买的商品
            user_items = self.ratings[user_id]
            similar_items = self.ratings[similar_user]
            
            # 未购买且相似用户评分高的商品
            new_items = np.where((user_items == 0) & (similar_items > 3))[0]
            
            recommendations.extend(new_items)
        
        return list(set(recommendations))[:k]

# 使用
ratings = np.array([
    [5, 3, 0, 1],
    [4, 0, 0, 1],
    [1, 1, 0, 5],
    [1, 0, 0, 4],
    [0, 1, 5, 4],
])

cf = CollaborativeFiltering(ratings)
recommendations = cf.recommend(user_id=0, k=5)
print(f"Recommended items: {recommendations}")
```

---

### 案例 7：情感分析

**场景**: 评论情感分析

**任务描述**:
```
情感分析系统，包含：
1. 文本预处理
2. 特征提取
3. 模型训练
4. 预测分析
```

**技术栈**:
- NLTK + TextBlob
- Transformers
- PyTorch

---

## 🔄 自动化案例

### 案例 8：自动化测试

**场景**: 自动化测试系统

**任务描述**:
```
创建自动化测试系统，包含：
1. 单元测试
2. 集成测试
3. E2E 测试
4. 性能测试
```

**实现代码**:
```python
# tests/test_api.py
import pytest
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

class TestAPI:
    def test_health_check(self):
        """测试健康检查"""
        response = client.get("/health")
        assert response.status_code == 200
        assert response.json()["status"] == "ok"
    
    def test_create_user(self):
        """测试创建用户"""
        response = client.post(
            "/api/users",
            json={
                "email": "test@example.com",
                "username": "testuser",
                "password": "password123"
            }
        )
        assert response.status_code == 201
        assert response.json()["email"] == "test@example.com"
    
    def test_login(self):
        """测试登录"""
        # 先创建用户
        client.post(
            "/api/users",
            json={
                "email": "test2@example.com",
                "username": "testuser2",
                "password": "password123"
            }
        )
        
        # 登录
        response = client.post(
            "/api/auth/login",
            json={
                "username": "testuser2",
                "password": "password123"
            }
        )
        assert response.status_code == 200
        assert "access_token" in response.json()

# 运行测试
if __name__ == "__main__":
    pytest.main([__file__, "-v"])
```

---

### 案例 9：定时任务

**场景**: 定时任务系统

**任务描述**:
```
创建定时任务系统，包含：
1. 定时执行
2. 任务调度
3. 错误处理
4. 日志记录
```

**技术栈**:
- Celery + Redis
- APScheduler
- Cron

---

### 案例 10：数据处理流水线

**场景**: ETL 流水线

**任务描述**:
```
创建 ETL 流水线，包含：
1. 数据提取
2. 数据转换
3. 数据加载
4. 监控告警
```

**实现代码**:
```python
# etl/pipeline.py
from datetime import datetime
import pandas as pd

class ETLPipeline:
    def __init__(self):
        self.data = None
    
    def extract(self, source):
        """提取数据"""
        print(f"[{datetime.now()}] Extracting data from {source}")
        
        if source.endswith('.csv'):
            self.data = pd.read_csv(source)
        elif source.endswith('.json'):
            self.data = pd.read_json(source)
        else:
            raise ValueError(f"Unsupported source: {source}")
        
        return self
    
    def transform(self):
        """转换数据"""
        print(f"[{datetime.now()}] Transforming data")
        
        # 数据清洗
        self.data = self.data.dropna()
        
        # 数据转换
        self.data['created_at'] = pd.to_datetime(self.data['created_at'])
        self.data['updated_at'] = datetime.now()
        
        return self
    
    def load(self, destination):
        """加载数据"""
        print(f"[{datetime.now()}] Loading data to {destination}")
        
        if destination.endswith('.csv'):
            self.data.to_csv(destination, index=False)
        elif destination.endswith('.json'):
            self.data.to_json(destination, orient='records')
        else:
            raise ValueError(f"Unsupported destination: {destination}")
        
        print(f"[{datetime.now()}] ETL completed successfully")
        return self
    
    def run(self, source, destination):
        """运行 ETL 流水线"""
        try:
            self.extract(source).transform().load(destination)
            return True
        except Exception as e:
            print(f"[{datetime.now()}] ETL failed: {e}")
            return False

# 使用
pipeline = ETLPipeline()
pipeline.run('data/source.csv', 'data/destination.csv')
```

---

## 📚 更多案例（100+）

### Web 开发（11-30）

11. 社交网络
12. 在线商城
13. 内容管理系统
14. 论坛系统
15. 预约系统
16. 在线支付
17. 即时通讯
18. 视频网站
19. 音乐平台
20. 新闻门户
21. 天气应用
22. 地图服务
23. 在线文档
24. 项目管理
25. CRM 系统
26. ERP 系统
27. HR 系统
28. 财务系统
29. 库存管理
30. 物流追踪

### 数据分析（31-50）

31. 用户画像
32. 销售预测
33. 库存优化
34. 价格分析
35. A/B 测试
36. 归因分析
37. 路径分析
38. 漏斗分析
39. 留存分析
40. 活跃度分析
41. RFM 分析
42. 同期群分析
43. 趋势分析
44. 异常检测
45. 时间序列
46. 文本挖掘
47. 社交网络分析
48. 地理分析
49. 竞品分析
50. 市场分析

### AI/ML（51-70）

51. 图像识别
52. 语音识别
53. 文本分类
54. 实体识别
55. 机器翻译
56. 问答系统
57. 对话系统
58. 知识图谱
59. 目标检测
60. 图像分割
61. 人脸识别
62. OCR 识别
63. 语音合成
64. 视频分析
65. 异常检测
66. 欺诈检测
67. 风险评估
68. 价格预测
69. 需求预测
70. 推荐系统

### 自动化（71-90）

71. 自动部署
72. 自动测试
73. 自动监控
74. 自动备份
75. 自动报告
76. 自动爬虫
77. 自动邮件
78. 自动消息
79. 自动同步
80. 自动归档
81. 自动清理
82. 自动重启
83. 自动扩容
84. 自动修复
85. 自动优化
86. 自动审查
87. 自动发布
88. 自动回滚
89. 自动告警
90. 自动恢复

### 其他（91-100）

91. 微服务架构
92. 事件驱动
93. CQRS 模式
94. 事件溯源
95. 分布式锁
96. 分布式事务
97. 消息队列
98. 服务网格
99. API 网关
100. 服务发现

---

<div align="center">
  <p>🎯 100+ 实战案例，快速上手！</p>
</div>
