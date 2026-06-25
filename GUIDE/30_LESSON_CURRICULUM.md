# OpenHuman 30 节课程大纲

> 基于 [AGENTS.md](../AGENTS.md) 和 [gitbooks/developing/architecture.md](../gitbooks/developing/architecture.md) 设计，覆盖项目全部核心知识点，循序渐进。

---

## 课程总览

| 阶段 | 课次 | 主题 | 难度 |
| --- | --- | --- | --- |
| 第一阶段：入门与全貌 | 1-5 | 环境搭建、架构总览、数据流 | ⭐ 入门 |
| 第二阶段：核心基础设施 | 6-10 | RPC、配置、事件总线、模块规范 | ⭐⭐ 基础 |
| 第三阶段：前端与桌面壳 | 11-15 | React UI、Redux、Tauri、传输层 | ⭐⭐ 基础 |
| 第四阶段：Agent 大脑 | 16-20 | Agent 运行时、Tool Loop、Prompt、Triage | ⭐⭐⭐ 核心 |
| 第五阶段：记忆与知识 | 21-24 | Memory Tree、图谱、嵌入、同步 | ⭐⭐⭐ 核心 |
| 第六阶段：推理与工具 | 25-28 | LLM 路由、可靠层、工具系统、MCP | ⭐⭐⭐ 进阶 |
| 第七阶段：安全与测试 | 29-30 | 安全模型、审批门控、测试体系 | ⭐⭐ 收尾 |

---

## 第一阶段：入门与全貌（第 1-5 课）

### 第 1 课：项目概览与产品定位

**知识点**：
- OpenHuman 是什么：AI 桌面超级助手
- 支持的平台：Windows / macOS / Linux 桌面（非移动端/Web）
- 本地优先 + 托管服务可选的混合架构
- 版本状态：Early Beta，活跃开发中

**阅读材料**：
- `README.md` — 产品介绍与安装方式
- `README.zh-CN.md` — 中文 README
- `gitbooks/overview/` — GitBook 产品概述

