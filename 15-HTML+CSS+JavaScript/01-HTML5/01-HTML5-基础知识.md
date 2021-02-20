# HTML5-基础知识
HTML5学习笔记，主要学习教程：《HTML5+CSS3+JavaScript从入门到精通(标准版)》未来科技编著。
## HTML5文档基本结构
&#8195;&#8195;HTML5文档省略了&#60;html>,&#60;head>,&#60;body>等元素，使用HTML5的DOCTYPE声明文档类型，简化&#60;meta>元素的charset属性值，省略&#60;p>元素的结束标记，使用&#60;元素/>的方式来结束&#60;meta>元素，以及&#60;br>元素等语法知识要点。   
一个简单的HTML5文档基本结构如下：
```html
<!DOCTYPE html>
<meta charset="UTF-8">
<title>HTML5文档基本结构</title>
<h1>HTML 的目标</h1>
<p>HTML5的目标是为了能够创建简单的WEB程序，书写出更简洁的HTML代码。
<br/>例如，为了使WEB应用程序的开发变得更容易，提供了很多API；为了使HTML变得更简洁，开发出新的属性、新的元素等。
```
## HTML5基本语法
与HTML4相比，HTML5在语法上发生了很大的变化。
### 内容类型
扩展名仍为“.html”或“.htm”，内容类型（ContentType）仍然为“text/html”。
### 文档类型
&#8195;&#8195;DOCTYPE命令声明文档的类型，它是HTML文档必须部分，且必须位于代码第一行。在HTML5中，文档类型声明方法如下：
```html
<!DOCTYPE html>
```
使用工具时，可以在DOCTYPE声明中加入SYSTEM识别符：
```html
<!DOCTYPE HTML SYSTEM "about:legacy-compat">
```
说明及注意：
- 在HTML5中，DOCTYPE声明方式不区分大小写，引号也不区分单引号或双引号
- 使用HTML5的DOCTYPE会触发浏览器以标准模式显示页面

### 字符编码
在HTML4中，使用meta元素定义文档的字符编码：
```html
<meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
```
在HTML5中，简化了charset属性写法：
```html
<meta charset="UTF-8">
```
说明及注意：
- 以上两种方法在HTML5中都有效，但是不能混用
- 从HTML5开始，对于文件的字符编码推荐使用UTF-8

### 标记省略
在HTML5中，元素的标记可以省略，分三种类型：
- 不允许写结束标记的元素：area,base,br,col,command,embed,hr,img,input,keygen,link,meta,param,source,track,wbr
- 可以省略结束标记的元素：li,dt,dd,p,rt,rp,optgroup,option,colgroup,thead,tbody,tfoot,tr,td,th
- 可以省略全部标记的元素：html,head,body,colgroup,tbody

&#8195;&#8195;不允许写结束标记的元素指不允许使用开始标记与结束标记将元素括起来形式，只允许使用&#60;元素/>形式书写，示例如下：
```html
<br></br>
```
在HTML5中正确书写方式：
```html
<br/>
```
&#8195;&#8195;可以省略全部标记的元素是指该元素可以完全被省略，但该元素还是以隐式的方式存在，例如body元素省略了，在文档结构中还是存在，可以使用document.body进行访问。
### 布尔值
示例说明如下：
```html
<!--只写属性，不写属性值，代表属性为true-->
<input type="checkbox" checked>
<!--不写属性值，代表属性为false-->
<input type="checkbox">
<!--属性值=属性名，代表属性为true-->
<input type="checkbox" checked="checked">
<!--属性值=空字符串，代表属性为true-->
<input type="checkbox" checked="">
```
### 属性值
&#8195;&#8195;属性值两边可以用双引号，也可以用单引号。在HTML5中，当属性值不包括空字符串、<、>、=、单引号、双引号等字符时，属性值两边的引号可以省略，示例：
```html
<input type="text">
<input type='text'>
<input type=text>
```
## HTML4元素
这些元素基本在HTML5中通用。
### 结构元素
结构元素用于构建网页的结构，多指块状元素：

