# YAML-基础学习
学习参考链接：
- [YAML version 1.2](https://yaml.org/spec/1.2.2/)
- [YAML Syntax](https://docs.ansible.com/ansible/2.9/reference_appendices/YAMLSyntax.html)
- [runoob.com YAML入门教程](https://www.runoob.com/w3cnote/yaml-intro.html)

## 基本语法
基本语法如下：
- 大小写敏感
- 使用缩进表示层级关系
- 缩进不允许使用tab，只允许空格
- 缩进的空格数不重要，只要相同层级的元素左对齐即可
- `#`表示注释
- 扩展名为`yml`

## 语言概述
### Collections
&#8195;&#8195;YAML的[Block Collection](https://yaml.org/spec/1.2.2/#block-collection-styles)使用[Indentation](https://yaml.org/spec/1.2.2/#indentation-spaces)来表示范围，并在每个条目自的开头。详细说明如下：
- `Block sequences`用破折号和空格`- `表示每个条目
- 映射使用冒号和空格`: `来标记每个键/值对
- 注释以`octothorpe`开头(也称为`hash`、`sharp`、`pound`或数字符号`#`)

标量序列示例：
```yaml
- Thor
- America Captain
- Thanos
```
将标量映射到标量示例：
```yaml
- Asgard: Thor # King of Asgard
- Earth: America Captain # Avengers captain
- Titan: Thanos # Love snaps fingers
```
将标量映射到序列示例：
```yaml
Marvel:
- Thor
- America Captain
- Thanos
Detective Comics:
- Batman
- Catwoman
- Wonder Woman
```
映射序列示例：
```yaml
-
  name: America Captain
  from: New York
  power: 9999
-
  name: Wonder Woman
  from: Amazon
  power: 9998
```
#### 流样式
&#8195;&#8195;YAML也有流样式，使用显式指示符而不是缩进来表示范围。流序列写为方括号内的逗号分隔列表。以类似的方式，流映射使用花括号。

序列中的序列示例：
```yaml
- [name        , from, power  ]
- [America Captain, New York, 9999]
- [Wonder Woman  , Amazon, 9998]
```
映射中的映射示例：
```yaml
America Captain: {from: New York, power: 9999}
Wonder Woman: {
    from: Amazon,
    power: 9998,
 }
```
### 结构