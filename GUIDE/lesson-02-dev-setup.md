# 第 2 课：开发环境搭建

> **难度**：⭐ 入门 | **预计时间**：90-120 分钟（取决于网速和 OS） | **前置要求**：第 1 课

---

## 学习目标

完成本课后，你将能够：

1. 在自己的电脑上安装全部必需工具链
2. 克隆仓库并初始化子模块
3. 配置 `.env` 环境变量
4. 成功运行 `pnpm typecheck`、`pnpm lint`、`cargo check` 通过验证
5. 启动 Web 开发模式 (`pnpm dev`) 或桌面开发模式 (`pnpm --filter openhuman-app dev:app`)

---

## 1. 工具链总览

在开始之前，先看一张完整的依赖关系图：

```
你的代码
   │
   ├── app/ (前端) ──────── Node.js 24+ + pnpm 10.10.0
   │      └── app/src-tauri/ (桌面壳) ── Rust 1.93.0 + CEF (Chromium)
   │             ├── whisper-rs-sys ─── CMake + (Windows: LLVM/Clang)
   │             └── cef-dll-sys ───── (macOS: Ninja)
   │
   └── src/ (Rust 核心) ─── Rust 1.93.0 + rustfmt + clippy
          └── 预推送钩子 ─── ripgrep (rg)
```

### 必需工具清单

| 工具 | 版本要求 | 用途 | 所有平台？ |
| --- | --- | --- | --- |
| **Git** | 最新稳定版 | 版本控制 + 子模块管理 | ✅ |
| **Node.js** | `>= 24.0.0` | 前端运行时 | ✅ |
| **pnpm** | `10.10.0` | JS 包管理器 | ✅ |
| **Rust** | `1.93.0` | 核心引擎编译 | ✅ |
| **rustfmt** | 与 Rust 配套 | 代码格式化 | ✅ |
| **clippy** | 与 Rust 配套 | 代码静态分析 | ✅ |
| **CMake** | 最新稳定版 | whisper.cpp 等原生依赖构建 | ✅ |
| **Ninja** | 最新稳定版 | macOS 上 CEF 构建加速 | 🍎 macOS |
| **ripgrep (rg)** | 最新稳定版 | 预推送钩子命令扫描 | ✅ |
| **Xcode CLI** | 最新 | macOS 桌面构建 | 🍎 macOS |
| **VS Build Tools** | 2022 (MSVC v143) | Windows C++ 链接器 | 🪟 Windows |
| **LLVM/Clang** | 最新 | Windows 上 whisper-rs-sys 编译 | 🪟 Windows |
| **GTK/WebKit 等** | 发行版默认 | Linux 桌面构建 | 🐧 Linux |

---

## 2. 分平台安装指南

### 2.1 macOS

```bash
# 1. 安装 Homebrew（如果没有）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2. 一次性安装所有工具
brew install node@24 pnpm rustup-init cmake ninja ripgrep

# 3. 安装 Rust 工具链
rustup toolchain install 1.93.0 --profile minimal
rustup component add rustfmt clippy --toolchain 1.93.0

# 4. Apple Silicon 需要额外安装 x86_64 target（CEF 构建通用二进制）
rustup target add x86_64-apple-darwin

# 5. 验证
node --version    # 应 ≥ v24.0.0
pnpm --version    # 应为 10.10.0
rustc --version   # 应为 1.93.0
cmake --version   # 任意最新版
ninja --version   # 任意最新版
rg --version      # 任意最新版
```

> ⚠️ **注意**：如果用 `nvm` 管理 Node，执行 `nvm install 24 && nvm use 24`。

### 2.2 Windows

Windows 工具链最复杂，**按顺序安装**，每步完成后重启终端：