元素|说明
:---:|:---
div|在文档中定义一块区域，即包含框、容器
ol|根据一定排序进行的列表
ul|没有排序的列表
li|每条列表项
dl|以定义的方式进行列表
dt|定义列表中的词条
dd|对定义的词条进行解释
del|定义删除的文本
ins|定义插入的文本
h1-h6|标题1到标题6，定义不同级别标题
p|定义段落结构
hr|定义水平线

### 内容元素
内容元素定义了元素在文本中表示内容的语义，一般指文本格式化元素，多是行内元素：

元素|说明
:---:|:---
span|在文本中定义一个区域，即行内包含框
a|定义超链接
abbr|定义缩写词
address|定义地址
dfn|定义术语，以斜体显示
kbd|定义键盘键
samp|定义样本
var|定义变量
tt|定义打印机字体
code|定义计算机源代码
pre|定义预定义格式文本，保留源代码格式
bolckquote|定义大块内容引用
cite|定义引文
q|定义引用短语
strong|定义重要文本
em|定义文本为重要

### 修饰元素
修饰元素定义了文本的显示效果：

元素|说明
:---:|:---
b|视觉提醒，显示为粗体
i|语气强调，显示为斜体
big|定义较大文本
small|表示细则一类的旁注，文本缩小显示
sup|定义上标
sub|定义下标
bdi和bdo|定义文本显示方向
br|定义换行
u|非文本注解，显示下划线

## HTML4属性
### 核心属性
核心属性主要包括以下三个，此三个属性为大部分元素所拥有：
- class：定义类规则或样式规则
- id：定义元素的唯一标识
- style：定义元素的样式声明

### 语言属性
语言属性主要定义元素的语言类型，包括两个属性：
- lang：定义元素的语言代码或编码
- dir：定义文本的方向，包括ltr和rtl取值，分别标识从左向右和从右向左

### 键盘属性
键盘属性定义元素的键盘访问方法，包括两个属性：
- accesskey：定义访问某元素的键盘快捷键
- tabindex：定义元素的Tab键索引编号

例如在文档中插入3个超链接，并分别定义tabindex属性，可以通过Tab键快速切换超链接：
```html
<a href="#" tabindex="1">Tab 1</a>
<a href="#" tabindex="2">Tab 2</a>
<a href="#" tabindex="3">Tab 3</a>
```
### 内容属性
&#8195;&#8195;内容属性定义元素包含内容的附加信息，可以避免元素本身包含信息不全而被误解，内容语义包括五个属性：
- alt：定义元素的替换文本
- title：定义元素的提示文本
- longdesc：定义元素包含内容的大段描述信息
- cite：定义元素包含内容的引用信息
- datetime：定义元素包含内容的日期和时间

alt和title比较常用，示例：
```html
<a href="URL" title="提示文本">超链接</a>
<img src="URL" alt="替换文本" title="提示文本" />
```
&#8195;&#8195;当图像无法显示时，必须准备替换的文本来替换无法显示的图像，alt属性只能用在img、area和input元素中（包括applet元素）。对于input元素，alt属性用来替换提交按钮的图片：
```html
<input type="image" src="URL" alt="替换文本" />
```
&#8195;&#8195;当浏览器被禁止显示、不支持或者无法下载图像时，通过替换文本可以给看不到图片的浏览者提供文本说明，这是一个很重要的预防和补救措施。如果是修饰性的图片，可以使用空值。

### 其它属性
两个比较实用的其它属性：
- rel：定义当前页面与其它页面的关系。表示从源文档到目标文档的关系
- rev：定义其它页面与当前页面之间的连接关系。表示从目标文档到源文档的关系

&#8195;&#8195;下面示例链接到同一个文件夹中的前一个文档，当搜索引擎检索到rel="prev"信息后，就知道当前文档与所连接的目标文档是平等关系，并处于相同的文件夹中：
```html
<a href="test.html" rel="prev">链接到集合中的前一个文档</a>
```
## HTML5元素
HTML5在HTML4基础上新增了很多元素。
### 结构元素
HTML5定义了一组新的语义化结构来描述网页内容。新增的语义化标记元素如下所示：

