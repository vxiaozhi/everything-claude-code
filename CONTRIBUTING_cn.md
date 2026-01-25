# 贡献给 Everything Claude Code

感谢您想要做出贡献。这个仓库旨在成为 Claude Code 用户的社区资源。

## 我们在寻找什么

### Agents（代理）

能够很好地处理特定任务的新代理：
- 特定语言的审查者（Python、Go、Rust）
- 框架专家（Django、Rails、Laravel、Spring）
- DevOps 专家（Kubernetes、Terraform、CI/CD）
- 领域专家（ML 流水线、数据工程、移动端）

### Skills（技能）

工作流定义和领域知识：
- 语言最佳实践
- 框架模式
- 测试策略
- 架构指南
- 特定领域知识

### Commands（命令）

调用有用工作流的斜杠命令：
- 部署命令
- 测试命令
- 文档命令
- 代码生成命令

### Hooks（钩子）

有用的自动化：
- 代码检查/格式化钩子
- 安全检查
- 验证钩子
- 通知钩子

### Rules（规则）

始终遵循的指导原则：
- 安全规则
- 代码风格规则
- 测试要求
- 命名约定

### MCP Configurations（MCP 配置）

新的或改进的 MCP 服务器配置：
- 数据库集成
- 云提供商 MCP
- 监控工具
- 通信工具

---

## 如何贡献

### 1. Fork 仓库

```bash
git clone https://github.com/YOUR_USERNAME/everything-claude-code.git
cd everything-claude-code
```

### 2. 创建分支

```bash
git checkout -b add-python-reviewer
```

### 3. 添加您的贡献

将文件放在适当的目录中：
- `agents/` - 用于新代理
- `skills/` - 用于技能（可以是单个 .md 文件或目录）
- `commands/` - 用于斜杠命令
- `rules/` - 用于规则文件
- `hooks/` - 用于钩子配置
- `mcp-configs/` - 用于 MCP 服务器配置

### 4. 遵循格式

**代理**应该有 frontmatter：

```markdown
---
name: agent-name
description: 它的功能
tools: Read, Grep, Glob, Bash
model: sonnet
---

说明内容...
```

**技能**应该清晰且可操作：

```markdown
# 技能名称

## 何时使用

...

## 如何工作

...

## 示例

...
```

**命令**应该解释它们的功能：

```markdown
---
description: 命令的简要描述
---

# 命令名称

详细说明...
```

**钩子**应该包含描述：

```json
{
  "matcher": "...",
  "hooks": [...],
  "description": "这个钩子的功能"
}
```

### 5. 测试您的贡献

在提交之前确保您的配置可以与 Claude Code 一起使用。

### 6. 提交 PR

```bash
git add .
git commit -m "Add Python code reviewer agent"
git push origin add-python-reviewer
```

然后打开 PR，包含：
- 您添加的内容
- 为什么它有用
- 您如何测试它

---

## 指导原则

### 应该做的

- 保持配置专注和模块化
- 包含清晰的描述
- 提交前测试
- 遵循现有模式
- 记录任何依赖项

### 不应该做的

- 包含敏感数据（API 密钥、令牌、路径）
- 添加过于复杂或小众的配置
- 提交未经测试的配置
- 创建重复功能
- 添加需要特定付费服务但没有替代方案的配置

---

## 文件命名

- 使用小写字母和连字符：`python-reviewer.md`
- 具有描述性：`tdd-workflow.md` 而不是 `workflow.md`
- 使代理/技能名称与文件名匹配

---

## 有问题吗？

在 X 上提出问题或联系：[@affaanmustafa](https://x.com/affaanmustafa)

---

感谢您的贡献。让我们一起构建一个很棒的资源。
