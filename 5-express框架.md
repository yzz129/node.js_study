# Express

## 1. 简介

Express 是一个简洁而灵活的 Node.js Web 应用框架，提供了一系列强大特性帮助你创建各种 Web 应用和丰富的 HTTP 工具。使用 Express 可以快速地搭建一个完整功能的网站。

### Express 框架核心特性

- 可以设置中间件来响应 HTTP 请求。
- 定义了路由表用于执行不同的 HTTP 请求动作。
- 可以通过向模板传递参数来动态渲染 HTML 页面。

## 2. 安装 Express

Express 可以通过 npm 安装：

```bash
npm install express 
```

也可以全局安装：

```bash
npm install -g express
```

查看 Express 使用的版本号：

```bash
npm list express
```

## 3. 基本使用

1. 引入 Express 模块  
2. 创建一个 Express 应用  
3. 定义路由  
4. 启动 Express 应用  

**express_demo.js 文件**：

```javascript
const express = require('express')
const app = express()
 
app.get('/', (req, res) => {
   res.send('Hello World')
});
 
const server = app.listen(8081, () => {
  const host = server.address().address;
  const port = server.address().port;
 
  console.log(`应用实例，访问地址为 http://${host}:${port}`)
})
```

运行 `node express_demo.js` 启动应用，在浏览器中访问 `http://localhost:8081/` 即可看到输出的 `Hello World` 内容。

## 4. 请求和响应

Express 应用使用回调函数的参数：`request`（请求）和 `response`（响应）对象来处理请求和响应的数据。

```javascript
app.get('/', function (req, res) {
   // 处理逻辑
})
```

### Request 对象

`request` 对象表示 HTTP 请求，包含了请求查询字符串、参数、内容、HTTP 头部等属性。常见属性有：

| 属性/方法                  | 说明                                                       |
| -------------------------- | ---------------------------------------------------------- |
| `req.app`                  | 当 callback 为外部文件时，用 `req.app` 访问 Express 的实例 |
| `req.baseUrl`              | 获取路由当前安装的 URL 路径                                |
| `req.body` / `req.cookies` | 获得请求主体 / Cookies                                     |
| `req.fresh` / `req.stale`  | 判断请求是否还「新鲜」（未过期）                           |
| `req.hostname` / `req.ip`  | 获取主机名和 IP 地址                                       |
| `req.originalUrl`          | 获取原始请求 URL                                           |
| `req.params`               | 获取路由的动态参数                                         |
| `req.path`                 | 获取请求路径                                               |
| `req.protocol`             | 获取协议类型（如 `http` 或 `https`）                       |
| `req.query`                | 获取 URL 的查询参数串（如 `?id=1&name=test`）              |
| `req.route`                | 获取当前匹配的路由                                         |
| `req.subdomains`           | 获取子域名                                                 |
| `req.accepts()`            | 检查可接受的请求文档类型                                   |
| `req.get()`                | 获取指定的 HTTP 请求头                                     |
| `req.is()`                 | 判断请求头 `Content-Type` 的 MIME 类型                     |

### Response 对象

`response` 对象表示 HTTP 响应，即在接收到请求时向客户端发送的 HTTP 响应数据。常见属性有：

| 属性/方法                            | 说明                                                        |
| ------------------------------------ | ----------------------------------------------------------- |
| `res.app`                            | 同 `req.app`                                                |
| `res.append()`                       | 追加指定 HTTP 头                                            |
| `res.cookie(name, value [, option])` | 设置 Cookie，`option` 可指定域名、过期时间等                |
| `res.clearCookie()`                  | 清除 Cookie                                                 |
| `res.download()`                     | 传送指定路径的文件                                          |
| `res.get()`                          | 返回指定的 HTTP 头                                          |
| `res.json()`                         | 传送 JSON 响应                                              |
| `res.jsonp()`                        | 传送 JSONP 响应（支持跨域）                                 |
| `res.location()`                     | 设置响应的 `Location` HTTP 头（不设置状态码）               |
| `res.redirect()`                     | 设置响应的 `Location` HTTP 头并设置状态码 302（重定向）     |
| `res.render()`                       | 渲染模板页面                                                |
| `res.send()`                         | 传送 HTTP 响应（自动根据内容类型设置 `Content-Type`）       |
| `res.sendFile()`                     | 传送指定路径的文件（自动根据文件扩展名设置 `Content-Type`） |
| `res.set()`                          | 设置 HTTP 头，传入对象可一次设置多个头                      |
| `res.status()`                       | 设置 HTTP 状态码                                            |
| `res.type()`                         | 设置 `Content-Type` 的 MIME 类型                            |

## 5. 路由

路由决定了由谁（指定脚本）去响应客户端请求。在 HTTP 请求中，我们可以通过路由提取出请求的 URL 以及 GET/POST 参数。

