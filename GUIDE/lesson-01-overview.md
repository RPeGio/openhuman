# 第 1 课：项目概览与产品定位

> **难度**：⭐ 入门 | **预计时间**：60-90 分钟 | **前置要求**：无

---

## 学习目标

完成本课后，你将能够：

1. 用自己的话描述 OpenHuman 是什么、解决什么问题
2. 列出 OpenHuman 支持的平台和主要功能
3. 理解"本地优先 + 托管服务可选"的混合架构理念
4. 知道项目当前的开发阶段和社区入口
5. 在自己的电脑上安装并运行 OpenHuman 桌面应用

---

## 1. OpenHuman 是什么？

OpenHuman 是一个**开源的 AI 桌面超级助手**——它不是聊天机器人，而是一个具备持久记忆、能操作你的电脑、能接入你所有工作工具的智能代理（Agent）。

用一句话概括：**把你的 AI 助手从浏览器标签页里解放出来，让它真正生活在你的桌面上。**

### 1.1 与普通聊天 AI 的区别

| | ChatGPT / Claude 网页版 | OpenHuman |
| --- | :-- | --- |
| **运行位置** | 浏览器标签页 | 桌面原生应用 |
| **记忆方式** | 对话上下文，关闭即丢失 | 本地 Memory Tree + Obsidian 知识库，持久存储 |
| **数据感知** | 只能看到你粘贴的内容 | 自动拉取 Gmail、Slack、GitHub、Notion 等 118+ 服务 |
| **执行能力** | 只能生成文本 | 可操作文件系统、运行命令、控制桌面应用 |
| **模型选择** | 单一模型 | 自动路由：推理/快速/视觉分别选最佳模型 |
| **隐私** | 数据全部上传云端 | 记忆树和配置存在本地，你做主 |

### 1.2 核心理念

OpenHuman 的设计围绕三个原则：

1. **Simple, not config-first** — 不需要先配 YAML、写 prompt、搭环境。装好就能用，几分钟上手。
2. **Local memory, managed where needed** — 你的记忆、知识库、配置存在你的机器上。模型路由、OAuth 认证、网页搜索代理走托管服务（你也可以换成自己的）。
3. **Persistent, not stateless** — Agent 有"脸"（桌面吉祥物）、有持续运行的潜意识循环（subconscious loop）、能记住你几周前说过的事。

---

## 2. 核心功能全景

### 2.1 118+ 第三方集成 + 自动拉取

OpenHuman 通过 Composio 集成层连接你的所有工具：

- **邮件**：Gmail
- **日历**：Google Calendar
- **代码**：GitHub、GitLab、Linear、Jira
- **文档**：Notion、Google Drive
- **支付**：Stripe
- **通讯**：Slack、Discord、Telegram、WhatsApp、微信、钉钉、飞书
- ……共 118+ 服务，一键 OAuth 连接

**自动拉取（Auto-fetch）**：每 20 分钟自动轮询所有已连接服务，把新数据拉取到本地记忆树中——你早上打开电脑，Agent 已经有今天的上下文了。

### 2.2 记忆树 + Obsidian 知识库

这是 OpenHuman 最核心的差异化能力：

```
外部数据 (Gmail/Slack/GitHub/...)
        ↓  自动拉取（每20分钟）
  记忆树 (Memory Tree)
  ├── L0: 原始 chunk（≤3k tokens / Markdown）
  ├── L1: 聚合摘要
  └── L2: 更高层压缩
        ↓  同时输出
  Obsidian Vault (.md 文件)
  — 你可以在 Obsidian 中打开、浏览、编辑
```

- 所有数据存本地 **SQLite** 数据库
- 自动分块、嵌入、建立知识图谱
- Obsidian 兼容的 Markdown 文件可直接打开编辑
- 可选 `agentmemory` 后端，与 Claude Code / Cursor / Codex 等 AI 编码工具共享记忆

### 2.3 智能 Token 压缩（TokenJuice）

在内容送入 LLM 之前，TokenJuice 层会：

- HTML → Markdown 转换
- 长 URL 缩短
- 冗余工具输出去重和摘要
- 通过三层规则（builtin → user → project）可配置压缩策略
- **保留 CJK 和 emoji**，不会丢弃多字节文字

宣称可减少 **最多 80%** 的 token 消耗，同时保留相同信息量。

### 2.4 自动模型路由

不需要手动选择模型：

- **推理任务** → 自动选最强的推理模型
- **快速任务** → 自动选低延迟模型
- **视觉任务** → 自动选多模态模型

