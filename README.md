<div align="center">
  <a href="https://suichuan.zxlee.cn" target="_blank" rel="noopener noreferrer">
    <img src="./imgs/logo.png" alt="随传" height="120" />
  </a>
  <h1 align="center">随传</h1>
  <p align="center"><b>自在互联，随传而至。</b></p>
  <p align="center">连接你的每一台设备，让文件、剪贴板与消息自由流转。跨平台互联，无需云端，轻松实现多设备协同。</p>
  <p align="center">
    <a href="../LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License: MIT" /></a>
    <a href="#"><img src="https://img.shields.io/badge/Version-1.0.15-brightgreen.svg" alt="Version" /></a>
    <a href="#"><img src="https://img.shields.io/badge/Platform-Windows%20|%20macOS%20|%20Android%20|%20iOS%20|%20Linux-lightgrey.svg" alt="Platform" /></a>
    <a href="#开源计划"><img src="https://img.shields.io/badge/开源状态-即将开源-orange.svg" alt="Status" /></a>
  </p>
  <p align="center">
    <a href="https://suichuan.zxlee.cn">官网</a> · 
    <a href="https://suichuan.zxlee.cn/#download">下载</a> · 
    <a href="https://suichuan.zxlee.cn/feedback/">意见反馈</a>
  </p>
</div>

---

> **⚠️ 注意：随传目前尚未正式开源。**
>
> 本文档为开源版 README 草案，旨在提前呈现项目最新概貌、工程架构与技术方案。源代码将在版本稳定后正式公开，敬请期待。届时本仓库将同步更新为完整的可构建状态。

---

## 目录

