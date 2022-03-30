# Navigator-数据交互
目前只写了三个功能:添加，编辑和删除。
## 模板渲染
将数据库中的数据传入到HTML中，是用过jinja2模板实现，即前面介绍的索引内容，此处不作详述。
## 数据交互
### 添加
用于处理POST请求在数据库中添加数据，navigator.py文件中添加：
```python
@bp.route('/', methods=('POST',))
def add():
    if request.method == 'POST':
        maincategory = request.form['maincategory']
        subcategory = request.form['subcategory']
        urlname = request.form['urlname']
        urllocation = request.form['urllocation']
        error = None
        if not maincategory:
            error = 'Main Category is required.'
        elif not subcategory:
            error = 'Sub Category is required.'
        elif not urlname:
            error = 'URL Name is required.'
        elif not urllocation:
            error = 'URL Location is required.'

        if error is not None:
            flash(error)
        else:
            db = get_db()
            db.execute(
                'INSERT INTO links (maincategory, subcategory, urlname, urllocation)'
                ' VALUES (?, ?, ?, ?)',
                (maincategory, subcategory, urlname, urllocation)
            )
            db.commit()
            return redirect(url_for('navigation.index'))
    return render_template('index.html')
```
说明：
- POST请求是通过添加模态框（&#60;form method="post">）发起的，此处接收默认的POST请求
- 示例中检查了是否获取的数据为空，如果都不为空，则调用数据库并插入数据

### 编辑
用于处理POST请求在数据库中编辑数据，navigator.py文件中添加：
```py
@bp.route('/<int:id>', methods=('GET', 'POST'))
def edit(id):
    if request.method == 'POST':
        maincategory = request.form['maincategory']
        subcategory = request.form['subcategory']
        urlname = request.form['urlname']
        urllocation = request.form['urllocation']
        error = None
        if not maincategory:
            error = 'Main Category is required.'
        elif not subcategory:
            error = 'Sub Category is required.'
        elif not urlname:
            error = 'URL Name is required.'
        elif not urllocation:
            error = 'URL Location is required.'

        if error is not None:
            flash(error)
        else:
            db = get_db()
            db.execute(
                'UPDATE links SET maincategory = ? , subcategory = ?, urlname = ?, urllocation = ?'
                ' WHERE id = ?',
                (maincategory, subcategory, urlname, urllocation, id)
            )
            db.commit()
            return redirect(url_for('navigation.index'))
    return render_template('index.html')
```
编辑同样采用POST请求，但是指定了一个ID，这样发起的POST就不会默认到添加视图中去了:
```html
<form action="" id="linkid" method="post">
```
对应的action属性，id是获取JS的内容传入到aciton中：
```js
$("#linkid").attr("action", link_id);
```
JS中的link_id变量获取：
```js
var link_id = button.data('link_id')
```
上面data内容中的link_id通过HTML中的data属性获取的：
```html
data-link_id="{{ link['id'] }}" 
```
最后数据是通过jinja2模板渲染从数据库中获取的。

### 删除
用于处理POST请求在数据库中删除数据，navigator.py文件中添加：
```py
@bp.route('/<int:id>/delete', methods=('GET', 'POST'))
def delete(id):
    db = get_db()
    db.execute('DELETE FROM links WHERE id = ?', (id,))
    db.commit()
    return redirect(url_for('navigation.index'))
```
&#8195;&#8195;删除数据同样获取了数据的ID进行对应，如果只有ID，POST请求会先到编辑视图里面，所以此处指向了一个页面，但是此页面是不存在的，对应HMTL中内容如下：
```html
href="{{ url_for('navigation.delete', id=link['id']) }}"
```
## 待补充
