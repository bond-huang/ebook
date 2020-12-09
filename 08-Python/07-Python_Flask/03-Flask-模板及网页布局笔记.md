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
```html
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
    - &#123;&#37; block title &#37;&#125;会改变显示在浏览器标签和窗口中的标题
    - &#123;&#37; block header &#37;&#125;类似于title,但是会改变页面的标题
    - &#123;&#37; block content &#37;&#125;是每个页面的具体内容，如登录表单或文章等
- 其他模板直接放在templates文件夹内
- 属于某个Blueprints的模板会被放在与Blueprints同名的文件夹内

### 注册
在tmplates目录下创建auth目录，在其下面并创建register.html文件：
```
$ mkdir auth
$ cd auth
$ touch register.html
$ pwd
/d/OS-Management/osmanagement/templates/auth
```
在register.html文件中写入：
```html
{% extends 'base.html' %}

{% block header %}
    <h1>{% block title %}Register{% endblock %}</h1>
{% endblock %}

{% block content %}
    <form method="post">
        <label for="username">Username</label>
        <input name="username" id="username" required>
        <label for="password">Password</label>
        <input type="password" name="password" id="password" required>
        <input type="submit" value="Register">
    </form>
{% endblock %}
```
说明：
- &#123;&#37; extends 'base.html' &#37;&#125;告诉Jinja这个模板基于基础模板base.html，并需要替换相应的块,所有替换的内容必须位于&#123;&#37; block &#37;&#125;标签之内
- 代码中把&#123;&#37; block title &#37;&#125;放在&#123;&#37; block header &#37;&#125;内部，这里不但可以设置title块，还可以把其值作为header块的内容
- `input`标记使用了required属性,告诉浏览器这些字段是必填的

### 登录
在auth目录下创建login.html文件，并写入如下内容：
```html
{% extends 'base.html' %}

{% block header %}
    <h1>{% block title %}Log In{% endblock %}</h1>
{% endblock %}

{% block content %}
    <form method="post">
        <label for="username">Username</label>
        <input name="username" id="username" required>
        <label for="password">Password</label>
        <input type="password" name="password" id="password" required>
        <input type="submit" value="Log In">
    </form>
{% endblock %}
```
### 注册用户
运行程序：
```
$ export FLASK_APP=osmanagement
$ export FLASK_ENV=development
$ flask run
 * Serving Flask app "osmanagement" (lazy loading)
 * Environment: development
 * Debug mode: on
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 142-805-651
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
运行后有报错：
```
File "D:\OS-Management\osmanagement\__init__.py", line 31, in create_app
app.register_blueprint(auth.bq)
AttributeError: module 'osmanagement.auth' has no attribute 'bq'
File "D:\OS-Management\osmanagement\auth.py", line 67
    return redirect(url_for('index'))
    ^
IndentationError: unexpected indent
```
缩进问题及字母写错了，修改后还是有报错：
```
File "D:\OS-Management\osmanagement\templates\base.html", line 23, in template
{% block content %}{% endblock %}
jinja2.exceptions.TemplateSyntaxError: Unexpected end of template. Jinja was looking for the following tags: 'endblock'. The innermost block that needs to be closed is 'block'.
```
一般是`{%`的问题，中间有空格或者缺少空格或者写错了，找了半天找到下面语句中两处错误：
```
<table>{%block title %}{$ endblock %} - osmanagement</table>
```
修改后进入`http://127.0.0.1:5000/auth/register`可以看到注册的页面,注册后报错了：
```
File "D:\OS-Management\osmanagement\auth.py", line 11, in register
password = request.form['password']
werkzeug.exceptions.BadRequestKeyError: 400 Bad Request: The browser (or proxy) sent a request that this server could not understand.
KeyError: 'password'
```
&#8195;&#8195;应该是`request.form['password']`用法不支持还是错了，改成`request.form.get('password')`后不报错，但是提示`Password is required!`,也就是相当于没获取到密码,查了半天是前端register.html和login.html里面这句都写错了，`name="username"`应该是`name="password"`：
```
<input type="password" name="username" id="password" required>
```
修改后运行报错变了：
```
File "D:\OS-Management\osmanagement\auth.py", line 23, in register
db.execute(
sqlite3.OperationalError: table user has no column named password
```
检查schema.sql文件发现下面内容手残打错了：
```
passwork TEXT NOT NULL
```
修改后重新初始化一下数据库就可以了。

## 静态文件
在base.html模板中使用了：
```
<link rel="stylesheet" href="{{ url_for('static',filename='style.css') }}">
```
说明：
- Flask自动添加一个static视图，视图使用相对于osmmanagerment/static的相对路径
- 除了CSS，还可以是JavaScript函数文件或者logo图片，都放置于flaskr/static文件夹中，并使用`url_for('static', filename='...')`来引用
- 改动静态文件后需要刷新页面，如果刷新无效，需要清除浏览器的缓存

