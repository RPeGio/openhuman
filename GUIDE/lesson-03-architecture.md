# 第 3 课：仓库结构与顶层架构

> **难度**：⭐⭐ 基础 | **预计时间**：60-90 分钟 | **前置要求**：第 1、2 课

---

## 学习目标

完成本课后，你将能够：

1. 画出 OpenHuman 的三层架构图并解释每层的职责
2. 说出仓库中每个顶级目录的作用
3. 理解 pnpm workspace 的单包结构
4. 知道 Rust 核心的 6 个二进制入口及其用途
5. 用 `codegraph explore` 追踪代码调用链

---

## 1. 三层架构

OpenHuman 不是一个单一进程，而是**三个独立构建、在运行时紧密协作的层次**：

```
┌──────────────────────────────────────────────┐
│                  React 前端                    │
│             app/src/ (TypeScript)             │
│    Vite + Redux + React Router + Tailwind     │
│         用户界面 · 导航 · 状态管理             │
└────────────────────┬─────────────────────────┘
                     │  Tauri IPC Bridge
                     │  invoke('core_rpc_relay', ...)
┌────────────────────┴─────────────────────────┐
│               Tauri 桌面壳                     │
│      app/src-tauri/ (Rust + CEF Chromium)     │
│   窗口管理 · CEF webview · CDP 扫描器         │
│   启动核心进程 · 传递 RPC token               │
└────────────────────┬─────────────────────────┘
                     │  HTTP JSON-RPC
                     │  http://127.0.0.1:7788/rpc
┌────────────────────┴─────────────────────────┐
│               Rust 核心引擎                    │
│               src/ (Rust + Tokio)             │
│   Agent · Memory · Inference · Tools · RPC    │
│      业务逻辑 · JSON-RPC · 数据持久化          │
└──────────────────────────────────────────────┘
```

### 1.1 每层的职责边界

| 层 | 语言 | 职责 | 不能做什么 |
| --- | --- | --- | --- |
| **React 前端** | TypeScript | UI 渲染、路由、状态展示、用户交互 | 不执行业务逻辑、不操作文件系统（通过 RPC 委托） |
| **Tauri 壳** | Rust | 窗口管理、CDP 扫描器、CEF 内嵌 webview、桌面通知 | 不含 Agent 逻辑、不含记忆管理 |
| **Rust 核心** | Rust | **所有业务逻辑**：Agent 运行时、记忆树、推理、工具执行、渠道接入 | 不渲染 UI |

> **关键原则**（来自 `AGENTS.md`）：**Rust 核心是权威的（authoritative），前端只做呈现和编排。**

### 1.2 通信方式

| 方向 | 协议 | 路径 |
| --- | --- | --- |
| 前端 → 核心 | HTTP JSON-RPC 2.0 | `http://127.0.0.1:7788/rpc` |
| 前端 ← 核心（事件推送） | SSE (Server-Sent Events) | `http://127.0.0.1:7788/events` |
| Tauri 壳 → 核心（启动） | 进程内 tokio task | `CoreProcessHandle` 管理生命周期 |
| 核心启动时 → 前端（token） | Tauri IPC | `invoke('core_rpc_token')` |

每次启动时，Tauri 壳生成一个随机 64 位 hex bearer token，通过内存传给内嵌的核心进程，前端通过 `core_rpc_token` Tauri 命令读取。所有 `/rpc` 请求带上 `Authorization: Bearer <token>` 头。

---

## 2. 仓库布局详解

### 2.1 顶级目录

```
openhuman/
├── app/               # 前端 + Tauri 壳 (pnpm workspace 唯一包)
├── src/               # Rust 核心 (lib + bin)
├── Cargo.toml         # Rust 核心 crate 定义
├── package.json       # 根 workspace (pnpm, 仅 devDeps)
├── pnpm-workspace.yaml # workspace 声明
├── tests/             # Rust 集成测试 (~100+ E2E 用例)
├── scripts/           # 构建/CI/调试脚本 (Bash/JS/Python)
├── docs/              # 设计文档、安全审计、计划
├── gitbooks/          # GitBook 用户文档 (中英文)
├── packages/          # 打包配置 (arch/deb/homebrew/npm)
├── remotion/          # Remotion 视频生成 (吉祥物动画)
├── e2e/               # Docker 化的 E2E 测试编排
├── .github/           # CI/CD 工作流 + Dockerfile
├── fastlane/          # iOS/macOS 自动化发布
├── design-previews/   # 设计预览 HTML
└── examples/          # 示例代码
```

