---
title: "zentao-mcp：让 AI 直接读禅道 Bug、修 Bug、回写禅道"
date: 2026-06-29
tags: ["Open Source", "MCP", "AI", "ZenTao", "禅道", "Cursor", "DevTools"]
keywords: ["zentao-mcp", "禅道 MCP", "Cursor 禅道", "AI 修 Bug", "MCP Server"]
description: "开源 zentao-mcp：一条命令接入 18 种 AI 客户端，让 Cursor / Claude 直接读取禅道 Bug 重现步骤、在 IDE 内修代码，并可选 dryRun 预演后回写「已解决」。"
cover:
  image: "images/posts/zentao-mcp/cover.png"
  alt: "zentao-mcp 封面：AI 与禅道 Bug 追踪系统连接"
  hiddenInList: true
  hiddenInSingle: true
draft: false
---

{{< figure src="/images/posts/zentao-mcp/cover.png" alt="zentao-mcp 封面" caption="zentao-mcp — 禅道与 AI 编程助手之间的桥梁" >}}

## 前言

每天开工前，你是不是都要打开禅道，翻一遍指派给自己的 Bug，复制重现步骤，再切回 IDE 开始改？

更麻烦的是——禅道里的重现步骤往往是 **HTML 富文本**，丢给 AI 时格式乱、上下文断，来回粘贴几次，半小时就没了。

如果你团队用 **禅道（ZenTao）** 管 Bug，又用 **Cursor / Claude / Windsurf** 写代码，这个问题应该不陌生。