```powershell
# ──── 第 1 步：Node.js ────
winget install CoreyButler.NVMforWindows
# 关闭并重新打开终端
nvm install 24
nvm use 24
node --version     # 应 ≥ v24.0.0

npm install -g pnpm@10.10.0
pnpm --version     # 应为 10.10.0

# ──── 第 2 步：VS Build Tools + Rust ────
# 先装 VS 2022 Build Tools（约 5.4 GB）
# 去 https://visualstudio.microsoft.com/downloads/ 下载
# 安装时勾选 "Desktop development with C++"（含 MSVC v143 + Windows 11 SDK）

winget install Rustlang.Rustup
# 关闭并重新打开终端
rustup toolchain install 1.93.0 --profile minimal
rustup component add rustfmt clippy --toolchain 1.93.0
rustc --version    # 应为 1.93.0

# ──── 第 3 步：LLVM/Clang（whisper-rs-sys 依赖）───
# 从 https://github.com/llvm/llvm-project/releases 下载 Windows x86_64 版（~822 MB）
# 安装时勾选 "Add LLVM to system PATH for all users"
# 如果 PATH 太长报错，手动设置环境变量：
#   LIBCLANG_PATH=C:\Program Files\LLVM\bin

# ──── 第 4 步：CMake ────
winget install Kitware.CMake
cmake --version

# ──── 第 5 步：ripgrep ────
winget install BurntSushi.ripgrep.MSVC

# ──── 最终验证 ────
rustc --version
cargo --version
clang --version
cmake --version
node --version
pnpm --version
rg --version
```

> ⚠️ **安装顺序很重要**：VS Build Tools → Rust → LLVM → CMake → Node + pnpm。每步后重启终端，否则 PATH 不会生效。

### 2.3 Linux (Arch)

```bash
# 一次性安装系统包
sudo pacman -S --needed nodejs npm rustup cmake base-devel clang openssl \
  alsa-lib xdotool libxtst libxi libevdev gtk3 webkit2gtk-4.1 \
  libayatana-appindicator librsvg patchelf nss nspr at-spi2-core \
  libcups libdrm libxkbcommon libxcomposite libxdamage libxfixes \
  libxrandr mesa pango cairo libxshmfence ripgrep

# pnpm
npm install -g pnpm@10.10.0

# Rust
rustup toolchain install 1.93.0 --profile minimal
rustup component add rustfmt clippy --toolchain 1.93.0

# 验证
node --version && pnpm --version && rustc --version && cmake --version && rg --version
```

### 2.4 Linux (Debian/Ubuntu)

```bash
# Tauri 桌面构建依赖
sudo apt-get install -y --no-install-recommends \
  libgtk-3-dev libwebkit2gtk-4.1-dev libayatana-appindicator3-dev \
  librsvg2-dev patchelf libsoup-3.0-dev libjavascriptcoregtk-4.1-dev \
  cmake build-essential curl wget file libssl-dev ripgrep

# Node.js 24（推荐用 nvm）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
# 重启终端
nvm install 24
nvm use 24

npm install -g pnpm@10.10.0

# Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# 重启终端
rustup toolchain install 1.93.0 --profile minimal
rustup component add rustfmt clippy --toolchain 1.93.0
```

---

## 3. 克隆仓库与初始化

### 3.1 Fork 上游仓库（如需贡献）

如果你计划提交 PR：

