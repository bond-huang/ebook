# SHM-AI编程阶段2
使用AI开始完善此项目。
## V2.3
### 系统主页内容优化
做了一下优化：
- Inspection Script按钮点击后可以看到脚本了，但是格式不好，优化下，shell脚本格式展现出来，就像markdown里面展示shell脚本一样，有颜色标记
- Reports List下面报告清单后面的三个按钮一行放不下换行了，修复下
- 点击Generate Report还是失败报错了，没法执行，并且我需要在Reports List下面报告清单历史的报告也一直展示，除非有新的生成
- Reports List下面报告清单后面的delete按钮做的和前面一样大，里面放个表示删除的图标，修复下
- 删除报告提示失败，但是实际上是删除了，修复下 
- 报告名称格式帮加上系统的IP地址，方便区分
- 点击系统进入Host Detail页面后，从系统获取的信息不显示了，点击刷新才有，帮我改成保留上一次获取的数据，刷新后更新
- Host Detail页面查看系统信息时候，还是没有上次的数据，保存展示最后一次刷新获取的数据，并且刷新经常报错，修复下
- 点击Inspection Script按钮查看巡检脚本，能不能去掉前面的编号，并且能不能左对齐，缩进啥的都按shell格式来
- 巡检脚本根据系统进行分类，这是Linux系统的，并且是RHEL8的，脚本名称改成Linux_RHEL8，以后系统只要是Linux系统，默认用这个脚本，在查看脚本页面，帮忙加上修改和下载按钮

### 脚本库功能
&#8195;&#8195;需求：之前弄了Script Library，帮我完善下，页面参考All Systems，包含筛选和add功能，数据库里面建立个脚本库，添加脚本文件包含以下信息：Name、System、Type、Description、Script。在数据表中，name是唯一的，System表示运行的环境，例如AIX，RHEL，Windows等，Type表示脚本类型，例如AIX系统脚本，MySQL脚本，redit脚本等，Description是个markdown文本，可以进行编辑，Script是脚本文件，需要上传文件。

目前脚本内容会同时存两份：
- 数据库scripts表的script_content字段（纯文本内容，用于查看和编辑）  
- 磁盘 uploads/scripts/目录（原始文件）

修复添加脚
- 添加脚本失败 — axios 全局默认 Content-Type: application/x-www-form-urlencoded 覆盖了
  multipart/form-data，导致文件上传的 boundary 丢失。在请求拦截器里加了判断：FormData 时自动删除
  Content-Type，让浏览器自动设置。
- Markdown 上传 — Description 区域加了 Upload .md 按钮，选择 .md 或 .txt  文件后内容自动填充到文本框，可以继续编辑。  

Script Library功能完整：
- 数据库 — scripts 表（name 唯一、run_system、script_type、description、script_content、filename）
- 列表页 — 参考 All Systems 布局：  - 搜索框（按名称/描述模糊搜索）
- System 下拉筛选（AIX、RHEL、Windows 等）  
- Type 下拉筛选（System Script、Database Script 等）                                             
- 分页 + 刷新
- 列：Name、System（彩色标签）、Type、Description、File、Updated
- 操作：View（终端风格代码查看）、Edit、Delete（二次确认）

Add Script 弹窗：
  - Name — 唯一名称
  - System — 下拉选择或自定义输入（AIX/RHEL/CentOS/Ubuntu/Windows/MySQL/Redis/Oracle/DB2）
  - Type — 下拉选择或自定义输入
  - Description — 支持 Markdown 的文本域
  - Script File — 文件上传（.sh/.py/.sql/.bat/.ps1 等，最大 5MB）
  - Or paste content — 手动粘贴脚本内容

Edit Script — 复用同一个弹窗，加载已有数据，支持更新文件或内容。

&#8195;&#8195;需求：添加脚本时候失败了，修复下，脚本说明弄个上传markdown文档，上传后可以编辑。
- 添加脚本失败 — axios 全局默认 Content-Type: application/x-www-form-urlencoded 覆盖了
  multipart/form-data，导致文件上传的 boundary 丢失。在请求拦截器里加了判断：FormData 时自动删除
  Content-Type，让浏览器自动设置。
- Markdown 上传 — Description 区域加了 Upload .md 按钮，选择 .md 或 .txt  文件后内容自动填充到文本框，可以继续编辑。 

&#8195;&#8195;需求：脚本Description如果是文件，在列表显示文件。后面view选型，查看Description用markdown格式显示
- 列表Description列 — 如果上传了.md 文件，显示文件名图标+文件名；否则显示纯文本描述。
- View 弹窗Description — 用marked渲染为Markdown格式：标题、列表、代码块、表格、引用等都有样式，带最大高度滚动。顶部显示 .md 文件名。
- 脚本Description文本太长了，直接显示文件名吧，点击文件名就是预览，后面view显示的markdown格式不对，跟我的markdown不一样，都没左对齐，改一下

