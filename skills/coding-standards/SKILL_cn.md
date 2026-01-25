---
name: coding-standards
description: TypeScript、JavaScript、React 和 Node.js 开发的通用编码标准、最佳实践和模式。
---

# 编码标准和最佳实践

适用于所有项目的通用编码标准。

## 代码质量原则

### 1. 可读性优先
- 代码被阅读的次数多于编写的次数
- 清晰的变量和函数名称
- 优先使用自文档化的代码而非注释
- 一致的格式

### 2. KISS（保持简单、愚蠢）
- 最简单的有效解决方案
- 避免过度工程
- 不要过早优化
- 易于理解 > 聪明的代码

### 3. DRY（不要重复自己）
- 将公共逻辑提取到函数中
- 创建可重用的组件
- 跨模块共享工具
- 避免复制粘贴编程

### 4. YAGNI（你不会需要它）
- 在需要之前不要构建功能
- 避免投机性泛化
- 仅在需要时添加复杂性
- 从简单开始，需要时重构

## TypeScript/JavaScript 标准

### 变量命名

```typescript
// ✅ 好：描述性名称
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// ❌ 坏：不清楚的名称
const q = 'election'
const flag = true
const x = 1000
```

### 函数命名

```typescript
// ✅ 好：动词-名词模式
async function fetchMarketData(marketId: string) { }
function calculateSimilarity(a: number[], b: number[]) { }
function isValidEmail(email: string): boolean { }

// ❌ 坏：不清楚或仅名词
async function market(id: string) { }
function similarity(a, b) { }
function email(e) { }
```

### 不可变模式（关键）

```typescript
// ✅ 始终使用展开运算符
const updatedUser = {
  ...user,
  name: 'New Name'
}

const updatedArray = [...items, newItem]

// ❌ 永远不要直接变异
user.name = 'New Name'  // 坏
items.push(newItem)     // 坏
```

### 错误处理

```typescript
// ✅ 好：全面的错误处理
async function fetchData(url: string) {
  try {
    const response = await fetch(url)
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }
    return await response.json()
  } catch (error) {
    console.error('Fetch failed:', error)
    throw new Error('Failed to fetch data')
  }
}
```

### 异步/等待最佳实践

```typescript
// ✅ 好：尽可能并行执行
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats()
])

// ❌ 坏：不必要的顺序执行
const users = await fetchUsers()
const markets = await fetchMarkets()
const stats = await fetchStats()
```

### 类型安全

```typescript
// ✅ 好：适当的类型
interface Market {
  id: string
  name: string
  status: 'active' | 'resolved' | 'closed'
  created_at: Date
}

// ❌ 坏：使用 'any'
function getMarket(id: any): Promise<any> {
  // 实施
}
```

## React 最佳实践

### 组件结构

```typescript
// ✅ 好：带类型的函数组件
interface ButtonProps {
  children: React.ReactNode
  onClick: () => void
  disabled?: boolean
  variant?: 'primary' | 'secondary'
}

export function Button({
  children,
  onClick,
  disabled = false,
  variant = 'primary'
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  )
}
```

### 自定义钩子

```typescript
// ✅ 好：可重用的自定义钩子
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}
```

### 状态管理

```typescript
// ✅ 好：适当的状态更新
const [count, setCount] = useState(0)

// 基于先前状态的功能更新
setCount(prev => prev + 1)

// ❌ 坏：直接状态引用
setCount(count + 1)  // 在异步场景中可能过时
```

### 条件渲染

```typescript
// ✅ 好：清晰的条件渲染
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// ❌ 坏：三元地狱
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

## API 设计标准

### REST API 约定

```
GET    /api/markets              # 列出所有市场
GET    /api/markets/:id          # 获取特定市场
POST   /api/markets              # 创建新市场
PUT    /api/markets/:id          # 更新市场（完整）
PATCH  /api/markets/:id          # 更新市场（部分）
DELETE /api/markets/:id          # 删除市场
```

### 响应格式

```typescript
// ✅ 好：一致的响应结构
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}
```

### 输入验证

```typescript
import { z } from 'zod'

// ✅ 好：架构验证
const CreateMarketSchema = z.object({
  name: z.string().min(1).max(200),
  description: z.string().min(1).max(2000),
  endDate: z.string().datetime(),
  categories: z.array(z.string()).min(1)
})

export async function POST(request: Request) {
  const body = await request.json()
  try {
    const validated = CreateMarketSchema.parse(body)
    // 使用经过验证的数据继续
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({
        success: false,
        error: 'Validation failed',
        details: error.errors
      }, { status: 400 })
    }
  }
}
```

## 文件组织

### 项目结构

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API 路由
│   ├── markets/           # 市场页面
│   └── (auth)/           # 认证页面（路由组）
├── components/            # React 组件
│   ├── ui/               # 通用 UI 组件
│   ├── forms/            # 表单组件
│   └── layouts/          # 布局组件
├── hooks/                # 自定义 React 钩子
├── lib/                  # 工具和配置
│   ├── api/             # API 客户端
│   ├── utils/           # 辅助函数
│   └── constants/       # 常量
├── types/                # TypeScript 类型
└── styles/              # 全局样式
```

## 性能最佳实践

### 记忆化

```typescript
import { useMemo, useCallback } from 'react'

// ✅ 好：记忆化昂贵的计算
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// ✅ 好：记忆化回调
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])
```

### 懒加载

```typescript
import { lazy, Suspense } from 'react'

// ✅ 好：懒加载重型组件
const HeavyChart = lazy(() => import('./HeavyChart'))

export function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyChart />
    </Suspense>
  )
}
```

## 测试标准

### 测试结构（AAA 模式）

```typescript
test('正确计算相似度', () => {
  // 安排
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // 行动
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // 断言
  expect(similarity).toBe(0)
})
```

## 代码异味检测

注意这些反模式：

### 1. 长函数
```typescript
// ❌ 坏：函数 > 50 行
function processMarketData() {
  // 100 行代码
}

// ✅ 好：拆分为较小的函数
function processMarketData() {
  const validated = validateData()
  const transformed = transformData(validated)
  return saveData(transformed)
}
```

### 2. 深度嵌套
```typescript
// ❌ 坏：5+ 级嵌套
if (user) {
  if (user.isAdmin) {
    if (market) {
      if (market.isActive) {
        if (hasPermission) {
          // 做某事
        }
      }
    }
  }
}

// ✅ 好：早期返回
if (!user) return
if (!user.isAdmin) return
if (!market) return
if (!market.isActive) return
if (!hasPermission) return

// 做某事
```

**记住**：代码质量是不可协商的。清晰、可维护的代码支持快速开发和自信重构。
