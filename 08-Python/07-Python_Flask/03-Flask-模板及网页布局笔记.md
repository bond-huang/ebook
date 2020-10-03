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
