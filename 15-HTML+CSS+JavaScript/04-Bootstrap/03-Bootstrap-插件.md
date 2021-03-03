# Bootstrap-插件
学习过程中记录的笔记，学习教程及参考链接：
- [runoob.com Bootstrap教程](https://www.runoob.com/bootstrap/bootstrap-environment-setup.html)
- [bootcss.com Bootstrap教程](https://v3.bootcss.com/getting-started/)
- [Bootstrap官方教程](https://getbootstrap.com/docs/5.0/getting-started/download/)
- [bootcss.com 图标使用](https://icons.bootcss.com/)

使用在线工具：
- [HTML/CSS/JS 在线工具](https://c.runoob.com/front-end/61)
- [Bootstrap可视化布局系统](https://www.bootcss.com/p/layoutit/)

## 模态框
&#8195;&#8195;模态框（Modal）是覆盖在父窗体上的子窗体。目的是显示来自一个单独的源的内容，可以在不离开父窗体的情况下有一些互动，子窗体可提供信息、交互等。
### 简单模态框
示例代码如下：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>模态框示例</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.4.0/font/bootstrap-icons.css">
    <script src="https://cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
    <!-- 按钮触发模态框 -->
    <button class="btn btn-primary btn-lg" data-toggle="modal" data-target="#myModal">
        <i class="bi bi-share-fill"></i>&nbsp;&nbsp;Share
    </button>
    <!-- 模态框（Modal） -->
    <div class="modal fade" id="myModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal" aria-hidden="true">
                &times;</button>
            <h4 class="modal-title" id="myModalLabel">Share this website</h4>
            </div>
            <div class="modal-body">
                <p>Share link: </p>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default" data-dismiss="modal">Close
                </button>
            </div>
        </div>
    </div>
    </div>
</body>
</html>
```
示例中采用button按钮方式出发，同样，可以采用链接方式进行触发,将button标签修改为：
```html
<a id="modal" href="#myModal" role="button" class="btn btn-primary" data-toggle="modal">
<i class="bi bi-share-fill"></i>&nbsp;&nbsp;Share</a>
```
示例说明：
- 使用按钮和链接方式几乎一样，除了样式有所差异
- 在&#60;button>标签中，`data-target="#myModal"`是想要在页面上加载的模态框的目标
- 在&#60;a>标签中，`href="#myModal"`是想要在页面上加载的模态框的目标
- 在`class="modal fade"`中：
    - `.modal`，用来把&#60;div> 的内容识别为模态框
    - `.fade`类，当模态框被切换时，它会引起内容淡入淡出，不想要删掉即可
- 属性`aria-labelledby="myModalLabel"`：引用模态框的标题
- 属性`aria-hidden="true"`：用于保持模态窗口不可见，直到触发器被触发为止
- &#60;div class="modal-header">中`modal-header`是为模态窗口的头部定义样式的类
- `class="close"`，`close`是一个CSS class，用于为模态窗口的关闭按钮设置样式
- `data-dismiss="modal"`是一个自定义的HTML5 data属性,在这里用于关闭模态窗口
- `class="modal-body"`是Bootstrap的一个CSS class，用于为模态窗口的主体设置样式
- `class="modal-footer"`是Bootstrap的一个CSS class，用于为模态窗口的底部设置样式
- `data-toggle="modal"`是HTML5自定义的data 属性`data-toggle`用于打开模态窗口

### 数据交互模态框
示例代码如下：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>模态框示例</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.4.0/font/bootstrap-icons.css">
    <script src="https://cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
    <!-- 按钮触发模态框 -->
    <a id="modal" href="#myModal" role="button" class="btn btn-primary" data-toggle="modal">
    <i class="bi bi-plus-circle"></i>&nbsp;&nbsp;Add</a>
    <!-- 模态框（Modal） -->
    <div class="modal fade" id="myModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal" aria-hidden="true">
                &times;</button>
            <h4 class="modal-title" id="myModalLabel">Add Data</h4>
            </div>
            <div class="modal-body">
                <div class="form-group">
                <label for="txt_classname">Class Name</label>
                <input type="text" name="txt_classname" class="form-control" id="txt_classname" placeholder="Class Name">
                </div>
                <div class="form-group">
                <label for="txt_typename">Type Name</label>
                <input type="text" name="txt_typename" class="form-control" id="txt_typename" placeholder="Type Name">
                </div>
                <div class="form-group">
                <label for="txt_name">Name</label>
                <input type="text" name="txt_name" class="form-control" id="txt_name" placeholder="Name">
                </div>
                <div class="form-group">
                <label for="txt_location">Location</label>
                <input type="text" name="txt_location" class="form-control" id="txt_location" placeholder="Location">
                </div>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default" data-dismiss="modal">Close
                </button>
                <button type="button" class="btn btn-primary" data-dismiss="modal">
                <span class="glyphicon glyphicon-floppy-disk" aria-hidden="true"></span>&nbsp;&nbsp;Save
                </button>
            </div>
        </div>
    </div>
    </div>
</body>
</html>
```
