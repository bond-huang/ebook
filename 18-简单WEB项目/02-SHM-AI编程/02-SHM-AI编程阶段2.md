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

## 待补充