- [项目简介](#项目简介)
- [核心功能](#核心功能)
- [技术架构与方案](#技术架构与方案)
- [平台支持](#平台支持)
- [仓库结构](#仓库结构)
- [开源计划](#开源计划)
- [参与贡献](#参与贡献)
- [联系我们](#联系我们)
- [许可证](#许可证)

---

## 项目简介

**随传**（SuiChuan）是一款**无广告、零云端、离线可用**的局域网多设备协作与文件流转工具。

专注于极速与安全的本地局域网设备流转，连接你的每一台设备，让文件、剪贴板与消息自由流转。跨平台互联，无需云端，轻松实现多设备协同，守护隐私安全。

---

## 核心功能

### 🚀 极速直传
基于局域网高速传输，充分释放网络带宽，无需经过互联网服务器。不限文件大小，离线亦可使用，大文件也能轻松秒传。

### 💬 即时对话
支持以聊天对话的方式在设备间传输文件与消息，跨设备沟通极速高效。提供阅后即焚模式，满足即传、即用、即删的隐私需求。

### 🔗 免装分享
支持通过网页直接共享文件，接收方无需安装应用即可扫码下载。支持手动审批、一次性链接和提取码，让分享安全便捷。

### 📋 无缝粘贴
毫秒级跨设备剪贴板自动同步，一处复制，多端粘贴，让文字、链接、代码自由流转，大幅提升办公与学习效率。

### ⚡ 智能连接
支持智能扫描局域网设备、扫码配对、手动绑定等多种连接方式，快速建立信任关系，零配置、开箱即用。

### 💻 全平台支持
全面支持 iOS、Android、macOS、Windows、Linux，无论手机、平板还是电脑，彻底打破生态壁垒。

### 🔒 隐私安全
采用 X25519 + AES-256-GCM 端对端加密，支持隐身与自定义密钥模式，数据不上云、不留痕，守护您的数据安全。

### 🧹 极简纯净
坚持简洁的设计理念，无广告、无打扰，界面清爽，让每一次连接、传输与分享都更加专注、纯粹、高效。

---

## 技术架构与方案

### 传输与安全握手流程

```
 传输握手与安全信道建立流程（Handshake Flow）
 ──────────────────────────────────────────────────────────
 Step 1: UDP Multicast Discovery   — 局域网广播组播，互相识别节点信息
 Step 2: Key Exchange (RSA / AES)  — 密钥协商握手
           AsymmetricKey:    "RSA-OAEP"
           SessionCipher:    "AES_256_GCM"
           SecurityCheck:    "SHA256_HASH_VALIDATED"
 Step 3: Secure WebSocket Session  — 物理信道协商完成，加密套接字建立
 // [SUCCESS] P2P channel established safely.
```

### 技术栈方案

为了使随传的开发更加轻量、运行更加高效，项目细分为了 Electron 完整包与 Tauri 极轻量包两种 PC 侧客户端，技术方案分工明确：

| 层级 | Electron 桌面端 (PC) | Tauri 桌面端 (PC-Lite) | 移动端 (App) |
|------|---------------------|-----------------------|-------------|
| **UI 框架** | Vue 3 + Element Plus | Vue 3 + Element Plus | Vue 3 + UniBest + UTS |
| **运行平台** | Electron + Node.js | Tauri 2.0 (Rust) + WebView | iOS / Android 客户端 |
| **数据通信** | TCP Socket / HTTP | TCP / HTTP (Rust 底层驱动) | TCP Socket / HTTP |
| **安全加密** | Node.js `crypto` (AES/RSA) | Rust Crypto / SubtleCrypto | WebCrypto API / Polyfill |
| **持久存储** | SQLite (better-sqlite3) | SQLite (Tauri Plugin) | SQLite (uni.openDatabase) |
| **设备发现** | UDP 局域网组播 | UDP 局域网组播 (Rust 驱动) | UDP 局域网组播 |

---

## 平台支持

| 平台 | 最低版本 | 获取方式 | 状态 |
|------|--------|---------|------|
| **Windows** | Windows 10 64位 | 官网 | ✅ 已支持 |
| **macOS** | macOS 11.0 (Big Sur) | 官网 | ✅ 已支持 |
| **Linux** | 主流 Linux 发行版 | 官网 | ✅ 已支持 |
| **iOS** | iOS 14.0 | App Store | ✅ 已支持 |
| **Android** | Android 5.0 (API 21) | 官网 | ✅ 已支持 |
| **HarmonyOS** | — | 官网 | 🚧 计划中 |

---

## 仓库结构

项目采用 **pnpm workspace 单仓库（Monorepo）** 进行模块化开发，业务领域逻辑、加密基元和协议层实现了彻底的平台与框架解耦：

```
suichuan/
├── pnpm-workspace.yaml            # 工作区清单
├── package.json                   # 根脚本 + 共享开发依赖 (devDependencies)
├── tsconfig.base.json             # TypeScript 严格编译基线
├── .npmrc                         # 包管理器镜像与 Electron/Tauri 二进制镜像配置
│
├── packages/
│   ├── protocol/                  # 局域网传输协议 v1 定义（Schemas、错误码、能力位）
│   ├── core/                      # 运行时核心逻辑（状态机、签名构建、重试机制、通用编解码）
│   ├── crypto/                    # 端到端加密基元接口与具体实现提供者（Node Crypto / WebCrypto 共享）
│   ├── database/                  # 数据库架构定义、SQL 语句与实体模型映射
│   ├── domain/                    # 业务领域层（设备身份、心跳维护、即时消息流、剪贴板同步）
│   ├── local-server/              # 本地 HTTP API 接口路由与业务处理器封装
│   ├── share/                     # 网页端提取码临时分享与审批流核心业务逻辑
│   └── branding/                  # Canonic 品牌资源（Logo、协议文档、隐私政策等）
│
├── pc/                            # Electron 桌面端客户端（包含 Vue 3 渲染进程）
├── pc-lite/                       # Tauri 2.0 桌面端客户端（极轻量，基于 Rust）
├── app/                           # uniapp 移动端客户端 + 原生插件
├── official-site/                 # Astro 5.0 + Tailwind CSS 官方宣传网站
├── docs/                          # 开发文档与设计手册
└── deploy/                        # 构建发布与自动化部署脚本
```

---

## 开源计划

随传目前正处于**积极迭代阶段**，核心功能已稳定运行于线上版本。我们计划在以下各阶段工作验证完毕后正式开放源代码：

| 阶段 | 内容 | 状态 |
|------|------|------|
| **Phase 1** | 协议层 `@suichuan/protocol`：制定通讯 Schema、常量与签名规范 | ✅ 完成 |
| **Phase 2a** | 运行时核心 `@suichuan/core`：封装通用状态机、重试与编解码 | ✅ 完成 |
| **Phase 2b** | 模块解耦：抽取抽象加密库 `@suichuan/crypto`、数据库层 `@suichuan/database` | ✅ 完成 |
| **Phase 2c** | 本地服务重构：完成 `@suichuan/local-server` 与临时分享包 `@suichuan/share` | ✅ 完成 |
| **Phase 3** | 桌面端重构：基于 Electron 实现高兼容度完整版客户端 | ✅ 完成 |
| **Phase 4** | 桌面轻量化：基于 Tauri 2.0 架构重构桌面端，推出 `pc-lite` 客户端 | ✅ 完成 |
| **Phase 5** | 移动端迁移：基于最新模块依赖重构 uniapp 移动端 | ✅ 完成 |
| **Phase 6** | 基础功能和体验完善 | 迭代中 |
| **🎉 开源发布** | 完整仓库代码公开，欢迎社区提交 Issue 与 PR 参与共建 | ⏳ 即将到来 |

**想在开源时第一时间收到通知？**
- ⭐ **Star** 本仓库
- 👀 **Watch** 仓库以获取后续更新动态

---

## 参与贡献

在代码开源之前，您仍然可以通过以下方式参与到随传的建设中：

- 🐛 **反馈 Bug**：在 [Issues](https://github.com/SmileZXLee/suichuan-open/issues) 提交您在使用中遇到的问题
- 💡 **功能建议**：在 [Discussions](https://github.com/SmileZXLee/suichuan-open/discussions) 分享您的改进想法
- 📖 **完善文档**：若发现文档中存在错漏，欢迎提交 Pull Request
- 📣 **推广分享**：将随传分享给更多有局域网极速文件互传需求的朋友

代码开源后，我们将正式发布完整的 **CONTRIBUTING.md** 贡献指南，期待社区共建。

---

## 联系我们

- 📧 **联系邮箱**：[admin@zxlee.cn](mailto:admin@zxlee.cn)
- 🌐 **项目官网**：[suichuan.zxlee.cn](https://suichuan.zxlee.cn)
- 💬 **意见反馈**：[https://suichuan.zxlee.cn/feedback](https://suichuan.zxlee.cn/feedback)

---

## 许可证

本项目基于 [MIT License](../LICENSE) 开源。详见根目录 [`LICENSE`](../LICENSE) 文件。

---

<div align="center">

**随传 · SuiChuan** — 自在互联，随传而至。

*Made with ❤️ by the SuiChuan Team*

</div>
