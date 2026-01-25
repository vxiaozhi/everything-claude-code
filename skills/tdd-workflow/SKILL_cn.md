---
name: tdd-workflow
description: 编写新功能、修复错误或重构代码时使用此技能。通过 80%+ 的覆盖率强制执行测试驱动开发，包括单元、集成和 E2E 测试。
---

# 测试驱动开发工作流

此技能确保所有代码开发都遵循 TDD 原则并具有全面的测试覆盖率。

## 何时激活

- 编写新功能或功能
- 修复错误或问题
- 重构现有代码
- 添加 API 端点
- 创建新组件

## 核心原则

### 1. 测试优于代码
始终首先编写测试，然后实施代码以通过测试。

### 2. 覆盖率要求
- 最低 80% 覆盖率（单元 + 集成 + E2E）
- 所有边缘情况都覆盖
- 错误场景经过测试
- 边界条件已验证

### 3. 测试类型

#### 单元测试
- 个别功能和工具
- 组件逻辑
- 纯函数
- 助手和工具

#### 集成测试
- API 端点
- 数据库操作
- 服务交互
- 外部 API 调用

#### E2E 测试 (Playwright)
- 关键用户流程
- 完整的工作流
- 浏览器自动化
- UI 交互

## TDD 工作流步骤

### 步骤 1：编写用户旅程
```
作为一个 [角色]，我想要 [行动]，以便 [利益]

示例：
作为一个用户，我想要语义搜索市场，
以便即使没有确切的关键词我也能找到相关的市场。
```

### 步骤 2：生成测试用例
为每个用户旅程，创建全面的测试用例：

```typescript
describe('语义搜索', () => {
  it('返回查询的相关市场', async () => {
    // 测试实施
  })

  it('优雅地处理空查询', async () => {
    // 测试边缘情况
  })

  it('当 Redis 不可用时回退到子字符串搜索', async () => {
    // 测试回退行为
  })
})
```

### 步骤 3：运行测试（它们应该失败）
```bash
npm test
# 测试应该失败 - 我们还没有实施
```

### 步骤 4：实施代码
编写最小的代码以通过测试：

```typescript
// 由测试指导的实施
export async function searchMarkets(query: string) {
  // 实施在这里
}
```

### 步骤 5：再次运行测试
```bash
npm test
# 测试现在应该通过
```

### 步骤 6：重构
在保持测试绿色时提高代码质量：
- 删除重复
- 改进命名
- 优化性能
- 增强可读性

### 步骤 7：验证覆盖率
```bash
npm run test:coverage
# 验证 80%+ 覆盖率
```

## 测试模式

### 单元测试模式 (Jest/Vitest)

```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from './Button'

describe('按钮组件', () => {
  it('使用正确的文本渲染', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByText('Click me')).toBeInTheDocument()
  })

  it('点击时调用 onClick', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>Click</Button>)

    fireEvent.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })
})
```

### API 集成测试模式

```typescript
import { NextRequest } from 'next/server'
import { GET } from './route'

describe('GET /api/markets', () => {
  it('成功返回市场', async () => {
    const request = new NextRequest('http://localhost/api/markets')
    const response = await GET(request)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data.success).toBe(true)
  })
})
```

### E2E 测试模式 (Playwright)

```typescript
import { test, expect } from '@playwright/test'

test('用户可以搜索和过滤市场', async ({ page }) => {
  // 导航到市场页面
  await page.goto('/')
  await page.click('a[href="/markets"]')

  // 搜索市场
  await page.fill('input[placeholder="搜索市场"]', 'election')

  // 验证搜索结果显示
  const results = page.locator('[data-testid="market-card"]')
  await expect(results).toHaveCount(5, { timeout: 5000 })

  // 过滤状态
  await page.click('button:has-text("Active")')

  // 验证过滤后的结果
  await expect(results).toHaveCount(3)
})
```

## 最佳实践

1. **首先编写测试** - 始终 TDD
2. **每个测试一个断言** - 专注于单个行为
3. **描述性测试名称** - 解释测试的内容
4. **安排-行动-断言** - 清晰的测试结构
5. **模拟外部依赖** - 隔离单元测试
6. **测试边缘情况** - Null、undefined、空、大
7. **测试错误路径** - 不仅仅是快乐路径
8. **保持测试快速** - 单元测试 < 50ms 每个
9. **测试后清理** - 没有副作用
10. **审查覆盖率报告** - 识别缺口

## 成功指标

- 实现 80%+ 代码覆盖率
- 所有测试通过（绿色）
- 没有跳过或禁用的测试
- 快速的测试执行（单元测试 < 30s）
- E2E 测试覆盖关键用户流程
- 测试在投产前捕获错误

**记住**：测试不是可选的。它们是支持自信重构、快速开发和生产可靠性的安全网。
