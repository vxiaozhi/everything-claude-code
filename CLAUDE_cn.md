# CLAUDE.md

此文件为 Claude Code (claude.ai/code) 在此仓库中使用代码时提供指导。

## 项目概述

这是一个 **Claude Code 插件**仓库，包含 Claude Code 的生产就绪配置，包括代理、技能、钩子、命令和规则。该插件具有跨平台支持（Windows、macOS、Linux），所有钩子和脚本都使用 Node.js 编写，以实现最大的兼容性。

**核心架构：**
- **代理** (`agents/`)：用于委托的有限范围的专门子代理（planner、architect、code-reviewer、security-reviewer、tdd-guide 等）
- **技能** (`skills/`)：代理/命令调用的工作流定义和领域知识
- **命令** (`commands/`)：用于快速执行的斜杠命令（/plan、/tdd、/code-review、/build-fix、/e2e 等）
- **钩子** (`hooks/`)：在 `hooks.json` 中配置的事件驱动自动化
- **规则** (`rules/`)：始终遵循的指导原则（安装时复制到用户的 `~/.claude/rules/`）
- **脚本** (`scripts/`)：用于钩子和包管理的跨平台 Node.js 实用程序

## 常用开发命令

### 运行测试
```bash
# 运行所有测试
node tests/run-all.js

# 运行单个测试文件
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

### 包管理器设置
```bash
# 交互式设置
node scripts/setup-package-manager.js

# 设置全局包管理器
node scripts/setup-package-manager.js --global pnpm

# 设置项目特定的包管理器
node scripts/setup-package-manager.js --project bun

# 检测当前设置
node scripts/setup-package-manager.js --detect
```

### 包管理器检测
插件使用以下优先级自动检测首选的包管理器：
1. 环境变量 `CLAUDE_PACKAGE_MANAGER`
2. 项目配置 `.claude/package-manager.json`
3. `package.json` 的 `packageManager` 字段
4. 锁定文件检测（pnpm-lock.yaml、bun.lockb、yarn.lock、package-lock.json）
5. 全局配置 `~/.claude/package-manager.json`
6. 回退到第一个可用的（pnpm > bun > yarn > npm）

## 架构深入分析

### 钩子系统
钩子在 `hooks/hooks.json` 中配置，并在特定事件时触发：

**PreToolUse**：工具执行前
- 阻止在 tmux 之外运行开发服务器
- 警告在 tmux 中运行长时间命令
- 阻止创建不必要的 .md 文件
- 定期间隔时建议手动压缩

**PostToolUse**：工具执行后
- 在 `gh pr create` 后记录 PR URL
- 在编辑后自动使用 Prettier 格式化 JS/TS
- 在编辑 .ts/.tsx 文件后运行 TypeScript 检查
- 警告 console.log 语句

**SessionStart**：新会话开始时
- 从会话文件加载先前的上下文
- 检测并报告包管理器
- 显示可用的学习技能

**SessionEnd**：会话结束时
- 将会话状态持久化到 `~/.claude/sessions/`
- 评估会话以提取可复用的模式

**PreCompact**：上下文压缩前
- 保存状态以在压缩期间维护上下文

**Stop**：每次响应后
- 检查修改后的文件中是否存在 console.log

### 脚本实用程序
位于 `scripts/lib/`：

**utils.js**：跨平台实用程序
- 文件操作（readFile、writeFile、findFiles）
- 目录管理（ensureDir、getClaudeDir、getSessionsDir）
- Git 操作（isGitRepo、getGitModifiedFiles）
- 日期/时间助手（getDateTimeString、getDateString）
- 安全加固的命令执行（commandExists、runCommand）

**package-manager.js**：包管理器抽象
- 从锁定文件、package.json 或配置自动检测
- 为 npm/pnpm/yarn/bun 生成命令
- 钩子匹配器的模式匹配（例如，`getCommandPattern('dev')`）

### 代理-技能-命令关系
- **命令**调用特定的**技能**或**代理**
- **代理**使用**技能**获取领域知识
- **技能**包含可重用的工作流和模式

示例流程：
1. 用户运行 `/plan` → 调用 `plan.md` 命令
2. 命令委托给 `planner.md` 代理
3. 代理使用 `tdd-workflow/SKILL.md` 获取方法论

### 内存持久化系统
详细指南（参见 README）实现：
- **会话文件**：存储在 `~/.claude/sessions/*.tmp`
- **学习技能**：自动提取模式到 `~/.claude/skills/learned/`
- **压缩建议**：战略性上下文缩减
- **验证循环**：持续评估检查点

### 插件清单
`.claude-plugin/plugin.json` 定义：
- 元数据（名称、描述、作者）
- 组件路径（命令、技能、代理、规则、钩子）
- `/plugin marketplace add` 的市场配置

## 文件组织约定

### 代理 (`agents/*.md`)
每个代理文件必须有一个 frontmatter 部分：
```markdown
---
name: agent-name
description: 这个代理的功能
tools: Read, Grep, Glob, Bash
model: opus
---

您是一个专门的代理...
```

### 技能 (`skills/*/SKILL.md`)
技能按领域组织：
- `coding-standards/`：语言最佳实践
- `backend-patterns/`：API、数据库、缓存模式
- `frontend-patterns/`：React、Next.js 模式
- `tdd-workflow/`：测试驱动开发方法论
- `continuous-learning/`：从会话中自动提取模式
- `security-review/`：安全检查清单
- `verification-loop/`：持续验证模式

### 命令 (`commands/*.md`)
斜杠命令遵循命名约定：`command-name.md` → `/command-name`

### 钩子 (`hooks/hooks.json`)
钩子匹配器使用表达式语法：
```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "node \"${CLAUDE_PLUGIN_ROOT}/scripts/hook.js\""
  }]
}
```

## 跨平台兼容性

所有脚本使用 Node.js，具有：
- `path.join()` 用于文件路径（没有硬编码的 `/`）
- `fs.existsSync()` 在文件操作之前
- 带有适当错误处理的 `spawnSync`/`execSync`
- 平台检测（`isWindows`、`isMacOS`、`isLinux`）

## 重要约束

- **无原生二进制文件**：所有钩子/脚本都使用 Node.js
- **安全性**：永远不要直接执行用户控制的 shell 命令
- **上下文窗口**：不要一次启用所有 MCP（建议每个项目启用 <10 个，<80 个工具）
- **钩子阻止**：退出代码为 1 的钩子将阻止工具执行
- **包管理器**：始终使用 `getPackageManager()` 生成命令

## 作为插件安装

```bash
# 添加市场
/plugin marketplace add affaan-m/everything-claude-code

# 安装插件
/plugin install everything-claude-code@everything-claude-code
```

或手动添加到 `~/.claude/settings.json`：
```json
{
  "extraKnownMarketplaces": {
    "everything-claude-code": {
      "source": {
        "source": "github",
        "repo": "affaan-m/everything-claude-code"
      }
    }
  },
  "enabledPlugins": {
    "everything-claude-code@everything-claude-code": true
  }
}
```

## 关键文件位置

- 插件清单：`.claude-plugin/plugin.json`
- 钩子配置：`hooks/hooks.json`
- 跨平台实用程序：`scripts/lib/utils.js`
- 包管理器抽象：`scripts/lib/package-manager.js`
- 钩子实现：`scripts/hooks/*.js`
- 测试套件：`tests/run-all.js`
- 市场配置：`.claude-plugin/marketplace.json`
