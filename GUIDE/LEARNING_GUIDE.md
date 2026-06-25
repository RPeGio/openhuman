# OpenHuman 学习路线图

基于对项目架构的深入分析，以下是针对不同目标的分层学习建议。

---

## 前置知识准备

| 技术 | 需要掌握的程度 | 这个项目里用在哪 |
| --- | --- | --- |
| **Rust** | 中级+ (async/tokio, traits, serde) | 整个核心引擎 `src/` |
| **TypeScript + React** | 中级 (hooks, Redux, Vite) | 前端 UI `app/src/` |
| **Tauri v2** | 入门即可 | `app/src-tauri/` — 桌面壳 |
| **LLM 基础概念** | 了解 prompt / token / embedding / tool-calling | agent、inference、memory |
| **MCP 协议** | 了解即可 | `mcp_client/` `mcp_server/` |

---

## 项目架构总览

这是一个 **AI 桌面代理平台**，由两大部分组成：**Rust 核心引擎** 和 **TypeScript/React 前端界面**，通过 Tauri 框架桥接。

### 顶层结构

| 目录 | 语言 | 角色 |
| --- | --- | --- |
| `src/` | Rust | 核心引擎 (`openhuman_core` lib + 多个 binary) |
| `app/` | TypeScript/React | 桌面 UI (Tauri + Vite) |
| `tests/` | Rust | E2E 集成测试 |
| `scripts/` | JS/Bash/Python | 构建/CI/诊断脚本 |
| `docs/` | Markdown | 设计文档 |
| `e2e/` | Docker | E2E 测试容器编排 |
| `remotion/` | TypeScript | 视频生成 (Remotion) |
| `packages/` | — | 各平台打包配置 (deb/homebrew/npm) |

### Rust 核心 (`src/`) 架构

**入口**：`src/main.rs` — Sentry 初始化 + 密钥脱敏 + 委托给 `openhuman_core::run_core_from_args`

```
src/
├── main.rs          # 二进制入口，Sentry + secret scrubbing
├── lib.rs           # 库根，导出 api / core / openhuman / rpc 四大模块
├── api/             # REST API (config, jwt, models, socket)
├── core/            # CLI 调度、事件总线 (event_bus)、JSON-RPC、日志
├── openhuman/       # ★ 核心业务逻辑 (~100+ 子模块)
└── rpc/             # RPC 分发层
```

**`src/openhuman/` 核心子模块**：

| 模块 | 职责 |
| --- | --- |
| `agent/` | Agent 运行时：harness、session、tool_loop、subagent_runner、prompts、triage |
| `agent_orchestration/` | 多 Agent 编排：command_center、agent_teams、workflow_runs、subagent 派发 |
| `agent_registry/` | 内置 Agent 定义（archivist、planner、researcher、orchestrator … 20+ 种） |
| `inference/` | LLM 推理：provider 路由（OpenAI/Anthropic/Ollama/Claude Code）、流式、可靠层 |
| `tools/` | 工具系统：computer、mcp、file、web |
| `memory/` → `memory_tree/` → `memory_sync/` | 记忆系统：图谱存储、向量嵌入、外部源同步 |
| `channels/` | 多渠道接入：Telegram/Slack/Discord/WhatsApp/微信/邮件… |
| `composio/` | Composio 集成平台：OAuth、工具发现、连接管理 |
| `mcp_client/` / `mcp_server/` / `mcp_registry/` | MCP 协议支持：客户端/服务端/注册中心 |
| `config/` | 配置系统：DaemonConfig、schema 定义、模型配置 |
| `accessibility/` | 桌面自动化（UI Automation / 视觉点击 / 键盘控制） |
| `learning/` | 用户学习/记忆：对话摘要、用户档案 |
| `approval/` | 工具执行审批门控 |
| `billing/` / `cost/` | 计费与成本追踪 |
| `credentials/` | 账户认证与 Session 管理 |
| `cron/` | 定时任务调度 |
| `keyring/` | 密钥存储（加密文件后端） |
| `embeddings/` | 嵌入模型（OpenAI/Cohere/Voyage/Ollama） |
| `voice/` | 语音：本地转写 (Whisper)、TTS、流式 |
| `agentbox/` | 沙箱代码执行 |
| `autocomplete/` | 智能自动补全 |
| `codegraph/` | 代码图索引与搜索 |
| `context/` | 对话上下文管理与摘要 |
| `heartbeat/` / `health/` | 心跳与健康检查 |

