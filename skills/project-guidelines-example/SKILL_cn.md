# 项目指南技能（示例）

这是一个项目特定技能的示例。将其用作您自己项目的模板。

基于真实的生产应用程序：[Zenith](https://zenith.chat) - AI 驱动的客户发现平台。

---

## 何时使用

在处理为其设计的特定项目时引用此技能。项目技能包含：
- 架构概述
- 文件结构
- 代码模式
- 测试要求
- 部署工作流

---

## 架构概述

**技术栈：**
- **前端**：Next.js 15（App Router）、TypeScript、React
- **后端**：FastAPI (Python)、Pydantic 模型
- **数据库**：Supabase (PostgreSQL)
- **AI**：Claude API 与工具调用和结构化输出
- **部署**：Google Cloud Run
- **测试**：Playwright (E2E)、pytest (后端)、React Testing Library

**服务：**
```
┌─────────────────────────────────────────────────────────────┐
│                         前端                            │
│  Next.js 15 + TypeScript + TailwindCSS                     │
│  已部署：Vercel / Cloud Run                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                         后端                             │
│  FastAPI + Python 3.11 + Pydantic                          │
│  已部署：Cloud Run                                       │
└─────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │ Supabase │   │  Claude  │   │  Redis   │
        │ Database │   │   API    │   │  Cache   │
        └──────────┘   └──────────┘   └──────────┘
```

---

## 文件结构

```
project/
├── frontend/
│   └── src/
│       ├── app/              # Next.js app router 页面
│       ├── components/       # React 组件
│       ├── hooks/            # 自定义 React 钩子
│       ├── lib/              # 工具
│       └── types/            # TypeScript 定义
│
├── backend/
│   ├── routers/              # FastAPI 路由处理程序
│   ├── models.py             # Pydantic 模型
│   ├── main.py               # FastAPI 应用入口
│   └── tests/                # pytest 测试
│
└── deploy/                   # 部署配置
```

---

## 代码模式

### API 响应格式 (FastAPI)

```python
from pydantic import BaseModel
from typing import Generic, TypeVar, Optional

T = TypeVar('T')

class ApiResponse(BaseModel, Generic[T]):
    success: bool
    data: Optional[T] = None
    error: Optional[str] = None

    @classmethod
    def ok(cls, data: T) -> "ApiResponse[T]":
        return cls(success=True, data=data)

    @classmethod
    def fail(cls, error: str) -> "ApiResponse[T]":
        return cls(success=False, error=error)
```

### Claude AI 集成（结构化输出）

```python
from anthropic import Anthropic
from pydantic import BaseModel

class AnalysisResult(BaseModel):
    summary: str
    key_points: list[str]
    confidence: float

async def analyze_with_claude(content: str) -> AnalysisResult:
    client = Anthropic()
    response = client.messages.create(
        model="claude-sonnet-4-5-20250514",
        max_tokens=1024,
        messages=[{"role": "user", "content": content}],
        tools=[{
            "name": "provide_analysis",
            "description": "Provide structured analysis",
            "input_schema": AnalysisResult.model_json_schema()
        }],
        tool_choice={"type": "tool", "name": "provide_analysis"}
    )
    return AnalysisResult(**tool_use.input)
```

---

## 测试要求

### 后端 (pytest)

```bash
# 运行所有测试
poetry run pytest tests/

# 运行带有覆盖率的测试
poetry run pytest tests/ --cov=. --cov-report=html
```

### 前端 (React Testing Library)

```bash
# 运行测试
npm run test

# 运行带有覆盖率的测试
npm run test -- --coverage

# 运行 E2E 测试
npm run test:e2e
```

---

## 部署工作流

### 预部署检查清单

- [ ] 所有测试在本地通过
- [ ] `npm run build` 成功（前端）
- [ ] `poetry run pytest` 通过（后端）
- [ ] 没有硬编码的机密
- [ ] 环境变量已记录
- [ ] 数据库迁移就绪

---

## 关键规则

1. **没有表情符号** 在代码、注释或文档中
2. **不可变性** - 永远不要变异对象或数组
3. **TDD** - 在实施之前编写测试
4. **80% 覆盖率** 最低
5. **许多小文件** - 典型 200-400 行，最多 800 行
6. **没有 console.log** 在生产代码中
7. **适当的错误处理** 使用 try/catch
8. **输入验证** 使用 Pydantic/Zod

---

## 相关技能

- `coding-standards.md` - 通用编码最佳实践
- `backend-patterns.md` - API 和数据库模式
- `frontend-patterns.md` - React 和 Next.js 模式
- `tdd-workflow/` - 测试驱动开发方法论
