---
description: 从单一真相来源 (package.json、.env.example) 同步文档。生成脚本参考、环境变量文档、贡献指南和操作手册。
---

# 更新文档命令

从代码同步文档。

## 工作流程

1. **读取 package.json**
   - 提取脚本部分
   - 生成脚本参考表
   - 包括描述注释

2. **读取 .env.example**
   - 提取环境变量
   - 记录目的和格式

3. **生成文档**
   - `docs/CONTRIB.md` - 开发工作流
   - `docs/RUNBOOK.md` - 部署、监控、回滚

4. **识别过时的文档**
   - 查找 90+ 天未修改的文档
   - 列出以供手动审查

5. **显示差异摘要**

## 使用

```bash
/update-docs
```

## 单一真相来源

- `package.json` - 脚本命令
- `.env.example` - 环境变量