**二进制入口**：
- `openhuman-core` — 主进程
- `slack-backfill` — Slack 数据回填
- `gmail-backfill-3d` — Gmail 回填
- `memory-tree-init-smoke` — 记忆树冒烟测试
- `inference-probe` — 推理探针
- `test-mcp-stub` — MCP 测试桩

### 前端 (`app/`) 架构

**技术栈**：React 19 + TypeScript + Vite + Tauri 2 + Redux + Tailwind

```
app/src/
├── main.tsx              # 前端入口
├── App.tsx / AppRoutes   # 根组件与路由
├── pages/                # 页面：Home, Conversations, Intelligence, Skills, Settings …
├── components/           # UI 组件库
│   ├── chat/             # 聊天界面（气泡、附件、审批卡）
│   ├── intelligence/     # 知识图谱可视化（记忆面板）
│   ├── settings/         # 设置面板
│   ├── skills/           # 技能浏览器/安装器
│   ├── channels/         # 渠道配置（Telegram/Slack/Discord…）
│   └── ...
├── services/             # 后端通信层
│   ├── chatService.ts    # 聊天 SSE 流
│   ├── coreRpcClient.ts  # JSON-RPC 客户端
│   ├── transport/        # 传输层（Local/LAN/Cloud/Tunnel）
│   └── api/              # REST API 封装
├── store/                # Redux 状态管理
├── hooks/                # 自定义 React Hooks
├── features/             # 功能模块（语音、屏幕智能、钱包…）
├── lib/                  # 工具库（i18n、MCP、Composio…）
└── utils/tauriCommands/  # Tauri 桥接命令
```

### 数据流简图

```
用户 ←→ React UI (app/) ←→ Tauri Bridge ←→ Rust Core (src/)
                                                 ├── inference (LLM 调用)
                                                 ├── agent (推理循环)
                                                 ├── tools (执行动作)
                                                 ├── memory (图谱存储)
                                                 ├── channels (消息收发)
                                                 └── mcp/composio (外部集成)
```

### 关键设计特点

1. **模块化 Agent 架构**：20+ 种内置 Agent 角色（orchestrator、planner、archivist…），通过 `agent_registry` 注册，`agent_orchestration` 编排
2. **三层规则系统**（`tokenjuice`）：builtin → user → project 优先级叠加
3. **多 LLM 路由**：OpenAI / Anthropic / Ollama / Claude Code / Claude Agent SDK，带可靠层（重试+降级）
4. **多渠道接入**：Telegram、Slack、Discord、WhatsApp、微信、邮件、iMessage 等统一抽象
5. **记忆图谱**：`memory_tree` 带嵌入向量搜索 + `memory_sync` 外部源同步
6. **桌面自动化**：Windows (UIA) + macOS (Accessibility API) + 视觉点击
7. **安全设计**：Sentry 脱敏、密钥加密存储、审批门控、沙箱执行

---

## 阶段一：俯瞰全局（1-2 天）

**目标：理解项目是什么、怎么跑起来的**

1. 读完 `AGENTS.md` —— 这是代码仓库的地图
2. 精读 `gitbooks/developing/architecture.md` —— 完整的架构文档
3. 浏览 `README.md` 了解产品定位
4. 看 `CONTRIBUTING-BEGINNERS.md` 搭本地环境

**关键理解**：这个项目分三层 ——
```
React UI (app/)  ←→  Tauri 壳 (app/src-tauri/)  ←→  Rust 核心 (src/)
```

---

## 阶段二：从前端入手（推荐初学者 3-5 天）

`CONTRIBUTING-BEGINNERS.md` 建议新人从 `app/src/`（React/TypeScript）开始，不动 Rust。这是正确的。

**按这 5 个文件递进阅读：**

| 顺序 | 文件 | 学什么 |
| --- | --- | --- |
| 1 | `app/src/main.tsx` | 前端入口，Redux Store、Provider 挂载 |
| 2 | `app/src/AppRoutes.tsx` | 路由结构，有哪些页面 |
| 3 | `app/src/services/coreRpcClient.ts` | 前端怎么和 Rust 核心通信（JSON-RPC） |
| 4 | `app/src/services/chatService.ts` | 聊天消息的 SSE 流式处理 |
| 5 | `app/src/store/index.ts` | Redux Store 的完整结构 |

**建议做的小任务**：改一个 UI 文案 → 跑 `pnpm dev` 看效果 → 写个测试

---

## 阶段三：深入 Rust 核心（5-10 天）

