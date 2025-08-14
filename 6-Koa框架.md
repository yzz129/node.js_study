# Koa 框架

## 介绍

Koa 是一个新的 web 框架，由 Express 原班人马打造，致力于成为一个更小、更富有表现力、更健壮的 Web 框架。

Koa 解决了 Express 存在的一些问题，例如：

- 中间件嵌套回调（callback hell）
- 错误处理不统一
- 上下文信息分散

Koa 通过使用 `async/await` 和一个统一的 `Context` 上下文对象，使得代码更简洁、可读性更高。

## 核心特点

| 特性         | 说明                                                                 |
|--------------|----------------------------------------------------------------------|
| 轻量无内置   | 不内置任何中间件（如路由、模板等），开发者按需选择                   |
| 全异步       | 支持 `async/await`，默认异步中间件                                   |
| 中间件机制   | 洋葱模型（onion model），控制流程清晰                               |
| 易扩展       | 社区插件丰富，灵活搭配功能                                           |
| Context 对象 | 封装了原生 `req/res`，更方便操作请求和响应                           |

## Koa 对比 Express

| 比较点       | Koa                              | Express                          |
|--------------|----------------------------------|----------------------------------|
| 中间件模型   | 洋葱模型                         | 线性模型（`use` 依次执行）       |
| 异步支持     | 原生 `async/await`               | 需要手动处理异步回调             |
| 内置中间件   | 无                               | 内置很多（如 `body-parser`）     |
| 上下文封装   | 有（`ctx` 封装 `req/res`）       | 无（直接使用 `req, res`）        |
| 灵活性       | 高（插件式）                     | 较低（结构固定）                 |

## 请求和响应

### 请求别名

以下访问器和别名与请求等效项：

```text
 请求别名
 以下访问器和别名请求等效项：
 ctx.header
 ctx.headers
 ctx.method
 ctx.method=
 ctx.url
 ctx.url=
 ctx.originalUrl
 ctx.origin
 ctx.href
 ctx.path
 ctx.path=
 ctx.query
 ctx.query=
 ctx.querystring
 ctx.querystring=
 ctx.host
 ctx.hostname
 ctx.fresh
 ctx.stale
 ctx.socket
 ctx.protocol
 ctx.secure
 ctx.ip
 ctx.ips
 ctx.subdomains
 ctx.is()
 ctx.accepts()
 ctx.acceptsEncodings()
 ctx.acceptsCharsets()
 ctx.acceptsLanguages()
 ctx.get()
```

### 响应别名

```text
响应别名
以下访问器和别名响应等效项：
ctx.body
ctx.body=
ctx.has()
ctx.status
ctx.status=
ctx.message
ctx.message=
ctx.length=
ctx.length
ctx.type=
ctx.type
ctx.vary()
ctx.headerSent
ctx.redirect()
ctx.attachment()
ctx.set()
ctx.append()
ctx.remove()
ctx.lastModified=
ctx.etag=
ctx.writable
```

## 路由

### 一、基本使用流程

#### 1. 安装依赖

```bash
npm install koa koa-router
```

#### 2. 简单示例

```javascript
const Koa = require('koa');
const Router = require('koa-router');
 
const app = new Koa();
const router = new Router(); // 创建路由实例
 
// 定义路由：GET 请求 + 路径 '/'
router.get('/', (ctx) => {
  ctx.body = 'Hello, Koa Router!'; // 设置响应体
});
 
// 注册路由到 Koa 应用
app.use(router.routes());
// 启用路由中间件的 HTTP 方法验证（如 405 Method Not Allowed）
app.use(router.allowedMethods());
 
app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

### 二、核心概念与用法

#### 1. 路由方法

koa-router 支持所有 HTTP 方法（GET、POST、PUT、DELETE 等），语法统一为：

```javascript
router.方法名(路径, 处理函数);
```

**示例**：

```javascript
// GET 请求
router.get('/users', (ctx) => {
  ctx.body = ['用户1', '用户2'];
});
 
// POST 请求（通常需解析请求体，可配合 koa-bodyparser）
router.post('/users', (ctx) => {
  const newUser = ctx.request.body; // 需要 koa-bodyparser 解析
  ctx.body = { message: '用户创建成功', data: newUser };
  ctx.status = 201; // 设置状态码
});
 
// 匹配所有方法
router.all('/test', (ctx) => {
  ctx.body = '支持所有 HTTP 方法';
});
```

#### 2. 路由路径

路径可以是字符串、字符串模式或正则表达式：

```javascript
// 精确匹配
router.get('/about', (ctx) => { ctx.body = '关于我们'; });
 
// 带参数的路径（动态路由）
router.get('/users/:id', (ctx) => {
  const userId = ctx.params.id; // 获取路径参数
  ctx.body = `用户 ID: ${userId}`;
});
 
