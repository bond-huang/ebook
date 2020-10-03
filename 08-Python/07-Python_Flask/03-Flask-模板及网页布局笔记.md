# Flask-模板及网页布局笔记
学习Flask项目搭建笔记，边学习边实践。
## 模板
&#8195;&#8195;应用已经写好验证视图，但是如果现在运行服务器的话，访问任何URL都会看到一个TemplateNotFound错误。这是因为视图调用了`render_template()`，但是模板还没有写。模板简单说明：
- 模板文件会储存在osmanagement包内的templates文件夹内
- 模板是包含静态数据和动态数据占位符的文件
- 模板使用指定的数据生成最终的文档，Flask使用Jinja模板库来渲染模板

### 基础布局
应用中的每一个页面主体不同，但是基本布局是相同的。创建tmplates目录并创建base.html文件：
```
$ mkdir templates
$ cd templates
$ touch base.html
$ pwd
/d/OS-Management/osmanagement/templates
$ ls
base.html
```
在base.html文件中写入：
```py
<!doctype html>
<table>{%block title %}{$ endblock %} - Operating System Management</table>
<link rel="stylesheet" href="{{ url_for('static',filename='style.css') }}">
<nav>
    <h1>Operating System Management</h1>
    <ul>
        {% if g.user %}
            <li><span>{{ g.user['username'] }}</span>
            <li><a href="{{ url_for('auth.logout') }}">Log Out</a>
        {% else %}
            <li><a href="{{ url_for('auth.register') }}">Register</a>
            <li><a href="{{ url_for('auth.login') }}">Log In</a>
        {% endif %}    
    </ul>
</nav>
<section class="content">
    <header>
        {% block header %}{% endblock %}
    </header>
    {% for message in get_flashed_messages() %}
        <div class="flash">{{ message }}</div>
    {% endfor %}
    {% block content %}{% endblock %}
</section>
```
说明：
- `g`在模板中自动可用。根据`g.user`是否被设置（在`load_logged_in_user`中进行），要么显示用户名和注销连接，要么显示注册和登录连接
- `url_for()`也是自动可用的，可用于生成视图的URL，而不用手动来指定
- 在标题下面，正文内容前面，模板会循环显示`get_flashed_messages()`返回的每个消息
- 在视图中使用`flash()`来处理出错信息，在模板中就可以这样显示出出来
- 模板中定义三个块，这些块会被其他模板重载:
    - `{% block title %}`会改变显示在浏览器标签和窗口中的标题
    - `{% block header %}`类似于title,但是会改变页面的标题
    - `{% block content %}`是每个页面的具体内容，如登录表单或文章等
- 其他模板直接放在templates文件夹内
- 属于某个Blueprints的模板会被放在与Blueprints同名的文件夹内
