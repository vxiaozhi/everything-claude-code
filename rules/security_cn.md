# 安全指导原则

## 强制性安全检查

在任何提交之前：
- [ ] 没有硬编码的机密（API 密钥、密码、令牌）
- [ ] 验证所有用户输入
- [ ] SQL 注入预防（参数化查询）
- [ ] XSS 预防（清理 HTML）
- [ ] 启用 CSRF 保护
- [ ] 验证认证/授权
- [ ] 所有端点上的速率限制
- [ ] 错误消息不泄露敏感数据

## 机密管理

```typescript
// 永远不要：硬编码机密
const apiKey = "sk-proj-xxxxx"

// 始终：环境变量
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

## 安全响应协议

如果发现安全问题：
1. 立即停止
2. 使用 **security-reviewer** 代理
3. 在继续之前修复关键问题
4. 轮换任何暴露的机密
5. 检查整个代码库是否存在类似问题
