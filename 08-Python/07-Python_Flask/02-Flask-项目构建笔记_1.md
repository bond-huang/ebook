# Flask-项目构建笔记_1
学习Flask项目搭建笔记，边学习边实践。
## 项目布局
项目包含如下内容（根据flask官方教程来的，内容基本一致）:
- `flaskr/`：包含应用代码和文件的Python包
- `tests/` :包含测试模块的文件夹
- `venv/` :Python虚拟环境，用于安装Flask和其他依赖的包
- 告诉 Python 如何安装项目的安装文件
- 版本控制配置，如 git，不管项目大小，应当养成使用版本控制的习惯
- 项目需要的其他文件

项目计划布局如下：
```
├── osmanagement/
│   ├── __init__.py
│   ├── db.py
│   ├── schema.sql
│   ├── auth.py
│   ├── osm.py
│   ├── templates/
│   │   ├── base.html
│   │   ├── auth/
│   │   │   ├── login.html
│   │   │   └── register.html
│   │   └── osm/
│   │       ├── create.html
│   │       ├── index.html
│   │       └── update.html
│   └── static/
│       └── style.css
├── tests/
│   ├── conftest.py
│   ├── data.sql
│   ├── test_factory.py
│   ├── test_db.py
│   ├── test_auth.py
│   └── test_osm.py
├── venv/
├── setup.py
└── MANIFEST.in
```
## 应用设置
&#8195;&#8195;可以在一个函数内部创建Flask实例来代替创建全局实例。这个函数被称为应用工厂。所有应用相关的配置、注册和其他设置都会在函数内部完成，然后返回这个应用。

### 应用工厂
创建osmanagement文件夹并添加`__init__.py`文件：
```
$ mkdir osmanagement
$ cd osmanagement
$ touch __init__.py
$ ls
__init__.py
```
在`__init__.py`文件写入内容如下：
```python
import os
from flask import Flask
def create_app(test_config=None):
    # create and configure the app
    app = Flask(__name__,instance_relative_config=True)
    app.config.from_mapping(
        SECRET_KEY='dev',
        DATABASE=os.path.join(app.instance_path,'osmanagment.sqlite'),
    )
    if test_config is None:
        # load the instance config, if it exists, when not testing
        app.config.from_pyfile('config.py',silent=True)
    else:
        # load the test config if passed in
        app.config.from_mapping(test_config)
    # ensure the instance folder exists
    try:
        os.makedirs(app.instance_path)
    except OSERROR:
        pass
    # a simple page that says gump
    @app.route('/gump')
    def gump():
        return 'Life was like a box of chocolates, \
        you never know what you\'re gonna get.'
    return app
```
### 运行应用
运行应用，注意在项目目录下，不是在`osmanagement`包里面:
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
&#8195;&#8195;在开发模式下，当页面出错的时候会显示一个可以互动的调试器；当你修改代码保存的时候会重启服务器。此次运行我出现了调试器，说明代码有问题，根据提示我修改了，提示如下：
```
    File "D:\OS-Management\osmanagement\__init__.py", line 5, in create_app

    app = Flask(__name__,instance_relative_config=Ture)

    NameError: name 'Ture' is not defined
```
修改后自动跳转恢复正常，注意页面地址是：http://127.0.0.1:5000/gump

## 定义操作和数据库
&#8195;&#8195;Python内置了SQLite数据库支持，相应的模块为sqlite3。使用SQLite的便利性在于不需要单独配置一个数据库服务器，并且Python 提供了内置支持,适合小应用。
### 连接数据库
创建文件db.py：
```
$ pwd
/d/OS-Management/osmanagement
$ touch db.py
```
在db.py中写入如下内容：
```python
def get_db():
    if 'db' not in g:
        g.db =  sqlite3.connect(
            current_app.config['DATAVASE'],
            detect_types=sqlite3.PARSE_DECLTYPES
        )
        g.db.row_factory = sqlite3.Row
    return g.db
def close_db(e=None):
    db = g.pop('db',None)
    if db is not None:
        db.close()
```
说明：
- `g`是一个特殊对象，在处理请求过程中，它可以用于储存可能多个函数都会用到的数据。把连接储存于其中，可以多次使用，而不用在同一个请求中每次调用`get_db`时都创建一个新的连接
- `current_app` 是另一个特殊对象，该对象指向处理请求的Flask应用。在处理一个请求时，`get_db`会被调用。这样就需要使用`current_app`
- `sqlite3.connect()` 建立一个数据库连接，该连接指向配置中的DATABASE指定的文件。这个文件目前还没有建立，后面会在初始化数据库的时候建立该文件
- `sqlite3.Row` 告诉连接返回类似于字典的行，这样可以通过列名称来操作数据。
- `close_db` 通过检查`g.db`来确定连接是否已经建立。如果连接已建立，那么就关闭连接。

