<div align="center">
  <a href="https://suichuan.zxlee.cn" target="_blank" rel="noopener noreferrer">
    <img src="./imgs/logo.png" alt="随传" height="120" />
  </a>
  <h1 align="center">随传</h1>
  <p align="center"><b>自在互联，随传而至。</b></p>
  <p align="center">连接你的每一台设备，让文件、剪贴板与消息自由流转。无需云端，开箱即用。</p>
  <p align="center">
    <a href="../LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License: MIT" /></a>
    <a href="#"><img src="https://img.shields.io/badge/Version-1.0.16-brightgreen.svg" alt="Version" /></a>
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

> **随传目前尚未正式开源。**
>
> 本文档提前呈现项目概貌与技术方案。源代码将在版本稳定后公开，届时本仓库同步更新为完整可构建状态。

---

## 目录

- [项目简介](#项目简介)
- [功能特性](#功能特性)
- [平台支持](#平台支持)
- [技术架构](#技术架构)
  - [连接与安全握手](#连接与安全握手)
  - [分层与三端分工](#分层与三端分工)
- [开发文档](#开发文档)
- [仓库结构](#仓库结构)
- [开源计划](#开源计划)
- [参与贡献](#参与贡献)
- [许可证](#许可证)

---

## 项目简介

随传（SuiChuan）是一款无广告、零云端、离线可用的局域网多设备协作工具，用于在自有设备之间传输文件、同步剪贴板与收发消息。

数据仅在局域网内的设备之间直接流转，不经过任何服务器，也无需注册账号。

---

## 功能特性

| 特性 | 说明 |
|------|------|
| 局域网直传 | 设备间直连传输，不限文件大小，无网络出口亦可使用；支持断点续传 |
| 即时对话 | 以聊天方式在设备间收发消息与文件，支持阅后即焚 |
| 免安装分享 | 接收方无需安装应用，扫码经网页下载；支持提取码、手动审批与一次性链接 |
| 剪贴板同步 | 跨设备剪贴板自动同步，一处复制、多端粘贴 |
| 多种连接方式 | 局域网自动扫描、扫码配对、手动输入 IP，无需额外配置 |
| 端到端加密 | X25519 密钥协商与 AES-256-GCM 加密，支持隐身模式与自定义私人密钥 |
| 跨平台 | Windows、macOS、Linux、iOS、Android |
| 无广告无推送 | 不含广告与推送，不需要账号体系 |

---

## 平台支持

| 平台 | 最低版本 | 获取方式 | 状态 |
|------|--------|---------|------|
| Windows | Windows 10 64 位 | 官网 | 已支持 |
| macOS | macOS 11.0 (Big Sur) | 官网 | 已支持 |
| Linux | 主流发行版 | 官网 | 已支持 |
| iOS | iOS 14.0 | App Store | 已支持 |
| Android | Android 5.0 (API 21) | 官网 | 已支持 |
| HarmonyOS | — | 官网 | 计划中 |

---

## 技术架构

### 连接与安全握手

| 步骤 | 环节 | 说明 |
|------|------|------|
| 1 | 设备发现 | UDP 38890 广播与 HTTP 38888 逐 IP 探测双通道并行，结果合并去重 |
| 2 | 配对绑定 | `POST /api/v1/pair`，对端弹窗确认后交换 X25519 公钥；会话密钥由双方各自经 ECDH 算出，不在网络上传输 |
| 3 | 身份认证 | `POST /api/v1/auth`，已绑定设备变更 IP 前完成挑战-应答证明，防止攻击者冒充设备 ID 将数据引向自身 |
| 4 | 加密传输 | AES-256-GCM 端到端信道，承载消息、剪贴板与文件分片；分片级与全文件级双重 SHA-256 校验，支持断点续传 |

### 分层与三端分工

业务逻辑、协议定义与加密基元均下沉至 `packages/` 共享包，三端共用同一份协议与判定逻辑，平台差异通过依赖注入解决。

| | PC (Electron) | PC-Lite (Tauri) | App (uni-app) |
|---|---|---|---|
| 运行平台 | Electron + Node.js | Tauri 2.0 (Rust) + WebView | iOS / Android 原生 |
| UI | Vue 3 + Element Plus | Vue 3 + Element Plus | Vue 3 + UniBest + UTS |
| 加密实现 | Node `crypto` | Rust（`aes-gcm`、`x25519-dalek`） | `@noble/*` 与 `tweetnacl`（纯 JS） |
| 持久存储 | SQLite (better-sqlite3) | SQLite (rusqlite) | SQLite (uni.openDatabase) |
| 设备发现 | Node `dgram` 与 HTTP 探测 | Rust `std::net` | UTS 原生插件 |

三端共用部分：局域网协议（HTTP/1.1 与 UDP 探测）、X25519 与 AES-256-GCM 加密方案、SQLite 数据模型。

---

## 开发文档

完整文档位于 [`docs/`](./docs/)，描述当前代码的实际行为，包含常量取值与实现位置。

| 文档 | 内容 |
|------|------|
| [API 接口文档](./docs/API.md) | HTTP 接口、响应信封、请求头、错误码、UDP 报文格式 |
| [局域网设备发现](./docs/device-discovery.md) | 双通道扫描、网卡筛选、在线状态判定、IP 跟随、隐身模式 |
| [设备配对与绑定](./docs/device-pairing.md) | 密钥协商流程、三种连接入口、IP 冲突处理、解绑与黑名单 |
| [设备身份认证](./docs/identity-auth.md) | 挑战-应答协议、威胁模型、密钥选择、失败语义 |
| [数据传输](./docs/data-transfer.md) | 消息、剪贴板、文件分片与续传、端到端加密线格式 |
| [设计规范与工程约定](./docs/conventions.md) | 分层依赖、安全约定、并发规则、各端平台注意事项 |

阅读顺序：设备发现 → 配对绑定 → 身份认证 → 数据传输。

---

## 仓库结构

项目采用 pnpm workspace 单仓库（Monorepo）组织：

```
suichuan/
├── packages/          共享包，三端复用，与平台无关
│   ├── protocol/        局域网协议定义：Schema、请求头、错误码、能力位
│   ├── core/            纯逻辑：状态机、编解码、重试策略
│   ├── crypto/          端到端加密基元，Provider 可插拔
│   ├── database/        SQLite Schema、语句与实体映射
│   ├── domain/          领域编排：身份、发现、配对、消息、传输、剪贴板
│   ├── local-server/    本地 HTTP 端点业务处理器
│   ├── share/           网页提取码分享与审批流
│   └── branding/        品牌静态资源
│
├── pc/                Electron 桌面端
├── pc-lite/           Tauri 2.0 桌面端（Rust）
├── app/               uni-app 移动端与原生插件
├── official-site/     Astro 官网
├── docs/              开发文档
└── deploy/            构建与部署脚本
```

---

## 开源计划

核心功能已稳定运行于线上版本，架构解耦与三端重构均已完成。

| 阶段 | 内容 | 状态 |
|------|------|------|
| 架构解耦 | 协议层、运行时核心、加密、数据库、本地服务、分享等共享包全部抽出 | 已完成 |
| 三端重构 | Electron 完整版、Tauri 轻量版、uni-app 移动端均基于共享包重写 | 已完成 |
| 功能与体验完善 | 持续迭代 | 进行中 |
| 开源发布 | 完整仓库公开，接受社区 Issue 与 Pull Request | 待发布 |

如需在开源时获得通知，可 Star 与 Watch 本仓库。

---

## 参与贡献

代码开源前，可通过以下方式参与：

- 反馈缺陷：[Issues](https://github.com/SmileZXLee/suichuan-open/issues)
- 功能建议：[Discussions](https://github.com/SmileZXLee/suichuan-open/discussions)
- 完善文档：发现错漏可提交 Pull Request
- 推广项目：向有局域网互传需求的用户介绍随传

代码开源后将发布完整的 CONTRIBUTING.md 贡献指南。

联系方式：[admin@zxlee.cn](mailto:admin@zxlee.cn) · [官网](https://suichuan.zxlee.cn) · [意见反馈](https://suichuan.zxlee.cn/feedback/)

---

## 许可证

基于 [MIT License](../LICENSE) 开源。

<div align="center">
<sub><b>随传 · SuiChuan</b> — 自在互联，随传而至。</sub>
</div>
