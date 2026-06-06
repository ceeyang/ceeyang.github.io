---
title: "开源邀请归因系统 share-installs：自托管延迟深度链接匹配方案"
date: 2026-06-01
tags: ["iOS", "Open Source", "Backend", "SDK", "Full-Stack"]
description: "分享一个开源自托管的邀请码 & 延迟深度链接归因系统——share-installs。用户点击邀请链接并安装 App 后，首次启动时邀请码自动识别回填。"
---

## 前言

你有没有遇到过这种场景：

> 用户 A 分享了一个邀请链接给用户 B。用户 B 在手机上点击链接，跳转到 App Store 下载了 App。安装打开后——**邀请码呢？**

传统方案要么依赖复杂的 Deep Link 配置（Universal Link / App Links），要么需要接入 Firebase Dynamic Links 这类第三方服务。前者配置繁琐，后者有被 deprecate 的风险。

这就是我开源 **share-installs** 的原因。

---

## 什么是 share-installs？

**share-installs** 是一个开源、自托管的 **邀请码 & 延迟深度链接归因系统**。

核心能力只有一句话：**用户点击邀请链接并安装 App 后，首次启动时邀请码会自动识别并回填——即使点击时 App 尚未安装。**

项目地址：[github.com/ceeyang/share-installs](https://github.com/ceeyang/share-installs)

```
⭐ 核心功能：浏览器指纹 → 后端匹配 → App 启动自动回填
🖥️ 三端 SDK：Web（TypeScript）+ iOS（Swift）+ Android（Kotlin）
🐳 一键部署：Docker Compose / Kubernetes / 裸机
🔐 自托管：数据存在你自己的服务器上
```

---

## 工作原理

整个流程分为四个阶段：

```
[阶段 1] 用户 A 分享邀请链接
         │
         ▼
[阶段 2] 用户 B 点击链接 → 你的落地页
         │  JS SDK 上报浏览器指纹到后端
         ▼
[阶段 3] 跳转 App Store / Play Store
         │
         ▼
[阶段 4] 用户 B 安装并打开 App
         │  移动 SDK 上报设备指纹
         ▼
         后端匹配指纹 → 返回邀请码
         │
         ▼
         App 自动填入邀请码 ✓
```

**关键技术点：**

1. **指纹匹配**：浏览器和设备的特征组合（时区、语言、屏幕尺寸、硬件参数等）被视为同一用户的指纹
2. **时间窗口**：默认 72 小时内，点击事件可被匹配
3. **置信度阈值**：模糊匹配默认 0.75，可配置

**职责边界：** share-installs 只负责指纹收集和匹配。邀请码的生成、跳转商店、使用次数统计等业务逻辑由你的系统处理。

---

## 快速启动

### Docker 一键启动（推荐）

```bash
git clone https://github.com/ceeyang/share-installs.git
cd share-installs

# 使用默认配置启动全部服务
docker compose up --build

# 验证
curl http://localhost:6066/api/health
# {"status":"ok","version":"1.0.0","timestamp":"..."}
```

### 本地调试模式（Docker 基础设施 + 本地代码热重载）

最适合有断点调试需求的日常开发：

```bash
# 1. 启动数据库 + 缓存
docker compose up db redis -d

# 2. 安装依赖
cd backend && npm install

# 3. 初始化数据库
npm run db:generate
npx prisma migrate dev --name init

# 4. 启动热重载开发服务器
npm run dev
# 服务运行在 http://localhost:6066
```

### 环境变量快速参考

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `MULTI_TENANT` | `false` | 自部署模式 / SaaS 模式 |
| `ADMIN_SECRET` | 空（不鉴权） | 项目管理端点保护密钥 |
| `FINGERPRINT_MATCH_THRESHOLD` | `0.75` | 匹配最低置信度 |
| `FINGERPRINT_MATCH_TTL_HOURS` | `72` | 匹配时间窗口（小时） |

---

## SDK 集成

### Web 端（邀请落地页）

```typescript
import { ShareInstallsSDK } from '@share-installs/js-sdk';

const sdk = new ShareInstallsSDK({
  apiBaseUrl: 'https://你的域名/api',
});

// 从 URL 提取邀请码
const inviteCode = location.pathname.split('/').pop()!;

// 上报指纹
await sdk.trackClick(inviteCode, {
  customData: { campaign: 'summer2024' },
});

// 然后由你的代码跳转 App Store / Play Store
```

### iOS（Swift）

```swift
import ShareInstallsSDK

// 在 AppDelegate 或 @main App 中初始化
ShareInstallsSDK.configure(with: ShareInstallsConfiguration(
    apiBaseURL: URL(string: "https://你的域名/api")!
))

// 首次启动后 resolve
if let result = try await ShareInstallsSDK.shared.resolveDeferred() {
    // result.inviteCode — 邀请码
    // result.customData — 落地页传入的自定义数据
    applyInviteCode(result.inviteCode)
}
```

### Android（Kotlin）

```kotlin
// 在 Application.onCreate 中初始化
ShareInstallsSDK.configure(
    context = applicationContext,
    configuration = ShareInstallsConfiguration(
        apiBaseUrl = "https://你的域名/api"
    )
)

// 首次启动后 resolve
val result = ShareInstallsSDK.instance.resolveDeferred()
if (result != null) {
    applyInviteCode(result.inviteCode)
}
```

---

## 部署选项

| 方式 | 适用场景 | 复杂度 |
|------|----------|--------|
| Docker Compose | 测试 & 自托管 | ⭐ |
| Docker Dev | 日常开发调试 | ⭐⭐ |
| Docker Compose（生产） | 小型生产部署 | ⭐⭐ |
| 裸机（Node.js + PG + Redis） | 已有基础设施 | ⭐⭐⭐ |
| Kubernetes + Terraform（AWS） | 生产环境 | ⭐⭐⭐⭐ |

---

## 项目结构一览

```
share-installs/
├── backend/           # Express + Prisma + Redis REST API
├── sdk/
│   ├── ios/           # Swift SDK（SPM + CocoaPods）
│   ├── android/       # Kotlin SDK
│   └── js/            # TypeScript Web SDK（npm）
├── landing/           # Next.js 邀请落地页（可选）
├── dashboard/         # 管理仪表盘
├── docs/              # 架构 & 集成文档
├── infrastructure/
│   ├── k8s/           # Kubernetes manifests
│   └── terraform/     # AWS 基础设施（EKS + RDS）
├── test/              # 内存测试服务器 & 测试页面
├── docker-compose.yml
└── docker-compose.dev.yml
```

**后端技术栈：**
- **运行时**：Node.js + TypeScript
- **框架**：Express
- **ORM**：Prisma
- **缓存**：Redis（用于指纹匹配加速 + 限流）
- **数据库**：PostgreSQL

---

## 适用场景

- **社交裂变**：App 用户邀请好友注册，自动归因
- **活动推广**：通过邀请链接分发优惠券码
- **内测分发**：控制邀请名额，追踪安装来源
- **渠道归因**：区分不同推广渠道的下载来源

---

## 与其他方案对比

| | share-installs | Firebase Dynamic Links | Branch.io | AppsFlyer |
|--|--|--|--|--|
| **开源** | ✅ | ❌ | ❌ | ❌ |
| **自托管** | ✅ | ❌ | ❌ | ❌ |
| **免费** | ✅ | ✅（有额度） | ❌ | ❌ |
| **多端 SDK** | Web/iOS/Android | Web/iOS/Android | Web/iOS/Android | Web/iOS/Android |
| **部署复杂度** | 低（Docker） | 零（托管） | 零（托管） | 零（托管） |

**选择建议：** 如果你的数据合规要求严格、不想依赖第三方服务、或者想完全控制归因逻辑，share-installs 是一个轻量且可扩展的选择。

---

## 结语

share-installs 是一个小而美的项目——它只做一件事，但把它做扎实了。从指纹采集、存储匹配、到多端 SDK，整个链路完整闭环。

项目是完全开源的（Apache 2.0 协议），欢迎 Star、Issue 和 PR：

👉 [https://github.com/ceeyang/share-installs](https://github.com/ceeyang/share-installs)

如果你有类似的场景需要处理邀请归因，不妨试试自部署一个，5 分钟就能跑起来。