### 2.2 核心源码目录：`src/`

这是最重要的目录，是你学习的主战场：

```
src/
├── main.rs            # ★ 二进制入口：Sentry 初始化 → 密钥脱敏 → run_core_from_args
├── lib.rs             # ★ 库根：导出 api/core/openhuman/rpc 四大模块
├── api/               # REST API：配置 (config.rs)、JWT (jwt.rs)、Socket
├── core/              # 传输层：CLI、JSON-RPC、事件总线、日志
├── openhuman/         # ★ 150+ 业务域模块（见下节）
└── rpc/               # RPC 分发层（已迁移到控制器注册表）
```

### 2.3 前端源码目录：`app/src/`

```
app/src/
├── main.tsx           # 前端入口
├── App.tsx            # 根组件：Provider 链
├── AppRoutes.tsx      # 路由定义 (HashRouter)
├── pages/             # 页面组件
├── components/        # UI 组件库（chat/intelligence/settings/skills/...）
├── services/          # 后端通信层（RPC 客户端、SSE、传输层）
├── store/             # Redux Toolkit 状态切片
├── hooks/             # 自定义 React Hooks
├── lib/               # 工具库（i18n、MCP、Composio）
├── features/          # 功能模块（语音、屏幕智能、钱包）
├── utils/             # Tauri 桥接命令封装
├── types/             # TypeScript 类型定义
├── providers/         # Context Provider 组件
├── constants/         # 常量
└── styles/            # CSS 主题
```

---

## 3. Rust 核心骨架

### 3.1 入口：`src/main.rs`

程序启动时做的事：

1. 加载 `.env`（`dotenvy::dotenv()`）
2. 初始化 Sentry（错误监控），配置多层密钥脱敏过滤器
3. 收集命令行参数，调用 `openhuman_core::run_core_from_args(&args)`

### 3.2 库根：`src/lib.rs`

只有 4 行 `pub mod`：

```rust
pub mod api;
pub mod core;
pub mod openhuman;
pub mod rpc;
```

`run_core_from_args()` 做两件事：
1. 加载 dotenv（`core::cli::load_dotenv_for_cli`）
2. 应用启动重启延迟（`service::apply_startup_restart_delay_from_env`）
3. 初始化主密钥（`keyring::init_master_key`）
4. 委托给 CLI 分发器（`core::cli::run_from_cli_args`）

### 3.3 `src/core/` — 传输层，不含业务逻辑

| 文件 | 作用 |
| --- | --- |
| `cli.rs` | 命令行参数解析 + 子命令分发 |
| `jsonrpc.rs` | JSON-RPC 2.0 HTTP 服务器（Axum），方法调用、SSE 事件流 |
| `all.rs` | **控制器注册表** — 所有域的 RPC handler 都在这里注册 |
| `dispatch.rs` | 遗留分发器（已废弃，返回 `None`，控制器注册表是权威路径） |
| `event_bus/` | 类型化发布/订阅 + 原生请求/响应 |
| `runtime.rs` | Tokio 运行时管理 |
| `logging.rs` / `rpc_log.rs` | 日志基础设施 |
| `observability.rs` | Sentry 过滤器（瞬态错误抑制） |

**控制器注册表（`all.rs`）** 是核心中的核心——前端调用的所有 RPC 方法最终都路由到这里。它把每个域的 `ControllerSchema`（方法名、参数 schema）和 handler 函数注册在一起，CLI 和 JSON-RPC 层都通过它统一调度。

### 3.4 `src/rpc/` — 分发层

`dispatch.rs` 现在是遗留兼容层，所有新方法走控制器注册表。它包含：
- `try_dispatch()`: 对任何方法返回 `None`，让调用方回退到注册表路径

### 3.5 `src/api/` — REST API

| 文件 | 作用 |
| --- | --- |
| `config.rs` | 配置相关端点 |
| `jwt.rs` | JWT 令牌管理 |
| `rest.rs` | REST 端点实现 |
| `socket.rs` | WebSocket / Socket.io 支持 |
| `models/` | 请求/响应数据模型 |

---

## 4. 多二进制入口

`Cargo.toml` 的 `[[bin]]` 定义了 6 个可执行文件：

