# 组件_深入了解
学习官方文档过程中记录的笔记。
# 组件_边界情况
官方参考文档链接：[处理边界情况](https://cn.vuejs.org/v2/guide/components-edge-cases.html)
## 循环引用
使用到实例：在`System-Health-Management`项目中的`Aside.vue`和`ChildrenMenu.vue`组件中使用到了。
### 递归组件
组件是可以在它们自己的模板中调用自身的，必需要设置`name`属性：
```js
export default {
  name: 'TestComponent'
}
```
使用`Vue.component`全局注册一个组件时，这个全局的`ID`会自动设置为该组件的`name`选项:
```js
Vue.component('TestComponent', {
  // ...
})
```
&#8195;&#8195;需要确保递归调用是条件性的 (例如使用最终会得到`false`的`v-if`)。直接引用不加条件时候，会导致`max stack size exceeded`错误，陷入死循环。
#### 实例
