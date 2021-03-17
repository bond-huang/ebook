# Navigator-网页模板
网页模板内容简介。
## 基础模板
### base.html
在nav目录下新建目录templates并创建文件base.html，内容如下：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Navigator-{% block title %}{% endblock %}</title>
    {% block css %}
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.4.0/font/bootstrap-icons.css">
    <link href="../static/style.css" type="text/css" rel="stylesheet"/>
    {% endblock %}
</head>
<body style="background: #f0f0f0">
    {% include "header.html" %}
    <div class="container-fluid">
        <div class="row-fluid">
            <!-- Category navigation -->
            <div class="col-sm-2">
            {% block category %}
            {% endblock %}
            </div>
            <!-- Main content -->
            <div class="col-sm-10">
            {% block content %}
            {% endblock %}
            </div>
        </div>
    </div>
    <!-- Add modal -->    
    {% include "modal/add.html" %}
    <!-- Edit modal -->    
    {% include "modal/edit.html" %}
    <!-- license modal -->    
    {% include "modal/license.html" %}
    <!-- Empty modal -->    
    {% include "modal/empty.html" %}
    <!-- footer -->    
    {% include "footer.html" %}
    
    {% block scripts %}
    <script src="https://cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
    <script type="text/javascript" src="../static/nav.js"></script>
    {% endblock %}
</body>
</html>
```
创建index.html文件,在index.html文件中写入如下内容：
```html
{% extends 'base.html' %}

{% block header %}
  <h1>{% block title %}Home{% endblock %}</h1>
{% endblock %}

<!-- 左侧导航 -->
{% block category %}
<div class="panel panel-primary">
    <div class="panel-heading">
        <h3 class="panel-title">Main Category</h3>
    </div>
    <ul class="nav nav-stacked">
        {% for item in maincg_list %}
            <li><a href="#{{ item['maincategory'] }}" data-toggle="tab">{{ item['maincategory'] }}</a></li>
        {% endfor %}
    </ul>
</div>
{% endblock %}

<!-- 右侧内容 -->
{% block content %}
<div id="myTabContent" class="tab-content">
    {% for maincg in links %}
        <div class="tab-pane fade in active" id="{{ maincg['main_cg'] }}">
            {% for subcg in maincg['sub_cg_list'] %}
            <div class="panel panel-primary">
                <div class="panel-heading">
                    <h3 class="panel-title">{{ subcg['sub_cg'] }}</h3>
                </div>
            <div class="panel-body" aria-hidden="true">
                {% for link in subcg['link'] %}
                    <div class="col-sm-3">
                        <a href="{{ link['urllocation'] }}" target="_blank">{{ link['urlname'] }}</a>&nbsp;
                        <a id="modal" href="#editModal" role="button" class="tooltip-show editicon" title="Edit this record" data-toggle="modal" data-link_id="{{ link['id'] }}" data-link_mcg="{{ link['maincategory'] }}" data-link_scg="{{ link['subcategory'] }}" data-link_name="{{ link['urlname'] }}" data-link_url="{{ link['urllocation'] }}">
                        <i class="bi bi-pencil-square"></i></a>
                        <a id="modal" role="button" class="action tooltip-show deleteicon" title="Delete this record" data-toggle="modal" href="{{ url_for('navigation.delete', id=link['id']) }}" onclick="return confirm('Are you sure to delete?');"><i class="bi bi-trash"></i></a>
                    </div>
                {% endfor %}
            </div>    
            </div>
          {% endfor %}
        </div>
    {% endfor %}