### 创建表
&#8195;&#8195;在SQLite中，数据储存在表和列中。Flaskr会把用户数据储存在user表中，把内容储存在post表中。下面创建一个文件储存用于创建空表的SQL命令：      
创建文件schema.sql：
```
$ pwd
/d/OS-Management/osmanagement
$ touch schema.sql
```
在schema.sql中写入如下内容：
```sql
DROP TABLE IF EXISTS user;
DROP TABLE IF EXISTS post;

CREATE TABLE user (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE NOT NULL,
    passwork TEXT NOT NULL
);

CREATE TABLE post (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    author_id INTEGER NOT NULL,
    created TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    title TEXT NOT NULL,
    body TEXT NOT NULL,
    FOREIGN KEY (author_id) REFERENCES user (id)
);
```
在`db.py`文件中添加运行这个SQL命令的函数：
```python
def init_db():
    db = get_db()
    with current_app.open_instance_resource('schema.sql') as f:
        db.executescript(f.read().decode('utf8'))

@click.command('init-db')
@with_appcontext
def init_db_command():
    # Clear the existing data and create new tables.
    init_db()
    click.echo('Initialized the database')
```
说明：
- `open_resource()`打开一个文件，该文件名是相对于flaskr包的，这样就不需要考虑以后应用具体部署在哪个位置
- `get_db`返回一个数据库连接，用于执行文件中的命令
- `click.command()`定义一个名为`init-db`命令行，它调用`init_db`函数

### 在应用中注册
&#8195;&#8195;`close_db`和`init_db_command`函数需要在应用实例中注册，否则无法使用。我们使用了工厂函数，那么在写函数的时候应用实例还无法使用。可以写一个函数，把应用作为参数，在函数中进行注册（`db.py`文件中）：
```python
def init_app(app):
    app.teardown_appcontext(close_db)
    app.cli.add_command(init_db_command)
```
说明：
- `app.teardown_appcontext()`告诉Flask在返回响应后进行清理的时候调用此函数
- `app.cli.add_command()`添加一个新的可以与flask一起工作的命令

在`__init__.py`中导入并调用这个函数:
```python
    from . import db
    db.init_app(app)
    
    return app
```
### 初始化数据库文件
停止之前的虚拟环境，然后激活环境：
```
(venv) D:\OS-Management>venv\Scripts\deactivate
D:\OS-Management>
D:\OS-Management>venv\Scripts\activate
(venv) D:\OS-Management>
```
设置`FLASK_APP`和`FLASK_ENV`并运行`init-db`命令：
```
$ export FLASK_APP=osmanagement
$ export FLASK_ENV=development
$ flask init-db
Initialized the database
```
&#8195;&#8195;刚开始运行有些报错，有些是明显写错了，`with current_app.open_instance_resource('schema.sql')`这句应该是`with current_app.open_resource('schema.sql')`：
```
 with current_app.open_instance_resource('schema.sql') as f:
  File "c:\users\qianhuang\appdata\local\programs\python\python38\lib\site-packages\flask\app.py", line 740, in open_instance_resource
    return open(os.path.join(self.instance_path, resource), mode)
FileNotFoundError: [Errno 2] No such file or directory: 'D:\\OS-Management\\instance\\schema.sql'
```
运行成功后instance文件夹中会出现osmanagment.sqlite文件。

## Blueprints和视图
&#8195;&#8195;视图是一个应用对请求进行响应的函数。Flask通过模型把进来的请求URL匹配到对应的处理视图。视图返回数据，Flask把数据变成出去的响应。Flask也可以反过来。
### 创建Blueprints
&#8195;&#8195;Blueprint是一种组织一组相关视图及其他代码的方式。Flaskr有两个蓝图，一个用于认证功能，另一个用于内容管理。每个蓝图的代码都在一个单独的模块中。     
创建auth.py文件 ：
```
$ pwd
/d/OS-Management/osmanagement
$ touch auth.py
```
写入如下内容：
```python
import functools
from flask import(Blueprint,flash,g,redirect,render_template,request,session,url_for)
from werkzeug.security import check_password_hash,generate_password_hash
from osmanagement.db import get_db
bp = Blueprint('auth',__name__,url_prefix='/auth')
```
说明：
- 创建了一个名称为'auth'的Blueprint。和应用对象一样，Blueprint需要知道是在哪里定义的，因此把`__name__`作为函数的第二个参数
- `url_prefix`会添加到所有与该Blueprint关联的URL前面

