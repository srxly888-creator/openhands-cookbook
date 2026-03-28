# OpenHands 数据处理完全指南

> 数据处理和分析

---

## 📋 目录

- [数据采集](#数据采集)
- [数据清洗](#数据清洗)
- [数据分析](#数据分析)
- [数据可视化](#数据可视化)

---

## 📥 数据采集

### 数据源集成

```python
# data/collection.py
import pandas as pd
from typing import Dict, List
import aiohttp
import asyncio

class DataCollector:
    """数据采集器"""
    
    def __init__(self):
        self.session = None
    
    async def collect_from_api(self, url: str, params: Dict = None) -> Dict:
        """从 API 采集数据"""
        if not self.session:
            self.session = aiohttp.ClientSession()
        
        async with self.session.get(url, params=params) as response:
            return await response.json()
    
    async def collect_from_database(self, query: str, connection: Dict) -> pd.DataFrame:
        """从数据库采集数据"""
        import sqlalchemy
        
        engine = sqlalchemy.create_engine(
            f"postgresql://{connection['user']}:{connection['password']}"
            f"@{connection['host']}:{connection['port']}/{connection['database']}"
        )
        
        return pd.read_sql(query, engine)
    
    def collect_from_csv(self, file_path: str) -> pd.DataFrame:
        """从 CSV 采集数据"""
        return pd.read_csv(file_path)
    
    def collect_from_json(self, file_path: str) -> pd.DataFrame:
        """从 JSON 采集数据"""
        return pd.read_json(file_path)
    
    async def collect_from_multiple_sources(
        self,
        sources: List[Dict]
    ) -> Dict[str, pd.DataFrame]:
        """从多个源采集数据"""
        results = {}
        
        for source in sources:
            name = source['name']
            type_ = source['type']
            
            if type_ == 'api':
                data = await self.collect_from_api(
                    source['url'],
                    source.get('params')
                )
                results[name] = pd.DataFrame(data)
            
            elif type_ == 'database':
                results[name] = await self.collect_from_database(
                    source['query'],
                    source['connection']
                )
            
            elif type_ == 'csv':
                results[name] = self.collect_from_csv(source['path'])
            
            elif type_ == 'json':
                results[name] = self.collect_from_json(source['path'])
        
        return results

# 使用
collector = DataCollector()

# 从 API 采集
data = await collector.collect_from_api(
    "https://api.example.com/data",
    params={"limit": 100}
)

# 从数据库采集
df = await collector.collect_from_database(
    "SELECT * FROM users",
    {
        'host': 'localhost',
        'port': 5432,
        'user': 'user',
        'password': 'password',
        'database': 'db'
    }
)

# 从多个源采集
sources = [
    {
        'name': 'users',
        'type': 'database',
        'query': 'SELECT * FROM users',
        'connection': {...}
    },
    {
        'name': 'orders',
        'type': 'api',
        'url': 'https://api.example.com/orders'
    },
    {
        'name': 'products',
        'type': 'csv',
        'path': '/data/products.csv'
    }
]

results = await collector.collect_from_multiple_sources(sources)
```

---

## 🧹 数据清洗

### 数据清洗器

```python
# data/cleaning.py
import pandas as pd
from typing import List, Dict, Optional

class DataCleaner:
    """数据清洗器"""
    
    def __init__(self, df: pd.DataFrame):
        self.df = df.copy()
    
    def remove_duplicates(self, subset: List[str] = None):
        """删除重复"""
        self.df = self.df.drop_duplicates(subset=subset)
        return self
    
    def handle_missing_values(
        self,
        strategy: str = 'drop',
        fill_value: any = None
    ):
        """处理缺失值"""
        if strategy == 'drop':
            self.df = self.df.dropna()
        elif strategy == 'fill':
            self.df = self.df.fillna(fill_value)
        elif strategy == 'mean':
            self.df = self.df.fillna(self.df.mean())
        elif strategy == 'median':
            self.df = self.df.fillna(self.df.median())
        elif strategy == 'mode':
            self.df = self.df.fillna(self.df.mode().iloc[0])
        
        return self
    
    def remove_outliers(
        self,
        columns: List[str],
        method: str = 'iqr',
        threshold: float = 1.5
    ):
        """删除异常值"""
        for col in columns:
            if method == 'iqr':
                Q1 = self.df[col].quantile(0.25)
                Q3 = self.df[col].quantile(0.75)
                IQR = Q3 - Q1
                
                lower = Q1 - threshold * IQR
                upper = Q3 + threshold * IQR
                
                self.df = self.df[
                    (self.df[col] >= lower) &
                    (self.df[col] <= upper)
                ]
            
            elif method == 'zscore':
                from scipy import stats
                z_scores = stats.zscore(self.df[col])
                self.df = self.df[abs(z_scores) < threshold]
        
        return self
    
    def normalize_column_names(self):
        """标准化列名"""
        self.df.columns = (
            self.df.columns
            .str.lower()
            .str.replace(' ', '_')
            .str.replace('[^a-z0-9_]', '', regex=True)
        )
        
        return self
    
    def convert_types(self, type_map: Dict[str, str]):
        """转换类型"""
        for col, dtype in type_map.items():
            if col in self.df.columns:
                if dtype == 'datetime':
                    self.df[col] = pd.to_datetime(self.df[col])
                else:
                    self.df[col] = self.df[col].astype(dtype)
        
        return self
    
    def remove_columns(self, columns: List[str]):
        """删除列"""
        self.df = self.df.drop(columns=columns, errors='ignore')
        return self
    
    def rename_columns(self, rename_map: Dict[str, str]):
        """重命名列"""
        self.df = self.df.rename(columns=rename_map)
        return self
    
    def filter_rows(self, condition: str):
        """过滤行"""
        self.df = self.df.query(condition)
        return self
    
    def get_cleaned_data(self) -> pd.DataFrame:
        """获取清洗后的数据"""
        return self.df

# 使用
df = pd.read_csv('data.csv')

cleaner = DataCleaner(df)

# 清洗流程
cleaned_df = (
    cleaner
    .remove_duplicates()
    .handle_missing_values(strategy='mean')
    .remove_outliers(['age', 'income'], method='iqr')
    .normalize_column_names()
    .convert_types({
        'date': 'datetime',
        'amount': 'float'
    })
    .remove_columns(['unnecessary_column'])
    .rename_columns({'old_name': 'new_name'})
    .filter_rows('age > 18')
    .get_cleaned_data()
)
```

---

## 📊 数据分析

### 数据分析器

```python
# data/analysis.py
import pandas as pd
import numpy as np
from typing import Dict, List, Optional

class DataAnalyzer:
    """数据分析器"""
    
    def __init__(self, df: pd.DataFrame):
        self.df = df
    
    def basic_statistics(self) -> Dict:
        """基础统计"""
        return {
            'count': len(self.df),
            'mean': self.df.mean(numeric_only=True).to_dict(),
            'median': self.df.median(numeric_only=True).to_dict(),
            'std': self.df.std(numeric_only=True).to_dict(),
            'min': self.df.min(numeric_only=True).to_dict(),
            'max': self.df.max(numeric_only=True).to_dict()
        }
    
    def correlation_analysis(self, columns: List[str] = None) -> pd.DataFrame:
        """相关性分析"""
        if columns:
            return self.df[columns].corr()
        return self.df.corr(numeric_only=True)
    
    def group_analysis(
        self,
        group_by: List[str],
        agg_dict: Dict
    ) -> pd.DataFrame:
        """分组分析"""
        return self.df.groupby(group_by).agg(agg_dict)
    
    def time_series_analysis(
        self,
        date_column: str,
        value_column: str,
        freq: str = 'D'
    ) -> pd.DataFrame:
        """时间序列分析"""
        df = self.df.copy()
        df[date_column] = pd.to_datetime(df[date_column])
        df = df.set_index(date_column)
        
        return df.resample(freq)[value_column].agg(['mean', 'sum', 'count'])
    
    def distribution_analysis(
        self,
        column: str,
        bins: int = 10
    ) -> pd.DataFrame:
        """分布分析"""
        return pd.cut(
            self.df[column],
            bins=bins
        ).value_counts().sort_index()
    
    def trend_analysis(
        self,
        date_column: str,
        value_column: str
    ) -> Dict:
        """趋势分析"""
        df = self.df.copy()
        df[date_column] = pd.to_datetime(df[date_column])
        df = df.sort_values(date_column)
        
        # 计算移动平均
        df['ma_7'] = df[value_column].rolling(window=7).mean()
        df['ma_30'] = df[value_column].rolling(window=30).mean()
        
        # 计算趋势
        from scipy import stats
        x = np.arange(len(df))
        y = df[value_column].values
        
        slope, intercept, r_value, p_value, std_err = stats.linregress(x, y)
        
        return {
            'slope': slope,
            'intercept': intercept,
            'r_squared': r_value ** 2,
            'p_value': p_value,
            'trend': 'increasing' if slope > 0 else 'decreasing',
            'data': df
        }
    
    def segment_analysis(
        self,
        segment_column: str,
        value_column: str,
        segments: Optional[List] = None
    ) -> pd.DataFrame:
        """分段分析"""
        if segments:
            df = self.df[self.df[segment_column].isin(segments)]
        else:
            df = self.df
        
        return df.groupby(segment_column)[value_column].agg([
            'count', 'mean', 'median', 'std', 'min', 'max'
        ])

# 使用
analyzer = DataAnalyzer(cleaned_df)

# 基础统计
stats = analyzer.basic_statistics()

# 相关性分析
corr = analyzer.correlation_analysis(['age', 'income', 'score'])

# 分组分析
grouped = analyzer.group_analysis(
    group_by=['category'],
    agg_dict={
        'amount': ['sum', 'mean'],
        'user_id': 'count'
    }
)

# 时间序列分析
time_series = analyzer.time_series_analysis(
    date_column='date',
    value_column='sales',
    freq='D'
)

# 趋势分析
trend = analyzer.trend_analysis(
    date_column='date',
    value_column='sales'
)
```

---

## 📈 数据可视化

### 可视化生成器

```python
# visualization/generator.py
import matplotlib.pyplot as plt
import seaborn as sns
from typing import List, Dict, Optional
import pandas as pd

class VisualizationGenerator:
    """可视化生成器"""
    
    def __init__(self, style: str = 'seaborn'):
        plt.style.use(style)
        self.figures = []
    
    def line_plot(
        self,
        df: pd.DataFrame,
        x: str,
        y: str,
        title: str = '',
        save_path: Optional[str] = None
    ):
        """折线图"""
        fig, ax = plt.subplots(figsize=(12, 6))
        
        ax.plot(df[x], df[y])
        ax.set_xlabel(x)
        ax.set_ylabel(y)
        ax.set_title(title)
        ax.grid(True)
        
        if save_path:
            fig.savefig(save_path)
        
        self.figures.append(fig)
        return fig
    
    def bar_plot(
        self,
        df: pd.DataFrame,
        x: str,
        y: str,
        title: str = '',
        save_path: Optional[str] = None
    ):
        """柱状图"""
        fig, ax = plt.subplots(figsize=(12, 6))
        
        ax.bar(df[x], df[y])
        ax.set_xlabel(x)
        ax.set_ylabel(y)
        ax.set_title(title)
        ax.grid(True, axis='y')
        
        if save_path:
            fig.savefig(save_path)
        
        self.figures.append(fig)
        return fig
    
    def scatter_plot(
        self,
        df: pd.DataFrame,
        x: str,
        y: str,
        hue: Optional[str] = None,
        title: str = '',
        save_path: Optional[str] = None
    ):
        """散点图"""
        fig, ax = plt.subplots(figsize=(10, 8))
        
        if hue:
            for category in df[hue].unique():
                subset = df[df[hue] == category]
                ax.scatter(subset[x], subset[y], label=category)
            ax.legend()
        else:
            ax.scatter(df[x], df[y])
        
        ax.set_xlabel(x)
        ax.set_ylabel(y)
        ax.set_title(title)
        ax.grid(True)
        
        if save_path:
            fig.savefig(save_path)
        
        self.figures.append(fig)
        return fig
    
    def histogram(
        self,
        df: pd.DataFrame,
        column: str,
        bins: int = 30,
        title: str = '',
        save_path: Optional[str] = None
    ):
        """直方图"""
        fig, ax = plt.subplots(figsize=(10, 6))
        
        ax.hist(df[column], bins=bins, edgecolor='black')
        ax.set_xlabel(column)
        ax.set_ylabel('Frequency')
        ax.set_title(title)
        ax.grid(True, axis='y')
        
        if save_path:
            fig.savefig(save_path)
        
        self.figures.append(fig)
        return fig
    
    def heatmap(
        self,
        df: pd.DataFrame,
        title: str = '',
        save_path: Optional[str] = None
    ):
        """热力图"""
        fig, ax = plt.subplots(figsize=(10, 8))
        
        sns.heatmap(
            df.corr(numeric_only=True),
            annot=True,
            fmt='.2f',
            cmap='coolwarm',
            ax=ax
        )
        ax.set_title(title)
        
        if save_path:
            fig.savefig(save_path)
        
        self.figures.append(fig)
        return fig
    
    def box_plot(
        self,
        df: pd.DataFrame,
        x: str,
        y: str,
        title: str = '',
        save_path: Optional[str] = None
    ):
        """箱线图"""
        fig, ax = plt.subplots(figsize=(12, 6))
        
        df.boxplot(column=y, by=x, ax=ax)
        ax.set_xlabel(x)
        ax.set_ylabel(y)
        ax.set_title(title)
        plt.suptitle('')  # 删除自动标题
        
        if save_path:
            fig.savefig(save_path)
        
        self.figures.append(fig)
        return fig
    
    def save_all(self, directory: str):
        """保存所有图表"""
        import os
        os.makedirs(directory, exist_ok=True)
        
        for i, fig in enumerate(self.figures):
            fig.savefig(f"{directory}/figure_{i}.png")
        
        print(f"✅ Saved {len(self.figures)} figures to {directory}")

# 使用
viz = VisualizationGenerator()

# 折线图
viz.line_plot(
    df=time_series.reset_index(),
    x='date',
    y='mean',
    title='Sales Trend',
    save_path='sales_trend.png'
)

# 柱状图
viz.bar_plot(
    df=grouped.reset_index(),
    x='category',
    y=('amount', 'sum'),
    title='Sales by Category',
    save_path='sales_by_category.png'
)

# 散点图
viz.scatter_plot(
    df=cleaned_df,
    x='age',
    y='income',
    hue='gender',
    title='Age vs Income',
    save_path='age_income_scatter.png'
)

# 热力图
viz.heatmap(
    df=cleaned_df,
    title='Correlation Heatmap',
    save_path='correlation_heatmap.png'
)

# 保存所有
viz.save_all('visualizations')
```

---

## 🎯 数据处理最佳实践

### 1. 数据采集原则

- **自动化** - 自动采集数据
- **定时** - 定时更新数据
- **验证** - 验证数据质量
- **备份** - 备份原始数据

### 2. 数据清洗原则

- **可重复** - 清洗流程可重复
- **可追溯** - 记录清洗步骤
- **可验证** - 验证清洗结果
- **可回滚** - 保留原始数据

### 3. 数据分析原则

- **目标明确** - 明确分析目标
- **方法合理** - 选择合适方法
- **结果验证** - 验证分析结果
- **文档记录** - 记录分析过程

### 4. 数据可视化原则

- **简洁** - 图表简洁明了
- **准确** - 数据准确无误
- **美观** - 设计美观大方
- **交互** - 支持交互探索

---

<div align="center">
  <p>📊 数据驱动，价值挖掘！</p>
</div>
