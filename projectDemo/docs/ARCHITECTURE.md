# ARCHITECTURE.md

## 1. 文档目的

本文档用于定义项目的系统边界、目录分层、依赖方向和核心架构约束。让人和 AI 在改动代码前快速知道：模块如何划分、依赖如何流动、新功能落在哪一层。

## 2. 项目概览

- 项目名称：projectDemo
- 业务目标：演示项目，验证 AI 协作开发文档体系的适用性
- 当前阶段：初始化 / 文档评估
- 核心技术栈：Next.js 15（App Router）· TypeScript · Shadcn/UI · Tailwind · Zustand · TanStack Query · Vercel AI SDK

## 3. 架构原则

- 按业务模块组织代码，不按技术类型堆文件
- 依赖方向单向流动，低层不依赖高层
- 外部数据在边界处完成类型校验和转换
- 共享能力集中沉淀，避免业务模块间复制实现

## 4. 目录结构

```txt
src/
├── app/            # Next.js App Router：路由、页面、全局 layout/provider
├── features/       # 按业务模块拆分
├── entities/       # 跨模块复用的核心业务实体
├── shared/         # 通用组件、工具、请求层、hooks
│   ├── ui/         # 基础 UI 组件（封装 Shadcn/UI）
│   ├── api/        # API 客户端和拦截器
│   └── hooks/      # 通用 hooks
└── generated/      # 自动生成代码，禁止手改
```

说明：
- `app/`：仅负责路由和应用装配，不承载具体业务实现
- `features/`：页面和业务流程的主要落点
- `entities/`：跨业务模块共享的业务对象和核心模型
- `shared/`：与具体业务弱相关的通用能力
- `generated/`：由脚本或契约生成的代码

## 5. 分层规则

推荐依赖方向：

```txt
app -> features -> entities -> shared
```

约束：
- `shared` 不能依赖 `features`
- `entities` 不能依赖具体页面实现
- `features` 间避免直接互相调用内部实现
- `app` 负责组装，不负责写业务逻辑

### 模块边界（支持多 Agent 并行）
- `features/` 下各模块相互独立，可并行开发
- `shared/` 为稳定公共层，变更需谨慎协调
- 跨模块通信通过类型化事件/API，不直接调用

## 6. 数据边界

- 所有接口响应在请求层完成解析，页面组件不处理原始后端结构
- 表单输入提交前统一校验（React Hook Form + Zod）
- URL 参数、Local Storage、Query 参数都视为不可信输入

## 7. 状态划分

- 全局状态：Zustand（用户信息、主题、全局配置）
- 页面级状态：useState / useReducer（UI 交互）
- 表单状态：React Hook Form
- 服务端状态：TanStack Query（从后端获取的数据，不重复存为全局状态）

推荐原则：
- 短生命周期状态优先本地化
- 服务端状态和客户端派生状态分离
- 筛选/分页/排序等可分享状态放 URL query

## 8. 核心约束

- 禁止在页面组件中直接发请求
- 禁止跨业务模块引用私有实现
- 禁止手改 `generated/` 下文件
- Server Component 不能使用 `useState`/`useEffect`，需标记 `'use client'`

## 9. 新功能落点指南

当新增一个功能时，默认判断顺序：
1. 是否属于已有业务模块 → 落入对应 `features/<模块>`
2. 是否可复用已有实体模型 → 落入 `entities/`
3. 是否应该进入共享层 → 落入 `shared/`
4. 是否需要补充新的架构说明 → 更新本文档
