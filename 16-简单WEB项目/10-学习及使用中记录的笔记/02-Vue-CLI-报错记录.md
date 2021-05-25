# Vue-CLI-报错记录
记录使用过程中遇到的报错及解决方法。
## 版本差异
### 示例一
报错示例：
```
`slot` attributes are deprecated
```
原因是在较新的Vue-CLI版本中使用了`slot-scope`，新版本使用`v-slot`取代了`slot`和`slot-scope`。 

参考链接：
- [ rfcs/active-rfcs/0001-new-slot-syntax.md](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0001-new-slot-syntax.md)
- [vue 插槽，`slot`和 `slot-scope`已被废弃](https://segmentfault.com/a/1190000019683759?utm_source=tag-newest)

## 语法错误
### 示例一
报错示例：
```
[vue/no-multiple-template-root]
The template root requires exactly one element.eslint-plugin-vue
```
vue的模版中只有能一个根节点，在`template`标签中加一个`div`标签即可

参考链接：[https://www.jianshu.com/p/6c6cc02a9001](https://www.jianshu.com/p/6c6cc02a9001)
## 待补充