</div>
{% endblock %}
```
### 创建头部文件
头部文件head.html：
```html
<nav class="navbar navbar-default">
    <div class="container-fluid">
        <div class="navbar-header">
        <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
        </button>
        <a class="navbar-brand" style="color: #337ab7;" href="#"><i class="bi bi-cursor-fill" style="font-size: 2rem; color: #337ab7;"></i>&nbsp;&nbsp;<strong>My Navigator </strong></a>
        </div>
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
        <ul class="nav navbar-nav">
            <li class="active"><a href="/"><i class="bi bi-house-fill"></i>&nbsp;&nbsp;Home<span class="sr-only">(current)</span></a></li>
            <li><a id="modal" href="#noInfo" role="button" data-toggle="modal">Profile</a></li>
            <li><a id="modal" href="#noInfo" role="button" data-toggle="modal">Message</a></li>
            <li class="dropdown">
                <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">Action<span class="caret"></span></a>
                <ul class="dropdown-menu">
                <li><a id="modal" href="#addModal" role="button" data-toggle="modal"><i class="bi bi-plus-circle"></i>&nbsp;&nbsp;Add</a></li>
                <li><a id="modal" href="#noInfo" role="button" data-toggle="modal"><i class="bi bi-arrow-down-up"></i>&nbsp;&nbsp;Import</a></li>
                <li role="separator" class="divider"></li>
                <li><a href="/"><i class="bi bi-arrow-repeat"></i>&nbsp;&nbsp;Refresh</a></li>
                <li role="separator" class="divider"></li>
                <li><a id="modal" href="#noInfo" role="button" data-toggle="modal"><i class="bi bi-download"></i>&nbsp;&nbsp;Download</a></li>
                </ul>
            </li>
        </ul>
        <ul class="nav navbar-nav navbar-right">
            <li class="dropdown">
            <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">More<span class="caret"></span></a>
            <ul class="dropdown-menu">
                <li><a id="modal" href="#noInfo" role="button" data-toggle="modal"><i class="bi bi-share-fill"></i>&nbsp;&nbsp;Share</a></li>
                <li><a href="https://github.com/bond-huang/navigator" target="_blank"><i class="bi bi-github"></i>&nbsp;&nbsp;GitHub</a></li>
                <li><a id="modal" href="#noInfo" role="button" data-toggle="modal"><i class="bi bi-envelope-fill"></i>&nbsp;&nbsp;Email</a></li>
                <li><a id="modal" href="#noInfo" role="button" data-toggle="modal"><i class="bi bi-question-octagon-fill"></i>&nbsp;&nbsp;Help</a></li>
                <li role="separator" class="divider"></li>
                <li><a id="modal" href="#license" role="button" data-toggle="modal">View License</a></li>
            </ul>
            </li>
        </ul>
        <form class="navbar-form navbar-right">
        <div class="form-group">
            <input type="text" class="form-control" placeholder="Key Words">
        </div>
        <button type="submit" class="btn btn-default"><i class="bi bi-search"></i>&nbsp;&nbsp;Search</button>
        </form>
        </div>
    </div>
</nav>
```
### 创建底部文件
底部文件footer.html，暂时没写内容。

## 模态框
### 添加模态框：
添加模态框内容如下：
```html
<!-- Add Modal -->
<div class="modal fade" id="addModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal" aria-hidden="true">
                &times;</button>
            <h4 class="modal-title" id="myModalLabel">Add Data</h4>
            </div>
            <form method="post">
            <div class="modal-body">
                <div class="form-group">
                    <label for="maincategory">Main Category</label>
                    <input type="text" name="maincategory" class="form-control" id="maincategory" placeholder="Main Category">
                </div>
                <div class="form-group">
                    <label for="subcategory">Sub Category</label>
                    <input type="text" name="subcategory" class="form-control" id="subcategory" placeholder="Sub Category">
                </div>
                <div class="form-group">
                    <label for="urlname">Name</label>
                    <input type="text" name="urlname" class="form-control" id="urlname" placeholder="Name of the URL">
                </div>
                <div class="form-group">
                    <label for="urllocation">Location</label>
                    <input type="text" name="urllocation" class="form-control" id="urllocation" placeholder="Location of the URL">
                </div>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default" data-dismiss="modal">Cancel</button>
                <input type="submit" class="btn btn-primary" value="Save">
            </div>
            </form>
        </div>
    </div>
</div>
```
### 编辑模态框：
编辑模态框内容如下：
```html
<!-- Edit Modal -->
<div class="modal fade" id="editModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal" aria-hidden="true">
                &times;</button>
            <h4 class="modal-title" id="myModalLabel">Edit the record</h4>
            </div>
            <form action="" id="linkid" method="post">
            <div class="modal-body">
                <div class="form-group">
                    <label for="maincategory">Main Category</label>
                    <input type="text" name="maincategory" class="form-control" value="" id="linkmcg">
                </div>
                <div class="form-group">
                    <label for="subcategory">Sub Category</label>
                    <input type="text" name="subcategory" class="form-control" value="" id="linkscg">
                </div>
                <div class="form-group">
                    <label for="urlname">Name</label>
                    <input type="text" name="urlname" class="form-control" value="" id="linkname">
                </div>
                <div class="form-group">
                    <label for="urllocation">Location</label>
                    <input type="text" name="urllocation" class="form-control" value="" id="linkurl">
                </div>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default" data-dismiss="modal">Cancel</button>
                <input type="submit" class="btn btn-primary" value="Confirm">
            </div>
            </form>
        </div>
    </div>
</div>
```
### 空模态框：
空模态框内容如下：
```html
<div class="modal fade" id="noInfo" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
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
```
