# 技术卡 2：Rust + Tokio 异步模型入门

## 一句话

**Rust** 是一个没有垃圾回收（GC）、保证内存安全的系统编程语言。**Tokio** 是 Rust 生态中最主流的异步运行时，相当于 Node.js 的 event loop，但更精细。

## 为什么核心要用 Rust？

OpenHuman 的核心引擎需要同时做很多事：

- 处理前端发来的 RPC 请求
- 调用 LLM API（网络 IO，等待响应可能几秒到几十秒）
- 在后台同步 Gmail/Slack/Notion 数据
- 运行记忆树的嵌入和压缩计算
- 管理多个 Agent 子任务

如果用 Node.js（单线程 event loop），一个 CPU 密集型操作（比如嵌入计算）会阻塞所有其他任务。Rust + Tokio 可以做**真正的多线程异步**。

## Tokio 是什么？

```rust
// 伪代码：Tokio 的 async/await 模式
#[tokio::main]
async fn main() {
    // 同时做三件事，不互相阻塞
    tokio::join!(
        handle_rpc_requests(),     // 任务 1
        sync_gmail(),              // 任务 2
        run_memory_compression(),  // 任务 3
    );
}
```

Tokio 的核心概念：

| 概念 | 类比 Node.js | 说明 |
| --- | --- | --- |
| `async fn` | `async function` | 声明一个异步函数 |
| `.await` | `await` | 等待异步操作完成 |
| `tokio::spawn` | `Promise` + 自动调度 | 在后台启动一个异步任务 |
| `tokio::join!` | `Promise.all` | 并发等待多个任务 |

## 为什么 Tokio 比 Node.js event loop 更强？

| | Node.js | Rust + Tokio |
| --- | --- | --- |
| 线程模型 | 单线程 event loop + worker threads | 多线程 work-stealing 调度 |
| CPU 密集型任务 | 会阻塞 event loop | 自动迁移到其他线程 |
| 内存控制 | GC，不可预测 | 无 GC，编译期确定 |

## OpenHuman 里的 Tokio

在 `openhuman-core` 启动时：

1. `main.rs` → `run_core_from_args()` → 创建 Tokio runtime
2. 核心服务器（JSON-RPC）在这个 runtime 上运行
3. 所有 Agent 推理、记忆树计算、渠道同步都在同一个 runtime 上以 `tokio::spawn` 方式并发

Rust 的学习曲线确实陡峭，但在这个项目中你不需要精通 Rust 才能理解架构——能读懂函数签名和控制流就够了。
