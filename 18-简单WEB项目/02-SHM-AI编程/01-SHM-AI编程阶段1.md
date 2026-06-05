# SHM-AI编程阶段1
## 版本更新说明
原始版本V1.0为之前边学边做的框架，后续主要由AI进行，版本更新说明如下：

V1.1版本：
- UI优化：登录页面及主页风格进行优化，头部及侧边导航进行优化

V2.0版本：
- 后端从Mock数据模拟改成MySQL数据库

V2.1版本：
- 新增用户管理模块
- 新增Linux系统检查模板
- 系统清单优化，细化系统信息，新增分类
- 修复无法翻页的问题
- 增加修改系统信息后成功提示，并且返回前一页面
- 增加删除条目需要管理员密码才行，避免随意删除
- 每个host的主页模板进行了确认，新增点击view就跳转到对应主页
- 新增System Class页面，对系统进行了多维度分类，可以选择同类型系统。

## AI环境配置
### 安装Claude
安装Claude，打开Powershell终端，执行：
```shell
winget install Anthropic.ClaudeCode
```
如果Claude命令提示没有，添加环境变量：
```shell
$userPath = [Environment]::GetEnvironmentVariable("PATH","User")
$newPath = "$userPath;$env:USERPROFILE\.local\bin[Environment]::SetEnvironmentVariable("PATH",$newPath,"User")
```
Claude常用命令：

| 命令                 | 作用                                       |
| :------------------- | :----------------------------------------- |
| `/help`              | 查看全部指令                               |
| `/model opus/sonnet` | 切换大模型：opus 复杂代码、sonnet 日常开发 |
| `/clear`             | 清空上下文，新开对话                       |
| `/exit`              | 退出 claude 终端                           |

