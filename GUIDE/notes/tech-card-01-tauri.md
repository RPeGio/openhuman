# 技术卡 1：Tauri 是什么？为什么不用 Electron？

## 一句话

**Tauri** 是一个用 Rust 驱动、用系统自带 WebView 渲染 UI 的桌面应用框架。它的角色和 Electron 一样——让你用前端技术（HTML/CSS/JS）写桌面应用——但实现方式完全不同。

## Electron 的问题

```
Electron 应用 = 你的 JS 代码 + 完整 Chromium 浏览器 + Node.js 运行时
                 └────────── ~150 MB ──────────┘
```

每个 Electron 应用都打包了一个**完整的 Chromium 浏览器**。如果你装了 5 个 Electron 应用（VS Code、Slack、Discord、Figma、Postman），你的电脑上跑了 5 个独立的 Chromium——内存和磁盘的巨大浪费。

## Tauri 的做法

```
Tauri 应用 = 你的前端代码 (HTML/CSS/JS) + 系统自带 WebView (~2 MB Rust 壳)
             │                           │
             └── 前端                     └── macOS: WKWebView (Safari)
                                              Windows: WebView2 (Edge)
                                              Linux: WebKitGTK
```

Tauri **不打包浏览器**，而是用操作系统自带的 WebView。Rust 后端编译成原生二进制（几 MB），通过 IPC 与前端通信。

## 对比

| | Electron | Tauri |
| --- | --- | --- |
| 包大小 | ~150 MB+ | ~2-10 MB |
| 内存占用 | 高（每个应用独立 Chromium） | 低（共享系统 WebView） |
| 后端语言 | Node.js (JS) | Rust |
| 原生 API 调用 | 通过 Node.js C++ addon | 直接 Rust FFI |
| 跨平台一致性 | ✅ 自己的 Chromium 保证一致 | ⚠️ WebView 差异需处理 |

## OpenHuman 为什么选 Tauri？

1. **性能**：Agent 核心需要高并发处理（LLM 流式推理、内存树计算）——Rust 比 Node.js 更适合
2. **安全**：Rust 的内存安全保证 + 更小的攻击面
3. **体积**：用户不会想下一个 200MB 的桌面助手

## OpenHuman 的例外：CEF

注意 `gitbooks/developing/cef.md` 提到——OpenHuman 的 Tauri 壳实际上打包了一个**裁剪版 Chromium (CEF)** 而非用系统 WebView。这是因为系统 WebView 不暴露 Chrome DevTools Protocol (CDP)，而 OpenHuman 需要用 CDP 来扫描 WhatsApp/Slack/Discord 等 Web 应用的数据。

这不是 Tauri 的默认行为——是 OpenHuman 的特殊需求驱动的定制。
