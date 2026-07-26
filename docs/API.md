# 随传 (SuiChuan) API 接口文档

## 1. 通用说明

- **基础路径**：`/api/v1`
- **默认端口**：HTTP `38888` / UDP `38888`
- **数据格式**：`application/json; charset=utf-8`

---

## 2. 响应信封格式

### 2.1 成功响应

```json
{
  "service": "suichuan",
  "code": 0,
  "msg": "success",
  "success": true,
  "data": {}
}
```

| 字段 | 类型 | 说明 |
| :--- | :--- | :--- |
| `service` | string | 固定值 `"suichuan"` |
| `code` | number | 状态码，成功为 `0` |
| `msg` | string | 状态描述，成功为 `"success"` |
| `success` | boolean | 请求是否成功标识（成功时为 `true`） |
| `data` | object / array | 响应数据体 |

---

### 2.2 失败响应

```json
{
  "service": "suichuan",
  "code": "AUTH_NOT_BOUND",
  "msg": "Device not paired",
  "success": false,
  "details": {}
}
```

| 字段 | 类型 | 说明 |
| :--- | :--- | :--- |
| `service` | string | 固定值 `"suichuan"` |
| `code` | string | 错误代码 |
| `msg` | string | 错误描述 |
| `success` | boolean | 请求是否成功标识（失败时为 `false`） |
| `details` | object | 错误上下文（可选） |

---

## 3. 请求头定义

### 3.1 基础 Request Header

| Header | 说明 |
| :--- | :--- |
| `X-Source-Device-Id` | 发送方设备 ID |
| `X-Request-Id` | 请求追踪 ID |

### 3.2 E2E 加密 Header

| Header | 说明 |
| :--- | :--- |
| `X-E2E-Encrypted` | `true` 表示 Body 已加密 |
| `X-E2E-IV` | Base64 编码的 12 字节 IV |
| `X-E2E-Tag` | Base64 编码的 16 字节 Auth Tag |

### 3.3 文件分片 Header

| Header | 说明 |
| :--- | :--- |
| `X-Transfer-Id` | 传输任务 ID (全场景唯一索引) |
| `X-Transfer-Scene` | 传输场景标识 (`direct` 直传 / `message` 消息文件 / `clipboard` 剪贴板文件) |
| `X-Chunk-Index` | 分片索引（从 `0` 开始） |
| `X-Chunk-Offset` | 当前分片起始字节偏移 |
| `X-Chunk-Count` | 分片总数 |
| `X-Chunk-Size` | 当前分片传输字节数 |
| `X-Plain-Size` | 解密后明文字节数 |
| `X-Chunk-Sha256` | 分片明文 SHA-256 (小写 Hex) |
| `X-File-Sha256` | 全文件 SHA-256 (小写 Hex) |
| `X-File-Name` | 文件名称 |
| `X-File-Size` | 文件总字节数 |

---

## 4. 接口列表

### 4.1 `GET /api/v1/info` — 获取设备信息

- **请求方法**：`GET`
- **请求参数**：无

#### 响应示例
```json
{
  "service": "suichuan",
  "code": 0,
  "msg": "success",
  "success": true,
  "data": {
    "id": "device-00000000-0000-0000-0000-000000000000",
    "name": "zxlee-macbook",
    "type": "computer",
    "platform": "macos",
    "ip": "192.168.0.104",
    "port": 38888,
    "version": "1.0.15"
  }
}
```

---

### 4.2 `GET /api/v1/heartbeat` — 心跳探测

- **请求方法**：`GET`
- **请求参数**：无

#### 响应示例
```json
{
  "service": "suichuan",
  "code": 0,
  "msg": "success",
  "success": true,
  "data": {
    "online": true,
    "timestamp": 1721000000000,
    "deviceId": "device-00000000-0000-0000-0000-000000000000"
  }
}
```

---

### 4.3 `POST /api/v1/pair` — 设备配对

- **请求方法**：`POST`

#### 请求 Body
```json
{
  "device": {
    "id": "device-00000000-0000-0000-0000-000000000000",
    "name": "iPhone",
    "type": "smartphone",
    "platform": "ios",
    "port": 38888
  },
  "publicKey": "BASE64_PUBLIC_KEY..."
}
```

