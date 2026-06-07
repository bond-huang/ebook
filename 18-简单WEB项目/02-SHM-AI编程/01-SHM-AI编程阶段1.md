# SHM-AI编程阶段1
使用AI开始完善此项目。
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

### System Class页面创建
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

## V2.2版本
### 登录报错
项目登陆后就弹出server error错误：
```
修复： 将 dashboard.js 第 10行和第 22-24 行的 status_info 改为status，与数据库实际表结构一致。
```
### Home页面优化
&#8195;&#8195;需求：home页面下我设计了四个按钮，分别是All hosts list，点击帮我跳转到刚才All systems页面，还要healthy hosts，点击展示筛选healthy状态的主机，后面类推。

全部完成！改了三处：
- 后端 server/routes/hosts.js — GET /api/allsystems 新增 status 查询参数，支持按状态筛选
- 前端 src/api/demo.js — getSystems() 新增 status 参数透传  
- 前端 src/views/allsystems/AllSystems.vue — 从路由 query.status 读取筛选条件，并监听路由变化自动刷新
- 前端 src/views/Dashboard.vue — 四个按钮绑定点击事件：   
    - All Hosts List → 跳转 /allsystems（全部主机）
    - Healthy Hosts → 跳转 /allsystems?status=Health
    - Warning Hosts → 跳转 /allsystems?status=Warning
    - Severe Hosts → 跳转 /allsystems?status=Severe

### System class分类优化
&#8195;&#8195;需求：在system class页面，我看你根据用途进行分类，帮我再单独弄几个分类，例如根据数据中心分类，根据系统类型分类

后端 server/routes/hosts.js — 新增4个接口：
  - GET /api/datacenters — 按数据中心分组统计  - GET /api/datacenters/:name/hosts — 某数据中心的主机列表
  - GET /api/hosttypes — 按系统类型（AIX/Linux/AS400）分组统计  - GET /api/hosttypes/:name/hosts — 某系统类型的主机列表

前端 src/api/demo.js — 新增 4 个 API 函数，前端 src/views/CategoryList.vue — 顶部加了三个 Tab 切换：
  - By Usage — 按用途分类（Database、Application 等）
  - By Data Center — 按数据中心分类（DC-Beijing、DC-Shanghai 等）
  - By System Type — 按系统类型分类（AIX、Linux、AS400）

### 连接测试系统
&#8195;&#8195;需求：我加入了一个真实的linux系统IP，192.168.31.17，我需要在系统主页获取到的CPU内存等信息是真实的，帮我写一下如何进入这个系统获取信息。

后端已重启。现在你需要给 192.168.31.17这台主机配置 SSH 登录凭据。在 MySQL 中执行：
```sql
UPDATE hosts SET ssh_user = '你的用户名', ssh_password = '你的密码' WHERE ip_address = '192.168.31.17';
```
比如 root 用户：
```sql
UPDATE hosts SET ssh_user = 'root', ssh_password = 'your_password' WHERE ip_address = '192.168.31.17';
```
&#8195;&#8195;配置好后，打开这台主机的详情页 → Performance Check tab，四个模块（CPU、内存、磁盘、网络）会自动通过SSH实时采集真实数据：
- CPU — 仪表盘（总使用率）+ 柱状图（user/system/iowait/idle 分项）
- 内存 — 仪表盘 + 堆叠柱状图（RAM/Swap）+ 详细数值面板
- 磁盘 — 表格展示各分区，带彩色进度条（绿/黄/红）
- 网络 — 柱状图（各网卡 RX/TX 流量 MB）+ 数据表格

&#8195;&#8195;需求：我不想每个系统都输入账户密码，帮我在系统页面的overview页面的Refresh按钮前面加个按钮，叫Authentication，用来输入系统账户密码。 

功能已完成。打开任意主机详情页 → Overview tab → 点击 Authentication 按钮：
- 弹窗中输入SSH用户名和密码
- 点 Test Connection — 会先保存凭据，然后实时SSH连接测试，成功会显示远程主机名 -点 Save — 仅保存凭据
- 保存后，切到 Performance Check tab 就会自动通过 SSH 采集真实的 CPU、内存、磁盘、网络数据。