默认走 OpenHuman 托管的路由服务。也支持**本地 AI**（Ollama），可在设置中切换到本地模型。

### 2.5 内置工具集

Agent 开箱即用：

| 工具类别 | 能力 |
| --- | --- |
| **编码** | 文件系统读写、Git 操作、lint、测试、grep 搜索 |
| **网页** | 搜索、抓取、内容提取 |
| **桌面控制** | 键盘鼠标操作、UI 自动化（Windows UIA / macOS Accessibility）、视觉点击 |
| **语音** | 本地语音转文字（Whisper）、TTS（ElevenLabs）、吉祥物唇形同步 |
| **定时任务** | Cron 调度、后台自动执行 |
| **浏览器** | 浏览器自动化控制 |

### 2.6 多渠道消息

Agent 可以通过你日常使用的消息平台接收和发送消息：
Telegram、Slack、Discord、WhatsApp、微信、iMessage、邮件、IRC、Matrix 等。

### 2.7 桌面吉祥物

OpenHuman 有一个名为 "The Tet" 的桌面吉祥物：
- 有表情和动画
- 可以语音对话（语音输入 → Whisper 转写 → LLM 处理 → ElevenLabs TTS 输出）
- 可以作为参与者加入 Google Meet 会议
- 持续在后台运行

---

## 3. 架构理念：本地优先 + 托管可选

OpenHuman 不是"纯本地"也不是"纯云端"，而是一个**混合架构**：

| 组件 | 存储位置 | 说明 |
| --- | --- | --- |
| Memory Tree | **本地** SQLite | 你的知识图谱在你自己机器上 |
| Obsidian Vault | **本地** 文件系统 | `.md` 文件，你完全掌控 |
| Workspace 配置 | **本地** `~/.openhuman/` | 账户无关的本地配置 |
| 账户登录 | 托管服务 | 默认走 OpenHuman 后端 |
| 模型路由 | 托管服务 | 可选切换到本地 Ollama |
| 网页搜索代理 | 托管服务 | 可选配置自己的搜索 API |
| OAuth 集成 | 托管服务 | 可选配置自己的 Composio API key |
| 实时触发器 webhook | 托管服务 | 自部署模式下需自己托管 |

这个设计让新手可以开箱即用（托管默认值），高级用户可以选择完全自主（BYO 模型/密钥/Composio）。

---

## 4. 项目状态与社区

### 4.1 当前阶段

- **状态**：Early Beta（早期测试版）
- **活跃开发中**：期望有"rough edges"（粗糙边缘）
- **版本**：`0.57.x`（参见 `Cargo.toml`、`app/package.json`）
- **许可证**：GNU General Public License

### 4.2 支持的平台

| 平台 | 状态 |
| --- | --- |
| macOS (x64 + ARM64) | ✅ 正式支持 |
| Windows (x64 + ARM64) | ✅ 正式支持 |
| Linux (x64 + ARM64) | ✅ 正式支持 |
| iOS | 🔬 实验性（不发货） |
| Android | 🚫 不支持 |
| Web | 🚫 不支持 |

> 安装包格式：macOS `.dmg` / Homebrew、Windows `.msi`、Linux `.deb` / `.AppImage` / AUR

### 4.3 社区入口