### 安装CC Switch
安装cc-switch，地址：[https://github.com/farion1231/cc-switch](https://github.com/farion1231/cc-switch)

### 添加AI模型
&#8195;&#8195;在CC Switch上配置模型，此阶段配置的是Xiaomi MiMo V2 Pro，申请的免费tokens，给了380亿，不过有时间限制，到期后再看。

### 插件安装
先添加Superpowers专属插件市场（必做）
```shell
/plugin marketplace add obra/superpowers-marketplace
```
然后安装：
```
/plugin install superpowers@superpowers-marketplace
```
&#8195;&#8195;安装完成操作`/exit` 退出当前会话重新输入 `claude` 新开对话加载插件。`/help` 查看，出现 `/superpowers:brainstorm、execute-plan` 即装好。

### Git配置
安装GIT后配置：
```shell
# 配置用户名(GitHub昵称)
git config --global user.name "xxxx-huang"
# 配置邮箱(Git注册邮箱)
git config --global user.email "huangxxxxxxxxx@163.com"
# 查看是否配置成功
git config --global --list
```
生产ssh key：
```shell
# ED25519加密（新版推荐，安全）
ssh-keygen -t ed25519 -C "huang18891023@163.com"
# 老系统报错就换RSA：ssh-keygen -t rsa -b 4096 -C "邮箱"
```
拷贝：
```shell
# Git Bash一键复制到剪贴板
clip < ~/.ssh/id_ed25519.pub
```
去GitHub上配置，配置好后验证：
```shell
$ ssh -T git@github.com
The authenticity of host 'github.com (20.205.243.166)' can't be established.
ED25519 key fingerprint is: SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? YES
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Hi bond-huang! You've successfully authenticated, but GitHub does not provide shell access.
```
克隆仓库到本地：
```shell
$ git clone git@github.com:bond-huang/shm.git
Cloning into 'shm'...
remote: Enumerating objects: 270, done.
remote: Counting objects: 100% (270/270), done.
remote: Compressing objects: 100% (178/178), done.
remote: Total 270 (delta 118), reused 223 (delta 71), pack-reused 0 (from 0)
Receiving objects: 100% (270/270), 292.81 KiB | 25.00 KiB/s, done.
Resolving deltas: 100% (118/118), done.
```
VScode打开项目，然后打开终端，输入Claude命令开启AI编程。

## V1.0版本
此为初始版本，框架搭建好后五年没更新，用AI对框架进行了说明。
### 项目框架说明
&#8195;&#8195;这是一个 AIX 服务器监控管理平台（SHM - Server Health Monitor），基于Vue 3技术栈构建。以下是项目概览：

技术栈：
  - 前端框架：Vue 3 + Vue Router + Vuex  - UI 组件库：Element Plus
  - 图表库：ECharts  - HTTP 客户端：Axios
  - 模拟数据：MockJS
  - 构建工具：Vue CLI

核心功能模块：

模块|说明
:---:|:---
Dashboard│主机概览仪表盘，展示 CPU、内存、PageSpace、文件系统使用率（环形图）及主机信息
All Systems|所有系统列表管理，支持主机信息更新
AIX Systems│AIX专用系统页面，含独立的主机详情
Performance|性能监控模块，含处理器、内存、磁盘、适配器等子项检查
Standardcheck|标准巡检功能
AutoPM|自动化巡检报告
Login|登录认证

项目特点
- 面向IBM AIX 小型机运维场景，监控 CPU、内存、磁盘、文件系统等关键指标
- 使用Mock数据模拟后端接口，方便前端独立开发
- 包含HTML格式的巡检报告（src/reports/）
- 采用Hash路由模式，支持静态部署

## V1.1版本
### AI启动项目
让AI帮我进行启动，提示需要基础软件，手动安装版本：
```shell
PS C:\Users\admin> node --version
v24.16.0
PS C:\Users\admin> npm --version
11.13.0
PS C:\Users\admin>
```
然后AI就帮我启动了项目。
### AI优化UI
需求：“在导航配置了Home按钮，但是点击不能跳转到主页”，AI修改结果：
```
● 改好了，做了三处修改：
  1. 添加点击事件 — @click="goHome" 绑定到 div 上
  2. 添加跳转方法 — goHome() 路由跳转到 /dashboard（即主页）
  3. 添加手型光标 — cursor: pointer 提示用户可点击
```
需求：“登录界面有点挫，帮我改高大上具有科技感一点？”，AI修改结果：
```
● 现在确实很朴素。我来给它改成科技感十足的深色主题 + 玻璃拟态 + 动态粒子背景。
```
其它优化：
- 主要导航进行优化，风格参考登录页面，右侧用户下拉选项进行了优化
- 左侧导航菜单及风格进行了优化

## V2.0版本
### 后端改造
需求：“那就使用mysql，可以根据我的后台数据，改成使用mysql吗？数据根据我的模拟类型进行创建。”

后端（server/ 目录），新建文件： 
- server/app.js — Express 入口
- server/config/db.js — MySQL 连接池  - server/middleware/auth.js — JWT 中间件                                            
- server/routes/auth.js — 登录接口
- server/routes/menu.js — 菜单接口
- server/routes/hosts.js — 主机 CRUD 接口
- server/routes/dashboard.js — 统计接口
- server/routes/perf.js — CPU 性能接口
- server/sql/schema.sql — 建表脚本
- server/sql/seed.sql — 初始数据（与 mock 一致）
- server/.env — 数据库配置
- server/README.md — 使用说明

前端改动：

文件|改动
:---:|:---
src/main.js|注释掉import '@/mock/mock.js'
src/plugins/axios.js|添加baseURL:'/api'
vue.config.js|添加devServer proxy指向localhost:3000

让AI自动安装了MySQL，然后启动了项目。继续下一步操作。

## V2.1版本
### 用户管理
&#8195;&#8195;需求：用户设置项帮我添加对用户修改密码，添加用户的内容，账号密码写入数据库，单独建一个用户表，包含之前的admin用户。

新增功能

功能|说明
:---:|:---
修改密码|用户可修改自己的密码
添加用户|Admin可添加新用户
用户列表|Admin可查看所有用户
删除用户|Admin可删除用户（不能删除自己）

新增/修改文件：

文件|说明
:---:|:---
server/routes/users.js|用户管理后端 API
server/sql/schema.sql|新增 role 字段 
src/api/user.js|前端用户 API
src/views/UserSettings.vue|用户管理页面 
src/router/index.js|新增 /user-settings 路由
src/components/Header/Tools.vue|"User Admin" 跳转到用户管理

完成后进行了进一步优化：
- 用户管理页面风格进行了优化，输入框进行了优化
- 左侧User Setting菜单点击到到用户管理页面
- admin用户可以修改其它用户的密码
- 其它小问题进行优化，例如文本对齐问题

### Linux系统检查模板
&#8195;&#8195;需求：AIXtest1页面是我模拟的系统数据收集的页面,参照这个帮我写一个Linux的，以RHEL8为例，页面风格完全一样，内容根据Linux系统和AIX系统差别进行修改。页面通过Linuxtest1菜单点进去我看看。

新增文件

文件|说明
:---:|:---
src/views/hostpage/Linuxbase.vue|Linux 主页面（6个 Tab）
src/components/Linuxpage/Dashboard.vue|概览仪表盘（CPU/Memory/Swap/FileSystem
src/components/Linuxpage/AutoPM.vue|自动巡检
src/components/Linuxpage/Standardcheck.vue|标准检查
src/components/Linuxpage/Performance.vue|性能图表汇总
src/components/Linuxpage/checkitem/DiskCheck.vue|磁盘/LVM 检查（替代 RootVG）
src/components/Linuxpage/checkitem/ProcessorPerf.vue|CPU 性能图
src/components/Linuxpage/checkitem/MemoryPerf.vue|内存/Swap 性能图
src/components/Linuxpage/checkitem/DiskPerf.vue|磁盘性能图（sda/sdb/dm-0）
src/components/Linuxpage/checkitem/NetworkPerf.vue|网络性能图（eth0/eth1/lo）

### 系统清单优化
&#8195;&#8195;需求：关于All Systems页面的内容，重新帮我建表，分类：HostType 、HostName、IP Addr、Status、 Category、Business Name、Data Center。在数据表中，IP Addr是唯一的，不能冲突，HostType 、HostName、IP Addr不能为空。页面风格啥的都不变。

新表结构：
字段|说明|约束
:---:|:---|:---
id|主键|自增
host_type|主机类型|NOT NULL
host_name|主机名|NOT NULL
ip_address|IP 地址|NOT NULL, UNIQUE
status|状态|Health/Warning/Severe
category|分类|Database/Application/Web Server等
business_name|业务名称|-   
data_center|数据中心|DC-Beijing/DC-Shanghai等

### System Class页面
&#8195;&#8195;需求：左侧导航的system class是我计划的系统分类主页，还没做，根据之前的数据类型和页面风格，帮我涉及一个分类导航页面，根据不同分类，点进去后显示是此分类的系统清单。 

新增功能，System Class分类页面：
  - 左侧导航 System Admin → System Class 进入分类主页  - 5 个分类卡片：Database、Application、Web Server、Middleware、Infrastructure
  - 每个卡片显示系统数量和健康状态统计
  - 点击卡片进入该分类的系统清单
  - 清单中可点击 View 跳转到主机详情页

新增文件

文件|说明
:---|:---
src/views/CategoryList.vue|分类主页（卡片列表） 
src/views/CategoryDetail.vue|分类详情（系统清单） 
src/api/demo.js |新增 getCategories()、getCategoryHosts()

新增API

接口|说明
:---|:---
GET /api/categories|获取分类统计（数量、健康状态）
GET /api/categories/:name/hosts|获取指定分类的主机列表 

## 待补充
