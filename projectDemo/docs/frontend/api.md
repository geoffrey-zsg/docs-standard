# API 请求层规范

> 当 API 调用复杂度增加时，从 FRONTEND.md 拆分到此文件。

## 架构原则

- **统一入口**：所有请求通过 `src/shared/api` 的客户端封装
- **类型安全**：接口类型从 OpenAPI 契约生成
- **错误处理**：统一拦截器处理认证失败、网络错误
- **三态处理**：loading、error、empty 必须完整

## 目录结构

```
src/shared/api/
├── client.ts          # 统一 fetch 封装
└── types.ts           # 公共类型

src/generated/api/     # 自动生成，禁止手改

src/features/<模块>/api/
└── <模块>.ts          # 业务模块的 API 封装
```

## 请求封装示例

```ts
// src/shared/api/client.ts
const BASE_URL = process.env.NEXT_PUBLIC_API_BASE_URL

export async function apiRequest<T>(
  path: string,
  options?: RequestInit
): Promise<T> {
  const res = await fetch(`${BASE_URL}${path}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    },
  })
  if (res.status === 401) {
    // 跳转登录
  }
  if (!res.ok) {
    throw new Error(`API Error: ${res.status}`)
  }
  return res.json()
}
```

## 业务 API 封装

```ts
// src/features/orders/api/orders.ts
import { apiRequest } from '@/shared/api/client'
import type { Order, OrderListParams } from '@/generated/api'

export const ordersApi = {
  list: (params: OrderListParams) =>
    apiRequest<Order[]>(`/orders?${new URLSearchParams(params as Record<string, string>)}`),

  getById: (id: string) =>
    apiRequest<Order>(`/orders/${id}`),

  create: (data: CreateOrderDto) =>
    apiRequest<Order>('/orders', { method: 'POST', body: JSON.stringify(data) }),
}
```

## 在组件中使用

```tsx
// ✅ 使用 TanStack Query
import { useQuery } from '@tanstack/react-query'
import { ordersApi } from '../api/orders'

function OrderList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['orders'],
    queryFn: () => ordersApi.list({})
  })

  if (isLoading) return <Loading />
  if (error) return <Error message={error.message} />
  if (!data?.length) return <Empty />

  return <Table data={data} />
}

// ❌ 避免：直接在组件中 fetch
function OrderList() {
  const [orders, setOrders] = useState([])
  useEffect(() => {
    fetch('/api/orders').then(/* ... */) // 不要这样做
  }, [])
}
```

## AI 流式请求

```tsx
// 使用 Vercel AI SDK
import { useChat } from 'ai/react'

function ChatPanel() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat({
    api: '/api/chat',
  })
  // isLoading 对应 loading 态，error 对应错误态
}
```