所以我开源了 **[zentao-mcp](https://github.com/ceeyang/zentao_mcp)** —— 一个专为禅道设计的 **MCP Server**，让 AI 直接：

- 查「指派给我的 Bug」
- 读重现步骤（自动 HTML → 纯文本）
- 在 IDE 里改代码
- 可选：dryRun 预演后回写「已解决」

```
⭐ 7 个 MCP 工具，覆盖 Bug 读写的完整闭环
🚀 一条命令安装，无需 clone 仓库
🤖 自动配置 18 种 AI 客户端（Cursor、Claude、Windsurf…）
🔒 写操作默认关闭 + dryRun 预演，防 AI 误操作
📦 MIT 开源 · 适配禅道 16.5+ / 18.12
```

**项目地址：** [github.com/ceeyang/zentao_mcp](https://github.com/ceeyang/zentao_mcp)

---

## 30 秒看懂：它能干什么？

| 你想… | 在 AI 里这样说 |
|--------|----------------|
| 确认装好了 | 「调用 zentao_health_check 检查禅道连接」 |
| 看我负责的 Bug | 「列出指派给我的 Bug，最新的 10 条」 |
| 读重现步骤修 Bug | 「读取 Bug #12345 详情，按 steps 帮我改代码」 |
| 查某产品下的 Bug | 「查产品 3 下 alice 的 active Bug」 |
| 修完回写禅道 | 「dryRun 预演把 #12345 标为 fixed，备注 xxx」 |

适配禅道开源版 **16.5+ / 18.12**，走官方 **REST API v1**。

---

## 为什么需要 zentao-mcp？

### 痛点 1：禅道 ↔ IDE 来回切换

传统流程：

```
打开禅道 → 找 Bug → 复制步骤 → 粘贴到 AI → 改代码 → 再打开禅道 → 手动标 resolved
```

**zentao-mcp 之后：**

```
「我有哪些 Bug？」→ AI 调 MCP → 直接读步骤 → 改代码 → 「标为 fixed」
```

全程在 AI 客户端内完成，**不用离开编辑器**。

### 痛点 2：重现步骤是 HTML，AI 读不懂

禅道 Bug 的 `steps` 字段通常是富文本 HTML。直接丢给 LLM，标签、样式、嵌套表格都会干扰理解。

`zentao_get_bug` 会自动输出 **`stepsPlain`** —— 把 HTML 转成干净的纯文本 checklist，AI 可以直接按步骤定位代码。

### 痛点 3：MCP 配置门槛高

手写 `~/.cursor/mcp.json`、找 node 绝对路径、处理 `spawn ENOENT`……同事推广成本很高。

zentao-mcp 的安装脚本会：

1. 下载到 `~/.local/share/zentao-mcp`
2. `npm install` + 构建
3. **自动检测本机 AI 客户端**并写入配置
4. 使用 **node 绝对路径**，避免 Cursor 找不到命令

---

## 架构一览

{{< figure src="/images/posts/zentao-mcp/architecture.svg" alt="zentao-mcp 架构图" caption="AI 客户端 ↔ MCP Server ↔ 禅道 REST API" >}}

整体链路很清晰：

1. **AI 客户端**（Cursor / Claude 等）通过 stdio 与 MCP Server 通信
2. **zentao-mcp** 提供 7 个工具，统一返回 `{ ok, data, error, meta }` JSON 信封
3. **禅道服务器** 通过 HTTPS 调用 `/api.php/v1/*` REST API

技术栈：**TypeScript · Node 18+ · @modelcontextprotocol/sdk · Zod**

---

## 典型 Bug 修复闭环

{{< figure src="/images/posts/zentao-mcp/workflow.svg" alt="Bug 修复五步闭环" caption="查 Bug → 读步骤 → 改代码 → dryRun 预演 → 回写禅道" >}}

**Morning standup 示例：**

```
你：我禅道上有哪些没关的 Bug？
AI：→ zentao_list_my_bugs → 返回 6 条列表

你：重点看 #12345，读步骤，在 src/auth 里找原因
AI：→ zentao_get_bug → 读 stepsPlain → 改代码

你：测试过了，dryRun 预演把 #12345 标为 fixed
AI：→ zentao_resolve_bug(dryRun=true) → 展示将要提交的内容

你：确认，正式提交
AI：→ zentao_resolve_bug(dryRun=false) → 禅道状态更新 ✓
```

---

## 一条命令，18 种客户端

{{< figure src="/images/posts/zentao-mcp/clients.svg" alt="支持的 AI 客户端" caption="安装脚本自动检测并写入 Cursor、Claude、Windsurf 等 18 种客户端的 MCP 配置" >}}

**macOS / Linux：**

```bash
curl -fsSL https://raw.githubusercontent.com/ceeyang/zentao_mcp/main/install.sh | bash
```

**Windows（PowerShell）：**

```powershell
irm https://raw.githubusercontent.com/ceeyang/zentao_mcp/main/install.ps1 | iex
```

装完 **完全重启** Cursor / Claude 等客户端，然后对话：

```
调用 zentao_health_check 检查连接
```

### 支持的客户端（18 种）

Cursor · Claude Desktop · Claude Code · Windsurf · Trae · VS Code · Cline · OpenCode · Continue · Zed · Kiro · Roo Code · Amazon Q · Copilot CLI · Gemini CLI · Kimi CLI · Codex CLI …

安装时 `INSTALL_PLATFORMS=auto` 自动检测，也可手动指定 `cursor,claude`。

> 详细安装文档：[docs/INSTALL.zh-CN.md](https://github.com/ceeyang/zentao_mcp/blob/main/docs/INSTALL.zh-CN.md)

---

## 7 个 MCP 工具

| 工具 | 能力 | 默认 |
|------|------|------|
| `zentao_health_check` | 连通性 + 写权限开关状态 | ✅ 只读 |
| `zentao_list_my_bugs` | 指派给当前账号的 Bug | ✅ 只读 |
| `zentao_list_bugs` | 按产品 / 项目 / 执行筛选 | ✅ 只读 |
| `zentao_get_bug` | 详情 + **stepsPlain** 纯文本步骤 | ✅ 只读 |
| `zentao_resolve_bug` | 标记已修复（fixed / duplicate 等） | 🔒 需开关 |
| `zentao_close_bug` | 关闭 Bug | 🔒 需开关 |
| `zentao_activate_bug` | 重新激活 | 🔒 需开关 |

> 完整功能说明与 AI 对话示例：[docs/FEATURES.zh-CN.md](https://github.com/ceeyang/zentao_mcp/blob/main/docs/FEATURES.zh-CN.md)

### 读 Bug 返回示例（简化）

```json
{
  "ok": true,
  "data": {
    "id": 12345,
    "title": "登录页 token 过期未刷新",
    "status": "active",
    "stepsPlain": "1. 打开登录页\n2. 等待 token 过期（30min）\n3. 点击任意菜单\n4. 预期：自动刷新；实际：白屏",
    "severity": 2,
    "assignedTo": "cee"
  },
  "meta": { "source": "zentao_get_bug" }
}
```

AI 拿到 `stepsPlain` 后，可以直接生成 checklist、定位文件、提出 patch。

---

## 安全设计：AI 能写，但默认不能乱写

写操作是双刃剑——AI 可能误关 Bug、写错备注。zentao-mcp 默认 **只读**：

| 环境变量 | 作用 | 默认值 |
|----------|------|--------|
| `ZENTAO_ALLOW_RESOLVE_BUG` | 允许标记已解决 | `false` |
| `ZENTAO_ALLOW_CLOSE_BUG` | 允许关闭 Bug | `false` |
| `ZENTAO_ALLOW_ACTIVATE_BUG` | 允许重新激活 | `false` |

额外保护：

- **`dryRun: true`** — 所有写工具支持预演，先看会提交什么
- **`ZENTAO_ALLOWED_PRODUCTS`** — 产品 ID 白名单，限制可操作范围
- **`ZENTAO_SKIP_SSL`** — 内网自签名证书场景
- **Token / 账号密码双模式**，401 自动刷新

企业内网部署时，建议先只开读权限，团队熟悉后再由管理员按需开启写权限。

---

## 手动配置（可选）

如果安装时跳过了账号配置，编辑 `~/.cursor/mcp.json`：

```json
{
  "mcpServers": {
    "zentao": {
      "command": "/absolute/path/to/node",
      "args": ["/Users/you/.local/share/zentao-mcp/dist/index.js"],
      "env": {
        "ZENTAO_URL": "https://your-zentao-host",
        "ZENTAO_ACCOUNT": "your_account",
        "ZENTAO_PASSWORD": "your_password",
        "ZENTAO_SKIP_SSL": "true"
      }
    }
  }
}
```

完整环境变量列表：[.env.example](https://github.com/ceeyang/zentao_mcp/blob/main/.env.example)

---

## IT 批量部署

```bash
ZENTAO_URL=https://zentao.example.com \
ZENTAO_ACCOUNT=bot \
ZENTAO_PASSWORD=secret \
ZENTAO_SKIP_SSL=true \
INSTALL_PLATFORMS=auto \
curl -fsSL https://raw.githubusercontent.com/ceeyang/zentao_mcp/main/install.sh | bash -s -- --yes
```

发给同事的一键文案：[docs/SHARE.md](https://github.com/ceeyang/zentao_mcp/blob/main/docs/SHARE.md)

---

## 与其他方案对比

| | zentao-mcp | 手动复制粘贴 | 通用 REST MCP |
|--|-----------|-------------|---------------|
| 禅道原生 API | ✅ REST v1 | — | 需自己封装 |
| stepsPlain 转换 | ✅ 内置 | ❌ | 需自己写 |
| 18 客户端一键安装 | ✅ | — | ❌ |
| 写操作安全门 | ✅ dryRun + 开关 | — | 看实现 |
| 中文文档 | ✅ | — | 不一定 |

---

## 路线图

**已实现 ✅**

- 查 Bug / 读步骤 / 写状态完整闭环
- 18 客户端自动配置安装器
- 自签名 HTTPS、产品白名单、Token 刷新

**规划中 📋**

- Bug 指派推送通知
- 群聊每日 Bug 汇总机器人

欢迎提 Issue 和 PR：[github.com/ceeyang/zentao_mcp/issues](https://github.com/ceeyang/zentao_mcp/issues)

---

## 快速链接

| 资源 | 链接 |
|------|------|
| **GitHub 仓库** | [github.com/ceeyang/zentao_mcp](https://github.com/ceeyang/zentao_mcp) |
| 功能说明 | [FEATURES.zh-CN.md](https://github.com/ceeyang/zentao_mcp/blob/main/docs/FEATURES.zh-CN.md) |
| 安装文档 | [INSTALL.zh-CN.md](https://github.com/ceeyang/zentao_mcp/blob/main/docs/INSTALL.zh-CN.md) |
| 发给同事 | [SHARE.md](https://github.com/ceeyang/zentao_mcp/blob/main/docs/SHARE.md) |
| 安装脚本 (sh) | [install.sh](https://raw.githubusercontent.com/ceeyang/zentao_mcp/main/install.sh) |
| 安装脚本 (ps1) | [install.ps1](https://raw.githubusercontent.com/ceeyang/zentao_mcp/main/install.ps1) |

---

## 结语

zentao-mcp 的目标很简单：**让禅道 Bug 和 AI 编程助手之间的墙消失**。

如果你团队在用禅道 + Cursor / Claude，花 30 秒跑一条 `curl | bash`，明天 standup 就可以跟 AI 说「我有哪些 Bug」了。

**Star 支持一下：** [github.com/ceeyang/zentao_mcp](https://github.com/ceeyang/zentao_mcp) ⭐

有问题欢迎留言，或在 GitHub 提 Issue。