在style.css文件中写入如下内容：
```
html { font-family: sans-serif; background: #eee; padding: 1rem; }
body { max-width: 960px; margin: 0 auto; background: white; }
h1 { font-family: serif; color: #377ba8; margin: 1rem 0; }
a { color: #377ba8; }
hr { border: none; border-top: 1px solid lightgray; }
nav { background: lightgray; display: flex; align-items: center; padding: 0 0.5rem; }
nav h1 { flex: auto; margin: 0; }
nav h1 a { text-decoration: none; padding: 0.25rem 0.5rem; }
nav ul  { display: flex; list-style: none; margin: 0; padding: 0; }
nav ul li a, nav ul li span, header .action { display: block; padding: 0.5rem; }
.content { padding: 0 1rem 1rem; }
.content > header { border-bottom: 1px solid lightgray; display: flex; align-items: flex-end; }
.content > header h1 { flex: auto; margin: 1rem 0 0.25rem 0; }
.flash { margin: 1em 0; padding: 1em; background: #cae6f6; border: 1px solid #377ba8; }
.post > header { display: flex; align-items: flex-end; font-size: 0.85em; }
.post > header > div:first-of-type { flex: auto; }
.post > header h1 { font-size: 1.5em; margin-bottom: 0; }
.post .about { color: slategray; font-style: italic; }
.post .body { white-space: pre-line; }
.content:last-child { margin-bottom: 0; }
.content form { margin: 1em 0; display: flex; flex-direction: column; }
.content label { font-weight: bold; margin-bottom: 0.5em; }
.content input, .content textarea { margin-bottom: 1em; }
.content textarea { min-height: 12em; resize: vertical; }
input.danger { color: #cc2f2e; }
input[type=submit] { align-self: start; min-width: 10em; }
```
## 页面Blueprints
完成每个视图时，保持开发服务器运行。当保存修改后，请尝试在浏览器中访问URL，并进行测试。
### Blueprints
定义blueprints并注册到应用工厂，在osmanagement目录下创建文件osm.py,并写入如下内容：
```python
from flask import(
    Blueprint,flash,g,redirect,render_templagte,request,url_for
)
from werkzeug.exceptions import abort
from osmanagement.auth import login_required
from osmanagement.db import get_db
bp = Blueprint('osm',__name__)
```
使用`app.register_blueprint()`在工厂中导入和注册蓝图,将新代码放在`__init__.py`的尾部，返回应用之前:
```python
    from . improt osm
    app.register_blueprint(osm.bp)
    app.add_url_rule('/',endpoint='index')
```
说明：
- 与验证blueprints不同，内容管理blueprints没有`url_prefix`。因此`index`视图会用于`/`，`create`会用于`/create`，以此类推
- 但下面的`index`视图的端点会被定义为`osm.index`。一些验证视图会指定向普通的`index`端点。 我们使用 `app.add_url_rule()`关联端点名称`'index'`和`/ URL`，这样`url_for('index')`或`url_for('osm.index')`都会有效，会生成同样的`/ URL`

### 索引
索引会显示所有帖子，最新的会排在最前面。为了在结果中包含user表中的作者信息，使用了一个JOIN。在osm.py中继续写入一下内容：
```python
@bp.route('/')
def index():
    db = get_db()
    posts = db.execute(
        'SELECT p.id, title, bode, created, author_id, username'
        'FROM post p JOIN user u ON p.author_id = u.id'
        'ORDER BY created DESC'
    ).fetchall()
    return render_template('osm/index.html',posts=posts)
```
创建index.html文件：
```
$ pwd
/d/OS-Management/osmanagement/templates/osm
$ touch index.html
```
在index.html文件中写入如下内容：
```html
{% extends 'base.html' %}

{% block header %}
  <h1>{% block title %}Posts{% endblock %}</h1>
  {% if g.user %}
    <a class="action" href="{{ url_for('osm.create') }}">New</a>
  {% endif %}
{% endblock %}

{% block content %}
  {% for post in posts %}
    <article class="post">
      <header>
        <div>
          <h1>{{ post['title'] }}</h1>
          <div class="about">by {{ post['username'] }} on {{ post['created'].strftime('%Y-%m-%d') }}</div>
        </div>
        {% if g.user['id'] == post['author_id'] %}
          <a class="action" href="{{ url_for('osm.update', id=post['id']) }}">Edit</a>
        {% endif %}
      </header>
      <p class="body">{{ post['body'] }}</p>
    </article>
    {% if not loop.last %}
      <hr>
    {% endif %}
  {% endfor %}
{% endblock %}
```
说明：
- 当用户登录后，`header`块添加了一个指向`create`视图的连接
- 当用户是 博客作者时，可以看到一个“ Edit ”连接，指向 update 视图
- loop.last 是一个 Jinja for 循环 内部可用的特殊变量，它用于在每个 博客帖子后面显示一条线来分隔帖子，最后一个帖子除外

### 创建