### 分析工具
&#8195;&#8195;需求：Analytical Tools打算放一些分析专用工具，例如在项目里面要跑python脚本用openpyx分析excel或csv文件，上传的csv文件，分析工具也是手动上传，例如python脚本文件，然后在web页面点击分析按钮，出现分析html的分析结果。Analytical Tool页面功能：
- 上传区：
  - Analysis Name — 分析名称
  - CSV Data File — 上传 CSV/Excel 数据文件  - Python Script — 上传 .py 分析脚本
  - Analyze 按钮 — 执行分析
- 执行流程：
    - 后端接收 CSV + Python 脚本
    - 用 child_process.execFile 调用本地 Python 执行脚本
    - Python 脚本读取 CSV，输出 HTML 结果
    - 结果存入数据库并展示在页面
- Python脚本约定：
    - 第一个参数：CSV 文件路径
    - 第二个参数：结果 HTML 输出路径
    - 脚本将 HTML 写入输出文件，或直接输出 HTML 到 stdout
- 使用方式：
    - 在 Analytical Tool 页面上传 CSV 文件 + 这个 Python 脚本，点击 Analyze 即可看到HTML分析报告

### 分析工具主页优化
&#8195;&#8195;需求：打算在Analytical Tool页面上放置多个工具，工具分类：Analysis Tool、Monitor Tool、Inspect Tool、Backup&Restore Tool、Deploy Tool、Operation Tool、Optimize Tool、Network Tool、Security Tool、Debug Tool，Analytical Tool主页设计对这些工具分类创建图标，点击图标到对应的主页，刚才的用python分析csv的那个放到Analysis Tool分类里面，分析结果用弹窗进行展示，并且提供下载。

编译通过。Analytical Tool页面重构完成。主页 — 10 个工具分类卡片，每个有独立图标和渐变色：

分类|图标|路由
:---|:---|:---
Analysis Tool    |bi-graph-up-arrow |/tool-analysis
Monitor Tool     |bi-activity       |/tool-monitor
Monitor Tool     |bi-activity       |/tool-monitor  
Inspect Tool     |bi-search-heart   |/tool-inspect  
Backup & Restore |bi-cloud-arrow-up |/tool-backup   
Deploy Tool      |bi-rocket-takeoff |/tool-deploy   
Operation Tool   |bi-sliders        |/tool-operation
Optimize Tool    |bi-speedometer    |/tool-optimize 
Network Tool     |bi-diagram-3      |/tool-network  
Security Tool    |bi-shield-lock    |/tool-security 
Debug Tool       |bi-bug            |/tool-debug  

Analysis Tool页面 — CSV分析功能完整：
- 上传CSV + Python脚本 → 点击 Analyze
- 结果用弹窗展示（85% 宽度，带滚动）
- 弹窗右上角 Download HTML 按钮下载报告
- 历史记录列表，支持查看和删除

其他9个分类 — 占位页，带返回按钮和 "Coming soon" 提示，后续可逐个开发。

&#8195;&#8195;需求：Analysis Tool点进去在帮我再弄一个分类，例如Performance Analysis、Log Analysis、Network Analysis、Data Analysis、Config Analysis、Root Cause Analysis、Capacity Analysis几个类型，刚才用python分析csv的那个放到Performance Analysis分类里面，并也做一个图标，叫SVC Performance Analysis，点进去就是分析工具主页。

Analytical Tool 三级导航结构：
- 第一层 — Analytical Tool 主页（10 个工具图标）
- 第二层 — Analysis Tool 页面（7 个分析分类图标）：  - Performance Analysis ← 已实现
    - Log Analysis / Network Analysis / Data Analysis / Config Analysis / Root Cause Analysis / Capacity  Analysis（占位）                                                                                 
- 第三层 — Performance Analysis 页面：
    - SVC Performance Analysis 工具卡片，上传 CSV 即可分析（内置 SVC-IOGRP-Perf.py 脚本）
    - 结果弹窗展示 + Download HTML 按钮
    - 历史记录列表

路径：Analytical Tools → Analysis Tool → Performance Analysis → SVC Performance Analysis

## V2.4
### 后端优化
重启电脑后数据全没了，项目也打不开，进行了修复，以后重装系统或换机器，只需要：
- 装 MySQL 8.4
- 放入 my.ini
- mysqld --initialize-insecure
- 注册服务
- 导入 schema.sql + seed.sql

