# Socket.IO 介绍

## 1. 简介

**Socket.IO** 是一个可以在 **客户端** 与 **服务器** 之间建立 **低延迟**、**双向**、**基于事件** 的实时通信通道的库。

它并不是单纯的 **WebSocket** 实现，而是建立在 **Engine.IO** 之上的一套高级通信协议，能够根据网络环境、浏览器支持情况，在不同传输方式之间切换，并提供更多高级功能。

核心特性：

* **多传输方式**：WebSocket、HTTP 长轮询、WebTransport（部分浏览器）
* **事件驱动**：使用 `emit` 和 `on` 进行事件发送与监听
* **自动重连**：连接意外中断后自动尝试重连
* **消息确认与回调**：保证消息可靠送达
* **广播与房间机制**：可实现群聊、频道功能
* **命名空间**：在同一连接内实现多路复用
* **数据缓冲**：断线期间发送的事件会在重连后自动发送

---

## 2. Socket.IO 与 WebSocket 的区别

* **相同点**

  * 都可以实现客户端与服务器之间的全双工通信
  * 都支持低延迟传输

* **不同点**

  | 对比项   | Socket.IO                           | WebSocket    |
  | ----- | ----------------------------------- | ------------ |
  | 协议    | 自定义协议（基于 Engine.IO）                 | 标准 RFC 6455  |
  | 传输方式  | WebSocket / HTTP 长轮询 / WebTransport | 仅 WebSocket  |
  | 自动重连  | ✅ 内置                                | ❌ 需要手动实现     |
  | 消息确认  | ✅ 内置回调与超时                           | ❌ 无此功能       |
  | 广播/房间 | ✅ 内置                                | ❌ 需自定义实现     |
  | 兼容性   | 高（可回退）                              | 依赖网络环境和浏览器支持 |

---

## 3. 安装

### 3.1 服务器端（Node.js）

```bash
npm install socket.io
```

**server.js**  

```js
const { Server } = require("socket.io");
const io = new Server(3000, {
  cors: { origin: "*" }
});

io.on("connection", (socket) => {
  console.log(`用户已连接: ${socket.id}`);

  socket.on("disconnect", () => {
    console.log(`用户断开连接: ${socket.id}`);
  });
});
```

---

### 3.2 客户端（浏览器）

```bash
npm install socket.io-client
```

**index.html**  

```html
<script src="https://cdn.socket.io/4.7.5/socket.io.min.js"></script>
<script>
  const socket = io("http://localhost:3000");

  socket.on("connect", () => {
    console.log("已连接到服务器", socket.id);
  });

  socket.on("disconnect", () => {
    console.log("与服务器断开连接");
  });
</script>
```

---

## 4. 事件机制

Socket.IO 基于事件驱动模型，主要方法：

* `socket.emit(event, data)`：发送事件
* `socket.on(event, callback)`：监听事件
* 回调与超时机制
* 广播
* 命名空间与房间

---

### 4.1 发送与监听

**服务器端：**

```js
io.on("connection", (socket) => {
  socket.on("chat message", (msg) => {
    console.log("收到消息:", msg);
    socket.emit("chat reply", `服务器已收到: ${msg}`);
  });
});
```

**客户端：**

```js
socket.emit("chat message", "你好，服务器");

socket.on("chat reply", (data) => {
  console.log(data);
});
```

---

### 4.2 回调与超时

```js
// 客户端
socket.emit("hello", "world", (response) => {
  console.log("服务器返回:", response);
});

// 服务器端
socket.on("hello", (msg, callback) => {
  callback("收到你的问候！");
});
```

带超时：

```js
socket.timeout(5000).emit("hello", "world", (err, response) => {
  if (err) console.log("服务器未在 5 秒内响应");
  else console.log(response);
});
```

---

### 4.3 广播

```js
// 向所有连接的客户端
io.emit("system message", "服务器广播通知");

// 向除自己以外的客户端
socket.broadcast.emit("user joined", `${socket.id} 加入了聊天室`);

// 向房间内客户端
io.to("room1").emit("room message", "来自 room1 的消息");
```

