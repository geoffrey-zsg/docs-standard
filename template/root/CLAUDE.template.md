# CLAUDE.md

> Claude 及其他 AI Agent 在本项目中的执行手册。

## 项目信息
- 项目名称：
- 技术栈：
- 当前阶段：
- 目录结构：

## 文档地图

### 必读文档（开始任何任务前）
- `docs/PLANS.md` - 当前任务、目标、验收标准、已知约束

### 核心规范（按需查阅）
- `docs/ARCHITECTURE.md` - 架构设计、分层规则、模块职责
- `docs/FRONTEND.md` - 前端开发规范总纲
- `docs/DESIGN.md` - 设计系统、组件规范、视觉标准
- `docs/QUALITY.md` - 代码质量、测试策略、性能要求

### 专项文档（特定场景查阅）
- `docs/frontend/api.md` - API 请求层详细规范
- `docs/frontend/components.md` - 组件开发详细规范
- `docs/frontend/state.md` - 状态管理详细规则
- `docs/frontend/styling.md` - 样式开发详细规范


## 指令优先级

冲突时按此优先级：项目规范文档 > 项目自定义 Skills > 第三方 Skills > AI 默认知识

**示例**：`FRONTEND.md` 规定用 TanStack Query，即使第三方 Skill 推荐 SWR，也必须用 TanStack Query。

## 工作原则
- 最小改动，复用优先
- 类型边界明确，避免 any 扩散
- 修复根因，不打补丁
- 契约先行，功能验收与视觉验收分离

## 问题记录

发现以下情况时，记录到 `CHANGELOG.md` 的"⚠️ 需要人工决策"章节：
- 破坏性变更（影响 API、数据结构、用户行为）
- 需求冲突、架构疑问、规范缺失

格式：`- [ ] **类型**：描述 | 时间 | 影响范围 | AI建议 | @决策人`

## 文件修改与校验

### 允许修改
任务相关的业务代码、测试、文档

### 禁止修改
- 生成文件（`src/generated` 或带 `AUTO-GENERATED` 标记）
- 与任务无关的模块
- 全局配置（除非任务明确要求）

### 校验命令
- UI 改动：`pnpm lint && pnpm typecheck`
- 逻辑改动：`pnpm lint && pnpm typecheck && pnpm test`
- 跨页面流程：`pnpm lint && pnpm typecheck && pnpm test && pnpm test:e2e`

## 服务管理

启动开发服务前：
1. 检查端口占用：`lsof -i :<端口>` 或 `netstat -ano | findstr <端口>`
2. 若已占用，检查服务健康：`curl http://localhost:<端口>/health`
3. 若不健康，清理后重启

禁止：不检查就重复启动、端口冲突时换端口、认为"端口可访问"就是"服务健康"

## 验收标准

### 功能验收
交互流程完整、测试通过、无明显报错

### 视觉验收
必须对截图复核：样式生效、布局合理、交互状态完整（hover/focus/disabled）、错误态/空态/加载态美观。若接近裸 HTML 或布局失衡，不得通过。

### 空仓库判断
开始前判断是"空仓库初始化"还是"已有工程迭代"。空仓库先搭基础，已有工程先读代码、最小改动。

## 自我完善

### 触发时机
- 同类错误出现 2 次以上 → 在本文件或 QUALITY.md 中新增约束
- 发现规范缺失 → 补充到对应文档
- 发现设计缺陷 → 记录到 CHANGELOG.md 并更新 ARCHITECTURE.md

### 文档行数控制
目标：每个 md 文件 ≤ 200 行。超过时识别冗余、合并相似内容、精简表达，必要时拆分文档。

## 多 Agent 协作

### 适用场景
✅ 独立功能模块、信息收集和校验、质量提升阶段
❌ 同一文件修改、架构未稳定前、强依赖任务、共享状态/目录/样式定义

核心原则：优先串行，确认瓶颈后再并行。并行主要用于信息收集和校验，而非代码实现。

### 契约要求
类型契约、API 契约、组件契约、状态契约必须先定义。共享契约在设计阶段完成，开发阶段只读不改。

### 文件边界
各 Agent 只修改分配的文件，共享类型文件由一个 Agent 负责。

### 服务管理
明确谁负责启动服务，统一使用固定端口，启动前先检查现有服务，禁止重复启动。

### 集成验证
各 Agent 完成后必须通过 typecheck，集成前运行 test，最终验证运行 build。

## 常见坑位
（根据项目填写，示例：）
- `src/shared/api/generated` 由 OpenAPI 生成，修改 schema 后重新生成
- 表格筛选状态统一放 URL query
- 开发服务启动前先检查端口占用

## 速查卡

| 场景 | 规则 |
|------|------|
| UI 改动 | `pnpm lint && pnpm typecheck` |
| 逻辑改动 | 上述 + `pnpm test` |
| 新代码落点 | shared → entities → features → app |
| 组件命名 | 文件 PascalCase，目录 kebab-case |
| 禁止 | 硬编码颜色 / any 扩散 / 组件内直接 fetch / 修改生成文件 |