#### 响应示例 (成功)
```json
{
  "service": "suichuan",
  "code": 0,
  "msg": "success",
  "success": true,
  "data": {
    "accepted": true,
    "device": {
      "id": "device-00000000-0000-0000-0000-000000000000",
      "name": "MacBook",
      "type": "computer",
      "platform": "macos",
      "port": 38888
    },
    "publicKey": "BASE64_PUBLIC_KEY..."
  }
}
```

#### 响应示例 (拒绝)
```json
{
  "service": "suichuan",
  "code": "PAIR_REJECTED",
  "msg": "User declined",
  "success": false,
  "details": {
    "reason": "user_declined"
  }
}
```

---

### 4.4 `POST /api/v1/messages` — 发送消息

- **请求方法**：`POST`

#### 请求 Body
```json
{
  "id": "msg-1721000000-xxxx",
  "type": "text",
  "content": "Hello World",
  "timestamp": 1721000000000
}
```

#### 响应示例
```json
{
  "service": "suichuan",
  "code": 0,
  "msg": "success",
  "success": true,
  "data": {
    "received": true,
    "msgId": "msg-1721000000-xxxx"
  }
}
```

---

### 4.5 `POST /api/v1/clipboard` — 推送剪贴板

- **请求方法**：`POST`

#### 请求 Body
```json
{
  "type": "text",
  "content": "剪贴板文本内容",
  "timestamp": 1721000000000
}
```

#### 响应示例
```json
{
  "service": "suichuan",
  "code": 0,
  "msg": "success",
  "success": true,
  "data": {
    "synced": true
  }
}
```

---

### 4.6 `GET /api/v1/files/offset` — 查询文件续传偏移量

- **请求方法**：`GET`
- **Query 参数**：
  - `transferId`: 传输任务 ID
  - `deviceId`: 设备 ID
  - `fileSize`: 文件总字节数（可选）
  - `chunkSize`: 分片大小（可选）
  - `chunkCount`: 分片总数（可选）

#### 响应示例
```json
{
  "service": "suichuan",
  "code": 0,
  "msg": "success",
  "success": true,
  "data": {
    "transferId": "transfer-9988",
    "offset": 5242880,
    "chunkIndex": 5
  }
}
```

---

### 4.7 `POST /api/v1/files/chunk` — 发送文件分片

- **请求方法**：`POST`
- **请求 Body**：二进制分片数据（元数据使用 Header 传输）

#### 响应示例
```json
{
  "service": "suichuan",
  "code": 0,
  "msg": "success",
  "success": true,
  "data": {
    "transferId": "transfer-9988",
    "chunkIndex": 5,
    "size": 1048576,
    "receivedBytes": 6291456
  }
}
```

---

### 4.8 `POST /api/v1/files/finish` — 完成文件传输

- **请求方法**：`POST`

#### 请求 Body
```json
{
  "transferId": "transfer-9988",
  "fileName": "demo.zip",
  "fileSize": 10485760,
  "fileSha256": "a591a6d40bf420404a011733cfb7b190d62c65bf0bcda32b57b277d9ad9f146e"
}
```

#### 响应示例
```json
{
  "service": "suichuan",
  "code": 0,
  "msg": "success",
  "success": true,
  "data": {
    "transferId": "transfer-9988",
    "fileName": "demo.zip",
    "size": 10485760,
    "savedPath": "/downloads/demo.zip"
  }
}
```

---

### 4.9 `POST /api/v1/files/cancel` — 取消文件传输

- **请求方法**：`POST`

#### 请求 Body
```json
{
  "transferId": "transfer-9988",
  "reason": "user_cancelled"
}
```

---

### 4.10 文件传输业务场景区分与关联机制

文件分片传输（`POST /api/v1/files/chunk`）为底层的通用二进制传输通道。接收端通过 Header `X-Transfer-Scene` 或对应接口载荷中的 `transferId` 进行场景关联与界面渲染区分：

