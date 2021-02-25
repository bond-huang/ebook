# OpenShift-应用部署
&#8195;&#8195;OpenShift是由红帽（Red Hat）开发的容器化软件解决方案，这是基于企业级 Kubernetes 管理的平台即服务（PaaS）。OpenShift支持通过Operators，通过Git使用S2I，使用Dockerfile，直接使用Image Registry的镜像等方式部署应用。

学习过程中整理的笔记，内容来自IBM在线实验室发布的学习链接：
- [在OpenShift平台上部署应用](https://csc.cn.ibm.com/labs/lab/062e8b38-74e9-43e8-8d48-85932d4317b1/ed635533-3627-4ce9-b22b-8b1163ab2286#fwaj-tab2)
- [OpenShift开发应用平台自动化部署](https://csc.cn.ibm.com/labs/lab/15b1cafe-3e66-4fa2-aae6-01eb02b8079f/3bf93d20-86d8-4400-968f-c496cbe9f8ef)
- [OpenShift容器平台高权限自由体验](https://csc.cn.ibm.com/labs/lab/6eb5b25b-7faa-4570-ab59-e6c9f3e5e6d7/d254e967-2644-43e5-902c-1adf37f26b86)
## 部署Git服务器
&#8195;&#8195;故事背景：某企业准备启动一个新的项目，需要为该项目准备一套环境供开发、测试使用。基于安全的考虑，需要使该项目独立于已有的产品环境。目前该项目已经在OpenShift平台上创建，接下来在这个平台上快速部署服务以支持项目的开发。

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

## 部署NodeJS MongoDB应用
&#8195;&#8195;故事背景：开发人员已经完成了该项目中的一个NodeJS应用，并且提交到了git服务器上，现在需要部署该应用到OpenShift环境供测试，该应用在git服务器上的位置是`/git/nodejs.git`。