---

### 4.4 命名空间

```js
const adminNamespace = io.of("/admin");
adminNamespace.on("connection", (socket) => {
  console.log("管理员连接:", socket.id);
});
```

客户端：

```js
const adminSocket = io("http://localhost:3000/admin");
```

---

### 4.5 房间机制

```js
io.on("connection", (socket) => {
  socket.on("join room", (room) => {
    socket.join(room);
    socket.to(room).emit("room notice", `${socket.id} 进入了房间`);
  });

  socket.on("leave room", (room) => {
    socket.leave(room);
  });
});
```

---

## 5. 聊天室示例

**服务器端：**

```js
io.on("connection", (socket) => {
  socket.on("join room", (room) => {
    socket.join(room);
    socket.to(room).emit("message", `${socket.id} 加入了 ${room}`);
  });

  socket.on("chat", ({ room, message }) => {
    io.to(room).emit("message", `${socket.id}: ${message}`);
  });
});
```

**客户端：**

```js
socket.emit("join room", "room1");

socket.on("message", (msg) => {
  console.log(msg);
});
```

---

## 6. 工作原理（Engine.IO + Socket.IO 协议）

Socket.IO 底层依赖 **Engine.IO** 实现连接管理与数据传输。

### 6.1 两层架构

1. **Engine.IO**（底层）

   * 负责低层传输（WebSocket / HTTP 长轮询）
   * 连接升级
   * 心跳检测
   * 断线与重连
2. **Socket.IO**（高层）

   * 提供事件 API
   * 房间与命名空间
   * 数据包缓冲
   * 广播机制

---

### 6.2 连接与升级流程

1. 客户端先使用 **HTTP 长轮询** 发起连接（提高兼容性）
2. 如果环境允许，自动升级到 **WebSocket**
3. 升级步骤：

   * 确保当前传输缓冲区为空
   * 尝试建立新的传输（WebSocket）
   * 成功后关闭旧的长轮询连接

---

### 6.3 握手数据包示例

```json
{
  "sid": "FSDjX-WRwSA4zTZMALqx",
  "upgrades": ["websocket"],
  "pingInterval": 25000,
  "pingTimeout": 20000
}
```

字段说明：

* `sid`：会话 ID
* `upgrades`：可升级的传输方式
* `pingInterval`：心跳间隔
* `pingTimeout`：心跳超时

---

### 6.4 心跳与断线检测

* **服务器 → 客户端**：定期发送 `PING`
* **客户端 → 服务器**：收到 `PING` 后立即回复 `PONG`
* 如果 `pingTimeout` 内没有收到响应，则认为连接断开
* 客户端断开后会 **指数回退重连**

---

### 6.5 数据包格式

Socket.IO 发送的消息会有额外协议前缀：

```text
42["eventName","data"]
```

* `4` → Engine.IO 消息类型
* `2` → Socket.IO 消息类型
* `["eventName","data"]` → JSON 序列化的事件名和数据

---

## 7. 多节点扩展

在多台服务器部署时，需要使用 **Adapter** 让不同节点共享房间信息。

官方提供的 Redis 适配器：

```bash
npm install @socket.io/redis-adapter ioredis
```

```js
const { createAdapter } = require("@socket.io/redis-adapter");
const { createClient } = require("ioredis");

const pubClient = createClient({ host: "localhost", port: 6379 });
const subClient = pubClient.duplicate();

io.adapter(createAdapter(pubClient, subClient));
```

---

## 8. 注意事项与最佳实践

* **移动端后台运行**：不要在后台维持长连接，使用 FCM 等消息推送服务。
* **大文件传输**：应使用 HTTP 上传，Socket.IO 适合小消息。
* **防止滥用**：使用 `socket.use` 做权限校验。
* **连接数限制**：在高并发场景下需结合负载均衡与 Redis 适配器。
