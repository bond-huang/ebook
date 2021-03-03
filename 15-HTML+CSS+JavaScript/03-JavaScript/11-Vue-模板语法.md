# Vue-模板语法
&#8195;&#8195;`Vue.js`使用了基于HTML的模板语法，允许开发者声明式地将DOM绑定至底层Vue实例的数据。在底层的实现上，Vue将模板编译成虚拟DOM渲染函数。结合响应系统，Vue能够智能地计算出最少需要重新渲染多少组件，并把DOM操作次数减到最少。

学习过程中记录的笔记，学习教程及参考链接：
- Vue.js官方中文文档：[https://cn.vuejs.org/v2/guide/syntax.html](https://cn.vuejs.org/v2/guide/syntax.html)
- Vue.js runoob网站教程:[https://www.runoob.com/vue2/vue-tutorial.html](https://www.runoob.com/vue2/vue-tutorial.html)

使用在线工具：
- [runoob.com菜鸟教程在线编辑器](https://www.runoob.com/try/try.php?filename=vue2-hw)

## 插值
### 文本
使用“Mustache”语法(双大括号)的文本插值方式：
```html
<!-- 默认可变的插值 -->
<span>Message: {{ message }}</span>
<!-- 一次性地插值 -->
<span v-once>Message: {{ message }}</span>
```
示例说明：
- Mustache标签将会被替代为对应数据对象上`message`的值，当绑定的数据对象上`message`发生了改变，插值处的内容都会更新
- 使用`v-once`指令，能执行一次性地插值，当数据改变时插值处的内容不会更新。注意这会影响到该节点上的其它数据绑定

### 原始HTML
&#8195;&#8195;双大括号会将数据解释为普通文本，而非HTML代码，如果是为了输出HTML代码，需要使用`v-html`指令。例如rawHtml内容为`<span stype="color:blue">I am blue</span>`，示例如下：
```html
<!-- 输出结果会将rawHtml内容作为文本完全展示 -->
<p>Using mustaches: {{ rawHtml }}</p>
<!-- 输出结果只有I am blue，并且字体颜色是蓝色的 -->
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```
### Attribute
Mustache语法不能作用在HTML attribute上，遇到这种情况应该使用`v-bind`指令：
```html
<div v-bind:id="dynamicId"></div>
```
对于布尔attribute(存在值就为true)，`v-bind`工作起来略有不同：
```html
<button v-bind:disabled="isButtonDisabled">Button</button>
```
&#8195;&#8195;如果`isButtonDisabled`的值是null、undefined或false，则`disabled attribute`甚至不会被包含在渲染出来的`button`元素中。
### 使用JavaScript表达式
&#8195;&#8195;目前学习的只绑定简单的property键值。对于所有的数据绑定，`Vue.js`提供了完全的JavaScript表达式支持，示例如下:
```js
{{ number + 1 }}
{{ ok ? 'YES' : 'NO' }}
{{ message.split('').reverse().join('') }}
<div v-bind:id="'list-' + id"></div>
```
无效的情况示例：
```html
<!-- 这是语句，不是表达式 -->
{{ var a = 1 }}
<!-- 流控制也不会生效，请使用三元表达式 -->
{{ if (ok) { return message } }}
```
&#8195;&#8195;每个绑定都只能包含单个表达式，所以上面两个例子不会生效。这些表达式会在所属Vue实例的数据作用域下作为JavaScript被解析。
## 指令
&#8195;&#8195;指令(Directives)是带有`v-`前缀的特殊attribute。指令attribute的值预期是单个JavaScript表达式 (v-for是例外情况)。指令的作用是，当表达式的值改变时，将其产生的连带影响，响应式地作用于DOM。例如之前学习的`v-if`和`v-bind`等。
### 参数
&#8195;&#8195;一些指令能够接收一个参数，在指令名称之后以冒号表示。例如`v-bind`指令可以用于响应式地更新HTML attribute：
```html
<!-- href是参数，告知v-bind指令将该元素的href attribute与表达式url的值绑定 -->
<a v-bind:href="url">...</a>
<!-- v-on指令，用于监听DOM事件，参数是监听的事件名-->
<a v-on:click="doSomething">...</a>
```
### 动态参数
从2.6.0开始，可以用方括号括起来的JavaScript表达式作为一个指令的参数：
```html
<a v-bind:[attributeName]="url"> ... </a>
<a v-on:[eventName]="doSomething"> ... </a>
```
示例说明：
- 示例中attributeName会被作为一个JavaScript表达式进行动态求值，求得的值将会作为最终的参数来使用
- 例如，如果你的Vue实例有一个data property attributeName，其值为"href"，那么这个绑定将等价于 `v-bind:href`
- 第二个示例中，当`eventName`的值为"focus"时，`v-on:[eventName]`等价于`v-on:focus`

#### 对动态参数的值的约束
&#8195;&#8195;动态参数预期会求出一个字符串，异常情况下值为null。这个特殊的null值可以被显性地用于移除绑定。任何其它非字符串类型的值都将会触发一个警告。
#### 对动态参数表达式的约束
&#8195;&#8195;某些字符，如空格和引号，放在HTML attribute名里是无效的。变通的办法是使用没有空格或引号的表达式，或用计算属性替代这种复杂表达式。例如：
```html
<!-- 这会触发一个编译警告 -->
<a v-bind:['foo' + bar]="value"> ... </a>
```
在DOM中使用模板时，需避免使用大写字符来命名键名，因为浏览器会把attribute名全部强制转为小写：
```html
<a v-bind:[someAttr]="value"> ... </a>
```
### 修饰符
&#8195;&#8195;修饰符(modifier)是以半角句号`.`指明的特殊后缀，用于指出一个指令应该以特殊方式绑定。例如`.prevent`修饰符告诉`v-on`指令对于触发的事件调用`event.preventDefault()`：
```html
<form v-on:submit.prevent="onSubmit">...</form>
```
## 缩写
`v-`前缀作为一种视觉提示，用来识别模板中Vue特定的attribute。
### v-bind缩写
示例如下：
```html
<!-- 完整语法 -->
<a v-bind:href="url">...</a>
<!-- 缩写 -->
<a :href="url">...</a>
<!-- 动态参数的缩写(2.6.0版本后) -->
<a :[key]="url"> ... </a>
```
### v-on缩写
示例如下：
```html
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>
<!-- 缩写 -->
<a @click="doSomething">...</a>
<!-- 动态参数的缩写 (2.6.0+) -->
<a @[event]="doSomething"> ... </a>
```
