# SQLite-基础知识笔记
&#8195;&#8195;SQLite是一款轻型的数据库，是遵守ACID的关系型数据库管理系统，它包含在一个相对小的C库中。此处是学习SQLite记的基础笔记。   
SQLite教程：[https://www.runoob.com/sqlite/sqlite-tutorial.html](https://www.runoob.com/sqlite/sqlite-tutorial.html)    
W3school SQL 教程：[https://www.w3school.com.cn/sql/index.asp](https://www.w3school.com.cn/sql/index.asp)    
官方网站：[https://www.sqlite.org/index.html](https://www.sqlite.org/index.html)
## 创建表
### 语法
SQLite中CREATE TABLE语句用于在任何给定的数据库创建一个新表，语法如下：
```sql
CREATE TABLE database_name.table_name(
   column1 datatype  PRIMARY KEY(one or more columns),
   column2 datatype,
   ...
   columnN datatype,
);
```
### 实例
在之前学习Flask过程中使用到过，如下所示：
```sql
DROP TABLE IF EXISTS user;
DROP TABLE IF EXISTS post;

CREATE TABLE user (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  username TEXT UNIQUE NOT NULL,
  password TEXT NOT NULL
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
上面示例中创建了两个表，说明如下：
- DROP TABLE IF EXISTS 表示如果表已经存在就DROP掉
- 首先创建了一个名为user表，ID作为主键
- INTEGER 表示属性类型，后面的TEXT及TIMESTAMP都是类型之一
- UNIQUE 表示此值是唯一的，不能重复
- primary key 表示这个属性是主键
- AUTOINCREMENT 表示这个值是自动增加的，默认的起始值是1
- NOT NULL的约束表示在表中创建纪录时这些字段不能为NULL
- 在created列中，定义了默认值，DEFAULT CURRENT_TIMESTAMP表示获取当前时间戳
- 最后一句是设置外键，作用是保持数据一致性，完整性；控制存储在外键表中的数据，使两张表形成关联

### 注意事项
最近创建时候发现报错：
```
  File "D:\navigator\nav\db.py", line 20, in init_db
    db.executescript(f.read().decode('utf8'))
sqlite3.OperationalError: near ")": syntax error
```
以为Python代码写错了，后来发现是在创建表的SQL语句中，最后一项加了逗号，去掉即可。

### 参考链接
创建表参考教程：
- [RUNOOB.COM SQLite创建表](https://www.runoob.com/sqlite/sqlite-create-table.html)
- [W3school SQL CREATE TABLE 语句](https://www.w3school.com.cn/sql/sql_create_table.asp)

## 待补充
