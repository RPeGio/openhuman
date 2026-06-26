# 技术卡 3：JSON-RPC 协议 vs REST/gRPC

## 一句话

**JSON-RPC** 是一种轻量级的远程调用协议——客户端发一个 JSON 对象描述"我要调用哪个方法，传什么参数"，服务器返回一个 JSON 对象描述结果。它比 REST 更简单直接。

## 三种协议对比

### REST

```
GET /users/123           →  获取用户 123
POST /users              →  创建用户
PUT /users/123           →  更新用户 123
DELETE /users/123        →  删除用户 123
```

- 以**资源**为中心（URL 就是资源路径）
- HTTP method 表示操作
- 适合 CRUD 场景
- 缺点：复杂操作难以映射（"开始推理"是哪个资源？哪个 method？）

### gRPC

```protobuf
// 需要先写 .proto 定义文件
service AgentService {
  rpc StartChat(StartChatRequest) returns (StartChatResponse);
}
```

- 以**服务和方法**为中心
- 用 Protocol Buffers（二进制）而非 JSON
- 需要代码生成
- 性能极高，但调试麻烦（二进制不可读）

### JSON-RPC

```json
// 请求（客户端 → 服务器）
{
  "jsonrpc": "2.0",
  "method": "openhuman.start_chat",
  "params": {
    "message": "帮我查一下明天天气",
    "thread_id": "abc123"
  },
  "id": 1
}

// 响应（服务器 → 客户端）
{
  "jsonrpc": "2.0",
  "result": {
    "response": "明天北京晴，15-25°C..."
  },
  "id": 1
}
```

- 以**方法调用**为中心
- 全部 JSON，人眼可读
- 不需要 URL 设计，不需要 proto 文件
- 一个端点处理所有请求：`POST /rpc`

## OpenHuman 为什么选 JSON-RPC？

| 需求 | JSON-RPC 的优势 |
| --- | --- |
| Agent 方法多且复杂（100+ 个） | 不需要为每个操作设计 URL——直接 `method: "openhuman.xxx"` |
| 需要流式输出（LLM 逐词返回） | JSON-RPC 返回可以走 SSE（Server-Sent Events） |
| 调试友好 | 所有请求/响应都是 JSON，Chrome DevTools 直接看 |
| 前后端都在本机（`127.0.0.1:7788`） | 不需要 REST 的 HTTP 缓存/代理等中间件优势 |

## OpenHuman 的具体实现

在 `src/core/jsonrpc.rs` 中：
- Axum HTTP 框架接收 `POST /rpc`
- 解析 `method` 字段 → 查 `all.rs` 控制器注册表
- 调用对应 handler → 返回 JSON 结果
- SSE 流通过 `text/event-stream` 响应类型返回

前端 `app/src/services/coreRpcClient.ts` 封装了这些调用，你只需 `rpc.call('openhuman.xxx', params)`。
