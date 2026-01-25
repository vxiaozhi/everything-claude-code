---
description: 测试驱动开发工作流。使用 tdd-guide 代理强制执行红-绿-重构循环，首先编写测试，确保 80%+ 覆盖率。
---

# TDD 命令

执行测试驱动开发工作流。

## TDD 循环

1. **红** - 编写失败的测试
2. **绿** - 编写最小代码以通过
3. **重构** - 改进代码质量
4. **验证** - 检查覆盖率（80%+）

## 工作流程

```bash
/tdd 添加用户认证
```

代理将：

### 1. 定义接口
```typescript
interface AuthService {
  login(email: string, password: string): Promise<User>
  register(email: string, password: string): Promise<User>
  logout(): Promise<void>
}
```

### 2. 首先编写测试
```typescript
test('登录有效用户', async () => {
  const user = await authService.login('user@example.com', 'password')
  expect(user).toBeDefined()
  expect(user.email).toBe('user@example.com')
})
```

### 3. 运行测试（应该失败）
```bash
npm test
# 测试失败 - 未实施
```

### 4. 实施最小代码
```typescript
async login(email: string, password: string): Promise<User> {
  // 最小实施
}
```

### 5. 运行测试（应该通过）
```bash
npm test
# 测试通过
```

### 6. 重构
- 删除重复
- 改进命名
- 优化性能
- 保持测试通过

### 7. 验证覆盖率
```bash
npm run test:coverage
# 目标：80%+ 覆盖率
```

## 测试类型

- **单元测试** - 函数、组件、工具
- **集成测试** - API 端点、数据库
- **E2E 测试** - 关键用户流程

与 **tdd-workflow** 技能一起使用。
