# SHM 系统健康管理平台 — 操作手册
本手册由AI自动生成。

## 一、项目简介

SHM（System Health Management）是一套企业级服务器健康管理平台，用于集中监控、巡检和运维管理基础设施（AIX、Linux、Windows、AS400 主机）。

### 技术栈

| 层级 | 技术 |
|------|------|
| 前端 | Vue 3 + Vue Router 4 + Vuex 4 + Element Plus + ECharts 5 |
| 后端 | Node.js + Express 5 + JWT + SSH2 |
| 数据库 | MySQL 8.4 |
| 分析工具 | Python 3 |

---

## 二、环境要求

在启动项目前，请确保已安装以下软件：

| 软件 | 版本要求 | 检查命令 |
|------|----------|----------|
| Node.js | >= 14.x | `node -v` |
| npm | >= 6.x | `npm -v` |
| MySQL | >= 8.0 | `mysql --version` |
| Python（可选） | >= 3.x | `python --version` |

---

## 三、首次部署（仅新环境需要）

### 3.1 安装依赖

```bash
# 安装前端依赖
cd D:\AI\shm
npm install

# 安装后端依赖
cd D:\AI\shm\server
npm install
```

### 3.2 初始化数据库

打开命令行，执行以下命令导入数据库结构和初始数据：

```bash
# 导入表结构
mysql -u root -p < D:\AI\shm\server\sql\schema.sql

# 导入初始数据（含默认用户和示例主机）
mysql -u root -p < D:\AI\shm\server\sql\seed.sql
```

> 执行时会提示输入 MySQL root 密码。如果 root 无密码，使用：`mysql -u root < 文件路径`

### 3.3 配置环境变量

编辑 `D:\AI\shm\server\.env` 文件：

```ini
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=          # 填写你的 MySQL 密码，无密码留空
DB_NAME=shm
JWT_SECRET=shm-secret-key-2024
PORT=3000
```

---

## 四、日常启动（每次使用）

按以下顺序依次启动三个服务：

### 第 1 步：启动 MySQL 数据库

**方式一：通过 Windows 服务启动（推荐）**

```bash
# 以管理员身份运行 CMD，执行：
net start MySQL80
```

**方式二：手动启动**

```bash
"C:\Program Files\MySQL\MySQL Server 8.4\bin\mysqld.exe" --defaults-file="C:\ProgramData\MySQL\MySQL Server 8.4\my.ini" --console
```

**验证是否启动成功：**

```bash
mysql -u root -p -e "SELECT 1;"
```

> 能正常返回结果即表示 MySQL 已启动。

### 第 2 步：启动后端服务

打开一个新的命令行窗口，执行：

```bash
cd D:\AI\shm\server
node app.js
```

**看到以下输出表示启动成功：**

```
SHM Server running on http://localhost:3000
[Scheduler] Starting perf collection every 10 min
[Scheduler] Collected X hosts at ...
```

### 第 3 步：启动前端服务

再打开一个新的命令行窗口，执行：

```bash
cd D:\AI\shm
npm run serve
```

**看到以下输出表示启动成功：**

```
  App running at:
  - Local:   http://localhost:8080/
  - Network: http://192.168.x.x:8080/
```

### 第 4 步：访问系统

浏览器打开 **http://localhost:8080/**

---

## 五、默认账号

| 用户名 | 密码 | 角色 |
|--------|------|------|
| admin | 123456 | 管理员 |

---

## 六、服务端口一览

| 服务 | 端口 | 说明 |
|------|------|------|
| 前端开发服务器 | 8080 | Vue CLI dev server |
| 后端 API 服务器 | 3000 | Express 服务 |
| MySQL 数据库 | 3306 | 数据库服务 |
| MySQL X Plugin | 33060 | MySQL 协议扩展（可忽略） |

---

## 七、关闭服务

按启动的**相反顺序**关闭：

1. 在前端命令行窗口按 `Ctrl + C` 停止前端
2. 在后端命令行窗口按 `Ctrl + C` 停止后端
3. 停止 MySQL：

```bash
# 如果是通过 net start 启动的：
net stop MySQL80

# 如果是手动启动的，在 MySQL 窗口按 Ctrl + C，或另开窗口执行：
"C:\Program Files\MySQL\MySQL Server 8.4\bin\mysqladmin.exe" -u root -p shutdown
```

---

## 八、常见问题排查

### 8.1 登录提示"服务错误"

**原因**：MySQL 未启动或连接配置有误。

**排查步骤**：

```bash
# 1. 检查 MySQL 是否在运行
mysql -u root -p -e "SELECT 1;"

# 2. 如果连不上，启动 MySQL
net start MySQL80

# 3. 重启后端服务（在 server 目录下）
cd D:\AI\shm\server
node app.js
```

### 8.2 前端启动报 OpenSSL 错误

项目已内置解决方案，`package.json` 中的 serve 脚本已包含 `--openssl-legacy-provider` 参数，一般不会遇到此问题。如仍报错：

```bash
set NODE_OPTIONS=--openssl-legacy-provider && npm run serve
```

### 8.3 npm install 安装失败

```bash
# 清除缓存后重试
npm cache clean --force
npm install
```

### 8.4 端口被占用

```bash
# 查找占用端口的进程
netstat -ano | findstr :3000    # 查后端
netstat -ano | findstr :8080    # 查前端
netstat -ano | findstr :3306    # 查 MySQL

# 根据 PID 终止进程（替换 <PID> 为实际值）
taskkill /PID <PID> /F
```

### 8.5 数据库表不存在

重新导入建表脚本：

```bash
mysql -u root -p < D:\AI\shm\server\sql\schema.sql
mysql -u root -p < D:\AI\shm\server\sql\seed.sql
```

> 注意：`schema.sql` 会先执行 `DROP DATABASE IF EXISTS shm`，会清除所有数据。如需保留数据，请手动导入单个表。

---

## 九、项目目录结构速查

```
D:\AI\shm\
├── src/                    # 前端源码
│   ├── api/                # API 接口定义
│   ├── views/              # 页面组件
│   ├── components/         # 公共组件
│   ├── router/             # 路由配置
│   ├── store/              # Vuex 状态管理
│   └── plugins/            # 插件配置（axios、element-plus）
├── server/                 # 后端源码
│   ├── routes/             # API 路由
│   ├── services/           # 业务逻辑（SSH、性能采集、巡检、报表）
│   ├── config/             # 数据库连接配置
│   ├── middleware/          # JWT 认证中间件
│   ├── sql/                # 数据库初始化脚本
│   ├── scripts/            # Python 分析脚本
│   ├── .env                # 环境变量配置
│   └── app.js              # 后端入口
├── public/                 # 静态资源
├── tools/                  # 独立分析工具
├── package.json            # 前端依赖
└── vue.config.js           # Vue CLI 配置
```

---

## 十、功能模块一览

| 模块 | 说明 |
|------|------|
| **Dashboard** | 系统总览，主机状态统计 |
| **主机管理** | AIX/Linux/Windows/AS400 主机的增删改查 |
| **性能监控** | 基于 SSH 实时采集 CPU/内存/磁盘/网络，每 10 分钟自动采集 |
| **自动巡检 (AutoPM)** | 一键 Linux 系统巡检，生成 HTML 报告 |
| **标准检查** | 按主机类型的标准化健康检查项 |
| **脚本库** | 集中管理运维脚本，支持上传和分类 |
| **分析工具** | CSV 数据上传 + Python 脚本分析，生成可视化报告 |
| **用户管理** | JWT 认证，管理员/普通用户角色 |

---

*最后更新：2026-06-13*
