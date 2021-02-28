# Bootstrap-下载及使用
学习过程中记录的笔记，学习教程及参考链接：
- [runoob.com Bootstrap教程](https://www.runoob.com/bootstrap/bootstrap-environment-setup.html)
- [bootcss.com Bootstrap教程](https://v3.bootcss.com/getting-started/)
- [bootcss.com Bootstrap教程](https://v3.bootcss.com/getting-started/)
- [Bootstrap官方教程](https://getbootstrap.com/docs/5.0/getting-started/download/)
- [bootcss.com 图标使用](https://icons.bootcss.com/)

使用在线工具：
- [HTML/CSS/JS 在线工具](https://c.runoob.com/front-end/61)
- [Bootstrap可视化布局系统](https://www.bootcss.com/p/layoutit/)

## 导航
### 示例一
根据教程写了一个简单网页头部导航：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>WEB导航器</title>
    <link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.4.0/font/bootstrap-icons.css">
    <script src="https://cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body style="background: #f0f0f0">
    <nav class="navbar navbar-default navbar-static-top" role="navigation">
    <div class="container-fluid"> 
       <div class="navbar-header">
            <a class="navbar-brand">
            <i class="bi bi-cursor-fill" style="font-size: 3rem; color: blue;"></i>
            </a>
        </div>
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
            <ul class="nav navbar-nav">                    
                <li class="active">
                    <a href="#">网站导航器</a>
                </li>
            </ul>
            <a class="btn pull-right js-toolbar-action" href="https://github.com/bond-huang/ebook">
                <i class="bi bi-github"></i>
            </a>
        </div>
    </div>
    </nav>
</body>
</html>
```
说明：
- 前面是一个导航图标：bi-cursor-fill
- 然后是网页名称，用列表形式做的
- 最后是github图标，在导航栏最右边，点击转向指定网站

### 示例二
同样是横向的导航，增加了一个搜索框：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>WEB导航器</title>
    <link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.4.0/font/bootstrap-icons.css">
    <script src="https://cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
    <nav class="navbar navbar-default navbar-static-top" role="navigation"> 
        <ul class="nav nav-pills">
            <li role="presentation" class="active"><a href="#">
            <i class="bi bi-house-fill"></i> Home</a></li>
            <li role="presentation"><a href="#">Profile</a></li>
            <li role="presentation"><a href="#">Messages</a></li>
            <li class="dropdown">
            <a class="dropdown-toggle" data-toggle="dropdown" href="#">
            More <span class="caret"></span>
            </a>
                <ul class="dropdown-menu">
                    <li><a href="#">About</a></li>
                    <li><a href="#">License</a></li>
                    <li><a href="#">Other</a></li>
                    <li class="divider"></li>
                    <li><a href="#">Contact us</a></li>
                </ul>
            </li>
        <div class="col search row-search-mobile">
            <form class="navbar-form navbar-right">
        <div class="form-group">
            <input type="text" class="form-control" placeholder="Key Words">
        </div>
            <button type="submit" class="btn btn-default">Search</button>
            </form>
        </div>
        </ul>
    </nav>
</body>
</html>
```
### 示例三