### 内容优化
进行了问题修复及页面优化：
- Analytical Tools页面有点大了，点开后覆盖了左边导航。问题原因：
  - 内层 el-container 没有明确设置 flex: 1，导致它无法正确计算宽度
  - 侧边栏没有设置 flex-shrink: 0，在 flex 布局中可能被压缩
- 分离图标里面Web Server和3 Systems改成靠左，看起来协调一点，和下面状态图标对齐
- 左侧导航在滚动右边滚动条后，下面有一段白色区域，不好看修复下
- 左侧导航不要弄滚动条，随页面右边滚动进行滚动
- 左侧导航展开后被页面挡住了：问题原因：之前侧边栏写死了 width: 60px，展开菜单时文字溢出被内容区遮挡。现在侧边栏宽度随状态变化，右侧content-wrapper用了flex: 1，会自动适应剩余空间
- 左侧导航展开后，下面的看不到了，帮我改成可以滑动翻到下面去

### 功能优化
&#8195;&#8195;需求：准备添加AIX系统，输入账户密码后Test Connection提示Real-time collection not supported for host type: AIX。帮我添加AIX支持，AIX也定制巡检脚本，根据系统类型不一样，放置不一样的脚本，点击生成巡检报告。Linux的不要删了

修改完成！修改总结
-  server/services/perfCollector.js
  - 添加了 collectAIX() 函数，用于采集AIX主机的实时性能数据
  - 支持 AIX 特有命令：svmon, lparstat, iostat, lsdev, lsconf 等  - 导出 collectAIX 函数
- server/routes/perf-realtime.js
  - 导入 collectAIX 函数
  - 添加 AIX 主机类型判断，支持 AIX 实时性能采集
- server/services/inspection.js
  - 重命名为 LINUX_SCRIPT_NAME 和 LINUX_INSPECTION_SCRIPT
  - 添加了 AIX_SCRIPT_NAME 和 AIX_INSPECTION_SCRIPT（AIX 巡检脚本）
  - 添加了 getInspectionConfig() 函数，根据系统类型返回对应配置
  - 添加了 runAIXInspection() 函数，执行 AIX 巡检命令
  - 修改 runInspection() 支持 hostType 参数
- server/routes/reports.js
  - 导入 getInspectionConfig 替代原来的 SCRIPT_NAME
  - 添加 hostType 查询参数支持
  - 移除 "Only Linux hosts are supported" 限制
  - 将 host_type 传递给 runInspection() 和 generateReport()
- server/services/reportGenerator.js
  - 更新报告模板支持 Linux 和 AIX
  - AIX 报告包含 LPAR、IO Stats、System Config、Paging Space 等 AIX 特有信息
  - 标题和元数据显示系统类型

现在系统支持：
- Linux: 完整的 CPU、内存、磁盘、网络采集
- AIX: 完整的 CPU、内存（svmon）、磁盘（df -g）、网络、LPAR、IO Stats 采集
- 两个系统都可以生成巡检报告

&#8195;&#8195;需求：AIX系统生成的报告不在Reports List里面显示，改一下。在AIX系统主页，Overview显示Host Information的条目还是linux，帮我参照AIXtest1页面改条目，除了几个关键信息是预填入的，其他信息通过Refresh获取，并且保留显示最后一次获取的值。

修复AIX报告不在Reports List中显示的问题
- server/routes/reports.js  - 生成报告时，目录名改用数据库中的host.host_name，而不是SSH获取的data.systemInfo.hostname
- 这样报告目录名与列表查询时使用的目录名一致

修复AIX系统主页Host Information显示Linux条目的问题
- src/components/hostpage/Dashboard.vue
- Host Information现在根据host.HostType显示不同条目：
  - Linux: SELinux、CPU Cores、Memory Size、Swap Size
  - AIX: LPAR、Memory Size、Paging Space、IO Stats
  - 公共字段（OS Version、Kernel、HostName、IP、Architecture、Status、Category、Data Center、Uptime、Load
  Average、Description）始终显示
  - sysInfo 数据结构添加了 AIX 特有字段：lpar、pagingSpace、ioStats
  - Refresh 时会正确获取并缓存 AIX 特有数据

修复AutoPM巡检脚本显示问题：
- src/components/hostpage/AutoPM.vue
  - 点击 "Inspection Script" 按钮时，根据主机类型加载对应的脚本
  - AIX 显示 AIX_Inspection.sh，Linux 显示 Linux_RHEL8.sh

