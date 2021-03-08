# Bootstrap-使用笔记
使用过程中遇到的一些需求，实现方法记录下来。
## 模态框&提示工具混用
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

## 待补充
