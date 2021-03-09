# Flask-操作请求数据
&#8195;&#8195;对于web应用来说对客户端向服务器发送的数据作出响应很重要。在Flask中由全局对象request来提供请求信息。
## 请求对象
### 基础用法
必须从flask模块导入请求对象:
```py
from flask import request
```
&#8195;&#8195;通过使用`method`属性可以操作当前请求方法，通过使用`form`属性处理表单数据（在POST或者PUT请求中传输的数据）,示例如下：
```py
@app.route('/login', methods=['POST', 'GET'])
def login():
    error = None
    if request.method == 'POST':
        if valid_login(request.form['username'],
                       request.form['password']):
            return log_the_user_in(request.form['username'])
        else:
            error = 'Invalid username/password'
    # 如果是POST请求方法执行以下代码
    # 如果是GET或凭据无效则返回错误
    return render_template('login.html', error=error)
```
说明：
- 当form属性中不存在这个键时会引发一个KeyError
- 如果你不像捕捉一个标准错误一样捕捉KeyError，那么会显示一个HTTP 400 Bad Request错误页面

要操作URL（如 ?key=value ）中提交的参数可以使用 args 属性:
```py
searchword = request.args.get('key', '')
```
&#8195;&#8195;用户可能会改变URL导致出现一个400请求出错页面，这样降低了用户友好度。推荐使用get或通过捕捉KeyError来访问URL参数。

请求对象方法和属性参见Request文档：[Incoming Request Data](https://dormousehole.readthedocs.io/en/latest/api.html#flask.Request)

### 