这是项目的精髓所在。

### 3.1 先理解"骨架"

| 文件 | 作用 |
| --- | --- |
| `src/main.rs` | 程序入口：Sentry 初始化、密钥脱敏 |
| `src/lib.rs` | 库根：只导出 4 大模块 `api / core / openhuman / rpc` |
| `src/core/cli.rs` | CLI 命令分发 |
| `src/core/jsonrpc.rs` | JSON-RPC 服务器（前端调用的入口） |
| `src/rpc/dispatch.rs` | RPC 方法路由 |
| `src/api/rest.rs` | REST API（配置、认证等） |

### 3.2 再理解"大脑"——Agent 系统

按调用链从外到内读：

```
用户发消息
  → channels/providers/  (消息接入)
  → agent/harness/session/  (会话管理)
  → agent/harness/tool_loop.rs  (推理循环：调 LLM → 解析 tool call → 执行 → 回传结果)
  → agent/triage/  (消息分类与路由)
  → agent/prompts/  (prompt 构建)
  → tools/  (工具执行)
```

**重点文件**：
- `src/openhuman/agent/harness/session/runtime.rs` — Agent 运行时主循环
- `src/openhuman/agent/harness/tool_loop.rs` — 工具调用循环
- `src/openhuman/agent/triage/routing.rs` — 消息路由

### 3.3 再理解"记忆"——Memory 系统

```
memory/  →  图谱数据结构 (GraphRelation)
  ↓
memory_tree/  →  向量嵌入 + 分层存储 (L0 buffer → L1/L2 压缩)
  ↓
memory_sync/  →  外部数据源同步 (Slack/Gmail/Notion...)
```

**重点文件**：
- `src/openhuman/memory/` — 基础数据结构
- `src/openhuman/memory_tree/` — 树形存储与检索
- `src/openhuman/memory_sync/` — 外部同步引擎

---

## 阶段四：按兴趣深入子系统

根据你的兴趣选择方向：

| 兴趣方向 | 模块 | 关键文件 |
| --- | --- | --- |
| **LLM 推理** | `inference/` | `provider/router.rs`、`provider/reliable.rs`（重试+降级） |
| **多 Agent 编排** | `agent_orchestration/` | `command_center/`、`agent_teams/`、`workflow_runs/` |
| **工具系统** | `tools/` | 看 `computer/`（桌面控制）、`mcp/`（MCP 协议工具） |
| **多渠道接入** | `channels/` | `providers/telegram/`、`providers/slack.rs` |
| **桌面自动化** | `accessibility/` | `uia_interact.rs`（Windows）、`ax_interact.rs`（macOS） |
| **MCP 协议** | `mcp_client/` `mcp_server/` | 理解 AI 工具协议标准 |
| **前端聊天** | `app/src/pages/Conversations.tsx` | 聊天界面完整实现 |

---

## 学习技巧

1. **用 `codegraph explore` 追踪调用链**。例如想知道"用户发一条消息后发生了什么"：
   ```
   codegraph explore "message from user to agent tool_loop session runtime"
   ```

2. **看测试理解行为**。`tests/` 目录下有大量 E2E 测试，每个测试文件都是一个完整的使用场景：
   - `tests/agent_tool_loop_raw_coverage_e2e.rs` — Agent 工具调用循环
   - `tests/channels_telegram_*.rs` — Telegram 集成

3. **从 GitBook 文档深入**。`gitbooks/developing/` 下有专题文档：
   - `architecture/` — 子模块架构
   - `building-rust-core.md` — 如何构建 Rust 核心
   - `testing-strategy.md` — 测试策略
   - `mcp-server.md` — MCP 服务端实现

4. **先改小东西**。找 `good first issue` 标签的 issue，从改一个文案、修一个简单 bug 开始。

5. **加入社区**。Discord: `discord.tinyhumans.ai`，Reddit: `r/tinyhumansai`

---

## 总结：推荐的学习路径

```
Week 1: 搭环境 → 读架构文档 → 前端小改动
Week 2: Rust 核心骨架 (main → cli → rpc → dispatch)
Week 3: Agent 系统 (session → tool_loop → prompts)
Week 4: Memory 系统 + 一个子系统的深度阅读
Week 5+: 认领 issue，做真正的贡献
```

这个项目代码量大（Rust 核心 100+ 子模块，前端 200+ 组件），不需要一次全看懂。**先理解数据流的主干，再按需深入枝叶**，比从头读到尾高效得多。
