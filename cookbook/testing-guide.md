# OpenHands 测试框架完整指南

> 从零到一构建完整的测试体系

---

## 📋 目录

- [测试策略](#测试策略)
- [单元测试](#单元测试)
- [集成测试](#集成测试)
- [E2E 测试](#e2e-测试)
- [测试自动化](#测试自动化)

---

## 🎯 测试策略

### 测试金字塔

```
        ┌───────┐
        │  E2E  │  10%
        ├───────┤
        │集成测试│  20%
        ├───────┤
        │单元测试│  70%
        └───────┘
```

### 覆盖率目标

| 类型 | 目标 | 度量方式 |
|------|------|---------|
| **语句覆盖** | > 80% | Jest/Vitest |
| **分支覆盖** | > 75% | Jest/Vitest |
| **函数覆盖** | > 85% | Jest/Vitest |
| **行覆盖** | > 80% | Jest/Vitest |

---

## 🧪 单元测试

### 1. 基础测试

```python
# test_calculator.py
import pytest
from calculator import Calculator

class TestCalculator:
    @pytest.fixture
    def calc(self):
        """每个测试前创建 Calculator 实例"""
        return Calculator()
    
    def test_add_positive_numbers(self, calc):
        """测试正数加法"""
        assert calc.add(2, 3) == 5
    
    def test_add_negative_numbers(self, calc):
        """测试负数加法"""
        assert calc.add(-1, -2) == -3
    
    def test_add_zero(self, calc):
        """测试加零"""
        assert calc.add(5, 0) == 5
    
    def test_add_floats(self, calc):
        """测试浮点数加法"""
        assert calc.add(1.5, 2.5) == 4.0
    
    def test_divide_by_zero(self, calc):
        """测试除以零"""
        with pytest.raises(ValueError, match="Cannot divide by zero"):
            calc.divide(10, 0)
```

### 2. 参数化测试

```python
import pytest

@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (-1, -2, -3),
    (0, 0, 0),
    (100, 200, 300),
])
def test_add_parameterized(a, b, expected):
    """参数化测试"""
    calc = Calculator()
    assert calc.add(a, b) == expected
```

### 3. Mock 测试

```python
from unittest.mock import Mock, patch
from openhands import Agent

def test_agent_with_mock():
    """使用 Mock 测试 Agent"""
    # 创建 Mock
    mock_model = Mock()
    mock_model.generate.return_value = "Hello World"
    
    # 使用 Mock
    agent = Agent(model=mock_model)
    result = agent.run("创建 Hello World")
    
    # 验证调用
    mock_model.generate.assert_called_once_with("创建 Hello World")
    assert result.output == "Hello World"
```

---

## 🔗 集成测试

### 1. API 集成测试

```python
import pytest
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

class TestAPI:
    def test_create_user(self):
        """测试创建用户"""
        response = client.post(
            "/api/users",
            json={
                "name": "Alice",
                "email": "alice@example.com"
            }
        )
        
        assert response.status_code == 201
        assert response.json()["name"] == "Alice"
    
    def test_get_user(self):
        """测试获取用户"""
        # 先创建用户
        create_response = client.post(
            "/api/users",
            json={"name": "Bob", "email": "bob@example.com"}
        )
        user_id = create_response.json()["id"]
        
        # 获取用户
        response = client.get(f"/api/users/{user_id}")
        
        assert response.status_code == 200
        assert response.json()["name"] == "Bob"
    
    def test_update_user(self):
        """测试更新用户"""
        # 创建用户
        create_response = client.post(
            "/api/users",
            json={"name": "Charlie", "email": "charlie@example.com"}
        )
        user_id = create_response.json()["id"]
        
        # 更新用户
        response = client.put(
            f"/api/users/{user_id}",
            json={"name": "Charlie Updated"}
        )
        
        assert response.status_code == 200
        assert response.json()["name"] == "Charlie Updated"
    
    def test_delete_user(self):
        """测试删除用户"""
        # 创建用户
        create_response = client.post(
            "/api/users",
            json={"name": "David", "email": "david@example.com"}
        )
        user_id = create_response.json()["id"]
        
        # 删除用户
        response = client.delete(f"/api/users/{user_id}")
        assert response.status_code == 204
        
        # 验证已删除
        get_response = client.get(f"/api/users/{user_id}")
        assert get_response.status_code == 404
```

### 2. 数据库集成测试

```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from database import Base, get_db
from models import User

# 测试数据库
SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"

engine = create_engine(SQLALCHEMY_DATABASE_URL)
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

@pytest.fixture
def db_session():
    """创建测试数据库会话"""
    # 创建表
    Base.metadata.create_all(bind=engine)
    
    session = TestingSessionLocal()
    try:
        yield session
    finally:
        session.close()
        # 清理表
        Base.metadata.drop_all(bind=engine)

def test_create_user(db_session):
    """测试创建用户"""
    user = User(
        name="Alice",
        email="alice@example.com"
    )
    
    db_session.add(user)
    db_session.commit()
    
    # 验证
    saved_user = db_session.query(User).first()
    assert saved_user.name == "Alice"
    assert saved_user.email == "alice@example.com"
```

---

## 🎭 E2E 测试

### 1. Playwright 测试

```typescript
// e2e/userFlow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Flow', () => {
  test('should complete user registration', async ({ page }) => {
    // 访问注册页面
    await page.goto('https://example.com/register');
    
    // 填写表单
    await page.fill('#name', 'Alice');
    await page.fill('#email', 'alice@example.com');
    await page.fill('#password', 'password123');
    
    // 提交表单
    await page.click('#submit-button');
    
    // 验证跳转
    await expect(page).toHaveURL('https://example.com/dashboard');
    await expect(page.locator('h1')).toContainText('Welcome, Alice!');
  });
  
  test('should login successfully', async ({ page }) => {
    // 访问登录页面
    await page.goto('https://example.com/login');
    
    // 填写表单
    await page.fill('#email', 'alice@example.com');
    await page.fill('#password', 'password123');
    
    // 提交表单
    await page.click('#login-button');
    
    // 验证登录成功
    await expect(page).toHaveURL('https://example.com/dashboard');
  });
});
```

### 2. Cypress 测试

```javascript
// cypress/e2e/userFlow.cy.js
describe('User Flow', () => {
  it('should complete user registration', () => {
    // 访问注册页面
    cy.visit('/register');
    
    // 填写表单
    cy.get('#name').type('Alice');
    cy.get('#email').type('alice@example.com');
    cy.get('#password').type('password123');
    
    // 提交表单
    cy.get('#submit-button').click();
    
    // 验证跳转
    cy.url().should('include', '/dashboard');
    cy.get('h1').should('contain', 'Welcome, Alice!');
  });
});
```

---

## 🤖 测试自动化

### 1. GitHub Actions

```yaml
# .github/workflows/test.yml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install Dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov
      
      - name: Run Unit Tests
        run: pytest tests/unit --cov=src --cov-report=xml
      
      - name: Run Integration Tests
        run: pytest tests/integration
      
      - name: Upload Coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
```

### 2. Pre-commit Hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
  
  - repo: https://github.com/psf/black
    rev: 23.3.0
    hooks:
      - id: black
  
  - repo: local
    hooks:
      - id: pytest
        name: pytest
        entry: pytest
        language: system
        pass_filenames: false
        always_run: true
```

---

## 📊 测试报告

### 1. 覆盖率报告

```bash
# 生成覆盖率报告
pytest --cov=src --cov-report=html --cov-report=term

# 输出示例
Name                    Stmts   Miss  Cover
-------------------------------------------
src/__init__.py             1      0   100%
src/calculator.py          20      2    90%
src/main.py                15      0   100%
-------------------------------------------
TOTAL                      36      2    94%
```

### 2. 测试报告

```python
# pytest.ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = 
    --verbose
    --tb=short
    --cov=src
    --cov-report=term-missing
    --cov-report=html
    --html=reports/test_report.html
```

---

## 🎯 测试检查清单

### ✅ 单元测试

- [ ] 所有公共方法都有测试
- [ ] 边界条件已测试
- [ ] 异常情况已测试
- [ ] Mock 使用得当
- [ ] 覆盖率 > 80%

### ✅ 集成测试

- [ ] API 端点已测试
- [ ] 数据库操作已测试
- [ ] 外部服务已测试
- [ ] 错误处理已测试

### ✅ E2E 测试

- [ ] 关键用户流程已测试
- [ ] 跨浏览器测试
- [ ] 响应式测试
- [ ] 性能测试

---

## 📚 相关资源

- [Pytest Documentation](https://docs.pytest.org/)
- [Playwright Documentation](https://playwright.dev/)
- [Testing Best Practices](https://testing.googleblog.com/)

---

<div align="center">
  <p>🧪 测试驱动开发，质量第一！</p>
</div>
