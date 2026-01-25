---
description: 交互式配置包管理器（npm、pnpm、yarn、bun）。设置全局或项目特定的首选项。自动检测并建议适当的包管理器。
---

# 设置包管理器命令

为 Claude Code 插件配置首选的包管理器。

## 包管理器支持

- **npm** - Node 默认包管理器
- **pnpm** - 快速、节省磁盘空间
- **yarn** - 流行、可靠
- **bun** - 超快、现代

## 检测优先级

1. 环境变量 `CLAUDE_PACKAGE_MANAGER`
2. 项目配置 `.claude/package-manager.json`
3. `package.json` 的 `packageManager` 字段
4. 锁定文件检测
5. 全局配置 `~/.claude/package-manager.json`
6. 回退到第一个可用的

## 使用

```bash
/setup-pm
```

命令将：
1. 检测当前的包管理器
2. 显示锁定文件
3. 建议适当的包管理器
4. 要求配置全局或项目特定设置

## 设置选项

### 全局设置
```bash
/setup-pm --global pnpm
```
影响所有项目。

### 项目特定设置
```bash
/setup-pm --project bun
```
仅影响当前项目。

### 检测当前设置
```bash
/setup-pm --detect
```
显示哪个包管理器将被使用。