| 业务场景 | 场景标识 (`X-Transfer-Scene`) | 关联与区分机制 | 界面渲染行为 |
| :--- | :--- | :--- | :--- |
| **设备直传文件** | `direct` | 发送方直接发起分片传输，`transferId` 在传输中心唯一建立 | 在【传输中心 / 文件列表】展示独立进度条与文件操作项 |
| **聊天消息文件** | `message` | 发送方先/同步发起 `POST /api/v1/messages` (`type: "file"`，包含 `transferId`) | 在【聊天会话窗口】的气泡中绑定消息卡片并展示传输进度 |
| **剪贴板文件同步** | `clipboard` | 发送方发起 `POST /api/v1/clipboard` (`type: "file" \| "image"`，包含 `transferId`) | 在【剪贴板同步列表】中绑定剪贴板卡片，自动下载/更新 |

---

### 4.11 网页提取码分享接口

网页提取码分享提供免配对的网页文件提取与审批流通道。

#### 4.11.1 `GET /s` — 提取码浏览器页面
- **说明**：获取提取码网页 UI 界面 (HTML)
- **Query 参数**：`code` (可选提取码)

---

#### 4.11.2 `GET /s/info` — 预览提取码文件元信息
- **说明**：查询提取码文件元数据（不触发审批流）
- **Query 参数**：`code` (提取码)

##### 响应示例
```json
{
  "service": "suichuan",
  "code": 0,
  "msg": "success",
  "success": true,
  "data": {
    "fileName": "document.pdf",
    "fileSize": 2048000,
    "requireApproval": true
  }
}
```

---

#### 4.11.3 `GET /s/submit` — 提交提取码申请
- **说明**：提交提取码。若需要所有者审批，返回 `waiting-approval` 及 `requestId`；若无需审批，直接返回单次 `downloadToken`。
- **Query 参数**：`code` (提取码)

##### 响应示例 (需审批)
```json
{
  "service": "suichuan",
  "code": 0,
  "msg": "success",
  "success": true,
  "data": {
    "status": "waiting-approval",
    "requestId": "req-share-7788",
    "fileName": "document.pdf",
    "fileSize": 2048000
  }
}
```

##### 响应示例 (直接下载)
```json
{
  "service": "suichuan",
  "code": 0,
  "msg": "success",
  "success": true,
  "data": {
    "status": "download",
    "downloadToken": "token-xyz-123456",
    "fileName": "document.pdf",
    "fileSize": 2048000
  }
}
```

---

#### 4.11.4 `GET /s/approval-check` — 轮询审批状态
- **说明**：轮询提现申请的审批通过状态。
- **Query 参数**：`requestId`

##### 响应示例 (审批通过)
```json
{
  "service": "suichuan",
  "code": 0,
  "msg": "success",
  "success": true,
  "data": {
    "status": "approved",
    "downloadToken": "token-xyz-123456",
    "fileName": "document.pdf",
    "fileSize": 2048000
  }
}
```

##### 响应示例 (审核中)
```json
{
  "service": "suichuan",
  "code": 0,
  "msg": "success",
  "success": true,
  "data": {
    "status": "pending"
  }
}
```

---

#### 4.11.5 `GET /s/dl/:code` — 提取码文件下载
- **说明**：传入 `token` 或提取码下载二进制文件流。
- **Query 参数 / 路径**：`code` 或 `token`

---

## 5. 错误代码定义

| 错误代码 | HTTP 状态码 | 说明 |
| :--- | :--- | :--- |
| `INVALID_SERVICE` | 403 | 非随传协议的无效服务 |
| `AUTH_NOT_BOUND` | 401 | 设备未配对 / 无访问权限 |
| `PAIR_REJECTED` | 403 | 配对请求被拒绝 |
| `PAIR_BLACKLISTED` | 403 | 设备已被加入黑名单 |
| `FILE_NOT_FOUND` | 404 | 文件或传输任务不存在 |
| `TRANSFER_CANCELLED`| 409 | 传输任务已被取消 |
| `PARAM_INVALID` | 400 | 请求参数或 Header 校验错误 |
| `CHECKSUM_MISMATCH` | 422 | 哈希校验不匹配 |
| `INTERNAL_ERROR` | 500 | 内部处理或 IO 错误 |

---

## 6. UDP 局域网广播

- **监听端口**：`38888`
- **广播报文格式**：

```json
{
  "service": "suichuan",
  "cmd": "DISCOVERY_PING",
  "port": 38888,
  "timestamp": 1721000000000
}
```
