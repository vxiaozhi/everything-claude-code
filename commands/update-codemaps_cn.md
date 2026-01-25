---
description: 分析代码库结构并更新架构文档。生成 token 精简的代码图 (docs/CODEMAPS/*) 和差异报告。
---

# 更新代码图命令

从代码库生成架构文档。

## 工作流程

1. **扫描源文件**
   - 导入、导出
   - 依赖关系
   - 文件结构

2. **生成代码图**
   - `codemaps/architecture.md` - 整体架构
   - `codemaps/backend.md` - 后端结构
   - `codemaps/frontend.md` - 前端结构
   - `codemaps/data.md` - 数据模型

3. **计算差异**
   - 与上一版本比较
   - 计算变化百分比

4. **请求批准**
   - 如果变化 > 30%，请求用户确认

5. **保存报告**
   - 添加新鲜度时间戳
   - 保存到 `.reports/codemap-diff.txt`

## 使用

```bash
/update-codemaps
```

命令将：
- 扫描代码库
- 生成代码图
- 显示差异
- 请求批准（如果重大更改）