&#8195;&#8195;需求：Authentication右边的Refresh按键，我点一次，就马上去系统获取一次数据，数据马上更新到仪表盘。

现在点Refresh按钮会：
- SSH 连接到目标系统  
- 采集 CPU、内存、Swap、磁盘数据
- 四个仪表盘实时更新（CPU 使用率、内存使用率、Swap 使用率、最高磁盘使用率）  4. 颜色自动变化：绿色 < 60%，黄色 60-80%，红色 > 80%
- 成功后提示远程主机名

&#8195;&#8195;需求：performance check里面的图标大小异常了，修复一下，并且定时从系统抓取的数据进行记录，保留一天时间，在这里进行展示，图标格式不用变：
- 图表尺寸修复： 四个组件统一用 height: 300px; width: 100% + autoresize，图表自适应容器。
  定时采集 + 历史展示：
- 新建 perf_history 表存储采集数据  - 后端定时任务每 10 分钟自动 SSH 采集所有已配置凭据的主机，数据保留 24 小时                      
- 四个图表组件改为从 /api/perf/history/:hostId 拉取历史数据，渲染为 24 小时折线图（和之前格式一致）
- AIX 主机和未配置凭据的主机仍显示默认数据

&#8195;&#8195;现在的流程：Overview 页点 Authentication 配置密码 → 点Refresh验证连通 → 切到Performance Check查看24小时趋势图，每10分钟自动更新新数据点。

### 系统主页信息页面优化
&#8195;&#8195;需求：在linux系统的主页的Host Information里面加点内容，参考我写的Linuxtest1页面里面的项目，加一点内容，数据从实际的系统获取。

Host Information现在参考了Linuxtest1页面的字段，点击Refresh会从真实系统SSH采集以下信息：

字段|来源命令
:---|:---
OS Version|/etc/os-release PRETTY_NAME
Kernel|uname -r
HostName|hostname
Architecture|uname -m
SELinux|getenforce
CPU Cores|nproc 
Memory Size|free -m
Swap Size|free -m
Uptime|uptime -p
Load Average|/proc/loadavg (1/5/15 min)

加上原来的Status、Category、Data Center、Description，一共15个字段，3列5行布局。

### 一键巡检功能
&#8195;&#8195;需求：操作系统主页Automatic PM页面，我的设计是点击Generate Report后自动生成html的报告，帮我写巡检脚本，并且脚本保留，Generate Report左边弄个Inspection Script按钮，点击后展示脚本内容。生成的报告参考我在reports目录下的html报告，报告保存在reports下，新建一个文件夹，文件夹名称用系统hostname加IP组成，报告记录在Reports List里面，记录日期和名称，报告名称包含：系统名称+日期时间。后面view点击后查看报告，Download就可以下载，delete删除，删除时候进行提示。

编译通过。AutoPM 页面功能完整：
- Inspection Script 按钮 — 点击弹窗展示 67 行 Linux 巡检脚本（暗色终端风格）
- Generate Report 按钮 — 点击后通过 SSH 连接目标系统执行巡检，采集：
  - 系统信息（OS、Kernel、Architecture、Uptime 等）  
  - CPU 使用率（user/system/iowait/idle）                                                          
  - 内存和 Swap
  - 磁盘各分区使用率
  - 系统错误日志
  - Top 进程
  - SELinux / Firewall 状态
  - 重启历史

&#8195;&#8195;生成HTML报告保存在server/reports/{hostname}_{ip}/ 目录，参考了现有 AIX报告的风格，带颜色状态分析（绿色正常/红色警告）：
- Reports List — 自动加载报告列表，每条显示日期和文件名，支持：
  - View — 新窗口打开 HTML 报告
  - Download — 浏览器下载
  - Delete — 二次确认弹窗后删除

## 待补充
