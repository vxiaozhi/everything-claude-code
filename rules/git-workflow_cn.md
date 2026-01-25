# Git 工作流

## 提交消息格式

```
<type>: <description>

<optional body>
```

类型：feat、fix、refactor、docs、test、chore、perf、ci

注意：通过 ~/.claude/settings.json 全局禁用归因。

## 拉取请求工作流

创建 PR 时：
1. 分析完整的提交历史（而不仅仅是最新的提交）
2. 使用 `git diff [base-branch]...HEAD` 查看所有更改
3. 起草全面的 PR 摘要
4. 包括带有待办事项的测试计划
5. 如果是新分支，使用 `-u` 标志推送

## 功能实施工作流

1. **首先计划**
   - 使用 **planner** 代理创建实施计划
   - 识别依赖关系和风险
   - 分解为阶段

2. **TDD 方法**
   - 使用 **tdd-guide** 代理
   - 首先编写测试（RED）
   - 实施以通过测试（GREEN）
   - 重构（IMPROVE）
   - 验证 80%+ 覆盖率

3. **代码审查**
   - 编写代码后立即使用 **code-reviewer** 代理
   - 解决关键和高问题
   - 尽可能修复中等问题

4. **提交和推送**
   - 详细的提交消息
   - 遵循传统提交格式
