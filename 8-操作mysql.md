# Node.js 操作 MySQL

## 一、什么是 MySQL？

MySQL 是一个开源的关系型数据库管理系统（RDBMS），使用结构化查询语言 SQL 进行数据操作。数据以表格（表）形式存储，适用于对数据一致性要求较高的系统。

### 特点

- **关系型数据库**：数据存储在表中，表之间可建立关系
- **严格的数据结构**：列名、类型必须定义
- **支持事务、视图、索引、存储过程等**
- **高性能、稳定性强、适合企业级应用**
- **支持多种存储引擎**（如 InnoDB）

## 二、MySQL 的功能概览

| 功能               | 说明                                  |
|--------------------|---------------------------------------|
| 表结构化           | 所有数据存储在“表”中，有固定字段类型  |
| SQL 查询语法       | 支持复杂 SQL 操作：连接、聚合、子查询等 |
| 多用户支持         | 权限控制精细，支持多用户操作          |
| ACID 事务支持      | 保证数据一致性和完整性                |
| 高可用与主从复制   | 支持主从复制、集群部署等高可用方案    |

## 三、MySQL 的安装与启动

### 安装 MySQL

官方网站下载安装：[https://dev.mysql.com/downloads/](https://dev.mysql.com/downloads/)  
使用 Docker 安装：

```bash
docker run --name mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -d mysql
```

### 启动服务

#### Windows 启动方式

##### 图形化界面方式

1. 打开 “服务” 窗口：按下 `Win + R`，输入 `services.msc` 并回车。
2. 在服务列表中找到 MySQL（名称可能为 MySQL80、MySQL57 等，取决于版本）。
3. 右键选择 “启动”（停止则选择 “停止”，重启选择 “重启”）。

##### 命令行方式（管理员权限）

按下 `Win + X`，选择 “Windows 终端（管理员）” 或 “命令提示符（管理员）”：

```bash
# 启动服务（服务名需替换为实际名称）
net start MySQL 
# 示例：net start MySQL80

# 停止服务
net stop MySQL
```

默认连接地址：`mysql://127.0.0.1:3306`

## 四、Node.js 如何连接 MySQL？

### 使用 mysql2 模块（推荐）

```bash
npm install mysql2
```

### 建立连接

```javascript
// db.js
const mysql = require('mysql2/promise');
 
async function connect() {
  const connection = await mysql.createConnection({
    host: 'localhost',
    user: 'root',
    password: '123456',
    database: 'testdb'
  });
  console.log("MySQL 连接成功");
  return connection;
}
 
module.exports = connect;
```

**说明**：

- `mysql2/promise` 支持 `async/await` 异步操作
- `testdb` 为数据库名称，需提前创建

## 五、创建数据表和插入数据（SQL 初始化）

```sql
CREATE DATABASE IF NOT EXISTS testdb;
 
USE testdb;
 
CREATE TABLE IF NOT EXISTS users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(50),
  password VARCHAR(50),
  age INT
);
```

## 六、Node.js 操作 MySQL（CRUD）

以下操作在 `async` 函数中执行，确保使用 `await` 等待数据库响应。

### 1️ 添加数据（Create）

```javascript
await connection.execute(
  'INSERT INTO users (username, password, age) VALUES (?, ?, ?)',
  ['Tom', '123456', 20]
);
```

### 2️ 查询数据（Read）

#### 查询全部

```javascript
const [rows] = await connection.execute('SELECT * FROM users');
console.log(rows);
```

#### 条件查询

```javascript
const [rows] = await connection.execute(
  'SELECT * FROM users WHERE age >= ?',
  [18]
);
```

#### 查询一条

```javascript
const [rows] = await connection.execute(
  'SELECT * FROM users WHERE username = ? LIMIT 1',
  ['Tom']
);
```

### 3️ 更新数据（Update）

```javascript
await connection.execute(
  'UPDATE users SET age = ? WHERE username = ?',
  [25, 'Tom']
);
```

### 4️ 删除数据（Delete）

```javascript
await connection.execute(
  'DELETE FROM users WHERE username = ?',
  ['Tom']
);
```

## 七、使用可视化工具查看数据库

| 工具               | 说明                                  |
|--------------------|---------------------------------------|
| MySQL Workbench    | 官方 GUI 工具，功能全面              |
| DBeaver            | 支持多种数据库，开源好用              |
| Navicat            | 商业软件，功能强大，体验良好          |
| HeidiSQL           | 免费轻量，适合基本操作                |

**使用方式**：连接 `localhost:3306`，输入账号密码后可管理数据库、表和数据。

## 八、实战项目结构示例

```text
project/
├── db.js               # 数据库连接配置
├── index.js            # 入口文件（包含 CRUD 操作）
```

### index.js 示例

```javascript
const connect = require('./db');
 
async function main() {
  const connection = await connect();
 
  // 添加
  await connection.execute(
    'INSERT INTO users (username, password, age) VALUES (?, ?, ?)',
    ['Alice', '123', 22]
  );
 
  // 查询
  const [users] = await connection.execute('SELECT * FROM users');
  console.log(users);
 
  // 更新
  await connection.execute(
    'UPDATE users SET age = ? WHERE username = ?',
    [23, 'Alice']
  );
 
  // 删除
  await connection.execute(
    'DELETE FROM users WHERE username = ?',
    ['Alice']
  );
 
  connection.end(); // 关闭连接
}
 
main();
```

## 总结

| 项目         | 内容                                  |
|--------------|---------------------------------------|
| 数据库       | MySQL（关系型数据库）                |
| Node 连接方式 | `mysql2`（Promise API）               |
| 操作方式     | 执行 SQL 语句（Insert、Select、Update、Delete） |
| 工具推荐     | MySQL Workbench、DBeaver、Navicat    |
| 数据结构     | 表（Table）、字段（Column）、记录（Row） |
| 常见端口     | 默认 3306                            |