// 多参数
router.get('/users/:userId/posts/:postId', (ctx) => {
  console.log(ctx.params); // { userId: '123', postId: '456' }
});
 
// 通配符匹配（? 匹配 0 或 1 个字符）
router.get('/user/?name', (ctx) => { ... });
 
// 正则表达式（匹配以 /api 开头的路径）
router.get(/^\/api/, (ctx) => { ... });
```

#### 3. 路由中间件

路由处理函数本质是 Koa 中间件，支持异步行多个中间件，并通过 `next()` 传递控制权：

```javascript
// 日志中间件
const logMiddleware = async (ctx, next) => {
  console.log(`访问路径: ${ctx.path}`);
  await next(); // 执行下一个中间件
};
 
// 权限中间件：先日志，再处理响应
router.get('/users', logMiddleware, async (ctx) => {
  ctx.body = ['用户1', '用户2'];
});
```

#### 4. 路由前缀

为一组路由统一添加前缀，简化路径定义：

```javascript
// 创建带前缀的路由实例
const userRouter = new Router({ prefix: '/users' });
 
// 实际路径为 /users
userRouter.get('/', (ctx) => { ... });
// 实际路径为 /users/123
userRouter.get('/:id', (ctx) => { ... });
 
// 注册到应用
app.use(userRouter.routes());
```

#### 5. 嵌套路由

通过 `use()` 实现路由嵌套，适合大型应用拆分路由模块：

```javascript
// userRoutes.js（子路由）
const Router = require('koa-router');
const userRouter = new Router();
 
userRouter.get('/', (ctx) => { ... });
userRouter.get('/:id', (ctx) => { ... });
 
module.exports = userRouter;
 
// 主文件
const Koa = require('koa');
const app = new Koa();
const userRouter = require('./userRoutes');
 
// 嵌套路由：所有 userRouter 路由会被挂载到 /api 下
const apiRouter = new Router({ prefix: '/api' });
apiRouter.use('/users', userRouter.routes()); // 实际路径：/api/users
 
app.use(apiRouter.routes());
```

#### 6. 响应处理

通过 `ctx` 对象操作请求和响应：

- `ctx.request`：请求对象（包含 `method`、`url`、`query`、`body` 等）
- `ctx.response`：响应对象（包含 `body`、`status`、`headers` 等）

简写：`ctx.body` 等价于 `ctx.response.body`，`ctx.status` 等价于 `ctx.response.status`

**示例**：

```javascript
router.get('/query', (ctx) => {
  // 获取查询参数（?name=tom&age=18）
  console.log(ctx.query); // { name: 'tom', age: '18' }
  ctx.body = { query: ctx.query };
});
```

#### 7. 路由重定向

使用 `ctx.redirect()` 实现重定向：

```javascript
router.get('/old', (ctx) => {
  ctx.redirect('/new'); // 重定向到 /new
  ctx.status = 301; // 可选：设置 301 永久重定向（默认 302）
});
 
router.get('/new', (ctx) => {
  ctx.body = '新页面';
});
```

### 三、`allowedMethods` 的作用

`router.allowedMethods()` 是一个中间件，用于处理不支持的 HTTP 方法，自动返回标准响应：

- 当请求方法不被支持时（如对 GET 路由发送 POST 请求），返回 `405 Method Not Allowed`
- 当请求的 HTTP 方法未在服务器实现时（如 PROPFIND），返回 `501 Not Implemented`

必须在 `router.routes()` 之后注册：

```javascript
app.use(router.routes());
app.use(router.allowedMethods()); // 放在 routes 后面
```

### 四、常见问题与最佳实践

#### 路由顺序

Koa 路由按定义顺序匹配，精确路径应放在模糊路径之前，避免被覆盖：

```javascript
// 正确：先精确匹配
router.get('/users/profile', (ctx) => { ... });
// 后模糊匹配
router.get('/users/:id', (ctx) => { ... });
```

#### 路由模块化

大型应用建议按功能拆分路由文件（如 `userRoutes.js`、`postRoutes.js`），再通过嵌套路由组合。

#### 请求体解析

处理 POST、PUT 等请求时，需使用 `koa-bodyparser` 中间件解析请求体：

```bash
npm install koa-bodyparser
```

```javascript
const bodyParser = require('koa-bodyparser');
app.use(bodyParser()); // 需在路由之前注册
```

#### 错误处理

在路由中间件中通过 `try/catch` 捕获错误，并统一处理：

```javascript
router.get('/users/:id', async (ctx) => {
  try {
    const user = await getUserById(ctx.params.id);
    if (!user) {
      ctx.status = 404;
      ctx.body = '用户不存在';
      return;
    }
    ctx.body = user;
  } catch (err) {
    ctx.status = 500;
    ctx.body = '服务器错误';
  }
});
```
