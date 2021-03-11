# Flask-常见问题
学习和使用过程中遇到的问题处理记录。
## 运行报错
### ValueError
报错示例如下：
```
ValueError: urls must start with a leading slash
```
原因：蓝图中路由写错了，没加斜杠：
```py
@bp.route('<int:id>', methods=('POST',))
```
正确代码：
```py
@bp.route('/<int:id>', methods=('POST',))
```
### KeyError
错误示例：
```
File "D:\navigator\nav\navigation.py", line 36, in add
maincategory = request.form['maincategory']
File "C:\Users\AppData\Local\Programs\Python\Python38\Lib\site-packages\werkzeug\datastructures.py", line 442, in __getitem__
    raise exceptions.BadRequestKeyError(key)
werkzeug.exceptions.BadRequestKeyError: 400 Bad Request: The browser (or proxy) sent a request that this server could not understand.
KeyError: 'maincategory'
```
明显是获取不到数据：
- 可能是在对应的位置没有传递值
- 可能是指向了错误的路由，比如一个FORM删除的功能路由到添加里面去了

## 待补充