1. 在 GitHub 上打开 [tinyhumansai/openhuman](https://github.com/tinyhumansai/openhuman)
2. 点击右上角 **Fork** 按钮
3. 克隆你 fork 的仓库（把 `YOUR_USERNAME` 替换为你的 GitHub 用户名）：

```bash
git clone git@github.com:YOUR_USERNAME/openhuman.git
cd openhuman
git remote add upstream git@github.com:tinyhumansai/openhuman.git
```

如果只是学习源码，直接克隆上游即可：

```bash
git clone https://github.com/tinyhumansai/openhuman.git
cd openhuman
```

### 3.2 初始化 Git 子模块

OpenHuman 依赖两个 **vendored（内置的）** 子模块：

| 子模块路径 | 作用 |
| --- | --- |
| `app/src-tauri/vendor/tauri-cef` | CEF（Chromium）感知的 Tauri CLI——桌面壳使用 Chromium 而非系统 WebView |
| `app/src-tauri/vendor/tauri-plugin-notification` | 通知插件补丁版 |

**必须**在 `pnpm install` 之前初始化它们，否则桌面构建会失败：

```bash
git submodule update --init --recursive
```

验证：

```bash
ls app/src-tauri/vendor/tauri-cef/Cargo.toml      # 应该存在
ls app/src-tauri/vendor/tauri-plugin-notification/  # 应该存在
```

### 3.3 安装 JS 依赖

```bash
pnpm install
```

如果看到 `Unsupported engine` 警告，说明 Node 版本不对——回到第 2 节确认版本。

---

## 4. 配置环境变量

OpenHuman 使用两级 `.env` 文件：

| 文件 | 作用域 | 内容 |
| --- | --- | --- |
| 根目录 `.env` | Rust 核心 + Tauri 壳 | 后端 URL、端口、模型配置、日志级别 |
| `app/.env.local` | 前端 Vite | `VITE_*` 变量 |

### 4.1 创建本地配置文件

```bash
# 从模板复制（不会提交到 git，已在 .gitignore 中）
cp .env.example .env
cp app/.env.example app/.env.local
```

### 4.2 默认值通常就够用

根目录 `.env` 中最重要的几个变量及其默认值：

```bash
# 核心 RPC 端口和地址
OPENHUMAN_CORE_PORT=7788
OPENHUMAN_CORE_RPC_URL=http://127.0.0.1:7788/rpc

# 运行模式：child（默认，内嵌启动核心进程）
OPENHUMAN_CORE_RUN_MODE=child

# 日志级别
RUST_LOG=info
RUST_BACKTRACE=1

# 每小时工具执行上限
OPENHUMAN_MAX_ACTIONS_PER_HOUR=20

# Token 留空——桌面壳会自动生成
# OPENHUMAN_CORE_TOKEN=
```

> ⚠️ **关键规则**：`.env` 和 `app/.env.local` 永远不要提交到 Git。

### 4.3 加载环境变量

每次打开新终端时：

```bash
source scripts/load-dotenv.sh
```

或者把这条加到你的 shell 配置里（`.zshrc` / `.bashrc`）。

---

## 5. 首次编译与验证

按下面顺序执行，全部通过才算环境搭建成功：

### 5.1 前端检查

```bash
# TypeScript 类型检查
pnpm typecheck

# ESLint
pnpm lint

# Prettier 格式检查
pnpm format:check
```

预期输出：三项都无报错。

### 5.2 Rust 核心检查

```bash
# 检查核心 crate
cargo check --manifest-path Cargo.toml

# 检查 Tauri 壳 crate
cargo check --manifest-path app/src-tauri/Cargo.toml
```

> 🍎 **Apple Silicon 特别提示**：如果 `cargo check` 在 whisper-rs-sys 或 llama.cpp 构建时失败，尝试：
> ```bash
> GGML_NATIVE=OFF cargo check --manifest-path Cargo.toml
> ```

首次 `cargo check` 会下载和编译所有 Rust 依赖，可能耗时 5-15 分钟——这是正常的。

### 5.3 启动开发模式

**Web 前端开发**（不启动桌面壳，浏览器中运行）：

```bash
pnpm dev
```

打开浏览器访问 `http://localhost:1420`，应该能看到 OpenHuman 的 Web 界面。

**桌面开发**（启动完整 Tauri + CEF 桌面应用）：

```bash
pnpm --filter openhuman-app dev:app
```

> 🍎 **macOS 首次桌面构建**：需要创建本地签名证书。运行一次：
> ```bash
> bash scripts/setup-dev-codesign.sh
> ```

### 5.4 验证清单

| 检查项 | 命令 | 预期结果 |
| --- | --- | --- |
| TypeScript 类型检查 | `pnpm typecheck` | 无错误 |
| ESLint | `pnpm lint` | 无错误 |
| 格式检查 | `pnpm format:check` | 无错误 |
| Rust 核心编译检查 | `cargo check --manifest-path Cargo.toml` | 无错误 |
| Tauri 壳编译检查 | `cargo check --manifest-path app/src-tauri/Cargo.toml` | 无错误 |
| Web 开发服务器 | `pnpm dev` | `http://localhost:1420` 可访问 |
| 桌面应用启动 | `pnpm --filter openhuman-app dev:app` | 窗口出现 |

---

## 6. 常见问题与解决

### 问题 1：`pnpm install` 报 "Unsupported engine"

```
Unsupported engine: wanted >=24.0.0 (current: v22.x.x)
```

**原因**：Node.js 版本低于 24。**解决**：

```bash
# nvm
nvm install 24 && nvm use 24

# 或 Homebrew
brew install node@24
```

### 问题 2：`cargo check` 编译失败（Apple Silicon）

**原因**：whisper-rs / llama.cpp 的 SIMD 指令集检测问题。

**解决**：
```bash
GGML_NATIVE=OFF cargo check --manifest-path Cargo.toml
```

### 问题 3：桌面应用报 "CEF cache is held by another instance"

```
CEF cache at ~/Library/Caches/com.openhuman.app/cef is held by another OpenHuman instance
```

**原因**：Chromium 的用户数据目录被锁（另一个 OpenHuman 进程还在跑）。

**解决**：
```bash
pkill -f "OpenHuman.app/Contents"
pkill -f "openhuman-core"
# 然后再试
pnpm --filter openhuman-app dev:app
```

### 问题 4：`cargo check --manifest-path app/src-tauri/Cargo.toml` 报子模块缺失

**原因**：忘记初始化 Git 子模块。

**解决**：
```bash
git submodule update --init --recursive
```

### 问题 5：Windows 上 `clang` 找不到

**原因**：LLVM 没装或 PATH 没生效。

**解决**：
```powershell
# 手动设置
$env:LIBCLANG_PATH = "C:\Program Files\LLVM\bin"
clang -v   # 验证
```

### 问题 6：macOS 桌面构建报 "OpenHuman Dev Signer: no identity found"

**原因**：缺少本地签名证书。

**解决**：
```bash
bash scripts/setup-dev-codesign.sh
```

### 问题 7：Linux 上桌面构建报 GTK/WebKit 缺失

**原因**：没装 Tauri 的桌面构建依赖。

**解决**：回到第 2 节，确保安装了对应发行版的 `gtk3`、`webkit2gtk-4.1`、`libayatana-appindicator` 等包。

---

## 动手任务

### 任务 1：完整搭建（必做）

从零开始，在你的操作系统上完成全部搭建流程，直到 `pnpm typecheck && cargo check --manifest-path Cargo.toml` 通过。

### 任务 2：探索配置文件

打开 `.env.example`，用注释中的英文关键词搜索每个变量的作用。重点关注：
- `OPENHUMAN_CORE_PORT` 和 `OPENHUMAN_CORE_RPC_URL` 的关系
- `RUST_LOG` 的可选值
- `OPENHUMAN_MAX_ACTIONS_PER_HOUR` 的安全含义

### 任务 3：运行测试

```bash
# 前端单元测试
pnpm test

# Rust 测试
cargo test --manifest-path Cargo.toml --lib
```

记录测试结果（通过/失败数量），熟悉测试命令。

### 任务 4：探索项目结构

在 IDE 中打开项目，浏览以下关键文件：
- `Cargo.toml` — 看 `[dependencies]` 部分，认识核心依赖
- `app/package.json` — 看 `scripts` 部分，了解所有可用命令
- `rust-toolchain.toml` — 理解 Rust 版本锁定机制
- `.env.example` — 了解完整的环境变量体系

---

## 思考题

1. 为什么 OpenHuman 要锁定 Rust `1.93.0` 而不是用最新版？`rust-toolchain.toml` 里的注释提到了什么具体原因？

2. Git 子模块 (`git submodule`) 和普通依赖（`cargo` / `npm`）有什么区别？为什么 OpenHuman 选择 vendored（内置）Tauri/CEF 而不是从 crates.io 下载？

3. 环境变量同时存在于 `.env`（Rust 核心）和 `app/.env.local`（Vite 前端）——为什么需要两个？`VITE_` 前缀有什么特殊含义？

4. 如果你在 Windows 上，`LIBCLANG_PATH` 环境变量是做什么的？为什么只有 Windows 需要它？

---

## 延伸阅读

| 材料 | 路径 |
| --- | --- |
| 初学者搭建指南 | [`CONTRIBUTING-BEGINNERS.md`](../CONTRIBUTING-BEGINNERS.md) |
| 完整贡献者指南 | [`CONTRIBUTING.md`](../CONTRIBUTING.md) |
| GitBook 搭建文档 | [`gitbooks/developing/getting-set-up.md`](../gitbooks/developing/getting-set-up.md) |
| 仅构建 Rust 核心 | [`gitbooks/developing/building-rust-core.md`](../gitbooks/developing/building-rust-core.md) |
| 架构文档入口 | [`gitbooks/developing/architecture.md`](../gitbooks/developing/architecture.md) |
| `.env` 完整模板 | [`.env.example`](../.env.example) |
| Rust 工具链锁定 | [`rust-toolchain.toml`](../rust-toolchain.toml) |

---

## 下一课预告

**第 3 课：仓库结构与顶层架构** — 深入理解三层架构（React → Tauri → Rust Core）、pnpm workspace 结构、多二进制入口设计。学会用 `codegraph` 探索代码调用链。
