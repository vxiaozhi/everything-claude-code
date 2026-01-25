---
name: security-review
description: 添加认证、处理用户输入、使用机密、创建 API 端点或实施支付/敏感功能时使用此技能。提供全面的安全检查清单和模式。
---

# 安全审查技能

此技能确保所有代码都遵循安全最佳实践并识别潜在的漏洞。

## 何时激活

- 实施认证或授权
- 处理用户输入或文件上传
- 创建新的 API 端点
- 使用机密或凭据
- 实施支付功能
- 存储或传输敏感数据
- 集成第三方 API

## 安全检查清单

### 1. 机密管理

#### ❌ 永远不要这样做
```typescript
const apiKey = "sk-proj-xxxxx"  // 硬编码的机密
const dbPassword = "password123" // 在源代码中
```

#### ✅ 始终这样做
```typescript
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// 验证机密存在
if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

#### 验证步骤
- [ ] 没有硬编码的 API 密钥、令牌或密码
- [ ] 所有机密都在环境变量中
- [ ] .env.local 在 .gitignore 中
- [ ] git 历史记录中没有机密
- [ ] 生产机密在托管平台（Vercel、Railway）

### 2. 输入验证

#### 始终验证用户输入
```typescript
import { z } from 'zod'

// 定义验证架构
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

// 在处理之前验证
export async function createUser(input: unknown) {
  try {
    const validated = CreateUserSchema.parse(input)
    return await db.users.create(validated)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.errors }
    }
    throw error
  }
}
```

### 3. SQL 注入预防

#### ❌ 永远不要连接 SQL
```typescript
// 危险 - SQL 注入漏洞
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
await db.query(query)
```

#### ✅ 始终使用参数化查询
```typescript
// 安全 - 参数化查询
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)
```

### 4. 认证和授权

#### JWT 令牌处理
```typescript
// ❌ 错误：localStorage（易受 XSS 攻击）
localStorage.setItem('token', token)

// ✅ 正确：httpOnly cookies
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

#### 授权检查
```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // 始终首先验证授权
  const requester = await db.users.findUnique({
    where: { id: requesterId }
  })

  if (requester.role !== 'admin') {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 403 }
    )
  }

  // 继续删除
  await db.users.delete({ where: { id: userId } })
}
```

### 5. XSS 预防

#### 清理 HTML
```typescript
import DOMPurify from 'isomorphic-dompurify'

// 始终清理用户提供的 HTML
function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

### 6. CSRF 保护

#### CSRF 令牌
```typescript
import { csrf } from '@/lib/csrf'

export async function POST(request: Request) {
  const token = request.headers.get('X-CSRF-Token')

  if (!csrf.verify(token)) {
    return NextResponse.json(
      { error: 'Invalid CSRF token' },
      { status: 403 }
    )
  }

  // 处理请求
}
```

### 7. 速率限制

#### API 速率限制
```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分钟
  max: 100, // 每个窗口 100 个请求
  message: 'Too many requests'
})

// 应用到路由
app.use('/api/', limiter)
```

### 8. 敏感数据暴露

#### 日志记录
```typescript
// ❌ 错误：记录敏感数据
console.log('User login:', { email, password })

// ✅ 正确：编辑敏感数据
console.log('User login:', { email, userId })
```

## 预部署安全检查清单

在任何生产部署之前：

- [ ] **机密**：没有硬编码的机密，所有都在环境变量中
- [ ] **输入验证**：所有用户输入都经过验证
- [ ] **SQL 注入**：所有查询都是参数化的
- [ ] **XSS**：清理用户内容
- [ ] **CSRF**：启用保护
- [ ] **认证**：适当的令牌处理
- [ ] **授权**：就位角色检查
- [ ] **速率限制**：在所有端点上启用
- [ ] **HTTPS**：在生产中强制执行
- [ ] **安全标头**：配置 CSP、X-Frame-Options
- [ ] **错误处理**：错误中没有敏感数据
- [ ] **日志记录**：没有记录敏感数据
- [ ] **依赖项**：最新，没有漏洞
- [ ] **行级安全性**：在 Supabase 中启用
- [ ] **CORS**：正确配置
- [ ] **文件上传**：验证（大小、类型）

**记住**：安全性不是可选的。一个漏洞可能会危及整个平台。如有疑问，请谨慎行事。