**动手**：从 [tinyhumans.ai/openhuman](https://tinyhumans.ai/openhuman) 下载安装桌面版，体验 10 分钟

---

### 第 2 课：开发环境搭建

**知识点**：
- 必需工具链：Rust 1.93.0、Node.js ≥ 24、pnpm 10.10.0、CMake、Ninja、ripgrep
- Windows 特殊要求：VS C++ Build Tools、LLVM/Clang
- 仓库克隆与子模块初始化
- `.env` 配置与加载机制（`scripts/load-dotenv.sh`）

**阅读材料**：
- `CONTRIBUTING-BEGINNERS.md` — 初学者完整搭建指南
- `CONTRIBUTING.md` — 贡献者参考
- `gitbooks/developing/getting-set-up.md` — 详细环境文档
- `gitbooks/developing/building-rust-core.md` — Rust 核心编译

**动手**：
```bash
pnpm install
cargo check --manifest-path Cargo.toml
pnpm dev                 # 启动 Vite 开发服务器
pnpm typecheck           # TypeScript 类型检查
```

---

### 第 3 课：仓库结构与顶层架构

**知识点**：
- 三层架构：React UI → Tauri Shell → Rust Core
- 仓库布局：`app/`、`src/`、`tests/`、`scripts/`、`docs/`、`gitbooks/`
- `Cargo.toml` 多二进制入口（openhuman-core、slack-backfill、gmail-backfill-3d）
- pnpm workspace 结构（`app/` 是独立 workspace）
- 运行时范围：核心进程内嵌为 tokio task（不再使用 sidecar）

**阅读材料**：
- `AGENTS.md` — Repository layout 章节
- `gitbooks/developing/architecture.md` — 完整架构文档
- `Cargo.toml` — 依赖与二进制定义

**动手**：画出项目的三层架构图，标注各层的技术栈

---

### 第 4 课：Rust 核心骨架

**知识点**：
- `src/main.rs`：程序入口、Sentry 初始化、密钥脱敏
- `src/lib.rs`：库根，导出 `api / core / openhuman / rpc` 四大模块
- `src/core/`：传输层（CLI 调度、事件总线、JSON-RPC、日志）
- `src/api/`：REST API（config、auth、socket）
- `src/rpc/`：RPC 分发层
- `src/openhuman/`：100+ 业务域模块

**阅读材料**：
- `src/main.rs` — 入口文件（含 Sentry + secret scrubbing 完整逻辑）
- `src/lib.rs` — 库根
- `AGENTS.md` — Domain layout 章节（完整域列表）

**动手**：
```bash
cargo build --bin openhuman-core
./target/debug/openhuman-core serve   # 启动核心服务器
curl http://127.0.0.1:<port>/health   # 健康检查
```

---

### 第 5 课：项目开发工作流

**知识点**：
- 常用命令：`pnpm dev`、`pnpm dev:app`、`pnpm build`、`pnpm typecheck`
- Agent 调试运行器：`scripts/debug/`
- Git 工作流：fork → branch off upstream/main → PR
- 代码规范：ESLint + Prettier + Husky pre-push hook
- 日志规范：稳定 grep 友好前缀 `[domain]`、禁止记录密钥/PII

**阅读材料**：
- `AGENTS.md` — Commands、Git workflow、Debug logging 章节
- `AGENTS.md` — Coding philosophy 章节
- `.husky/pre-push` — Git hook

**动手**：完成一次完整的 fork → branch → 小改动 → pre-push 验证流程

---

## 第二阶段：核心基础设施（第 6-10 课）

### 第 6 课：JSON-RPC 通信机制

**知识点**：
- JSON-RPC 作为前后端通信协议
- 命名规范：`openhuman.<namespace>_<function>`
- `src/core/jsonrpc.rs` — JSON-RPC 服务器实现
- `src/rpc/dispatch.rs` — RPC 方法路由
- `app/src/services/coreRpcClient.ts` — 前端 RPC 客户端
- `core_rpc_relay` Tauri IPC 命令
- 认证：per-launch hex bearer token

**阅读材料**：
- `src/core/jsonrpc.rs` — 服务端
- `src/rpc/dispatch.rs` — 路由分发
- `src/core/jsonrpc_tests.rs` — 测试用例
- `app/src/services/coreRpcClient.ts` — 前端客户端

**动手**：追踪一个 RPC 调用从 `invoke('core_rpc_relay', ...)` 到 Rust handler 的完整路径

---

### 第 7 课：配置系统

**知识点**：
- `Config` 结构体（`src/openhuman/config/schema/types.rs`）
- TOML 文件 + 环境变量覆盖（`load.rs`）
- 前端配置中心：`app/src/utils/config.ts`（禁止直接读 `import.meta.env`）
- `.env.example` / `app/.env.example` 的分工
- 关键路径：`action_dir`（Agent 读/写根）、`workspace_dir`（内部状态）
- `is_workspace_internal_path` 写保护机制

**阅读材料**：
- `src/openhuman/config/schema/types.rs` — Config 结构体
- `src/openhuman/config/ops/loader.rs` — 配置加载
- `app/src/utils/config.ts` — 前端配置
- `.env.example` — 环境变量模板

**动手**：修改一个配置项（如 API endpoint），追踪它如何从 env → Rust Config → 前端生效

---

### 第 8 课：事件总线系统

**知识点**：
- 两种模式：Broadcast（发布/订阅）+ Native Request/Response（一对一）
- 核心类型：`DomainEvent`、`EventBus`、`NativeRegistry`、`EventHandler`
- 事件域分类：agent、memory、channel、cron、skill、tool、webhook、system
- 约定：`<Purpose>Subscriber`、`name()` → `"<domain>::<purpose>"`
- 零序列化 Native 通道（`Send + 'static`，不要求 `Serialize`）

**阅读材料**：
- `src/core/event_bus/bus.rs` — 事件总线核心
- `src/core/event_bus/events.rs` — 事件类型定义
- `src/core/event_bus/native_request.rs` — Native 请求/响应
- `src/core/event_bus/subscriber.rs` — 订阅者 trait

**动手**：添加一个自定义事件类型，在 `DomainEvent` 中注册，写 subscriber 并验证

---

### 第 9 课：域模块规范与 Controller 模式

**知识点**：
- 标准模块形态：`mod.rs` / `types.rs` / `store.rs` / `ops.rs` / `schemas.rs` / `tools.rs` / `bus.rs`
- `mod.rs` 规则：仅导出，不含业务逻辑
- Controller 迁移清单：`mod.rs` → `schemas.rs` → 接入 `src/core/all.rs` → 从 `dispatch.rs` 移除
- 工具所有权：域工具放域内 `tools.rs`，通过 `src/openhuman/tools/mod.rs` 重导出
- 禁止在 `cli.rs`/`jsonrpc.rs` 中分叉逻辑

**阅读材料**：
- `AGENTS.md` — Canonical module shape 章节
- `AGENTS.md` — Controller migration checklist 章节
- 选一个简单域（如 `health/`、`doctor/`）阅读完整文件结构

**动手**：分析 `src/openhuman/health/` 域，画出其文件→职责映射图

---

### 第 10 课：日志、监控与可观测性

**知识点**：
- Rust 端：`log` / `tracing` crate，`debug`/`trace` 级别
- 前端：命名空间 `debug` 日志
- Sentry 集成：DSN 解析、环境检测（staging/production）、before_send 过滤器
- 事件抑制规则：transient HTTP failures、session expired、budget errors 等
- 密钥脱敏：Bearer token、API key、sk-* 模式匹配
- `src/core/observability.rs`：Sentry 事件分类器

**阅读材料**：
- `src/main.rs` — Sentry 初始化与 secret scrubbing 完整代码
- `src/core/observability.rs` — 事件过滤器
- `AGENTS.md` — Debug logging 章节

**动手**：模拟一个错误场景，观察日志输出格式，确认 Sentry 脱敏规则生效

---

## 第三阶段：前端与桌面壳（第 11-15 课）

### 第 11 课：前端入口与 Provider 链

**知识点**：
- Provider 挂载顺序：`Sentry.ErrorBoundary` → `Redux Provider` → `PersistGate` → `BootCheckGate` → `CoreStateProvider` → `SocketProvider` → `ChatRuntimeProvider` → `HashRouter` → `CommandProvider` → `ServiceBlockingGate` → `AppShell`
- 认证在 `CoreStateProvider` 通过 `fetchCoreAppSnapshot()` 完成
- HashRouter 路由表
- 启动检查流程（`BootCheckGate`）

**阅读材料**：
- `app/src/main.tsx` — 前端入口
- `app/src/App.tsx` — Provider 链
- `app/src/components/BootCheckGate/` — 启动检查
- `app/src/providers/CoreStateProvider.tsx` — 核心状态

**动手**：画出完整的 Provider 挂载链，标注每个 Provider 的职责

---

### 第 12 课：Redux 状态管理

**知识点**：
- Redux Toolkit slices：`accounts`、`channelConnections`、`chatRuntime`、`coreMode`、`mascot`、`notification`、`providerSurface`、`socket`、`thread` 等
- `app/src/store/index.ts` — Store 组装
- 选择器模式（`connectivitySelectors`、`socketSelectors`）
- `userScopedStorage` — 用户隔离持久化
- 偏好 Redux 而非 ad-hoc `localStorage`

**阅读材料**：
- `app/src/store/index.ts` — Store 结构
- `app/src/store/chatRuntimeSlice.ts` — 聊天运行时核心 slice
- `app/src/store/accountsSlice.ts` — 账户状态

**动手**：在 Redux DevTools 中观察一次聊天会话的 state 变化序列

---

### 第 13 课：前端通信服务层

**知识点**：
- `apiClient.ts` — HTTP 请求封装
- `socketService.ts` — Socket.io 双工通信
- `chatService.ts` — SSE 流式聊天
- `coreRpcClient.ts` — JSON-RPC 桥接
- `coreCommandClient.ts` — 核心命令接口
- 传输策略：`LocalTransport` / `LanHttpTransport` / `CloudHttpTransport` / `TunnelTransport`
- `TransportManager` — 多 Profile 切换

**阅读材料**：
- `app/src/services/chatService.ts` — SSE 流处理
- `app/src/services/coreRpcClient.ts` — RPC 客户端
- `app/src/services/transport/TransportManager.ts` — 传输管理

**动手**：追踪一条聊天消息从前端发送到 Rust 核心再返回的完整通信路径

---

### 第 14 课：React 组件与页面架构

**知识点**：
- 页面路由：Welcome → Onboarding → Home / Human / Intelligence / Skills / Chat / Channels / Settings
- 核心组件族：`chat/`（聊天界面）、`intelligence/`（知识图谱）、`skills/`（技能市场）、`settings/`（配置面板）
- UI 组件库：`Button`、`Card`、`ModalShell`、`PanelScaffold`、`TwoPanelLayout`
- 视觉设计 Token：ocean `#4A83DD`、sage/amber/coral 语义色、Inter + Cabinet Grotesk + JetBrains Mono 字体

**阅读材料**：
- `app/src/AppRoutes.tsx` — 路由定义
- `app/src/pages/Conversations.tsx` — 聊天页面
- `app/src/components/ui/` — UI 组件库
- `app/tailwind.config.js` — 设计 Token

**动手**：在 Intelligence 页面中找一个组件（如 `MemoryControls`），追踪它如何获取数据并渲染

---

### 第 15 课：Tauri 桌面壳

**知识点**：
- `app/src-tauri/` 结构：薄桌面宿主
- 核心模块：`core_process`、`core_rpc`、`cdp`、`cef_preflight`、`window_state`
- IPC 命令：`greet`、`core_rpc_relay`、`core_rpc_token`、`start_core_process`、`restart_core_process`
- CEF 子 WebView 管控：禁止新增 JS 注入
- 平台扫描器：`discord_scanner`、`slack_scanner`、`telegram_scanner`、`whatsapp_scanner`
- 深度链接：macOS `.app` bundle、Windows 注册表

**阅读材料**：
- `gitbooks/developing/architecture/tauri-shell.md` — Tauri 壳架构
- `app/src-tauri/src/core_process.rs` — 核心进程管理
- `app/src-tauri/src/core_rpc.rs` — RPC 中继

**动手**：运行 `pnpm dev:app`，观察 Tauri 日志中 core process 的启动序列

---

## 第四阶段：Agent 大脑（第 16-20 课）

### 第 16 课：Agent 运行时架构

**知识点**：
- Agent 生命周期：创建 → 会话 → 推理循环 → 终止
- `agent/harness/`：Agent 核心引擎
  - `session/` — 会话管理
  - `tool_loop.rs` — 工具调用循环
  - `parse.rs` — 响应解析
  - `instructions.rs` — 指令构建
- `agent/harness/engine/` — 引擎核心
- `agent/schemas.rs` — Agent RPC 接口

**阅读材料**：
- `gitbooks/developing/architecture/agent-harness.md` — Agent 引擎文档
- `src/openhuman/agent/harness/mod.rs` — 引擎入口
- `src/openhuman/agent/schemas.rs` — Agent RPC schemas

**动手**：从 `agent/schemas.rs` 的 RPC handler 开始，追踪创建 Agent 的完整流程

---

### 第 17 课：Tool Loop —— Agent 推理循环

**知识点**：
- Tool Loop 主循环：调 LLM → 解析响应 → 提取 tool call → 执行工具 → 回传结果 → 循环
- 工具调用策略：并行 vs 串行
- 最大迭代次数限制与终止条件
- `tool_result_artifacts/` — 工具结果产物管理
- `stop_hooks.rs` — 停止条件判断
- `tool_filter.rs` — 工具过滤与准入

**阅读材料**：
- `src/openhuman/agent/harness/tool_loop.rs` — 工具循环
- `src/openhuman/agent/harness/tool_loop_tests.rs` — 测试
- `src/openhuman/agent/stop_hooks.rs` — 停止钩子
- `src/openhuman/agent/harness/tool_filter.rs` — 工具过滤

**动手**：画出一轮 Tool Loop 的时序图，标注每个步骤涉及的模块

---

### 第 18 课：Prompt 构建系统

**知识点**：
- Prompt 分层结构：SOUL.md / IDENTITY.md / USER.md
- `agent/prompts/builder.rs` — Prompt 组装器
- `agent/prompts/sections.rs` — Prompt 分段
- 动态上下文注入：`memory_context`、`worktree_context`、`connected_identities`
- Token 预算管理（`token_budget.rs`）
- `agent/harness/model_vision_context.rs` — 多模态上下文

**阅读材料**：
- `src/openhuman/agent/prompts/` — 完整 prompt 系统
- `src/openhuman/agent/prompts/SOUL.md` — Agent 灵魂定义
- `src/openhuman/agent/prompts/USER.md` — 用户上下文模板
- `src/openhuman/agent/prompts/IDENTITY.md` — 身份模板
- `src/openhuman/agent/harness/token_budget.rs` — Token 预算

**动手**：修改 `SOUL.md` 中的一行，观察对最终 prompt 的影响（用 `scripts/debug-agent-prompts.sh` 导出 prompt）

---

### 第 19 课：Triage 消息分类与路由

**知识点**：
- `agent/triage/`：消息入口的三级路由
- `decision.rs` — 分类决策
- `routing.rs` — 路由分发
- `escalation.rs` — 升级策略
- `evaluator.rs` — 质量评估
- `envelope.rs` — 消息信封结构
- `events.rs` — Triage 事件

**阅读材料**：
- `src/openhuman/agent/triage/mod.rs` — Triage 入口
- `src/openhuman/agent/triage/routing.rs` — 路由逻辑
- `src/openhuman/agent/triage/decision.rs` — 决策引擎
- `src/openhuman/agent/triage/routing_tests.rs` — 路由测试

**动手**：分析一条 Telegram 消息如何经过 Triage → Agent → Tool Loop 处理

---

### 第 20 课：Subagent 与多 Agent 编排

**知识点**：
- Subagent 派发机制：`spawn_subagent`、`spawn_parallel_agents`、`spawn_worker_thread`
- `agent_orchestration/command_center/` — 指挥中心
- `agent_orchestration/agent_teams/` — Agent 团队
- `agent_orchestration/workflow_runs/` — 工作流引擎
- `agent_orchestration/running_subagents.rs` — 运行中的子 Agent 追踪
- `agent/harness/subagent_runner/` — 子 Agent 执行器

**阅读材料**：
- `src/openhuman/agent_orchestration/tools/spawn_subagent.rs` — 子 Agent 派发
- `src/openhuman/agent_orchestration/command_center/control.rs` — 指挥控制
- `src/openhuman/agent_orchestration/agent_teams/runtime.rs` — 团队运行时

**动手**：追踪一次 `spawn_parallel_agents` 调用，理解多个 Agent 如何并发执行并汇总结果

---

## 第五阶段：记忆与知识（第 21-24 课）

### 第 21 课：Memory 数据模型

**知识点**：
- 知识图谱基础：`GraphRelation`（subject, predicate, object）
- `memory/` — 基础数据结构和 API
- `memory_store/` — 持久化存储（SQLite via `rusqlite`）
- `memory_graph/` — 图谱遍历与查询
- `memory_entities/` — 实体管理
- `memory_queue/` — 写入队列
- `memory_diff/` — 差异计算
- `memory_tools/` — Agent 可调用的记忆工具

**阅读材料**：
- `src/openhuman/memory/mod.rs` — 记忆域入口
- `src/openhuman/memory_store/` — 存储层
- `src/openhuman/memory_graph/` — 图谱层

**动手**：创建一个 `GraphRelation`，调用 `add_relation` 写入，再用 `query_relations` 读出

---

### 第 22 课：Memory Tree —— 分层向量存储

**知识点**：
- 三层架构：L0（缓冲区）→ L1（压缩层）→ L2（归档层）
- Seal 机制：当 L0 超过 token 阈值时自动封存
- `LlmSummariser`：L0 → L1 压缩摘要
- 嵌入检索：向量相似度搜索
- `memory_tree/` 核心数据结构

**阅读材料**：
- `gitbooks/developing/architecture/memory-tree.md` — Memory Tree 架构
- `src/openhuman/memory_tree/` — Memory Tree 实现
- `docs/memory-sync-functions.md` — 同步函数说明

**动手**：理解 "seal-cascade" 流程：一条消息进入 L0 → 触发 seal → LlmSummariser 压缩 → embedder 生成向量

---

### 第 23 课：Embeddings 嵌入模型

**知识点**：
- 嵌入提供者：OpenAI、Cohere、Voyage、Ollama（本地）
- `embeddings/provider_trait.rs` — 统一 trait
- `embeddings/factory.rs` — 工厂创建
- 速率限制与重试（`rate_limit.rs`、`retry_after.rs`）
- 云端 vs 本地嵌入的选择策略

**阅读材料**：
- `src/openhuman/embeddings/mod.rs` — 入口
- `src/openhuman/embeddings/openai.rs` — OpenAI 适配
- `src/openhuman/embeddings/ollama.rs` — Ollama 本地适配
- `src/openhuman/embeddings/provider_trait.rs` — Provider trait

**动手**：配置本地 Ollama 嵌入模型，验证向量生成与相似度搜索

---

### 第 24 课：外部数据源同步

**知识点**：
- `memory_sync/`：与外部平台同步记忆
- 同步引擎：Slack、Gmail、Notion 等
- `memory_sources/` — 数据源注册
- `memory_conversations/` — 对话记忆提取
- `memory_archivist/` — 归档 Agent
- 去重与冲突解决

**阅读材料**：
- `src/openhuman/memory_sync/` — 同步引擎
- `src/bin/slack_backfill.rs` — Slack 回填工具
- `src/openhuman/composio/ops/execute.rs` — Composio 执行

**动手**：运行 `slack-backfill` 二进制，观察 Slack 消息如何流入记忆系统

---

## 第六阶段：推理与工具（第 25-28 课）

### 第 25 课：LLM 推理提供者体系

**知识点**：
- 支持的提供者：OpenAI、Anthropic、Ollama（本地）、Claude Code、Claude Agent SDK、OpenHuman Backend
- `inference/provider/` — 统一抽象
  - `traits.rs` — Provider trait 定义
  - `factory.rs` — 工厂与注册
  - `router.rs` — 路由选择
- `inference/provider/compatible.rs` — OpenAI 兼容层
- `inference/local/ollama.rs` — Ollama 本地推理
- `inference/model_context.rs` — 模型上下文窗口管理

**阅读材料**：
- `src/openhuman/inference/provider/mod.rs` — Provider 入口
- `src/openhuman/inference/provider/traits.rs` — 统一接口
- `src/openhuman/inference/provider/router.rs` — 路由逻辑
- `src/openhuman/inference/schemas.rs` — 推理配置

**动手**：添加一个新 LLM 提供者（如自定义 endpoint），注册到 factory 并验证

---

### 第 26 课：可靠层与容错机制

**知识点**：
- `inference/provider/reliable.rs` — 可靠推理层
- 重试策略：指数退避 + 抖动
- 降级链路：主提供者 → 备用提供者 → 本地模型
- 瞬态错误分类（429/408/502/503/504）
- `provider/ops/` — 错误处理与重试
- `provider/billing_error.rs` — 计费错误
- `provider/config_rejection.rs` — 配置拒绝

**阅读材料**：
- `src/openhuman/inference/provider/reliable.rs` — 可靠层
- `src/openhuman/inference/provider/reliable_tests.rs` — 测试
- `src/openhuman/inference/provider/ops/` — 操作层

**动手**：模拟一个服务端 429 错误，观察可靠层的退避重试行为

---

### 第 27 课：工具系统架构

**知识点**：
- 工具分类：内置工具 vs 外部工具（MCP/Composio）
- `tools/mod.rs` — 工具注册表
- `tools/impl/` — 内置工具实现
- `tool_registry/` — 工具注册中心
- `tool_timeout/` — 工具超时控制
- `agent/tools.rs` — Agent 级工具定义
- `agent/tool_policy.rs` — 工具使用策略

**阅读材料**：
- `src/openhuman/tools/mod.rs` — 工具注册
- `src/openhuman/tool_registry/` — 注册中心
- `src/openhuman/tool_timeout/` — 超时控制

**动手**：注册一个自定义工具（如 `get_weather`），在 Tool Loop 中验证 LLM 能正确调用

---

### 第 28 课：MCP 协议与 Composio 集成

**知识点**：
- MCP 协议三件套：`mcp_client/`、`mcp_server/`、`mcp_registry/`
- MCP 工具发现与调用流程
- `mcp_registry/registries/` — 官方注册表 + Smithery
- Composio 集成平台：
  - `composio/ops/` — 连接管理、工具发现、Trigger
  - OAuth 握手（`oauth_handoff.rs`）
  - 直连模式 vs 托管模式
- `integrations/` — 第三方集成（Apify、Google Places、Twilio）

**阅读材料**：
- `src/openhuman/mcp_client/` — MCP 客户端
- `src/openhuman/mcp_server/` — MCP 服务端
- `src/openhuman/composio/mod.rs` — Composio 入口
- `gitbooks/developing/architecture/mcp-registry.md` — MCP 注册架构

**动手**：配置一个 MCP 服务器，观察工具自动发现与调用的完整流程

---

## 第七阶段：安全与测试（第 29-30 课）

### 第 29 课：安全模型与审批机制

**知识点**：
- `security/policy.rs` — 安全策略引擎
- 自主权层级：`readonly` / `supervised` / `full`
- 路径隔离：`is_workspace_internal_path` + `action_dir` 限制
- 命令分类：`classify_command` → `Read`/`Write`/`Network`/`Install`/`Destructive`
- `approval/gate.rs` — 审批门控（默认开启）
  - 10 分钟 TTL → 自动拒绝
  - 交互式 vs 后台/Cron 差异化处理
- 沙箱后端：Docker / Landlock / Seatbelt / AppContainer
- `cwd_jail/` — 工作目录监禁
- `prompt_injection/` — 提示注入防护
- `keyring/` — 密钥加密存储（`encrypted_file_backend.rs`）
- TLS 配置（`tls/`）

**阅读材料**：
- `src/openhuman/security/policy.rs` — 安全策略
- `src/openhuman/approval/gate.rs` — 审批门控
- `src/openhuman/cwd_jail/` — 目录监禁
- `src/openhuman/prompt_injection/` — 注入防护
- `gitbooks/developing/architecture/security.md` — 安全架构

**动手**：修改自主权层级为 `readonly`，验证工具调用被正确拦截

---

### 第 30 课：测试体系与质量保障

**知识点**：
- 四层测试金字塔：
  - Rust 单元测试：`#[cfg(test)] mod tests` 同文件
  - Rust 集成测试：`tests/*.rs`（`json_rpc_e2e.rs` 为核心）
  - Vitest 前端测试：`*.test.ts(x)` 同目录
  - WDIO E2E：`app/test/e2e/specs/*.spec.ts`
- 覆盖率要求：变更行 ≥ 80%（`diff-cover` + `cargo-llvm-cov`）
- CI 检查：类型检查、lint、测试、覆盖率、i18n 完整性
- Mock 后端：`scripts/mock-api-core.mjs` + `scripts/mock-api-server.mjs`
- E2E 测试运行：Linux `tauri-driver`、macOS Appium Mac2
- i18n 强制完整性：14 种语言翻译同步

**阅读材料**：
- `gitbooks/developing/testing-strategy.md` — 测试策略
- `gitbooks/developing/e2e-testing.md` — E2E 测试
- `tests/json_rpc_e2e.rs` — JSON-RPC E2E 测试
- `docs/TEST-COVERAGE-MATRIX.md` — 覆盖矩阵

**动手**：为一个域模块编写完整的四层测试（Rust unit → Rust integration → Vitest → WDIO E2E roadmap）

---

## 附录

### A. 知识点覆盖矩阵

| 域模块 | 覆盖课时 |
| --- | --- |
| `api/` | 第 4 课 |
| `core/` (cli, jsonrpc, event_bus) | 第 4、6、8 课 |
| `rpc/` | 第 6 课 |
| `config/` | 第 7 课 |
| `app/src/` (React, Redux, Services) | 第 11-14 课 |
| `app/src-tauri/` | 第 15 课 |
| `agent/` (harness, tool_loop, prompts, triage) | 第 16-19 课 |
| `agent_orchestration/` | 第 20 课 |
| `agent_registry/` | 第 20 课 |
| `memory/` + `memory_store/` + `memory_graph/` | 第 21 课 |
| `memory_tree/` | 第 22 课 |
| `embeddings/` | 第 23 课 |
| `memory_sync/` + `memory_sources/` | 第 24 课 |
| `inference/` (provider, local) | 第 25-26 课 |
| `tools/` + `tool_registry/` + `tool_timeout/` | 第 27 课 |
| `mcp_client/` + `mcp_server/` + `mcp_registry/` + `composio/` | 第 28 课 |
| `security/` + `approval/` + `cwd_jail/` + `keyring/` + `prompt_injection/` | 第 29 课 |
| 测试体系（Rust + Vitest + WDIO） | 第 30 课 |
| `channels/` | 第 2、15、19 课 |
| `credentials/` | 第 11 课 |
| `cron/` | 第 29 课 |
| `autocomplete/` | 第 14 课 |
| `accessibility/` | 第 27 课 |
| `learning/` | 第 21 课 |
| `billing/` + `cost/` | 第 26 课 |
| `voice/` | 第 25 课 |
| `agentbox/` | 第 29 课 |
| `context/` | 第 18 课 |
| `health/` + `heartbeat/` | 第 10 课 |
| `update/` | 第 15 课 |
| `sandbox/` | 第 29 课 |
| `tokenjuice/` | 第 18 课 |

### B. 推荐学习节奏

| 模式 | 频率 | 周期 |
| --- | --- | --- |
| 全日制 | 每天 2 课 | 3 周 |
| 业余深入学习 | 每周 3-4 课 | 8-10 周 |
| 业余快速浏览 | 每天 1 课 | 1 个月 |

### C. 每个阶段结束的检查点

- **第一阶段结束**：能独立搭建环境、运行 `pnpm dev:app`、理解三层架构
- **第二阶段结束**：能添加一个带 RPC 接口的新域模块
- **第三阶段结束**：能修改前端 UI 并追踪到 Rust 核心的数据变更
- **第四阶段结束**：能理解 Agent 完整生命周期并编写自定义 Agent 行为
- **第五阶段结束**：能操作记忆图谱、配置嵌入模型和外部同步
- **第六阶段结束**：能添加 LLM 提供者、自定义工具、对接 MCP 服务器
- **第七阶段结束**：能为项目贡献完整的、带测试覆盖的 PR
