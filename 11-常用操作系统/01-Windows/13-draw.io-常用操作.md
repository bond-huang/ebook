# draw.io-常用操作
特点：
- 完全开源免费，无水印；
- 网页直接打开，也有Windows/Linux/mac桌面离线客户端；
- 支持私有化部署；
- 内置大量IT图标：服务器、存储 V7000、防火墙、密码机、负载均衡、K8s、云图标；导出 PNG/SVG/PDF 矢量图，
- 能打开Visio文件

相关参考链接：
- 官方主页：[https://www.drawio.com/](https://www.drawio.com/)
- 下载地址：[https://github.com/jgraph/drawio-desktop/releases/tag/v31.4.4](https://github.com/jgraph/drawio-desktop/releases/tag/v31.4.4)
- 开源图标下载地址1：[https://github.com/JF-Dumont/drawio-libs](https://github.com/JF-Dumont/drawio-libs)
- 开源图标下载地址2：[https://gitcode.com/gh_mirrors/dr/drawio-libs](https://gitcode.com/gh_mirrors/dr/drawio-libs)
- 图标下载：[https://icons.diagrams.net/](https://icons.diagrams.net/)
- 腾讯云图标：[https://cloud.tencent.com/act/event/icons](https://cloud.tencent.com/act/event/icons)

## draw.io图标
### 图标转换
&#8195;&#8195;`.vss`：Visio旧版二进制形状库（模具/符号库stencil），不是完整绘图图纸，里面存一堆硬件图标（存储、服务器、网络设备图标）。区分后缀：
- `.vsd`：旧Visio绘图图纸
- `.vss`：图标模具库（一堆设备图标）
- `.vsdx`：新版图纸，`draw.io`原生直接支持打开
- `.vssx`：新版图标库

vss下载及转换工具：
- vss图标下载地址：[https://www.visiocafe.com](https://www.visiocafe.com)
- 在线转换工具：[https://vss.draw.io/](https://vss.draw.io/)

### 图标导入
从vss.draw.io下载得到的xml库文件导入步骤如下：
- 点击draw.io 菜单：`File → Open Library from → Device`（文件 → 从设备打开库）
- 成功后左侧有分类，展开后可以看到图标

## 画布
### 容器
#### 容器添加
添加容器方法：
- 左侧点【更多图形】
- 拉到下面找到`C4 Model`，勾选启用
- C4模型里面有`Container`，就是架构图标准容器框，拖入后把服务器丢进去，移动容器，里面所有服务器跟着一起移动

## 常见问题
### 画布问题
#### 整体非常卡
&#8195;&#8195;通常是由于元素太多了，本次我弄一个存储，只有一个框，里面加入了多个硬盘柜，组合后再复制，由于每块硬盘也是个svg，导致整体元素非常多，解决办法：
- 将后期作为一个整体的图标全选中（例如我这次的存储，几百个SVG），进行组合
- 组合后右键，复制为SVG，然后粘贴，或直接加入到新的库里面去
- 或者保存为PNG，保存在本地，然后新建一个库统一加入
- 这样一台存储就是一个SVG了，有些服务器也是，服务器框架和里面硬盘都是单独的SVG，建议组合

参数优化也有点用，找到运行程序，右键属性，目标后面加参数，示例：
```
"C:\Program Files\draw.io\draw.io.exe" --disable-acceleration --disable-spell-check --disable-update
```
参数说明：
- `--disable-acceleration`：关闭GPU硬件加速，避免复杂图形渲染冲突
- `--disable-spell-check`：禁用拼写检查
- `--disable-update`：关闭自动更新检查

参考链接：[解决Drawio桌面版超大文件卡顿：从根源优化到实战方案](https://blog.csdn.net/gitblog_00768/article/details/151422689)

## 待补充