&#8195;&#8195;需求：AIX除了基础的HostType、HostName、IP Addr、Status、Category、BusinessName、DataCenter项外，AIX从系统获取信息还有：Machine Type 、Serial Number、Platform Firmware Level、AIX Level、CPU Entitled Capacity、Memory Size、和Page Space Size ，这些给我展示在系统主页Host Information里面，一共14项，其他的都不需要了，prtconf、lparstat、uname -Mu、oslevel -s、lsps等命令获取这些信息

后端server/services/perfCollector.js
- AIX性能采集现在使用以下命令获取信息：  
  - prtconf → Machine Type、Serial Number、Platform Firmware Level、Memory Size  
  - uname -Mu → 备用获取 Machine Type 和 Serial Number
  - oslevel -s → AIX Level
  - lparstat → CPU Entitled Capacity
  - lsps -a → Page Space Size
  - svmon → Memory Size

前端src/components/hostpage/Dashboard.vue。AIX系统主页Host Information现在显示14项：

字段|来源
:---|:---
Host Type|数据库
Host Name|数据库
IP Address|数据库
Status|数据库
Category|数据库
Description|数据库
Data Center|数据库
Machine Type|prtconf
Serial Number|prtconf/uname
Platform Firmware Level|prtconf
AIX Level|oslevel -s
CPU Entitled Capacity|lparstat
Memory Size|prtconf/svmon
Page Space Size|lsps

### AIX snap日志分析
&#8195;&#8195;需求：在shm目录下放置一个名为1snap.pax.Z的文件，这是AIX系统snam -gc收集的系统日志，能帮我写一个工具，分析这个日志，生成分析报告，Python或shell分析都行，把脚本给我，我回头加到分析工具里面去进行测试 

分析结果
  - Node: sapecp01
  - Partition: 740A_1  - AIX Level: 7100-00-10-1334
  - CPUs: 16 VCPUs, 16.00 Entitled  - Memory: 127744 MB (128GB)
  - Error Events: 312 total (298 PERM, 5 TEMP, 9 INFO)
  - Disks: 17

报告已生成：D:\AI\shm\1snap_report.html

脚本位置：D:\AI\shm\tools\snap_analyzer.py

使用方法：
  1. 先解压 .Z 文件：uncompress 1snap.pax.Z（在 Git Bash 中）
  2. 运行分析：python tools/snap_analyzer.py 1snap.pax
  3. 查看生成的 HTML 报告

报告包含：
  - 系统信息（Node、Partition、OS Level、CPU、Memory）
  - 错误日志汇总（按类型统计、Top 错误标签、最近 30 条错误）
  - 磁盘设备列表
  - HTML 格式，可以直接在浏览器中查看

此方法暂未添加

### 系统功能添加
&#8195;&#8195;需求：每个系统规划了一个Standard Check页面，描述为System Online Standardized Check，这个我想改造下。改成从Script Library里面添加脚本，弄一个Add Script按钮，点击后从Script Library根据分类一层一层选择脚本，添加在Standard Check页面后以列表的形式展示，第一列是Name(脚本名称)，第二列是Readme，弄一个按钮，点击查看的是脚本上传的markdown内容，第三列是Script，同样东一个按钮，叫View Script，点击查看脚本内容，第四列是Description，从Script Library添加脚本时候写个简单描述，第五列是功能按钮，分三个，四个按钮是Run，点击后在当前系统运行脚本，结果用弹窗展示，第二个按钮是History，点击显示脚本执行结果的历史记录，第三个是Delete，点击就删除此脚本。

Standard Check改造完成！功能说明
- Add Script 按钮 — 点击后弹窗，从Script Library 按 System > Type > Script 三层选择脚本，可填写简要描述
- Run — 通过 SSH 在目标主机执行脚本，弹窗显示执行结果
- History — 显示脚本执行历史记录，可点击查看每次执行的输出
- Delete — 从当前主机移除该脚本关联

脚本列表表格：

列|说明
:---|:---
Name|脚本名称（来自 Script Library）
Readme|查看按钮，弹窗显示 markdown 渲染内容
Script|View Script 按钮，弹窗显示脚本代码（语法高亮）
Description|添加时填写的描述
Actions|Run/History/Delete三个按钮

新增的数据库表
- host_scripts — 主机与脚本的关联关系
- script_run_history — 脚本执行历史记录

新增的后端 API
  - GET /api/standard-check/:hostId — 获取主机关联的脚本
  - POST /api/standard-check/:hostId — 添加脚本关联
  - DELETE /api/standard-check/:hostId/:scriptId — 删除关联
  - POST /api/standard-check/:hostId/:scriptId/run — 执行脚本
  - GET /api/standard-check/:hostId/:scriptId/history — 查看执行历史

## 待补充