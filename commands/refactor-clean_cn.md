---
description: 检测并删除死代码、未使用的导出和重复代码。使用 refactor-cleaner 代理运行 knip、depcheck、ts-prune 并安全地删除未使用的代码。
---

# 重构清理命令

检测并删除死代码和重复项。

## 工作流程

1. **运行检测工具**
   ```bash
   npx knip
   npx depcheck
   npx ts-prune
   ```

2. **分析结果**
   - 未使用的导出
   - 未使用的依赖
   - 未使用的文件
   - 重复代码

3. **安全删除**
   - 确认代码确实未使用
   - 删除死代码
   - 更新导入
   - 验证构建通过

## 使用

```bash
/refactor-clean
```

代理将：
- 运行检测工具
- 分析结果
- 要求确认
- 安全删除代码
- 记录在 DELETION_LOG.md 中

## 检测工具

- **knip** - 未使用的文件、导出、依赖
- **depcheck** - 未使用的 npm 包
- **ts-prune** - 未使用的导出
- **eslint** - 未使用的变量
