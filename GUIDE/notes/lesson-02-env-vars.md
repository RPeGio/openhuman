# 第 2 课笔记：环境变量详解

> 来源：`.env.example` 文件分析，按功能分组解释每个环境变量的用法与默认值。

---

## 一、应用环境

| 变量 | 默认值 | 说明 |
| --- | --- | --- |
| `OPENHUMAN_APP_ENV` | `production` | 设为 `staging` 时指向 staging 后端，使用独立 `~/.openhuman-staging` 工作目录。⚠️ 两套环境凭证不互通 |

## 二、后端 API

| 变量 | 默认值 | 说明 |
| --- | --- | --- |
| `BACKEND_URL` | `https://api.tinyhumans.ai` | Rust 核心使用的后端 URL |
| `VITE_BACKEND_URL` | 同上 | Vite 前端使用的后端 URL |
| `VITE_CONSUMER_FIRST_SESSION` | `false` | 消费级首次会话 UX |

## 三、认证

| 变量 | 默认值 | 说明 |
| --- | --- | --- |
| `JWT_TOKEN` | 空 | Session JWT，供技能沙箱和调试脚本使用 |

## 四、核心进程

| 变量 | 默认值 | 说明 |
| --- | --- | --- |
| `OPENHUMAN_CORE_HOST` | `127.0.0.1` | 绑定地址。Docker/云部署时设 `0.0.0.0` |
| `OPENHUMAN_CORE_PORT` | `7788` | JSON-RPC 服务器端口 |
| `OPENHUMAN_CORE_RPC_URL` | `http://127.0.0.1:7788/rpc` | 核心 RPC 地址 |
| `OPENHUMAN_CORE_TOKEN` | 空 | RPC bearer token。桌面壳留空，Docker 必设 |
| `OPENHUMAN_CORE_RUN_MODE` | `child` | `child`（内嵌）/ `inprocess` |
| `OPENHUMAN_DOTENV_PATH` | 空 | 显式指定 .env 文件路径 |

## 五、运行时配置

| 变量 | 默认值 | 说明 |
| --- | --- | --- |
| `OPENHUMAN_MAX_ACTIONS_PER_HOUR` | `20` | 工具调用安全上限，`0` 禁止所有副作用 |
| `OPENHUMAN_MODEL` | 空 | 覆盖默认模型 |
| `OPENHUMAN_WORKSPACE` | `~/.openhuman` | 工作目录路径 |
| `OPENHUMAN_TEMPERATURE` | `0.7` | LLM 温度 |
| `OPENHUMAN_OUTPUT_LANGUAGE` | 空 | Agent 输出语言（如 `zh-CN`） |
| `OPENHUMAN_TOOL_TIMEOUT_SECS` | `120` | 工具超时（1-3600 秒） |

## 六、运行时标志

| 变量 | 默认值 | 说明 |
| --- | --- | --- |
| `OPENHUMAN_BROWSER_ALLOW_ALL` | `0` | `1` 允许全部 URL |
| `OPENHUMAN_LOG_PROMPTS` | `0` | `1` 记录完整 prompt |
| `OPENHUMAN_REASONING_ENABLED` | 空 | 启用推理模式 |

## 七、网页搜索

| 变量 | 默认值 | 说明 |
| --- | --- | --- |
| `OPENHUMAN_WEB_SEARCH_MAX_RESULTS` | `5` | 搜索结果上限 |
| `OPENHUMAN_WEB_SEARCH_TIMEOUT_SECS` | `10` | 搜索超时 |

支持的搜索引擎 API 密钥：`SELTZ_API_KEY`、`QUERIT_API_KEY`、`BRAVE_API_KEY`、`PARALLEL_API_KEY`，以及自托管 `OPENHUMAN_SEARXNG_*` 系列。

## 八、代理

`OPENHUMAN_PROXY_ENABLED` / `OPENHUMAN_HTTP_PROXY` / `OPENHUMAN_HTTPS_PROXY` / `OPENHUMAN_ALL_PROXY` / `OPENHUMAN_NO_PROXY`

## 九、本地 AI

| 变量 | 默认值 | 说明 |
| --- | --- | --- |
| `OPENHUMAN_LOCAL_AI_TIER` | 空 | `low` / `medium` / `high` |
| `OPENHUMAN_OLLAMA_BASE_URL` | `http://127.0.0.1:11434` | Ollama 地址 |
| `OPENHUMAN_LM_STUDIO_BASE_URL` | `http://localhost:1234/v1` | LM Studio 地址 |

二进制路径覆盖：`WHISPER_BIN`、`PIPER_BIN`、`OLLAMA_BIN`

## 十、Telegram

`OPENHUMAN_TELEGRAM_BOT_USERNAME`（默认 `openhuman_bot`）、`OPENHUMAN_TELEGRAM_API_BASE`

## 十一、钱包 RPC

`OPENHUMAN_WALLET_RPC_EVM` / `_BTC` / `_SOLANA` / `_TRON`，各有内置公共节点默认值

## 十二、x402 机器支付

预算上限（原子 USDC）：`OPENHUMAN_X402_PER_REQUEST_MAX`（1 USDC）、`_DAILY_MAX`（10 USDC）、`_MONTHLY_MAX`（100 USDC）

## 十三、技能

`SKILLS_REGISTRY_URL`、`SKILLS_LOCAL_DIR`（开发用最高优先级）、`OPENHUMAN_SKILLS_WORKING_MEMORY_ENABLED`

## 十四、Python 运行时

`OPENHUMAN_RUNTIME_PYTHON_*` 系列，控制托管 Python 的行为

## 十五、Sentry

`OPENHUMAN_CORE_SENTRY_DSN`、`SENTRY_URL`、`OPENHUMAN_BUILD_SHA`、`OPENHUMAN_ANALYTICS_ENABLED`

## 十六、Composio 记忆同步

按 provider 配置同步间隔（`OPENHUMAN_COMPOSIO_*_SYNC_INTERVAL_SECS`）和每日请求预算（`OPENHUMAN_COMPOSIO_DAILY_REQUEST_LIMIT`）

## 十七、日志

`RUST_LOG`（默认 `info`）、`RUST_BACKTRACE`（默认 `0`）

## 十八、测试

`OPENHUMAN_FILE_STATE_GUARD`、`OPENHUMAN_SERVICE_MOCK` 系列

---

## 日常开发最常用

```bash
RUST_LOG=debug                  # 详细日志
RUST_BACKTRACE=1                # 完整堆栈
OPENHUMAN_CORE_PORT=7788        # 端口
OPENHUMAN_APP_ENV=staging       # staging 环境
OPENHUMAN_OUTPUT_LANGUAGE=zh-CN # 中文输出
OPENHUMAN_MAX_ACTIONS_PER_HOUR=0 # 禁用工具执行
```