| 二进制名 | 入口文件 | 用途 |
| --- | --- | --- |
| `openhuman-core` | `src/main.rs` | ★ 主进程：桌面应用的核心服务器 |
| `slack-backfill` | `src/bin/slack_backfill.rs` | Slack 历史数据回填工具 |
| `gmail-backfill-3d` | `src/bin/gmail_backfill_3d.rs` | Gmail 最近 3 天数据回填 |
| `memory-tree-init-smoke` | `src/bin/memory_tree_init_smoke.rs` | 记忆树冒烟测试 |
| `inference-probe` | `src/bin/inference_probe.rs` | LLM 推理探测器 |
| `test-mcp-stub` | `src/bin/test_mcp_stub.rs` | MCP 测试桩 |

它们共享同一个 `openhuman_core` 库 crate。编译方式：

```bash
cargo build --bin openhuman-core    # 主进程
cargo build --bin slack-backfill    # Slack 工具
# 或一次性全部
cargo build --workspace
```

---

## 5. pnpm Workspace 结构

OpenHuman 的 workspace 比较特殊——**只有一个业务包**：

```yaml
# pnpm-workspace.yaml
packages:
  - "app"                        # 前端 + Tauri 壳
  - "packages/tauri-plugin-ptt"  # PTT 语音插件（仅 iOS 实验功能）
```

### 根 `package.json`

- 名称：`openhuman-repo`（private，不发布）
- 只含 `devDependencies`（husky、tsx、ws）
- 所有 `scripts` 都通过 `pnpm --filter openhuman-app` 转发给 `app/`

### `app/package.json`

- 名称：`openhuman-app`（版本与 Rust 核心同步 `0.57.x`）
- 完整的前端依赖 + Tauri 相关脚本
- 独立的 ESLint、Prettier、Tailwind 配置

### 为什么是单包 workspace？

历史上有多个包（如 npm 发布包），但 `packages/npm/` 的 `postinstall` 会下载预构建二进制导致 CI 失败，所以 workspace 不会用通配符 `packages/*` 引入所有子目录——只显式声明需要的包。

---

## 6. `src/openhuman/` 域模块速览

这是 150+ 个域模块的目录，按功能分组：

| 组别 | 模块 | 职责关键词 |
| --- | --- | --- |
| **Agent 系统** | `agent/`, `agent_orchestration/`, `agent_registry/`, `agent_tool_policy/`, `agentbox/` | 运行时、多 Agent 编排、20+ 种内置 Agent、工具策略、沙箱 |
| **推理** | `inference/`, `model_council/` | LLM 提供者路由、流式响应、可靠层、本地 AI (Ollama) |
| **记忆** | `memory/`, `memory_tree/`, `memory_sync/`, `memory_graph/`, `memory_store/`, `memory_search/`, `memory_sources/`, `embeddings/` | 知识图谱、向量嵌入、分层存储、外部源同步 |
| **渠道** | `channels/`, `webview_accounts/`, `webview_apis/`, `webview_notifications/` | Telegram/Slack/Discord/微信/邮件 等多渠道消息 |
| **集成** | `composio/`, `integrations/`, `mcp_client/`, `mcp_server/`, `mcp_registry/` | Composio OAuth 集成、MCP 协议支持 |
| **工具** | `tools/`, `tool_registry/`, `tool_timeout/`, `accessibility/`, `skills/`, `skill_registry/`, `skill_runtime/` | 工具定义与执行、桌面自动化、技能系统 |
| **安全** | `security/`, `approval/`, `cwd_jail/`, `keyring/`, `keyring_consent/`, `prompt_injection/`, `sandbox/` | 安全策略、审批门控、路径监狱、密钥加密 |
| **基础设施** | `config/`, `credentials/`, `cron/`, `health/`, `heartbeat/`, `service/`, `startup/`, `update/` | 配置、认证、定时任务、健康检查、更新 |
| **语音** | `voice/`, `audio_toolkit/` | Whisper 转写、TTS、流式处理 |
| **学习** | `learning/`, `context/`, `autocomplete/` | 用户档案学习、上下文管理、自动补全 |
| **其他** | `billing/`, `cost/`, `dashboard/`, `devices/`, `doctor/`, `encryption/`, `image/`, `javascript/`, `meet/`, `notifications/`, `people/`, `profiles/`, `referral/`, `team/`, `threads/`, `wallet/`, `webhooks/`, `workflows/` | 计费、成本、设备、诊断、加密、图像、会议、钱包…… |

