# 随传 (SuiChuan) API 接口文档

## 目录

- [1. 概述](#1-概述)
  - [1.1 服务地址与端口](#11-服务地址与端口)
  - [1.2 接口总览](#12-接口总览)
- [2. 通用约定](#2-通用约定)
  - [2.1 响应信封](#21-响应信封)
  - [2.2 通用请求头](#22-通用请求头)
  - [2.3 端到端加密](#23-端到端加密)
  - [2.4 文件分片请求头](#24-文件分片请求头)
  - [2.5 公共数据结构](#25-公共数据结构)
  - [2.6 准入与加密策略](#26-准入与加密策略)
- [3. 设备接口](#3-设备接口)
  - [3.1 GET /api/v1/info](#31-get-apiv1info)
  - [3.2 GET /api/v1/heartbeat](#32-get-apiv1heartbeat)
  - [3.3 GET /api/v1/ping](#33-get-apiv1ping)
  - [3.4 POST /api/v1/pair](#34-post-apiv1pair)
  - [3.5 POST /api/v1/auth](#35-post-apiv1auth)
- [4. 业务数据接口](#4-业务数据接口)
  - [4.1 POST /api/v1/messages](#41-post-apiv1messages)
  - [4.2 POST /api/v1/clipboard](#42-post-apiv1clipboard)
- [5. 文件传输接口](#5-文件传输接口)
  - [5.1 GET /api/v1/files/offset](#51-get-apiv1filesoffset)
  - [5.2 POST /api/v1/files/chunk](#52-post-apiv1fileschunk)
  - [5.3 POST /api/v1/files/finish](#53-post-apiv1filesfinish)
  - [5.4 POST /api/v1/files/cancel](#54-post-apiv1filescancel)
  - [5.5 POST /api/v1/files/upload](#55-post-apiv1filesupload)
  - [5.6 传输场景区分](#56-传输场景区分)
- [6. 网页提取码分享接口](#6-网页提取码分享接口)
  - [6.1 GET /s](#61-get-s)
  - [6.2 GET /s/info](#62-get-sinfo)
  - [6.3 GET /s/submit](#63-get-ssubmit)
  - [6.4 GET /s/approval-check](#64-get-sapproval-check)
  - [6.5 GET /s/dl/{token}](#65-get-sdltoken)
  - [6.6 GET /s/icon](#66-get-sicon)
- [7. 服务状态接口](#7-服务状态接口)
- [8. 错误码](#8-错误码)
  - [8.1 完整错误码表](#81-完整错误码表)
  - [8.2 协议外错误码](#82-协议外错误码)
- [9. UDP 设备发现协议](#9-udp-设备发现协议)
- [10. 跨端差异](#10-跨端差异)

相关流程文档：[设备发现](./device-discovery.md)、[配对绑定](./device-pairing.md)、[身份认证](./identity-auth.md)、[数据传输](./data-transfer.md)、[设计规范](./conventions.md)。
---

## 1. 概述

### 1.1 服务地址与端口

每台设备各自运行一个本地 HTTP 服务，设备之间点对点直连，不经过中心服务器。请求地址形如 `http://<对端IP>:38888/api/v1/<接口>`。

| 项 | 值 |
| :--- | :--- |
| 基础路径 | `/api/v1` |
| HTTP 端口 | `38888`（常量 `DEFAULT_API_PORT`） |
| UDP 发现端口 | `38890`（常量 `DEFAULT_DISCOVERY_PORT`） |
| 请求 Content-Type | `application/json; charset=utf-8`；加密 JSON 为 `text/plain`；二进制为 `application/octet-stream` |
| 响应 Content-Type | `application/json; charset=utf-8`；网页分享为 `text/html`；服务状态为 `text/plain` |
| 字符编码 | UTF-8 |
| 版本标识 | 路径中的 `/api/v1` |

### 1.2 接口总览

| 接口 | 方法 | 说明 | 需绑定 | 明文 |
| :--- | :--- | :--- | :--- | :--- |
| [`/api/v1/info`](#31-get-apiv1info) | GET | 获取设备公开身份信息 | 否 | 允许 |
| [`/api/v1/heartbeat`](#32-get-apiv1heartbeat) | GET | 存活探测 | 否 | 允许 |
| [`/api/v1/ping`](#33-get-apiv1ping) | GET | 本机服务自检 | 否 | 允许 |
| [`/api/v1/pair`](#34-post-apiv1pair) | POST | 设备配对，交换公钥 | 否 | 固定明文 |
| [`/api/v1/auth`](#35-post-apiv1auth) | POST | 设备身份认证 | 是 | 固定明文 |
| [`/api/v1/messages`](#41-post-apiv1messages) | POST | 发送聊天消息 | 是 | 受策略约束 |
| [`/api/v1/clipboard`](#42-post-apiv1clipboard) | POST | 推送剪贴板内容 | 是 | 受策略约束 |
| [`/api/v1/files/offset`](#51-get-apiv1filesoffset) | GET | 查询续传进度 | 是 | 允许 |
| [`/api/v1/files/chunk`](#52-post-apiv1fileschunk) | POST | 上传文件分片 | 是 | 受策略约束 |
| [`/api/v1/files/finish`](#53-post-apiv1filesfinish) | POST | 完成文件传输 | 是 | 允许 |
| [`/api/v1/files/cancel`](#54-post-apiv1filescancel) | POST | 取消文件传输 | 是 | 允许 |
| [`/api/v1/files/upload`](#55-post-apiv1filesupload) | POST | 单请求整文件上传 | 是 | 受策略约束 |
| [`/s`](#61-get-s) | GET | 提取码网页 | 否 | 允许 |
| [`/s/info`](#62-get-sinfo) | GET | 提取码文件元信息 | 否 | 允许 |
| [`/s/submit`](#63-get-ssubmit) | GET | 提交提取码 | 否 | 允许 |
| [`/s/approval-check`](#64-get-sapproval-check) | GET | 轮询审批状态 | 否 | 允许 |
| [`/s/dl/{token}`](#65-get-sdltoken) | GET | 下载分享文件 | 否 | 允许 |
| [`/s/icon`](#66-get-sicon) | GET | 分享页图标 | 否 | 允许 |
| [`/`](#7-服务状态接口) | GET | 服务运行状态 | 否 | 允许 |

「需绑定」为「是」时，服务端校验请求头 `X-Source-Device-Id` 是否在本机绑定列表内，不在则返回 `AUTH_NOT_BOUND`。「受策略约束」表示是否接受明文取决于接收端的「允许接收不安全的数据」开关，见 [2.6](#26-准入与加密策略)。

---

## 2. 通用约定

### 2.1 响应信封

全部 JSON 响应使用统一信封，定义于 [`packages/protocol/src/response.ts`](../packages/protocol/src/response.ts)。

**成功响应**

```json
{
  "service": "suichuan",
  "success": true,
  "data": {}
}
```

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `service` | string | 是 | 固定值 `"suichuan"`，标识对端为随传服务 |
| `success` | boolean | 是 | 固定 `true` |
| `data` | object \| array | 否 | 业务载荷。仅返回操作结果的接口不带该字段 |
| `requestId` | string | 否 | 回显请求的 `X-Request-Id` |

**失败响应**

```json
{
  "service": "suichuan",
  "success": false,
  "code": "AUTH_NOT_BOUND",
  "message": "Device is not bound"
}
```

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `service` | string | 是 | 固定值 `"suichuan"` |
| `success` | boolean | 是 | 固定 `false` |
| `code` | string | 是 | 机器可读错误码，取值见[第 8 章](#8-错误码) |
| `message` | string | 否 | 英文错误描述，不参与判定 |
| `retryable` | boolean | 否 | 是否可重试。缺省时取错误码的默认可重试性，见 [8.1](#81-完整错误码表) |
| `details` | object | 否 | 结构化上下文，取值随接口而定，如 `{ "field": "requestId" }`、`{ "missingChunks": [3, 7] }` |
| `requestId` | string | 否 | 回显请求的 `X-Request-Id` |

HTTP 状态码：成功恒为 `200`；失败由错误码经 `ERROR_HTTP_STATUS` 映射得出，见 [8.1](#81-完整错误码表)。

`/api/v1/info` 与 `/api/v1/heartbeat` 在 pc、app 与 pc-lite 三端均统一返回标准信封。

### 2.2 通用请求头

| Header | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `X-Source-Device-Id` | string | 业务接口必填 | 发送方设备 ID。服务端用其做绑定列表过滤 |
| `X-Request-Id` | string | 可选 | 请求关联 ID，格式 `req-<时间戳>-<随机串>`。接收端用于跨端去重，并在响应中回显 |
| `Content-Type` | string | 有请求体时必填 | 取值见 [1.1](#11-服务地址与端口) |
| `Content-Length` | number | 有请求体时必填 | 加密分片为「明文长度 + 16」 |

`X-Source-Device-Id` 与 `X-Request-Id` 只出现在请求头，不写入请求体。

### 2.3 端到端加密

| Header | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `X-E2E-Encrypted` | string | 加密时必填 | 固定 `"true"`。缺失或取值非 `"true"` 时按明文处理 |
| `X-E2E-IV` | string | 加密时必填 | Base64 编码的 12 字节 AES-GCM IV，每请求随机生成 |
| `X-E2E-Tag` | string | JSON 类加密请求必填 | Base64 编码的 16 字节认证标签。分片请求不使用该头，标签位于请求体末尾 |
| `X-E2E-Keycheck` | string | 可选 | Base64 编码的 `{iv, ciphertext, tag}` JSON，明文为双方约定的固定校验串。接收端用其区分密钥不匹配与数据损坏 |

**请求体编码方式**

| 场景 | Content-Type | 请求体内容 |
| :--- | :--- | :--- |
| JSON 加密 | `text/plain` | `base64(AES-256-GCM(整段 JSON 字符串))` |
| JSON 明文 | `application/json` | 明文 JSON |
| 二进制加密 | `application/octet-stream` | `ciphertext ‖ tag(16 字节)`，不作 Base64 |
| 二进制明文 | `application/octet-stream` | 原始字节 |

加密算法为 AES-256-GCM，密钥为 32 字节最终密钥：由配对时协商的传输密钥经 HKDF-SHA256 再混入应用内置密钥与用户端对端加密私人密钥派生得出，不落库。派生结构见 [device-pairing.md 第 3.2 节](./device-pairing.md#32-密钥派生)。

发送端无会话密钥时回退为明文，接收端是否接受取决于 [2.6](#26-准入与加密策略) 的策略。

### 2.4 文件分片请求头

用于 [`/api/v1/files/chunk`](#52-post-apiv1fileschunk) 与 [`/api/v1/files/finish`](#53-post-apiv1filesfinish)。

| Header | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `X-Transfer-Id` | string | 是 | 传输任务 ID，全场景唯一。前缀决定业务场景，见 [5.6](#56-传输场景区分) |
| `X-File-Name` | string | 是 | 文件名，百分号编码的 UTF-8 |
| `X-File-Size` | number | 是 | 文件总字节数，空文件为 `0` |
| `X-Chunk-Index` | number | chunk 必填 | 分片索引，从 `0` 开始 |
| `X-Chunk-Offset` | number | chunk 必填 | 该分片在文件中的起始字节偏移，等于 `chunkIndex × chunkSize` |
| `X-Chunk-Size` | number | 是 | 分片大小，默认 `4194304`（4 MiB） |
| `X-Chunk-Plain-Size` | number | chunk 必填 | 该分片解密后的明文字节数，末片小于 `chunkSize` |
| `X-Chunk-Count` | number | 是 | 分片总数，等于 `ceil(fileSize / chunkSize)`，空文件为 `0` |
| `X-Chunk-Sha256` | string | chunk 必填 | 该分片明文的 SHA-256，64 位小写 Hex |
| `X-File-Sha256` | string | finish 必填 | 全文件 SHA-256，64 位小写 Hex |

服务端校验上述参数的自洽性，不符时返回 `TRANSFER_RANGE_INVALID`。

### 2.5 公共数据结构

#### DeviceInfo

出现在 [`/api/v1/info`](#31-get-apiv1info) 响应、[`/api/v1/pair`](#34-post-apiv1pair) 的请求体与响应，以及 UDP 应答包中。

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `id` | string | 是 | 设备唯一 ID，格式 `device-<UUIDv4>`，生成后持久不变 |
| `name` | string | 是 | 设备名称，最长 64 字符，用户可修改 |
| `type` | string | 是 | 设备形态，取值见 [DeviceType 枚举](#devicetype-枚举type) |
| `platform` | string | 是 | 操作系统，取值见 [Platform 枚举](#platform-枚举platform) |
| `model` | string | 否 | 设备型号，如 `iPhone`、`MacBook Pro`，最长 64 字符 |
| `ip` | string | 是 | 设备当前主 IP |
| `port` | number | 是 | HTTP 服务端口，默认 `38888` |
| `version` | string | 否 | 应用版本号，语义化版本，如 `1.0.16` |
| `remark` | string | 否 | 备注名 |
| `status` | string | 否 | 固定 `"online"`，仅 pc-lite 端下发 |
| `lastSeen` | number | 否 | 生成该响应时的 Unix 毫秒时间戳，仅 pc-lite 端下发 |

`publicKey` 与 `ips`（本机全部地址）在 `/api/v1/info` 响应中被移除。

#### DeviceType 枚举（`type`）

| 取值 | 含义 | 产出端 |
| :--- | :--- | :--- |
| `smartphone` | 手机 | app |
| `tablet` | 平板 | app |
| `laptop_mac` | Mac 笔记本 | pc、pc-lite |
| `desktop_mac` | Mac 台式机 | pc、pc-lite |
| `laptop_windows` | Windows 笔记本 | pc、pc-lite |
| `desktop_windows` | Windows 台式机 | pc、pc-lite |
| `tablet_mac` | iPad 类设备 | pc |
| `computer` | 未细分的计算机，Linux 归此类 | pc、pc-lite |
| `unknown` | 未知 | 兜底值 |

未列出的取值按 `unknown` 处理，使用默认图标。

#### Platform 枚举（`platform`）

| 取值 | 说明 |
| :--- | :--- |
| `iOS` | iOS 与 iPadOS |
| `Android` | Android |
| `macOS` | macOS |
| `Windows` | Windows |
| `Linux` | Linux |
| `HarmonyOS` | HarmonyOS |
| 其它 | 系统原值透传，按未知处理 |

### 2.6 准入与加密策略

绑定准入：需绑定的接口在处理前校验 `X-Source-Device-Id` 是否属于本机绑定列表，不属于则返回 `AUTH_NOT_BOUND`（403）。

明文策略：接收端默认仅接受加密数据。收到明文请求且未开启「允许接收不安全的数据」时返回 `E2E_PLAINTEXT_REJECTED`（403）。以下接口不受该策略约束：

| 接口 | 说明 |
| :--- | :--- |
| `/api/v1/pair` | 配对发生在密钥建立之前 |
| `/api/v1/auth` | 应答方须先读取 `sourceDeviceId` 才能确定密钥 |
| `/api/v1/info`、`/api/v1/heartbeat`、`/api/v1/ping` | 公开探测接口，不含业务数据 |
| `/api/v1/files/offset`、`/api/v1/files/finish`、`/api/v1/files/cancel` | 仅传输控制元数据，不含文件内容 |
| `/s/*` | 面向浏览器 |

隐身模式：启用后 `/api/v1/info` 返回 404 且不响应 UDP 探测，其余接口行为不变。

---

## 3. 设备接口

### 3.1 GET /api/v1/info

获取对端公开身份信息。用于设备发现的 HTTP 探测通道，以及手动输入 IP 时的连通性确认。

**请求**

| 项 | 值 |
| :--- | :--- |
| 方法与路径 | `GET /api/v1/info` |
| 需绑定 | 否 |
| 路径参数 | 无 |
| Query 参数 | 无 |
| 请求头 | 无 |
| 请求体 | 无 |
| 客户端超时 | 探测 350ms（pc-lite 扫描）／ 1500ms（单播探测）／ 3000ms（手动输入 IP） |

**响应**

`data` 为 [DeviceInfo](#deviceinfo) 对象。

```json
{
  "service": "suichuan",
  "success": true,
  "data": {
    "id": "device-00000000-0000-0000-0000-000000000000",
    "name": "MacBook Pro",
    "type": "laptop_mac",
    "platform": "macOS",
    "model": "MacBook Pro",
    "ip": "192.168.1.10",
    "port": 38888,
    "version": "1.0.16"
  }
}
```

**错误码**

| 错误码 | HTTP | 触发条件 |
| :--- | :--- | :--- |
| `ROUTE_NOT_FOUND` | 404 | 本机处于隐身模式 |

### 3.2 GET /api/v1/heartbeat

存活探测。由心跳轮询按 3000ms 间隔调用（常量 `HEARTBEAT_INTERVAL_MS`）。

**请求**

| 项 | 值 |
| :--- | :--- |
| 方法与路径 | `GET /api/v1/heartbeat` |
| 需绑定 | 否 |
| 路径参数 | 无 |
| Query 参数 | 无 |
| 请求头 | 无 |
| 请求体 | 无 |
| 客户端超时 | 1500ms（常量 `HEARTBEAT_TIMEOUT_MS`） |

**响应**

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `data.online` | boolean | 是 | 固定 `true` |
| `data.timestamp` | number | 是 | 服务端当前 Unix 毫秒时间戳 |
| `data.deviceId` | string | 是 | 本机设备 ID |

```json
{
  "service": "suichuan",
  "success": true,
  "data": {
    "online": true,
    "timestamp": 1721000000000,
    "deviceId": "device-00000000-0000-0000-0000-000000000000"
  }
}
```

在线状态由连续失败次数判定，达到 2 次（常量 `HEARTBEAT_FAILURE_THRESHOLD`）转为离线，规则见 [device-discovery.md 第 6 节](./device-discovery.md#6-在线状态判定)。

### 3.3 GET /api/v1/ping

本机 HTTP 服务自检，启动流程中经 `127.0.0.1` 调用，确认服务处于监听状态。仅 app 端提供。

**请求**

| 项 | 值 |
| :--- | :--- |
| 方法与路径 | `GET /api/v1/ping` |
| 需绑定 | 否 |
| 路径参数 | 无 |
| Query 参数 | 无 |
| 请求体 | 无 |

**响应**

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `data.pong` | boolean | 是 | 固定 `true` |

```json
{ "service": "suichuan", "success": true, "data": { "pong": true } }
```

### 3.4 POST /api/v1/pair

设备配对。发送方提交自身设备信息与 X25519 公钥，接收方弹窗等待用户确认，确认后双方各自完成 ECDH 密钥协商并互相写入绑定列表。流程说明见 [device-pairing.md](./device-pairing.md)。

**请求**

| 项 | 值 |
| :--- | :--- |
| 方法与路径 | `POST /api/v1/pair` |
| 需绑定 | 否 |
| Content-Type | `application/json` |
| 加密 | 固定明文 |
| 路径参数 | 无 |
| Query 参数 | 无 |
| 客户端超时 | 65000ms（对端弹窗等待上限 60s） |

**请求体**

| 字段 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `device` | object | 是 | 发送方的 [DeviceInfo](#deviceinfo) |
| `device.id` | string | 是 | 发送方设备 ID |
| `device.name` | string | 是 | 发送方设备名 |
| `device.type` | string | 是 | 取值见 [DeviceType 枚举](#devicetype-枚举type) |
| `device.platform` | string | 是 | 取值见 [Platform 枚举](#platform-枚举platform) |
| `device.model` | string | 否 | 设备型号 |
| `device.port` | number | 是 | 发送方 HTTP 端口 |
| `device.ip` | string | 否 | 接收方以本次请求的 socket 地址为准，忽略该字段 |
| `publicKey` | string | 是 | 发送方 X25519 公钥，Base64 编码的 32 字节原始公钥 |

```json
{
  "device": {
    "id": "device-00000000-0000-0000-0000-000000000000",
    "name": "iPhone",
    "type": "smartphone",
    "platform": "iOS",
    "model": "iPhone",
    "port": 38888
  },
  "publicKey": "BASE64_X25519_PUBLIC_KEY"
}
```

**响应**

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `data.accepted` | boolean | 是 | 固定 `true`。拒绝走失败响应 |
| `data.device` | object | 是 | 接收方的 [DeviceInfo](#deviceinfo)，其 `id` 为写入绑定列表的权威设备 ID |
| `data.publicKey` | string | 是 | 接收方 X25519 公钥，Base64。发起方用其完成镜像 ECDH 计算 |

```json
{
  "service": "suichuan",
  "success": true,
  "data": {
    "accepted": true,
    "device": {
      "id": "device-11111111-1111-1111-1111-111111111111",
      "name": "MacBook Pro",
      "type": "laptop_mac",
      "platform": "macOS",
      "ip": "192.168.1.10",
      "port": 38888
    },
    "publicKey": "BASE64_X25519_PUBLIC_KEY"
  }
}
```

**错误码**

| 错误码 | HTTP | 触发条件 | `details` |
| :--- | :--- | :--- | :--- |
| `PAIR_BLACKLISTED` | 403 | 发送方设备 ID 在接收方黑名单中 | `{ "reason": "blacklisted" }` |
| `PAIR_REJECTED` | 403 | 用户拒绝，或弹窗 60s 内未确认 | `{ "reason": "user_declined" \| "blacklisted" \| "timeout" }` |
| `PROTO_BAD_REQUEST` | 400 | 请求体非合法 JSON，或缺少必填字段 | 无 |

```json
{
  "service": "suichuan",
  "success": false,
  "code": "PAIR_REJECTED",
  "message": "User declined pairing request",
  "details": { "reason": "user_declined" }
}
```

### 3.5 POST /api/v1/auth

设备身份认证。挑战方发送一次性随机串，应答方以双方会话密钥加密证明回传，挑战方解密比对后确认对端身份。

调用时机：已绑定设备的 IP 变更时，验证通过后才更新其 IP。流程说明与威胁模型见 [identity-auth.md](./identity-auth.md)。

**请求**

| 项 | 值 |
| :--- | :--- |
| 方法与路径 | `POST /api/v1/auth` |
| 需绑定 | 是，挑战方须在应答方的绑定列表内 |
| Content-Type | `application/json` |
| 加密 | 固定明文 |
| 路径参数 | 无 |
| Query 参数 | 无 |
| 客户端超时 | 3000ms |

**请求体**

| 字段 | 类型 | 必填 | 说明 | 取值约束 |
| :--- | :--- | :--- | :--- | :--- |
| `sourceDeviceId` | string | 是 | 挑战方设备 ID。应答方用其定位对端并取出会话密钥 | 长度 1–128 |
| `nonce` | string | 是 | 一次性随机挑战串，每次挑战重新生成 | 固定 7 位，字符集 `[0-9a-zA-Z]` |
| `targetIp` | string | 是 | 挑战方认为应答方所处的地址。应答方校验该地址是否属于本机 | 长度 7–45 |

```json
{
  "sourceDeviceId": "device-00000000-0000-0000-0000-000000000000",
  "nonce": "a1B2c3D",
  "targetIp": "192.168.1.77"
}
```

**响应**

`data` 为 AES-256-GCM 信封：

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `data.iv` | string | 是 | Base64 编码的 12 字节 IV |
| `data.ciphertext` | string | 是 | Base64 编码的密文 |
| `data.tag` | string | 是 | Base64 编码的 16 字节认证标签 |

```json
{
  "service": "suichuan",
  "success": true,
  "data": {
    "iv": "BASE64_12_BYTES",
    "ciphertext": "BASE64_CIPHERTEXT",
    "tag": "BASE64_16_BYTES"
  }
}
```

密文解密后为下述 JSON：

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `nonce` | string | 是 | 原样回显请求中的挑战串 |
| `ip` | string | 是 | 应答方已确认属于本机的地址，即请求中的 `targetIp` |
| `deviceId` | string | 是 | 应答方自身的设备 ID |

```json
{
  "nonce": "a1B2c3D",
  "ip": "192.168.1.77",
  "deviceId": "device-11111111-1111-1111-1111-111111111111"
}
```

加解密密钥为最终密钥，与消息、文件传输一致，见 [2.3](#23-端到端加密)。

**挑战方的判定规则**

解密后比对三项，全部相等为验证通过；任一项不等、或请求失败、超时、解密失败，均为验证不通过：

| 序号 | 比对项 | 期望值 |
| :--- | :--- | :--- |
| 1 | `nonce` | 本次发出的挑战串 |
| 2 | `ip` | 本次待验证的候选地址 |
| 3 | `deviceId` | 待确认设备的 ID |

对端为无此接口的旧版本时返回 `ROUTE_NOT_FOUND`（404），按验证不通过处理。

**错误码**

| 错误码 | HTTP | 触发条件 |
| :--- | :--- | :--- |
| `PROTO_BAD_REQUEST` | 400 | 字段缺失，或 `nonce` 长度、字符集不符 |
| `AUTH_NOT_BOUND` | 403 | 挑战方不在应答方的绑定列表中 |
| `AUTH_IP_MISMATCH` | 403 | `targetIp` 不属于应答方本机地址 |
| `E2E_SESSION_MISSING` | 400 | 绑定关系存在，但本地无该对端的会话密钥 |

---

## 4. 业务数据接口

### 4.1 POST /api/v1/messages

发送一条聊天消息。消息内的文件不经本接口传输，仅携带 `transferId` 元信息，文件字节走分片通道。

**请求**

| 项 | 值 |
| :--- | :--- |
| 方法与路径 | `POST /api/v1/messages` |
| 需绑定 | 是 |
| Content-Type | `application/json`；加密时为 `text/plain` |
| 加密 | 受策略约束，见 [2.6](#26-准入与加密策略) |
| 必填请求头 | `X-Source-Device-Id` |
| 路径参数 | 无 |
| Query 参数 | 无 |
| 客户端超时 | 30000ms |

**请求体**

| 字段 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `id` | string | 是 | 消息 ID，格式 `msg-<时间戳>-<随机串>`。撤回操作按该 ID 定位消息 |
| `type` | string | 是 | 消息类型，取值见 [MessageType 枚举](#messagetype-枚举type) |
| `content` | string | 否 | 文本内容。`type` 为 `text`、`link` 时必填 |
| `timestamp` | number | 是 | 发送时间，Unix 毫秒 |
| `sourceDeviceName` | string | 否 | 发送方设备名。接收端无本地备注时用其展示 |
| `fileSize` | number | 否 | 文件字节数。`type` 为 `file`、`image` 时填写 |
| `mimeType` | string | 否 | 文件 MIME 类型，如 `image/png` |
| `transferId` | string | 否 | 关联的文件传输任务 ID，前缀为 `chat-transfer-`。文件类消息填写 |
| `autoDestruct` | boolean | 否 | 是否启用阅后即焚 |
| `autoDestructSeconds` | number | 否 | 阅后即焚倒计时秒数 |
| `autoDestructAt` | number | 否 | 绝对销毁时间，Unix 毫秒。与 `autoDestructSeconds` 二者其一 |
| `imageWidth` | number | 否 | 图片像素宽度，接收端在下载完成前用其占位 |
| `imageHeight` | number | 否 | 图片像素高度 |
| `videoDuration` | number | 否 | 视频时长，毫秒。仅 app 端产出 |
| `videoThumbnail` | string | 否 | 视频缩略图。仅 app 端产出 |

`sourceDeviceId` 与 `requestId` 不在请求体内，见 [2.2](#22-通用请求头)。

#### MessageType 枚举（`type`）

| 取值 | 说明 | 配套字段 |
| :--- | :--- | :--- |
| `text` | 纯文本消息 | `content` |
| `file` | 文件消息 | `transferId`、`fileSize` |
| `image` | 图片消息 | `transferId`、`fileSize`、`imageWidth`、`imageHeight` |
| `link` | 链接消息 | `content` |
| `recall` | 撤回指令，接收端删除 `id` 对应的消息 | `id` |

```json
{
  "id": "msg-1721000000000-a1b2c3",
  "type": "text",
  "content": "Hello",
  "timestamp": 1721000000000,
  "sourceDeviceName": "iPhone"
}
```

**响应**

成功时不带 `data`：

```json
{ "service": "suichuan", "success": true }
```

**错误码**

| 错误码 | HTTP | 触发条件 |
| :--- | :--- | :--- |
| `AUTH_NOT_BOUND` | 403 | 发送方不在接收方绑定列表中 |
| `E2E_PLAINTEXT_REJECTED` | 403 | 收到明文且接收端未允许不安全数据 |
| `E2E_SESSION_MISSING` | 400 | 接收端无该发送方的会话密钥 |
| `E2E_DECRYPTION_FAILED` | 400 | 解密失败，两端私人密钥不一致时出现 |
| `PROTO_BAD_REQUEST` | 400 | 解密后内容非合法 JSON |
| `MESSAGE_TOO_LARGE` | 413 | 消息内容超出接收端限制 |

### 4.2 POST /api/v1/clipboard

推送剪贴板内容。去重与回环抑制在应用层完成，本接口为单向投递。

**请求**

| 项 | 值 |
| :--- | :--- |
| 方法与路径 | `POST /api/v1/clipboard` |
| 需绑定 | 是 |
| Content-Type | `application/json`；加密时为 `text/plain` |
| 加密 | 受策略约束 |
| 必填请求头 | `X-Source-Device-Id` |
| 路径参数 | 无 |
| Query 参数 | 无 |
| 客户端超时 | 30000ms |

**请求体**

| 字段 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `type` | string | 是 | 内容类型，取值见 [ClipboardType 枚举](#clipboardtype-枚举type) |
| `content` | string | 是 | 文本内容；`type` 为 `image` 时为图片的关联信息 |
| `timestamp` | number | 是 | 复制时间，Unix 毫秒。接收端用其判定新旧 |
| `isDirectSend` | boolean | 否 | `true` 为用户主动一次性直发，缺省或 `false` 为自动同步广播。接收端据此决定是否写入历史与是否提示 |
| `sourceDeviceId` | string | 否 | 体内冗余字段，接收端优先取请求头 |
| `targetDeviceId` | string | 否 | 目标设备 ID，发送端用其选择加密密钥 |

#### ClipboardType 枚举（`type`）

| 取值 | 说明 |
| :--- | :--- |
| `text` | 文本、链接、代码等纯文本内容 |
| `image` | 图片。字节走分片通道，本接口投递元信息 |

```json
{
  "type": "text",
  "content": "https://example.com",
  "timestamp": 1721000000000,
  "isDirectSend": false
}
```

**响应**

```json
{ "service": "suichuan", "success": true }
```

**错误码**

除 [4.1](#41-post-apiv1messages) 所列外，另有：

| 错误码 | HTTP | 触发条件 |
| :--- | :--- | :--- |
| `CLIPBOARD_DISABLED` | 403 | 接收端关闭了剪贴板同步 |

---

## 5. 文件传输接口

分片参数定义于 [`packages/protocol/src/file-chunk.ts`](../packages/protocol/src/file-chunk.ts)：

| 参数 | 值 |
| :--- | :--- |
| 分片大小 | 4 MiB（`4194304` 字节） |
| 并发分片数 | 4 |
| AES-GCM IV 长度 | 12 字节 |
| AES-GCM Tag 长度 | 16 字节 |

### 5.1 GET /api/v1/files/offset

查询接收端已收到的分片进度。发送端在开始传输前与重连后调用。

**请求**

| 项 | 值 |
| :--- | :--- |
| 方法与路径 | `GET /api/v1/files/offset` |
| 需绑定 | 是 |
| 路径参数 | 无 |
| 请求体 | 无 |
| 客户端超时 | 1500ms |

**Query 参数**

| 参数 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `transferId` | string | 是 | 传输任务 ID |
| `deviceId` | string | 是 | 发送方自身的设备 ID。接收端用其定位缓存目录 |
| `fileSize` | number | 否 | 文件总字节数。接收端用其校验已存布局是否匹配 |
| `chunkSize` | number | 否 | 分片大小，缺省按 4 MiB |
| `chunkCount` | number | 否 | 分片总数，缺省由 `fileSize` 与 `chunkSize` 推导 |

**响应**

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `data.offset` | number | 是 | 已接收的明文字节总数，无缓存时为 `0` |
| `data.chunkSize` | number | 是 | 接收端记录的分片大小，发送端沿用该值 |
| `data.chunkCount` | number | 是 | 接收端记录的分片总数 |
| `data.receivedChunks` | number[] | 是 | 已完整接收的分片索引，升序。发送端跳过这些索引 |
| `data.completed` | boolean | 是 | `true` 表示该文件此前已传输完成 |

```json
{
  "service": "suichuan",
  "success": true,
  "data": {
    "offset": 12582912,
    "chunkSize": 4194304,
    "chunkCount": 8,
    "receivedChunks": [0, 1, 2],
    "completed": false
  }
}
```

发送端声明的布局与接收端已存元信息不一致时，接收端清空缓存，按全新传输处理。

**错误码**

| 错误码 | HTTP | 触发条件 |
| :--- | :--- | :--- |
| `AUTH_NOT_BOUND` | 403 | `deviceId` 不在接收方绑定列表中 |

### 5.2 POST /api/v1/files/chunk

上传单个文件分片。请求体为二进制，元数据经请求头传递。同一任务最多 4 路并发。

**请求**

| 项 | 值 |
| :--- | :--- |
| 方法与路径 | `POST /api/v1/files/chunk` |
| 需绑定 | 是 |
| Content-Type | `application/octet-stream` |
| 加密 | 受策略约束 |
| 路径参数 | 无 |
| Query 参数 | 无 |

**请求头**

[2.2](#22-通用请求头) 的通用头，加 [2.4](#24-文件分片请求头) 中标注为「chunk 必填」的全部头。加密时另有 `X-E2E-Encrypted`、`X-E2E-IV`，以及可选的 `X-E2E-Keycheck`。

**请求体**

| 场景 | 内容 | 长度 |
| :--- | :--- | :--- |
| 明文 | 分片原始字节 | 等于 `X-Chunk-Plain-Size` |
| 加密 | `ciphertext ‖ tag(16 字节)` | 等于 `X-Chunk-Plain-Size + 16` |

**服务端处理顺序**

1. 绑定准入与明文策略校验；
2. 分片参数自洽性校验（`chunkCount`、`chunkOffset`、末片大小、`Content-Length`）；
3. 按需解密，含 `X-E2E-Keycheck` 校验；
4. 分片明文 SHA-256 与 `X-Chunk-Sha256` 比对；
5. 落盘：先写临时文件，再重命名为 `<index>.chunk`。

**响应**

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `data.transferId` | string | 是 | 回显传输任务 ID |
| `data.chunkIndex` | number | 是 | 回显本次分片索引 |
| `data.size` | number | 是 | 本次落盘的明文字节数 |
| `data.receivedBytes` | number | 是 | 该任务已接收的明文字节总数 |

```json
{
  "service": "suichuan",
  "success": true,
  "data": {
    "transferId": "transfer-1721000000000-a1b2c3",
    "chunkIndex": 3,
    "size": 4194304,
    "receivedBytes": 16777216
  }
}
```

**错误码**

| 错误码 | HTTP | 触发条件 |
| :--- | :--- | :--- |
| `AUTH_NOT_BOUND` | 403 | 发送方不在绑定列表中 |
| `E2E_PLAINTEXT_REJECTED` | 403 | 收到明文且接收端未允许不安全数据 |
| `PROTO_BAD_REQUEST` | 400 | 必填头缺失，或 `X-Chunk-Sha256` 非 64 位 Hex |
| `TRANSFER_RANGE_INVALID` | 416 | 分片参数不自洽、偏移错误，或 `Content-Length` 与声明不符 |
| `TRANSFER_CHECKSUM_MISMATCH` | 422 | 分片 SHA-256 校验失败 |
| `TRANSFER_CANCELLED` | 410 | 该任务已被取消 |
| `E2E_SESSION_MISSING` | 400 | 无该发送方的会话密钥 |
| `E2E_DECRYPTION_FAILED` | 400 | 分片解密失败 |
| `IO_WRITE_FAILED` | 500 | 接收端写盘失败 |
| `IO_DISK_FULL` | 507 | 接收端磁盘空间不足 |

### 5.3 POST /api/v1/files/finish

声明分片全部发送完毕，接收端合并并校验全文件。参数经请求头传递，请求体为空。

**请求**

| 项 | 值 |
| :--- | :--- |
| 方法与路径 | `POST /api/v1/files/finish` |
| 需绑定 | 是 |
| 路径参数 | 无 |
| Query 参数 | 无 |
| 请求体 | 空 |

**请求头**

| Header | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `X-Source-Device-Id` | string | 是 | 发送方设备 ID |
| `X-Transfer-Id` | string | 是 | 传输任务 ID |
| `X-File-Name` | string | 是 | 文件名，百分号编码 |
| `X-File-Size` | number | 是 | 文件总字节数，空文件为 `0` |
| `X-Chunk-Size` | number | 是 | 分片大小 |
| `X-Chunk-Count` | number | 是 | 分片总数，空文件为 `0` |
| `X-File-Sha256` | string | 是 | 全文件 SHA-256，64 位小写 Hex |

**响应**

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `data.transferId` | string | 是 | 回显传输任务 ID |
| `data.fileName` | string | 是 | 实际落盘文件名，重名时带 `(1)`、`(2)` 等后缀 |
| `data.size` | number | 是 | 落盘文件字节数 |

```json
{
  "service": "suichuan",
  "success": true,
  "data": {
    "transferId": "transfer-1721000000000-a1b2c3",
    "fileName": "demo (1).zip",
    "size": 10485760
  }
}
```

接收端已写入完成标记且 SHA-256 一致时直接返回成功，重复调用幂等。

**错误码**

| 错误码 | HTTP | 触发条件 | `details` |
| :--- | :--- | :--- | :--- |
| `PROTO_BAD_REQUEST` | 400 | 必填头缺失，或 `X-File-Sha256` 格式错误 | 无 |
| `TRANSFER_RANGE_INVALID` | 416 | 分片不齐全，或参数不自洽 | `{ "missingChunks": [3, 7] }` |
| `TRANSFER_CHECKSUM_MISMATCH` | 422 | 合并后全文件 SHA-256 不匹配 | 无 |
| `IO_WRITE_FAILED` | 500 | 合并或重命名失败 | 无 |

### 5.4 POST /api/v1/files/cancel

取消一个进行中的传输任务。接收端置取消标记，进行中的分片写入随即中止。

**请求**

| 项 | 值 |
| :--- | :--- |
| 方法与路径 | `POST /api/v1/files/cancel` |
| 需绑定 | 是 |
| Content-Type | `application/json` |
| 路径参数 | 无 |

`transferId` 可经 Query 参数或请求体传递，服务端优先读取 Query。

**Query 参数**

| 参数 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `transferId` | string | 与请求体二者其一 | 待取消的传输任务 ID |

**请求体**

| 字段 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `transferId` | string | 与 Query 二者其一 | 待取消的传输任务 ID |
| `reason` | string | 否 | 取消原因，写入日志，如 `user_cancelled` |

```json
{ "transferId": "transfer-1721000000000-a1b2c3", "reason": "user_cancelled" }
```

**响应**

```json
{ "service": "suichuan", "success": true }
```

未找到对应任务时同样返回成功。

### 5.5 POST /api/v1/files/upload

单请求整文件上传。分片通道为当前默认传输方式，本接口用于兼容旧版本对端与小文件场景。

**请求**

| 项 | 值 |
| :--- | :--- |
| 方法与路径 | `POST /api/v1/files/upload` |
| 需绑定 | 是 |
| Content-Type | `application/octet-stream` |
| 加密 | 受策略约束 |
| 路径参数 | 无 |
| Query 参数 | 无 |

**请求头**

| Header | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `X-Source-Device-Id` | string | 是 | 发送方设备 ID |
| `X-File-Name` | string | 是 | 文件名，百分号编码 |
| `X-File-Size` | number | 是 | 文件总字节数 |
| `X-Transfer-Id` | string | 否 | 传输任务 ID，缺省时接收端自行生成 |
| `X-E2E-Encrypted` | string | 否 | 加密时为 `"true"` |
| `X-E2E-IV` | string | 加密时必填 | Base64 编码的 12 字节 IV |
| `X-Resume-Offset` | number | 否 | 续传起始偏移，接收端自该偏移追加写入 |

**请求体**：文件字节流。加密时为 `ciphertext ‖ tag(16 字节)`。

**响应**

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `data.transferId` | string | 是 | 传输任务 ID |
| `data.fileName` | string | 是 | 实际落盘文件名 |
| `data.size` | number | 是 | 落盘字节数 |

**错误码**：与 [5.2](#52-post-apiv1fileschunk) 一致。

### 5.6 传输场景区分

分片通道为通用二进制传输，三类业务共用。接收端依据 `transferId` 前缀判定场景与界面归属，前缀常量定义于 [`packages/core/src/utils/id.ts`](../packages/core/src/utils/id.ts)。

| 场景 | `transferId` 前缀 | 关联方式 | 界面归属 |
| :--- | :--- | :--- | :--- |
| 设备直传文件 | `transfer-` | 发送端直接发起分片传输 | 传输中心与文件列表，独立进度条 |
| 聊天消息内文件 | `chat-transfer-` | 同时发起 `/api/v1/messages`，`type` 为 `file` 或 `image`，携带同一 `transferId` | 聊天气泡内的文件卡片，不计入传输历史 |
| 剪贴板文件与图片 | 由剪贴板流程携带 | 同时发起 `/api/v1/clipboard`，`type` 为 `image`，携带同一 `transferId` | 剪贴板同步列表 |

收发两端均以 `isChatTransferId()` 判定聊天文件并将其排除在传输历史之外。

---

## 6. 网页提取码分享接口

面向未安装随传的浏览器，不依赖设备绑定，以提取码与可选的所有者审批作为访问控制。该组接口不在 `/api/v1` 路径下。提取码大小写不敏感，服务端统一归一化。

### 6.1 GET /s

返回提取码输入网页。

**请求**

| 项 | 值 |
| :--- | :--- |
| 方法与路径 | `GET /s`，或 `GET /s/{code}` |
| 需绑定 | 否 |

| 参数 | 位置 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- | :--- |
| `code` | 路径 | string | 否 | 提取码。以路径形式提供时页面自动填入 |
| `code` | Query | string | 否 | 提取码，作用同上 |

**响应**：HTTP 200，`Content-Type: text/html; charset=utf-8`，返回分享页面 HTML。

### 6.2 GET /s/info

查询提取码对应文件的元信息，不触发审批流程。

**Query 参数**

| 参数 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `code` | string | 是 | 提取码 |

**响应**

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `data.fileName` | string | 是 | 文件名 |
| `data.fileSize` | number | 是 | 文件字节数 |
| `data.requireApproval` | boolean | 是 | 下载是否需要所有者审批 |

```json
{
  "service": "suichuan",
  "success": true,
  "data": { "fileName": "document.pdf", "fileSize": 2048000, "requireApproval": true }
}
```

**错误码**

| 错误码 | HTTP | 触发条件 |
| :--- | :--- | :--- |
| `SHARE_NOT_FOUND` | 404 | 提取码不存在，或对应文件已从磁盘移除 |
| `SHARE_INACTIVE` | 403 | 分享已被所有者停用 |
| `SHARE_EXPIRED` | 410 | 分享已过期，或一次性分享已被使用 |

### 6.3 GET /s/submit

提交提取码。需审批时返回 `requestId`，免审批时返回一次性下载凭据。

**Query 参数**

| 参数 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `code` | string | 是 | 提取码 |

**响应**

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `data.status` | string | 是 | `waiting-approval` 或 `download` |
| `data.requestId` | string | 条件 | `status` 为 `waiting-approval` 时返回，用于轮询审批结果 |
| `data.downloadToken` | string | 条件 | `status` 为 `download` 时返回，一次性下载凭据 |
| `data.fileName` | string | 是 | 文件名 |
| `data.fileSize` | number | 是 | 文件字节数 |

需审批：

```json
{
  "service": "suichuan",
  "success": true,
  "data": {
    "status": "waiting-approval",
    "requestId": "req-share-0001",
    "fileName": "document.pdf",
    "fileSize": 2048000
  }
}
```

免审批：

```json
{
  "service": "suichuan",
  "success": true,
  "data": {
    "status": "download",
    "downloadToken": "TOKEN_PLACEHOLDER",
    "fileName": "document.pdf",
    "fileSize": 2048000
  }
}
```

**错误码**：同 [6.2](#62-get-sinfo)。

### 6.4 GET /s/approval-check

轮询审批结果。分享页按 1–2 秒间隔轮询。

**Query 参数**

| 参数 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- |
| `requestId` | string | 是 | [6.3](#63-get-ssubmit) 返回的审批请求 ID |

**响应**

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `data.status` | string | 是 | `pending` 或 `approved` |
| `data.downloadToken` | string | 条件 | `status` 为 `approved` 时返回 |
| `data.fileName` | string | 条件 | `status` 为 `approved` 时返回 |
| `data.fileSize` | number | 条件 | `status` 为 `approved` 时返回 |

审核中：

```json
{ "service": "suichuan", "success": true, "data": { "status": "pending" } }
```

已通过：

```json
{
  "service": "suichuan",
  "success": true,
  "data": {
    "status": "approved",
    "downloadToken": "TOKEN_PLACEHOLDER",
    "fileName": "document.pdf",
    "fileSize": 2048000
  }
}
```

**错误码**

| 错误码 | HTTP | 触发条件 | `details` |
| :--- | :--- | :--- | :--- |
| `PROTO_BAD_REQUEST` | 400 | `requestId` 缺失或不存在 | `{ "field": "requestId" }` |
| `SHARE_APPROVAL_REJECTED` | 403 | 所有者拒绝了本次下载 | 无 |
| `SHARE_APPROVAL_TIMEOUT` | 408 | 审批超时，请求已失效 | 无 |

### 6.5 GET /s/dl/{token}

下载分享文件。

**请求**

| 参数 | 位置 | 类型 | 必填 | 说明 |
| :--- | :--- | :--- | :--- | :--- |
| `token` | 路径 | string | 是 | 下载凭据，由 [6.3](#63-get-ssubmit) 或 [6.4](#64-get-sapproval-check) 返回 |

**响应**

成功时为二进制文件流：

| 响应头 | 值 |
| :--- | :--- |
| `Content-Type` | `application/octet-stream` |
| `Content-Length` | 文件字节数 |
| `Content-Disposition` | `attachment; filename="..."; filename*=UTF-8''...` |

凭据为一次性，使用后即标记为已用，且设有过期时间。分享配置为一次性时，下载完成后自动停用。

**错误码**

| 错误码 | HTTP | 触发条件 |
| :--- | :--- | :--- |
| `SHARE_NOT_FOUND` | 404 | 凭据不存在，或对应文件已从磁盘移除 |
| `SHARE_EXPIRED` | 410 | 凭据已过期或已被使用 |
| `SHARE_INACTIVE` | 403 | 分享已停用 |

### 6.6 GET /s/icon

返回分享页面使用的图标。

**响应**：图片二进制，`Content-Type: image/png`。app 端不提供该资源，返回 `ROUTE_NOT_FOUND`（404）。

---

## 7. 服务状态接口

`GET /` 返回纯文本运行状态。

| 项 | 值 |
| :--- | :--- |
| 方法与路径 | `GET /` |
| 需绑定 | 否 |
| HTTP 状态码 | 200 |
| Content-Type | `text/plain; charset=utf-8` |
| Cache-Control | `no-store` |

```
✅ Suichuan server is running at http://192.168.1.10:38888
Version: 1.0.16
```

---

## 8. 错误码

错误码定义于 [`packages/protocol/src/errors.ts`](../packages/protocol/src/errors.ts)，含 `ErrorCode` 枚举与 `ERROR_HTTP_STATUS` 映射表。响应的 HTTP 状态码由该映射表得出。

### 8.1 完整错误码表

共 46 个。「中文文案」列取自 `error-messages.ts`，为界面展示文案；标注「无」表示未配置文案。

| 错误码 | HTTP | 中文文案 |
| :--- | :--- | :--- |
| `AUTH_MISSING_HEADERS` | 400 | 无 |
| `AUTH_UNKNOWN_DEVICE` | 401 | 对方不认识本设备，请重新配对 |
| `AUTH_INVALID_SIGNATURE` | 401 | 无 |
| `AUTH_INVALID_BODY_HASH` | 400 | 无 |
| `AUTH_CLOCK_SKEW` | 401 | 无 |
| `AUTH_REPLAYED_NONCE` | 401 | 无 |
| `AUTH_NOT_BOUND` | 403 | 对方未绑定本设备，请重新配对 |
| `AUTH_REVOKED` | 403 | 对方已解除与本设备的绑定 |
| `AUTH_IP_MISMATCH` | 403 | 身份验证失败：该地址上的设备不是要找的设备 |
| `PAIR_INVALID_CODE` | 401 | 无 |
| `PAIR_CODE_EXPIRED` | 410 | 无 |
| `PAIR_REJECTED` | 403 | 无 |
| `PAIR_BLACKLISTED` | 403 | 无 |
| `PAIR_LOCKED_OUT` | 429 | 无 |
| `PAIR_ALREADY_BOUND` | 409 | 无 |
| `PAIR_SESSION_NOT_FOUND` | 404 | 无 |
| `PROTO_VERSION_UNSUPPORTED` | 426 | 对方版本不兼容，请升级到最新版本 |
| `PROTO_CAPABILITY_MISSING` | 400 | 无 |
| `PROTO_BAD_REQUEST` | 400 | 请求格式错误 |
| `PROTO_UNSUPPORTED_MEDIA` | 415 | 无 |
| `TRANSFER_NOT_FOUND` | 404 | 无 |
| `TRANSFER_ALREADY_COMPLETE` | 409 | 无 |
| `TRANSFER_CANCELLED` | 410 | 传输已取消 |
| `TRANSFER_SIZE_EXCEEDED` | 413 | 文件超出对方允许的大小限制 |
| `TRANSFER_CHECKSUM_MISMATCH` | 422 | 文件校验失败，数据可能在传输中损坏 |
| `TRANSFER_RANGE_INVALID` | 416 | 无 |
| `TRANSFER_DECLINED` | 403 | 对方拒绝了本次传输 |
| `MESSAGE_TOO_LARGE` | 413 | 消息内容过大 |
| `MESSAGE_DUPLICATE` | 409 | 无 |
| `CLIPBOARD_DISABLED` | 403 | 无 |
| `E2E_SESSION_MISSING` | 400 | 端对端加密会话缺失，请重新配对 |
| `E2E_CIPHER_INIT_FAILED` | 400 | 端对端加密初始化失败 |
| `E2E_TAG_MISMATCH` | 422 | 端对端加密密钥不匹配 |
| `E2E_DECRYPTION_FAILED` | 400 | 端对端加密密钥不匹配 |
| `E2E_PLAINTEXT_REJECTED` | 403 | 对方仅接收加密数据，请开启端对端加密后重试 |
| `ROUTE_NOT_FOUND` | 404 | 对方不支持该操作，请升级到最新版本 |
| `SHARE_NOT_FOUND` | 404 | 无 |
| `SHARE_EXPIRED` | 410 | 无 |
| `SHARE_INACTIVE` | 403 | 无 |
| `SHARE_APPROVAL_PENDING` | 202 | 无 |
| `SHARE_APPROVAL_REJECTED` | 403 | 无 |
| `SHARE_APPROVAL_TIMEOUT` | 408 | 无 |
| `RATE_LIMITED` | 429 | 请求过于频繁，请稍后再试 |
| `IO_DISK_FULL` | 507 | 对方磁盘空间不足 |
| `IO_WRITE_FAILED` | 500 | 对方写入文件失败 |
| `INTERNAL_ERROR` | 500 | 对方处理时发生错误 |

默认可重试的错误码为 `RATE_LIMITED`、`SHARE_APPROVAL_PENDING`、`INTERNAL_ERROR`、`IO_WRITE_FAILED`。响应中的 `retryable` 字段覆盖该默认值。

### 8.2 协议外错误码

所有端均严格产出 `ErrorCode` 枚举内定义的标准错误码，无非标准遗留错误码。

---

## 9. UDP 设备发现协议

| 项 | 值 |
| :--- | :--- |
| 监听端口 | `38890` |
| 传输方式 | UDP，广播发送，单播应答 |

**探测包**：向各网卡的广播地址发送，两种形式等效。

```
SUICHUAN_DISCOVER            裸文本
{"type":"discover"}          JSON
```

**应答包**：单播回发至来源 `addr:port`。

| 字段 | 类型 | 必有 | 说明 |
| :--- | :--- | :--- | :--- |
| `type` | string | 是 | 固定 `"SUICHUAN_RESPONSE"` |
| `device` | object | 是 | [DeviceInfo](#deviceinfo) 对象 |

```json
{
  "type": "SUICHUAN_RESPONSE",
  "device": {
    "id": "device-00000000-0000-0000-0000-000000000000",
    "name": "MacBook Pro",
    "type": "laptop_mac",
    "platform": "macOS",
    "ip": "192.168.1.10",
    "port": 38888,
    "version": "1.0.17"
  }
}
```

扫描方以实际收到应答的源地址为设备地址，`device.ip` 不参与该判定。隐身模式下不响应探测包。完整发现流程见 [device-discovery.md](./device-discovery.md)。

---

## 10. 跨端差异

三端实现遵循同一协议，以下为已知的表现差异。

| 项 | pc (Electron) | pc-lite (Tauri) | app (uni-app) |
| :--- | :--- | :--- | :--- |
| `/api/v1/ping` | 未提供 | 未提供 | 提供 |
| `/s/icon` | 提供 | 提供 | 返回 404 |
| 设备 `type` 取值 | 桌面类取值 | 桌面类取值 | `smartphone`、`tablet` |
