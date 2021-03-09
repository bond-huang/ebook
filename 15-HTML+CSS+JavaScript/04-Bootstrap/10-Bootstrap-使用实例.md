# Bootstrap-使用实例
使用过程中遇到的一些需求，实现方法记录下来。
## 模态框相关
### 模态框&提示工具混用
使用示例代码如下：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>模态框&提示工具混用</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.4.0/font/bootstrap-icons.css">
    <script src="https://cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
    <style type="text/css">
    .deleteicon i {
        color: rgba(255, 255, 255, 0);
    }
    .deleteicon i:hover {
        color: red;
    }
    </style>
</head>
<body>
<div style="padding: 100px 100px 10px;">
    <p>垃圾桶：<a id="modal" href="#deleteModal" role="button" data-toggle="modal" class="tooltip-show deleteicon" title="Delete this record">
    <i class="bi bi-trash"></i></a></p>
</div>
<!-- 删除模态框（Modal） -->
<div class="modal fade" id="deleteModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content ">
            <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal" aria-hidden="true">
                &times;</button>
            <h4 class="modal-title" id="myModalLabel">No infomation</h4>
            </div>
            <div class="modal-body">
                Infomation is adding... 
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-primary" data-dismiss="modal">Close
                </button>
            </div>
        </div>
    </div>
</div>
</body>
</html>
```
示例说明：
- 示例中，使用CSS样式将图标隐藏，使用`:hover`选择器设置鼠标指向的颜色是红色
- 鼠标指向图标后不仅显示了，还弹出提示框
- 点击图标，就会弹出对应的模态框
- 提示工具可以结合JavaScript做出更多样式或者效果

### 向模态框传入数据
代码示例如下：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>向模态框传入数据</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css">
    <script src="https://cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<div style="padding: 100px 100px 10px;">
    <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#exampleModal" data-whatever="@delete">Delele</button>
    <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#exampleModal" data-whatever="@Share">Share</button>
    <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#exampleModal" data-whatever="@github">GitHub</button>
</div>
<!-- 模态框（Modal） -->
<div class="modal fade" id="exampleModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel">
    <div class="modal-dialog" role="document">
        <div class="modal-content">
        <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
            <h4 class="modal-title" id="exampleModalLabel">New message</h4>
        </div>
        <div class="modal-body">
        <form>
            <div class="form-group">
                <label for="recipient-name" class="control-label">Recipient:</label>
                <input type="text" class="form-control" id="recipient-name">
            </div>
            <div class="form-group">
                <label for="message-text" class="control-label">Message:</label>
                <textarea class="form-control" id="message-text"></textarea>
            </div>
        </form>
        </div>
        <div class="modal-footer">
            <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
            <button type="button" class="btn btn-primary">Send message</button>
        </div>
        </div>
    </div>
</div>
<script>
    $('#exampleModal').on('show.bs.modal', function (event) {
        var button = $(event.relatedTarget) // Button that triggered the modal
        var recipient = button.data('whatever') // Extract information from data-* attributes
        var modal = $(this)
        modal.find('.modal-title').text('New message to ' + recipient)
        modal.find('.modal-body input').val(recipient)
    })
</script>
</body>
</html>
```
示例说明:
- 示例中，`exampleModal`可以自定义，按钮中、模态框中及JS中要对应一致
- 示例中，`data-whatever="@github"`中的`whatever`可以自定义，同样需要和JS中对应
- 如果需要传入多个数据，可以继续加`data`属性,例如`data-test="license"`，需要在JS中增加对应内容：`var test = button.data('test')`
- 通过jinja2传入了一个数据，是从一个字典对象通过key截取而来的数据，尝试直接传入字典对象，显示的只是个对象

参考及详细用法说明链接：[https://v3.bootcss.com/javascript/#modals-related-target](https://v3.bootcss.com/javascript/#modals-related-target)

### 以上示例结合到一起
要求效果就是：
- 是一个图标，例如垃圾桶图标
- 图标不显示，鼠标指向就显示出来并且指定颜色，例如红色
- 鼠标指向图标后出现提示工具提示相应内容
- 点击图标后，弹出对应的模态框
- 弹出的模态框中，有通过图标那边传入的数据

满足以上需求的图标HTML写法示例如下（其它全省略参考上面示例）：
```html
<a id="modal" href="#deleteModal" role="button" class="tooltip-show deleteicon" title="Delete this record" data-toggle="modal" data-whatever="{{ link['id'] }}">
    <i class="bi bi-trash"></i>
</a>
```
## 待补充