---

## 7. codegraph 入门

`codegraph` 是这个项目的代码索引工具，让你不用 grep 就能追踪调用链。

### 7.1 基础用法

```bash
# 探索一个符号或问题
codegraph explore "how does the agent tool_loop work"

# 查看单个文件的结构（等同于 read_file + outline）
codegraph node src/main.rs

# 查看一个函数的定义 + 调用者
codegraph node --symbol run_core_from_args --file src/lib.rs

# 搜索符号
codegraph search --query "memory_tree" --kind function
```

### 7.2 实战示例

追踪"前端发来一条聊天消息后发生了什么"：

```bash
codegraph explore "message from chatService to agent tool_loop session runtime"
```

这会输出相关符号的源代码 + 调用路径，比你手动 grep + read_file 快 10 倍。

### 7.3 最佳实践

- 在改一个函数之前，先用 `codegraph explore` 看它的所有调用者
- 探索一个新域模块时，先用 `codegraph node <模块>/mod.rs` 看入口
- `codegraph explore` 一次能覆盖最多 12 个文件，通常一次调用就够了

---

## 动手任务

### 任务 1：画出三层架构图

在纸上或绘图工具中画出 React → Tauri → Rust Core 的三层架构，标注每层用到的技术和通信协议。

### 任务 2：用 codegraph 探索入口

```bash
# 看看 main.rs 调用了哪些函数
codegraph explore "main function entry point run_core_from_args"

# 看看 JSON-RPC 服务器如何启动
codegraph explore "jsonrpc server axum Router"
```

### 任务 3：追踪一个 RPC 调用

从 `app/src/services/coreRpcClient.ts` 开始，追踪一个 RPC 调用如何到达 Rust 核心的 handler：

1. 前端：`coreRpcClient.ts` → `invoke('core_rpc_relay', ...)`
2. Tauri 壳：`core_rpc.rs` → HTTP 转发到 `127.0.0.1:7788/rpc`
3. Rust 核心：`jsonrpc.rs` → `all.rs` 控制器注册表 → 具体 handler

用 `codegraph explore` 验证你的理解。

### 任务 4：浏览域模块

在 `src/openhuman/` 下随机选 3 个子目录，用 `codegraph node` 看它们的 `mod.rs`，然后解释这个域大致做什么。

---

## 思考题

1. 为什么 `src/rpc/dispatch.rs` 对所有方法返回 `None`？这种"遗留兼容"设计有什么好处和代价？

2. OpenHuman 为什么把核心作为 tokio task 嵌入 Tauri 壳进程（`child` 模式），而不是作为独立的 sidecar 进程？提示：`AGENTS.md` 提到 sidecar 已在 PR #1061 中移除。

3. pnpm workspace 为什么不用 `packages/*` 通配符？`packages/npm/` 的 `postinstall` 会引发什么问题？

4. `src/core/all.rs` 的控制器注册表设计——为什么要把所有域的 RPC handler 集中注册，而不是让每个域自己注册？这和 Rust 的编译模型有什么关系？

---

## 延伸阅读

| 材料 | 路径 |
| --- | --- |
| AGENTS.md（仓库地图） | [`AGENTS.md`](../AGENTS.md) |
| 架构文档 | [`gitbooks/developing/architecture.md`](../gitbooks/developing/architecture.md) |
| 前端架构 | [`gitbooks/developing/architecture/frontend.md`](../gitbooks/developing/architecture/frontend.md) |
| Tauri 壳架构 | [`gitbooks/developing/architecture/tauri-shell.md`](../gitbooks/developing/architecture/tauri-shell.md) |
| Agent Harness 架构 | [`gitbooks/developing/architecture/agent-harness.md`](../gitbooks/developing/architecture/agent-harness.md) |
| Cargo.toml | [`Cargo.toml`](../Cargo.toml) |
| pnpm workspace | [`pnpm-workspace.yaml`](../pnpm-workspace.yaml) |

---

## 下一课预告

**第 4 课：数据流全景追踪** — 从前端点击"发送消息"一路追踪到 LLM 返回结果。理解 chatService → coreRpcClient → JSON-RPC → Agent runtime → tool_loop 的完整数据流。
