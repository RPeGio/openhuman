# 技术卡 5：CEF (Chromium Embedded Framework) 是什么？

## 一句话

**CEF**（Chromium Embedded Framework）是一个让你把 Chromium 浏览器嵌入到自己应用里的库。你可以把它理解为"一个可以塞进任何桌面应用的 Chrome 内核"。

## 为什么要嵌入浏览器？

假设你要写一个桌面应用，需要：

- 显示 Slack 的 Web 界面
- 从 WhatsApp Web 读取消息
- 让用户登录 Google 账号

你有两种做法：

**做法 A：写一个完整的 Slack/WhatsApp/Gmail 客户端** — 荒谬，工作量巨大。

**做法 B：在应用里开一个浏览器窗口，加载 Slack 的网页版** — 聪明。这就是 CEF 做的事。

## CEF vs 系统 WebView vs Electron

| | 系统 WebView | CEF | Electron |
| --- | --- | --- | --- |
| 本质 | 操作系统自带的浏览器组件 | 你打包的 Chromium | 你打包的 Chromium + Node.js |
| 大小 | 几乎为 0 | ~100 MB | ~150 MB |
| CDP 支持 | ❌ 不暴露 | ✅ 完整 CDP | ✅ 完整 CDP |
| 可控性 | 低（随 OS 更新变化） | 高（你固定版本） | 高 |

## CDP 是什么？

CDP = Chrome DevTools Protocol。就是 Chrome 开发者工具和浏览器通信用的协议。通过 CDP 你可以编程式地：

- 打开一个网页
- 读取页面的 DOM
- 执行 JavaScript
- 读取 IndexedDB（网页的本地数据库）
- 拦截网络请求

简而言之：**CDP 让你用代码控制一个网页，就像你在 DevTools 里手动操作一样。**

## OpenHuman 为什么需要 CEF？

OpenHuman 需要**扫描你登录的 Web 应用**来建立记忆：

```
WhatsApp Web → CEF webview → CDP scanner → 读取消息 → 提取摘要 → Memory Tree
Slack       → CEF webview → CDP scanner → 读取频道 → 提取摘要 → Memory Tree
Discord     → CEF webview → CDP scanner → 读取消息 → 提取摘要 → Memory Tree
```

系统 WebView 不提供 CDP，所以 OpenHuman 无法用系统 WebView 来做这些扫描。这就是为什么它要打包一个 CEF。

具体实现见 `gitbooks/developing/cef.md` 和 `app/src-tauri/src/*_scanner.rs`。