```javascript
let express = require('express')
let app = express()
 
// 主页输出 "Hello World"
app.get('/', (req, res) => {
   console.log("主页 GET 请求")
   res.send('Hello World')
})
 
// POST 请求
app.post('/', (req, res) => {
   console.log("主页 POST 请求")
   res.send('POST 请求')
})
 
// /del_user 页面响应
app.get('/del_user', (req, res) => {
   console.log("/del_user 响应 DELETE 请求")
   res.send('删除页面')
})
 
// /list_user 页面 GET 请求
app.get('/list_user', (req, res) => {
   console.log("/list_user GET 请求")
   res.send('用户列表页面')
})
 
// 对页面 abcd, abxcd, ab123cd 等响应 GET 请求（正则匹配）
app.get('/ab*cd', (req, res) => {   
   console.log("/ab*cd GET 请求")
   res.send('正则匹配')
})
 
var server = app.listen(8081, () => {
  console.log("启动成功")
})
```

**执行代码**：

```bash
$ node express_demo2.js 
启动成功
```

- 访问 `http://127.0.0.1:8081/list_user`，会看到 `用户列表页面`。  
- 访问 `http://127.0.0.1:8081/abcd`，会看到 `正则匹配`。  
- 访问未定义的路由（如 `http://127.0.0.1:8081/abcdefg`），会看到 `Cannot GET /abcdefg`。

## 6. 中间件

中间件是一种函数，它可以访问请求对象（`req`）、响应对象（`res`）和应用程序的中间件堆栈中的下一个中间件函数（`next`）。

### 中间件的常见任务

1. 执行任何代码。  
2. 对请求和响应对象进行更改。  
3. 结束请求-响应循环。  
4. 调用堆栈中的下一个中间件函数。  

### 中间件分类

中间件分为**应用级中间件**、**路由级中间件**、**错误处理中间件**、**内置中间件**和**第三方中间件**等。

#### 应用级中间件

应用级中间件函数在应用启动时执行，并且在每个请求之前执行。

```javascript
let express = require('express')
let app = express()
 
// 应用级中间件
app.use((req, res, next) => {
   console.log('应用级中间件')
   next() // 调用下一个中间件
})
 
// 路由
app.get('/', (req, res) => {
   console.log('主页 GET 请求')
   res.send('Hello World')
});
 
app.listen(8081, () => {
   console.log('启动成功')
})
```

**执行结果**：  
访问 `http://127.0.0.1:8081` 时，控制台输出：

```node
应用级中间件
主页 GET 请求
```

#### 路由级中间件

绑定到 `express.Router()` 实例，使用 `router.use()` 或 `router.METHOD()` 绑定，用于特定路由的预处理。

```javascript
let express = require('express')
 
let app = express()
let router = express.Router()
 
// 路由级中间件
router.use((req, res, next) => {
  console.log('路由级中间件')
  next()
})
 
// 路由处理函数
router.get('/', (req, res) => {
  console.log('路由处理函数')
  res.send('用户列表')
})
 
// 挂载路由
app.use('/list_user', router)
 
app.listen(8081, () => {
  console.log('启动成功')
})
```

**执行结果**：  
访问 `http://127.0.0.1:8081/list_user` 时，控制台输出：

```node
路由级中间件
路由处理函数
```

#### 错误处理中间件

专门用于捕获和处理错误，必须接收四个参数（`err`、`req`、`res`、`next`），且需定义在所有路由之后。

```javascript
const express = require('express');
const app = express();
 
// 错误处理中间件
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
 
// 触发错误的路由
app.get('/', (req, res) => {
  throw new Error('Something went wrong!');
});
 
app.listen(8081, () => {
  console.log('Server is running on port 8081');
});
```

**执行结果**：  
访问 `http://127.0.0.1:8081` 时，页面显示 `Something broke!`，控制台输出错误堆栈。

#### 内置中间件

由 Express 提供，用于处理常见任务，如解析请求体、提供静态文件等。

```javascript
// 解析 POST 参数（键值对格式，如 username=yzz&password=123456）
app.use(express.urlencoded({ extended: false }))

// 解析 POST 参数（JSON 格式，如 {"username":"yzz","password":"123456"}）
app.use(express.json())

// 配置静态资源目录（public 和 static 目录下的文件可直接访问）
app.use(express.static("public"))
app.use(express.static("static"))
```

静态资源访问示例：  
若 `public` 目录下有 `image.jpg`，可通过 `http://127.0.0.1:8081/image.jpg` 直接访问。

#### 第三方中间件

由社区提供的实用中间件，需通过 npm 安装，如：

- `morgan`：日志记录中间件  
- `cors`：跨域资源共享中间件  
- `helmet`：安全相关的 HTTP 头设置中间件  

安装示例：

```bash
npm install morgan cors helmet
```

使用示例：

```javascript
const express = require('express');
const morgan = require('morgan');
const cors = require('cors');
const helmet = require('helmet');

const app = express();

// 使用第三方中间件
app.use(morgan('combined')); // 日志记录
app.use(cors()); // 允许跨域
app.use(helmet()); // 增强安全性
```