使用`app.register_blueprint()`导入并注册Blueprint。在`__init__.py`中加上如下内容：
```python
    from . import auth
    app.register_blueprint(auth.bq)

    return app
```
### 注册试图
&#8195;&#8195;当用户访问`/auth/register`URL时， register视图会返回用于填写注册内容的表单的HTML。当用户提交表单时会验证表单内容，验证失败会显示表单并显示一个出错信息，成功创建新用户并显示登录页面，在auth.py中继续写入如下内容:
```python
@bq.route('/register',methods=('GET','POST'))
def register():
    if request.method == 'POST':
        username = request.form['username']
        passwork = request.form['password']
        db = get_db()
        error = None
        if not username:
            error = 'Username is required!'
        elif not password:
            error = 'Password is required!'
        elif db.execute(
            'SELECT id FROM user WHERE username = ?',(username,)
        ).fetchone()is not None:
            error = 'User {} is already registered,please enter another!'.format(username)
        if error is None:
            db.execute(
                'INSERT INTO user (username,password) VALUES (?,?)',
                (username,generate_password_hash(password))
            )
            db.commit()
            return redirect(url_for('auth.login'))
        flash(error)
    return render_template('auth/register.html')
```
说明：
- `@bp.route`关联了URL:`/register`和register视图函数。当Flask收到一个指向/auth/register的请求时就会调用register视图并把其返回值作为响应
- 如果用户提交了表单，那么request.method将会是'POST'，然后会开始验证用户的输入内容。
- `request.form`是一个特殊类型的dict，其映射了提交表单的key和vaule
- 代码中还验证了username和password是否为空
- 通过查询数据库，检查是否有查询结果返回来验证username是否已被注册。`db.execute`使用了带有`?`占位符的SQL查询语句
- `fetchone()`根据查询返回一个记录行,如果查询没有结果，则返回 None。后面还用到`fetchall()`它返回包括所有结果的列表
- 如果验证成功，那么在数据库中插入新用户数据。使用`generate_password_hash()`生成安全的哈希值并储存到数据库中
- 然后调用`db.commit()`保存修改
- 用户数据保存后将转到登录页面。 `url_for()`根据登录视图的名称生成相应的URL
- 如果验证失败，那么会向用户显示一个出错信息。`flash()`用于储存在渲染模块时可以调用的信息
- 当用户最初访问`auth/register`或者注册出错时，应用会显示一个注册表单
- `render_template()`会渲染一个包含HTML的模板

### 登录视图
和register视图差不多，在auth.py中继续写入如下内容：
```python
@bp.route('/login',methods=('GET','POST'))
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        db = get_db()
        error = None
        user = db.execure(
            'SELECT * FROM user WHERE username = ?',(username,)
        ).fetchone()
        if user is None:
            error = 'Incorrect username,please try again!'
        elif not check_password_hash(user['password'],password):
            errot = 'Incorrect password,please try again!'
        if error is None:
            session.clear()
            session['user_id'] = user['id']
            return redirect(url_for('index'))

        flash(error)
    return render_template('auth/login.html')
```
说明：
- 首先需要查询用户并存放在变量中，以备后用
- `check_password_hash()`以哈希值方法保存密码并比较哈希值
- `session`是一个dict，用于储存横跨请求的值。当验证成功后，用户的id被储存于一个新的会话中，会话数据被储存到一个向浏览器发送的cookie中，在后继请求中浏览器会返回它。
- Flask会安全对数据进行签名以防数据被篡改

&#8195;&#8195;用户的id已被储存在`session`中，可以被后续的请求使用。在个请求的开头，如果用户已登录，那么其用户信息应当被载入，以使其可用于其他视图，在auth.py中继续写入如下内容：
```python
@bp.before_app_request
def load_logged_in_user():
    user_id =  session.get('user_id')
    if user_id is None:
        g.user = None
    else:
        g.user = get_db().execute(
            'SELECT * FROM user WHERE id = ?',(user_id,)
        ).fetchone()
```
说明：
- `bp.before_app_request()`注册一个在视图函数之前运行的函数，不论其URL是什么
- `load_logged_in_user`检查用户id是否已经储存在`session`中，并从数据库中获取用户数据，然后储存在`g.user`中
- `g.user`的持续时间比请求要长
- 如果没有用户id ，或者id不存在，那么`g.user`将会是None 

### 注销视图
&#8195;&#8195;注销的时候需要把用户id从`session`中移除。然后`load_logged_in_user`就不会在后继请求中载入用户了,在auth.py中继续写入如下内容：
```python
@bp.route('/logout')
def logout():
    session.clear()
        return redirect(url_for('index'))
```
### 其它视图中验证
&#8195;&#8195;只能用户登录后才能在其它网页中进行操作，在每个视图中可以用装饰器来完成这个工作，在auth.py中继续写入如下内容：
```python
def login_required(view):
    @functools.wraps(view)
    def wrapped_view(**kwargs):
        if g.user is None:
            return redirect(url_for('auth.login'))
        return view(**kwargs)
    return wrapped_view
```
说明：
- 装饰器返回一个新的视图，该视图包含了传递给装饰器的原视图
- 新的函数检查用户是否已载入，如果已载入，那么就继续正常执行原视图，否则就重定向到登录页面
- 在后续视图中会使用到这个装饰器

### Endpoints和URLs
说明如下：
- `url_for()`函数根据视图名称和arguments生成URL,视图相关联的名称亦称为Endpoint，默认Endpoint名称与视图函数名称相同
- 例如，前文被加入应用工厂的 gump() 视图端点为'gump'，可以使用url_for('gump')来连接
- 当使用Blueprints的时候，Blueprints的名称会添加到函数名称的前面。上面的login函数的端点为 'auth.login'，因为它已被加入'auth' Blueprints中
