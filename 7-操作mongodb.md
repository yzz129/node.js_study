# Node.js 操作 MongoDB

## 一、什么是 MongoDB？

MongoDB 是一个开源的 NoSQL 文档型数据库，它使用类似 JSON 的 BSON 格式存储数据，每一条记录称为一个“文档（Document）”，多个文档组成“集合（Collection）”。

### 特点

- **非关系型**：无需预定义表结构（灵活）
- **文档存储**：结构类 JSON，支持嵌套
- **高性能、高可扩展性**
- **支持索引、聚合、地理位置查询等强大功能**

## 二、MongoDB 的功能概览

- 文档式结构（BSON）
- 动态模式：可以存不同结构的数据
- 高可用性与分片机制：适用于大数据量场景
- 内置副本集：数据安全
- 丰富的查询语法：支持条件、排序、分页、聚合等

## 三、MongoDB 的安装与启动

### 安装 MongoDB（以本地安装为例）

Windows/macOS/Linux 可从官网下载：[Download MongoDB Community Server](https://www.mongodb.com/try/download/community)  
也可使用 Docker 安装：

```bash
docker run -d -p 27017:27017 --name mongo mongo
```

### 启动 MongoDB

本地方式：

```bash
mongod --dbpath 数据文件路径
```

默认监听地址为：`mongodb://127.0.0.1:27017`

## 四、Node.js 如何连接 MongoDB？

### 使用 Mongoose ODM 工具

```bash
npm install mongoose
```

### 建立连接

```javascript
// db.js
const mongoose = require("mongoose");
 
mongoose.connect("mongodb://127.0.0.1:27017/mydb")
  .then(() => console.log("MongoDB 连接成功"))
  .catch((err) => console.error("MongoDB 连接失败", err));
```

**说明**：

- `127.0.0.1:27017` 是 MongoDB 默认端口
- `mydb` 是数据库名，不存在会自动创建

## 五、定义数据模型（Model）

```javascript
// model/UserModel.js
const mongoose = require("mongoose");
 
const UserSchema = new mongoose.Schema({
  username: String,
  password: String,
  age: Number
});
 
// 对应集合 users（自动将模型名转为复数形式）
const UserModel = mongoose.model("user", UserSchema); 
module.exports = UserModel;
```

## 六、Node.js 操作 MongoDB（CRUD）

所有操作需在 `await connect()` 后使用，或包裹在 `async` 函数中。

### 1️ 添加数据（Create）

```javascript
await UserModel.create({
  username: "Tom",
  password: "123456",
  age: 20
});
```

### 2️ 查询数据（Read）

#### 查询全部

```javascript
const users = await UserModel.find(); 
```

#### 条件查询

```javascript
// 查询年龄 >= 18 的用户
const users = await UserModel.find({ age: { $gte: 18 } });
```

#### 查询一条

```javascript
const user = await UserModel.findOne({ username: "Tom" });
```

### 3️ 更新数据（Update）

```javascript
await UserModel.updateOne(
  { username: "Tom" }, // 查询条件
  { $set: { age: 25 } } // 更新内容
);
```

### 4️ 删除数据（Delete）

```javascript
await UserModel.deleteOne({ username: "Tom" });
```

## 七、使用可视化工具查看数据库

推荐以下图形化 MongoDB 管理工具：

| 工具               | 说明                                  |
|--------------------|---------------------------------------|
| MongoDB Compass    | 官方 GUI 工具，功能强大，免费        |
| Robo 3T            | 轻量 GUI，常用于开发调试              |
| NoSQLBooster       | 支持 MongoShell 脚本和图表            |
| MongoDB Atlas      | 云端托管 + 可视化操作                 |

**使用方式**：连接 `mongodb://127.0.0.1:27017/mydb`，即可看到所有集合与数据。

## 八、实战推荐结构示例

```test
project/
├── db.js               # 数据库连接配置
├── model/
│   └── UserModel.js    # 用户模型
├── index.js            # 入口文件（包含增删改查）
```

### index.js 示例

```javascript
const connect = require('./db');
const UserModel = require('./model/UserModel');
 
async function main() {
  await connect();
 
  // 添加
  await UserModel.create({ username: "Alice", password: "123", age: 22 });
 
  // 查询
  const users = await UserModel.find();
  console.log(users);
 
  // 更新
  await UserModel.updateOne({ username: "Alice" }, { age: 23 });
 
  // 删除
  await UserModel.deleteOne({ username: "Alice" });
 
  process.exit();
}
 
main();
```

## 总结

| 项目         | 内容                                  |
|--------------|---------------------------------------|
| 数据库       | MongoDB（非关系型，文档型）          |
| Node 连接方式 | `mongoose.connect()`                  |
| 操作方式     | create、find、updateOne、deleteOne    |
| 工具推荐     | MongoDB Compass、Robo 3T、NoSQLBooster 等 |
| 数据结构     | 文档（Document）、集合（Collection）  |
| 常见端口     | 默认 27017                            |