| 渠道 | 链接 |
| --- | --- |
| Discord | [discord.tinyhumans.ai](https://discord.tinyhumans.ai/) |
| Reddit | [r/tinyhumansai](https://www.reddit.com/r/tinyhumansai/) |
| X/Twitter | [@tinyhumansai](https://x.com/tinyhumansai) |
| 文档 | [tinyhumans.gitbook.io/openhuman](https://tinyhumans.gitbook.io/openhuman/) |
| 创始人 | [@senamakel](https://x.com/senamakel) |
| GitHub | [github.com/tinyhumansai/openhuman](https://github.com/tinyhumansai/openhuman) |

---

## 5. 与同类产品的对比

| | Claude Cowork | OpenClaw | Hermes Agent | **OpenHuman** |
| --- | --- | --- | --- | --- |
| **开源** | 🚫 闭源 | ✅ MIT | ✅ MIT | ✅ **GNU** |
| **上手难度** | ✅ 桌面 + CLI | ⚠️ 终端优先 | ⚠️ 终端优先 | ✅ **分钟级上手** |
| **费用** | ⚠️ 订阅 + 附加费 | ⚠️ 自备模型 | ⚠️ 自备模型 | ✅ **一个订阅** + TokenJuice |
| **记忆** | ✅ 对话范围 | ⚠️ 依赖插件 | ✅ 自学习 | 🚀 **记忆树 + Obsidian** |
| **集成** | ⚠️ 少量连接器 | ⚠️ 自建 | ⚠️ 自建 | 🚀 **118+ OAuth** |
| **自动拉取** | 🚫 无 | 🚫 无 | 🚫 无 | ✅ **20 分钟循环** |
| **模型路由** | 🚫 单模型 | ⚠️ 手动 | ⚠️ 手动 | ✅ **内置自动** |
| **原生工具** | ✅ 仅编码 | ✅ 仅编码 | ✅ 仅编码 | ✅ **编码+搜索+抓取+语音** |

核心差异化优势：**记忆持久化 + 自动数据拉取 + Token 压缩 + 内置模型路由**。

---

## 动手任务

### 任务 1：安装桌面应用

根据你的操作系统选择安装方式：

**macOS（推荐 Homebrew）**：
```bash
brew tap tinyhumansai/core
brew install openhuman
```

**Linux（推荐 apt 签名源）**：
```bash
sudo apt-get install -y --no-install-recommends gnupg2 curl ca-certificates
curl -fsSL https://tinyhumansai.github.io/openhuman/apt/KEY.gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/openhuman.gpg
echo "deb [signed-by=/etc/apt/keyrings/openhuman.gpg arch=amd64] \
  https://tinyhumansai.github.io/openhuman/apt stable main" \
  | sudo tee /etc/apt/sources.list.d/openhuman.list
sudo apt-get update
sudo apt-get install -y openhuman
```

**Windows**：从 [GitHub Releases](https://github.com/tinyhumansai/openhuman/releases/latest) 下载 `.msi` 安装包。

**或者直接从官网下载**：[tinyhumans.ai/openhuman](https://tinyhumans.ai/openhuman)

### 任务 2：初次体验（15 分钟）

安装完成后，打开 OpenHuman，做以下事情：

1. **注册/登录账号** — 走完 onboarding 流程
2. **和 Agent 对话** — 发送 3 条消息，观察回复
3. **浏览界面** — 点击左侧导航，了解有哪些页面（Home / Intelligence / Skills / Channels / Settings）
4. **观察吉祥物** — 注意 "The Tet" 的动画和反应
5. **查看设置** — 进入 Settings 页面，浏览有哪些可配置项

### 任务 3：记录你的第一印象

创建一个笔记，回答以下问题：
- 你对 OpenHuman 的第一印象是什么？
- 和 ChatGPT / Claude 网页版相比，最大的不同是什么？
- 你最感兴趣的功能是哪个？为什么？

---

## 思考题

1. OpenHuman 的"本地记忆 + 托管服务"混合架构相比纯本地方案（如 Ollama + 本地向量数据库）和纯云端方案（如 ChatGPT）各有什么优劣？

2. TokenJuice 声称可节省 80% token 消耗——这在实际场景中可能吗？什么类型的内容最适合压缩？什么类型的内容压缩效果最差？

3. 自动每 20 分钟从 118+ 服务拉取数据——这会产生大量 API 调用和存储写入。你认为项目可能面临哪些工程挑战？

4. 如果让你给 OpenHuman 增加一个功能，你会加什么？为什么？

---

## 延伸阅读

| 材料 | 路径 |
| --- | --- |
| 产品 README | [`README.md`](../README.md) |
| 中文 README | [`README.zh-CN.md`](../README.zh-CN.md) |
| 架构文档 (入口) | [`gitbooks/developing/architecture.md`](../gitbooks/developing/architecture.md) |
| GitBook 功能总览 | [`gitbooks/SUMMARY.md`](../gitbooks/SUMMARY.md) |
| 记忆树详解 | [GitBook: Memory Trees](https://tinyhumans.gitbook.io/openhuman/features/obsidian-wiki/memory-tree) |
| Token 压缩 | [GitBook: Token Compression](https://tinyhumans.gitbook.io/openhuman/features/token-compression) |
| 模型路由 | [GitBook: Model Routing](https://tinyhumans.gitbook.io/openhuman/features/model-routing) |
| 隐私与安全 | [GitBook: Privacy & Security](https://tinyhumans.gitbook.io/openhuman/features/privacy-and-security) |

---

## 下一课预告

**第 2 课：开发环境搭建** — 安装 Rust、Node.js、pnpm、CMake 等工具链，克隆仓库，完成首次编译。为后续的源码学习做好准备。

> 💡 **提示**：在第 2 课之前，可以去 [CONTRIBUTING-BEGINNERS.md](../CONTRIBUTING-BEGINNERS.md) 预览一下搭建流程，提前了解需要的工具。