元素|说明
:---:|:---
header|表示页面中一个内容区块或整个页面的标题
footer|表示整个页面或页面中一个内容区块的脚注。一般包含创作者姓名、创作日期及联系信息等
section|表示页面中的一个内容区块，如章节、页眉、页脚或页面中其它部分。可以与h1等标题元素结合使用，标识文档结构
article|表示页面中的一块与上下文不相关的独立内容，如博客中的一篇文章或报纸中的一篇文章
aside|表示article元素的内容之外的、与article元素的内容相关的辅助信息
nav|表示页面总导航链接的部分
main|表示网页中的主要内容。主要内容区域指与网页标题或应用程序中本页面主要功能直接相关或进行扩展的内容
figure|表示一段独立的流内容，一般表示文档主体流内容中的一个独立单元

### 功能元素
HTML5新增了很多专用元素，简单说明如下。
####  hgourp元素
用于对整个页面或页面中一个内容区块的标题进行组合。在HTML5.1中废除：
```html
<hgroup>
  <h1>Welcome to my blog</h1>
  <h2>AIX system check</h2>
</hgroup>
```
在HTML4中表示为：
```html
<div>...</div>
```
#### video元素
定义视频，例如电源片段或其它视频流：
```html
<video src="ssn-030.avi" controls="controls">video 元素</video>
```
在HTML4中使用object元素表示。
#### audio元素
定义音频，比如音乐或其它音频流：
```html
<audio src="yesterday once nore.mp3">audio 元素</video>
```
在HTML4中使用object元素表示。
#### embed元素
用来插入各种多媒体，格式可以是Midi、Wav、AIFF、AU、MP3等：
```html
<embed src="home.wav" />
```
在HTML4中使用object元素表示。
#### marky元素
用来在视觉上向用户呈现那些需要突出显示或高亮的显示文本，例如在搜索结构中高亮关键字：
```html
<mark></mark>
```
在HTML4中表示为：
```html
<span></span>
```
#### dialog元素
定义对话框或窗口：
```html
<dialog open>这是打开的对话窗口</dialog>
```
在HTML4中表示为：
```html
<div id="dialog">这是打开的对话窗口</div>
```
#### bdi元素
定义文本的文本方向，使其脱离其周围文本的方向设置：
```html
<ul>
<li>Username <bdi>Thor</bdi>:80 prints</li>
<li>Username <bdi>Batman</bdi>:78 prints</li>
</ul>
```
#### figcaption元素
定义figure元素的标题：
```html
<figure>
    <figcaption>这是一个图片</figcaption>
    <img src="test.jpg" width="500" heigth="800" />
</figure>
```
在HTML4中表示为：
```html
<div id = "figure">
    <h2>这是一个图片</h2>
    <img src="test.jpg" width="500" heigth="800" />
</div>
```
#### time元素
表示日期或时间，也可以同时表示：
```html
<time></time>
```
在HTML4中表示为：
```html
<span></span>
```
#### canvas元素
&#8195;&#8195;表示图形，如图表和其它图像。此元素本身没有行为，仅提供一块画布，但是可以把一个绘图API展现给客户端JavaScript，使脚本能够把想绘制的东西绘制到此画布上：
```html
<canvas id="myCanvas" width="400" heigh="400"></canvas>
```
在HTML4中表示为：
```html
<object data="inc/hdr.svg" type="image/svg+xml" width="400" heigh="400">
</object>
```
#### output元素
表示不同类型的输出，比如脚本的输出：
```html
<output>></output>
```
在HTML4中表示为：
```html
<span></span>
```
#### source元素
为媒介元素（例如video和audio）定义媒介资源：
```html
<source>
```
在HTML4中表示为：
```html
<param>
```
#### menu元素
表示菜单列表。当希望列出表单控件时使用该标签：
```html
<menu>
    <li><input type="checkbox" />Green</li>
    <li><input type="checkbox" />Red</li>
</menu>
```
在HTML4中，menu元素不推荐使用。
#### reby元素
表示ruby注释（中文注音或字符）：
```html
<ruby>美<rt><rp>(</rp>ㄇㄟˇ<rp>)</rp></rt></ruby>
```
#### rt元素
表示字符（中文注音或字符）的解释或发音：
```html
<ruby>美<rt>ㄇㄟˇ</rt></ruby>
```
#### rp元素
在ruby注释中使用，以定义不支持ruby元素的浏览器所显示的内容：
```html
<ruby>美<rt><rp>(</rp>ㄇㄟˇ<rp>)</rp></rt></ruby>
```
