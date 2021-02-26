# OpenShift-应用部署
&#8195;&#8195;OpenShift是由红帽（Red Hat）开发的容器化软件解决方案，这是基于企业级 Kubernetes 管理的平台即服务（PaaS）。OpenShift支持通过Operators，通过Git使用S2I，使用Dockerfile，直接使用Image Registry的镜像等方式部署应用。

学习过程中整理的笔记，内容来自IBM在线实验室发布的学习链接：
- [在OpenShift平台上部署应用](https://csc.cn.ibm.com/labs/lab/062e8b38-74e9-43e8-8d48-85932d4317b1/ed635533-3627-4ce9-b22b-8b1163ab2286#fwaj-tab2)
- [OpenShift开发应用平台自动化部署](https://csc.cn.ibm.com/labs/lab/15b1cafe-3e66-4fa2-aae6-01eb02b8079f/3bf93d20-86d8-4400-968f-c496cbe9f8ef)
- [OpenShift容器平台高权限自由体验](https://csc.cn.ibm.com/labs/lab/6eb5b25b-7faa-4570-ab59-e6c9f3e5e6d7/d254e967-2644-43e5-902c-1adf37f26b86)

## 部署Git服务器
&#8195;&#8195;部署背景：某企业准备启动一个新的项目，需要为该项目准备一套环境供开发、测试使用。基于安全的考虑，需要使该项目独立于已有的产品环境。目前该项目已经在OpenShift平台上创建，接下来在这个平台上快速部署服务以支持项目的开发。

### 查看项目
操作步骤：
- 登录OpenShift图形界面
- 在左侧导航栏，选择`Projects`，在右侧列表可以看到一个项目(PR)`cscdemoXX`
- 点击`cscdemoXX`项目后，通过`Overview`查看项目的概要信息
- 点击`Workloads`可以查看项目中部署的应用情况，目前为空

### 部署Git服务器
操作步骤：
- 点击左侧导航栏顶部的`Administrator`下拉菜单，切换到`Developer`视图
- 在右侧的部署方式中选择`From Catalog`
- 在可部署的项目中选择`Git Server (Ephemeral)`
- 点击`Instantiate Template`准备部署
- 在`Application Hostname`中填入`git-(Project Name).apps.kvm-ocp.example.com`
- 确认正确后点击`Create`开始部署
- 点击左侧导航栏的`Events`可以查看部署过程中的重要事件

### 查看部署的Git服务器
操作步骤：
- 在左侧导航栏顶部的从`Developer`切换到`Administrator`视图
- 点击`Projects`后在右侧点击`cscdemoXX`项目
- 点击`Workloads`打开该选项卡
- 点击`git-server-example`下面列出的DC项目可以看到刚刚部署的Git服务器应用信息
- 点击对应DC项目，然后点击`Resources`选项卡，确认`Pods`组里的`P`资源是`Running`状态，表示部署成功
- 在`Resources`最下部可以看到`Routes`组信息，其中`Location`即是我们刚刚部署时输入的`Application Hostname`
- 点击`Location`里面链接打开页面，可以看到`403 Forbidden`的页面，表示正常工作

## 部署Node.JS MongoDB应用
&#8195;&#8195;部署背景：开发人员已经完成了该项目中的一个NodeJS应用，并且提交到了git服务器上，现在需要部署该应用到OpenShift环境供测试，该应用在git服务器上的位置是`/git/nodejs.git`。  
### 部署Node.JS MongoDB
操作步骤：
- 在左侧导航栏顶部的从`Administrator`切换到`Developer`视图
- 点击`+Add`菜单项，在右侧的部署方式中选择`From Catalog`
- 选中`TYPE`下的`Template`过滤条件，仅显示模板项目
- 在可部署的项目中选择`Node.js + MongoDB (Ephemeral)`
- 点击`Instantiate Template`准备部署
- 在`Git Repository URL`中填入`http://git-(Project Name).apps.kvm-ocp.example.com/git/nodejs.git`
- 确认正确后点击`Create`开始部署

### 查看部署的Node.JS MongoDB
操作步骤：
- 在左侧导航栏顶部的从`Developer`切换到`Administrator`视图
- 点击`Projects`后在右侧点击`cscdemoXX`项目
- 点击`Workloads`打开该选项卡
- 点击`(DC) nodejs-mongodb-example`项目可以看到刚刚部署的`Node.JS`应用信息
- 点击`Resources`选项卡，可以看到`Builds`组下面`Build`任务的运行状态
- 点击`Build`任务后面的`View Logs`可以查看`Build`任务的运行日志
- 几分钟后，最后一行将出现`Push successful`表示成功；
- 返回到`Resources`选项卡页，确认`Pods`组里的`P`资源是`Running`状态
- 在资源的最下部可以看到`Routes`组信息，点击`Location`中的链接，确认页面最上方出现“Welcome to your Node.js....”表示部署成功

## 部署Java MySQL应用
&#8195;&#8195;部署背景：开发人员已经完成了该项目中的一个Java应用，并且提交到了我们的git服务器上，现在需要部署该应用到OpenShift环境供测试，该应用在git服务器上的位置是`/git/java.git`。
### 部署Java MySQL
操作步骤：
- 在左侧导航栏顶部的从`Administrator`切换到`Developer`视图
- 点击`+Add`菜单项，在右侧的部署方式中选择`From Catalog`
- 选中`TYPE`下的`Template`过滤条件，仅显示模板项目
- 在可部署的项目中选择`OpenJDK + MySQL (Ephemeral)`
- 点击`Instantiate Template`准备部署，
- 在`Git Repository URL`中填入`http://git-(Project Name).apps.kvm-ocp.example.com/git/java.gitCopy to clipboard`
- 将`Context Directory`项内容清空
- 确认正确后，点击`Create`开始部署

### 查看部署的Java MySQL
操作步骤：
- 在左侧导航栏顶部的从`Developer`切换到`Administrator`视图
- 点击`Projects`后在右侧点击`cscdemoXX`项目
- 点击`Workloads`打开该选项卡
- 点击`(DC) openjdk-app-mysql`项目可以看到刚刚部署的Java应用信息
- 点击`Resources`选项卡，可以看到`Builds`组下面`Build`任务的运行状态
- 点击`Build`任务后面的`View Logs`可以查看`Build`任务的运行日志
- 几分钟后，最后一行将出现`Push successful`，表示成功
- 返回到`Resources`选项卡页，确认`Pods`组里的`P`资源是`Running`状态
- 在资源的最下部可以看到`Routes`组信息，点击`Location`中给出的链接，确认页面最上方出现“Number for user 'NONE' is not found”表示部署成功

## 代码更改后的自动化部署
&#8195;&#8195;实验背景：因为代码的快速迭代，为了加快开发速度，希望可以在代码提交后，可以自动实现新代码的部署而不需要人工干预。在 OpenShift 中只需要通过Webhook的方式触发这一功能，下面的实验是用来演示如何获得Webhook的URL及自动部署的过程。   
操作步骤如下：
- 点击桌面左上方`Applcations`，选择`Terminal Emulator`启动终端窗口
- 并运行如下命令来克隆java代码库，并确认克隆成功。
    ```
    ls
    git clone http://git-(Project Name).apps.kvm-ocp.example.com/git/java.git
    ls
    cd java
    ```
- 回到OpenShift的`Administrator`界面中
- 在左侧导航栏选择`Builds`下面的`Build Configs`项
- 在右侧页面中点击`openjdk-app-mysql`项目
- 在页面的底部可以看到`Webhooks`组
- 点击`Generic`的`Copy URL with Secret`复制Webhook的URL
- 回到终端窗口，用`vi`创建`webhook`文件，并将拷贝的`URL`粘贴在文件中保存退出
- 再用`vi`修改`java`源文件`src/main/java/org/openshift/quickstarts/undertow/servlet/PhoneBookServlet.java`文件
    ```
    vi src/main/java/org/openshift/quickstarts/undertow/servlet/PhoneBookServlet.java
    将第 87 行的
    name = "NONE";
    改为
    name = "NULL";
    ```
- 修改完成后，运行以下命令提交修改:
    ```
    git add .
    git commit -m "Test auto build"
    git push
    ```
- 如果提示用户名和密码，请输入用户名：git，密码：demo
- 返回OpenShift的`Administrator`界面
- 选择`Projects`下的cscdemoXX
- 打开`Workloads`选项卡
- 选择`(DC) openjdk-app-mysql`后
- 打开`Resources`选项卡，查看`Builds`组下面的任务项，这时会看到有一个新的`Build`任务在运行
- 待其运行完成后，`Pods`组里的`Running`状态项目会被停止并重新生成新的项目
- 当新项目变成`Running`状态后，点击底部的`Routes`组的`Location`链接，页面输出已经从
`Number for user 'NONE' is not found`变成了`Number for user 'NULL' is not found`

## 部署Python
再次在OpenShift上部署应用进行学习，这次选择部署Python。操作步骤如下：
- 登录OpenShift图形界面，进入到`Developer`视图
- 在`Topology`点击`Samples`，选择希望部署的应用，此处我选择的Python
- 如果初次部署应用，直接点击`Create`即可开始部署
- 部署开始后，一般需要几分钟，可以看到圆形的应用，点击`圆形`图标可以查看详情
- 当`Builds #1`状态由`running`变为`completed`后，表示部署完成
- 点击`圆形`图标右上角的`Open URL`，可以打开应用的`Welcome`页面，至此该应用部署成功
- 接下来，可以通过左导航的`+Add`，然后在右侧点击`Samples`，继续添加其他应用

## Build失败
记录下bulid失败几种情况。
### Git Repository URL填写错误
部署应用的步骤不多，最容易出现错误的就是`Git Repository URL`写错了。

日志查看步骤：
- 在左侧导航栏顶部选择`Administrator`视图
- 点击`Projects`后在右侧点击`cscdemoXX`项目
- 点击`Workloads`打开该选项卡
- 点击`(DC) nodejs-mongodb-example`项目可以看到刚刚部署的`Node.JS`应用信息
- 点击`Build`任务后面的`View Logs`可以查看`Build`任务的运行日志

修改配置步骤：
- 点击右上角`Action`展开下拉菜单
- 选择`Edit Build Config`选项
- 在`YAML`选项中会显示配置文件，找到`type:Git`,下面`uri`中有之前输入的Git Repository URL
- 检查修改成正确地址后，点击左下角`save`保存
- 点击右上角`Action`展开下拉菜单
- 选择`Start Build`选项重新开始build
- 通过上面的查看步骤查看状态

## 红帽官方链接
更多操作参考红帽官方：[GETTING STARTED GUIDE](https://access.redhat.com/documentation/en-us/red_hat_codeready_containers/1.18/html/getting_started_guide/index?_ga=2.142747381.190581808.1613978048-540765612.1577072940)
