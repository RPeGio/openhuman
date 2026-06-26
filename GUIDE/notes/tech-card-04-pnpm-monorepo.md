# 技术卡 4：pnpm workspace / monorepo

## 一句话

**Monorepo**（单一仓库）就是把多个相关项目放在一个 Git 仓库里管理。**pnpm workspace** 是 pnpm 提供的 monorepo 管理机制。

## 为什么用 monorepo？

分开管理 vs 放在一起：

```
多仓库 (polyrepo)：              单仓库 (monorepo)：
  openhuman-frontend/               openhuman/
  openhuman-core/                   ├── app/          ← 前端
  openhuman-docs/                   ├── src/          ← Rust 核心
  openhuman-scripts/                ├── scripts/      ← 共用脚本
                                    └── packages/     ← 打包配置
```

| | 多仓库 | monorepo |
| --- | --- | --- |
| 版本同步 | 需要手动对齐版本号 | 一个 PR 改前后端 |
| CI/CD | 多个仓库各自配 | 一套 CI 跑全部 |
| 代码共享 | 通过 npm 发布 | 直接 `import` |
| 缺点 | 改前后端联调要开两个 PR | 仓库体积大，clone 慢 |

## pnpm 的三个关键概念

### 1. workspace（工作空间）

`pnpm-workspace.yaml` 声明哪些目录是 workspace 的包：

```yaml
packages:
  - "app"
  - "packages/tauri-plugin-ptt"
```

### 2. filter（过滤器）

根目录 `pnpm dev` 其实是：

```json
// 根 package.json
"dev": "pnpm --filter openhuman-app dev"
```

`--filter` 让你在根目录就能操作子包，不需要 `cd` 进去。

### 3. 硬链接 / 内容寻址存储

pnpm 不会像 npm 那样把 `node_modules` 复制 N 份。它把所有包存在全局 store，项目里只放**硬链接**。效果：

| | npm | pnpm |
| --- | --- | --- |
| 磁盘占用 | 大（每个项目独立复制） | 小（共享全局 store） |
| 安装速度 | 慢 | 快 |
| `node_modules` 结构 | 扁平化（能访问未声明的依赖） | 严格（不能访问未声明的） |

## OpenHuman 的 workspace 为什么只有两个包？

主要是历史原因——原来 `packages/npm/` 也有一个包，但它的 `postinstall` 脚本会自动下载预构建的 openhuman 二进制，在 CI 环境里会失败（release 还没发布）。所以 `pnpm-workspace.yaml` 不写 `packages/*` 通配符，而是显式声明需要的包。

## 日常命令对应

| 你输入的（根目录） | 实际执行的 |
| --- | --- |
| `pnpm dev` | `cd app && pnpm dev` |
| `pnpm typecheck` | `cd app && pnpm compile` |
| `pnpm test` | `cd app && pnpm test` |
