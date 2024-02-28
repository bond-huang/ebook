# SQLite-基础知识笔记
&#8195;&#8195;SQLite是一款轻型的数据库，是遵守ACID的关系型数据库管理系统，它包含在一个相对小的C库中。此处是学习SQLite记的基础笔记。   
- SQLite教程：[https://www.runoob.com/sqlite/sqlite-tutorial.html](https://www.runoob.com/sqlite/sqlite-tutorial.html)
- W3school SQL 教程：[https://www.w3school.com.cn/sql/index.asp](https://www.w3school.com.cn/sql/index.asp)
- vue5.com SQLite 教程：[http://www.vue5.com/sqlite/sqlite_select_query.html](http://www.vue5.com/sqlite/sqlite_select_query.html)
- 官方网站：[https://www.sqlite.org/index.html](https://www.sqlite.org/index.html)
- SQLite数据库查看工具：[https://sqlitebrowser.org/](https://sqlitebrowser.org/)

## 基础操作
### 创建数据库
SQLite的sqlite3命令用来创建新的SQLite数据库：
```
$ sqlite3 test.db
```
或者使用`.open`来新建数据库文件：
```
sqlite> .open test.db
```
### 打开数据库
命令同创建，如果数据库存在就会直接打开，示例如下：
```
[root@VM-0-6-centos tmp]# sqlite3 nav.sqlite
SQLite version 3.7.17 2013-05-20 00:56:22
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> .databases
seq  name             file                                                      
---  ---------------  ----------------------------------------------------------
0    main             /tmp/nav.sqlite 
```
### 退出
退出sqlite提示符示例：
```
sqlite> .quit
[root@VM-0-6-centos tmp]# 
```
## CREATE创建表
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

### 参考链接
创建表参考教程：
- [RUNOOB.COM SQLite创建表](https://www.runoob.com/sqlite/sqlite-create-table.html)
- [W3school SQL CREATE TABLE 语句](https://www.w3school.com.cn/sql/sql_create_table.asp)

### 重置序列值
注意表里面没有数据了，SQL说明如下：
```sql
DELETE FROM sqlite_sequence WHERE name = 'table-name'; -- 删除当前序列值记录
UPDATE table-name SET id = NULL; -- 清空所有已存在的ID值
INSERT INTO sqlite_sequence (name, seq) VALUES ('table-name', 0); -- 设置新的起始序列值为0（或其他初始值）
```
示例如下：
```sql
sqlite> select count(*) from links;
0
sqlite> select * from links;
sqlite> DELETE FROM sqlite_sequence WHERE name = 'links';
sqlite> UPDATE links SET id = NULL;
sqlite> INSERT INTO sqlite_sequence (name, seq) VALUES ('links', 0);
sqlite> INSERT INTO links (maincategory, subcategory, urlname, urllocation) VALUES ('IBM','Power System','PowerE980','e980.com');
sqlite> select * from links;      
1|IBM|Power System|PowerE980|e980.com
```
## 常用命令：
### 数据库查看命令
列出数据库的名称及其所依附的文件：
```
sqlite> .databases
main: /home/navusr/navigator/instance/nav.sqlite
```
## Select语句
### 基本语法
基本语法：
```sql
-- 获取指定的字段
SELECT column1, column2, columnN FROM table_name; 
-- 获取所有可用字段
SELECT * FROM table_name;
```
### 示例
获取所有可用字段：
```
sqlite> SELECT * FROM links;
1|IBM|IBM Home Page|IBM官方中文网站|https://www.ibm.com/cn-zh
2|IBM|IBM Home Page|IBM Knowledge Center|https://www.ibm.com/support/knowledgecenter
3|IBM|IBM Home Page|IBM Fix Central|https://www-945.ibm.com/support/fixcentral/
...
```
获取指定字段：
```sql
sqlite> SELECT urlname FROM links;
IBM官方中文网站
IBM Knowledge Center
IBM Fix Central
IBM Software Support
...
```
根据关键字获取数据：
```sql
sqlite> SELECT * FROM links WHERE id = 1;
1|IBM|IBM Home Page|IBM官方中文网站|https://www.ibm.com/cn-zh
sqlite> SELECT * FROM links WHERE subcategory = 'Docker';
64|容器平台|Docker|Docker官方首页|https://www.docker.com/
65|容器平台|Docker|Docker官方文档|https://docs.docker.com/
66|容器平台|Docker|Docker中文社区|https://www.docker.org.cn/index.html
67|容器平台|Docker|Docker runoob教程|https://www.runoob.com/docker/docker-tutorial.html
76|容器平台|Docker|Docker官方博客|https://www.docker.com/blog/
```
获取某一列数据并过滤去重：
```
sqlite> SELECT distinct maincategory FROM links;
IBM
Linux_System
Python
WEB前端
虚拟化平台
容器平台
SAN_Switch
```
获取字段并根据某一列进行排序:
```sql
sqlite> SELECT * FROM links order by maincategory;
1|IBM|IBM Home Page|IBM官方中文网站|https://www.ibm.com/cn-zh
2|IBM|IBM Home Page|IBM Knowledge Center|https://www.ibm.com/support/knowledgecenter
3|IBM|IBM Home Page|IBM Fix Central|https://www-945.ibm.com/support/fixcentral/
...
```
## Insert语句
### 基本语法
INSERT INTO 语句有两种基本语法，如下所示：
```sql
-- 向所有列添加值
INSERT INTO TABLE_NAME VALUES (value1,value2,value3,...valueN);
-- 向指定的列添加值，确保值的顺序与列在表中的顺序一致
INSERT INTO TABLE_NAME (column1, column2, column3,...columnN) 
VALUES (value1, value2, value3,...valueN);
```
### 示例
insert一条数据示例：
```sql
sqlite> INSERT INTO links (maincategory, subcategory, urlname, urllocation) \
VALUES ('IBM','Power System','PowerE980','e980.com');
sqlite> SELECT * FROM links WHERE urlname = 'PowerE980';
77|IBM|Power System|PowerE980|e980.com
sqlite> INSERT INTO links VALUES ('78','IBM','Power System','PowerE880','e880.com');
sqlite> SELECT * FROM links WHERE id = 78;
78|IBM|Power System|PowerE880|e880.com
```
插入多条数据，SQL示例：
```sql
INSERT INTO links (maincategory, subcategory, urlname, urllocation) \
VALUES ('IBM','Power System','PowerE980','e980.com'),('IBM','Power System','PowerE880','e880.com');
```
使用示例：
```sql
sqlite> INSERT INTO links (maincategory, subcategory, urlname, urllocation) VALUES (
'IBM','Power System','PowerE980','e980.com'),('IBM','Power System','PowerE880','e880.com');sqlite> SELECT * FROM links;
281|IBM|Power System|PowerE980|e980.com
282|IBM|Power System|PowerE880|e880.com
```
## Update语句
### 基本语法
基本语法如下
```sql
UPDATE table_name
SET column1 = value1, column2 = value2...., columnN = valueN
WHERE [condition];
```
### 示例
示例如下：
```sql
sqlite> UPDATE links 
SET maincategory = 'IBM', subcategory = 'Storage System', urlname = '
DS8888', urllocation = 'ds8888.com' 
WHERE id = 78;
sqlite> SELECT * FROM links WHERE id = 78;
78|IBM|Storage System|DS8888|ds8888.com
```
## Delete语句
### 基本语法
带有WHERE子句的DELETE查询基本语法：
```sql
DELETE FROM table_name
WHERE [condition];
```
### 示例
删除一条记录示例如下：
```sql
sqlite> DELETE FROM links WHERE id = 78;
sqlite> SELECT * FROM links WHERE id = 78;
sqlite> 
```
## insert or replace
### 基本语法
从table_2中取所有记录INSERT到table_1中：
```sql
INSERT INTO table_1 SELECT * FROM table_2
```
如果两张表有重复的内容，主要是ID一样的：
```sql
INSERT OR REPLACE INTO table_1 SELECT * FROM table_2
```
如果有重复的就REPLACE，合并两张表的方法，注意会覆盖数据，慎重操作。

## 数据库备份及恢复
### 备份数据库
最近发现导航丢数据了，想备份下。首先查找库文件所在目录：
```
[root@centos82 /]# find . -name nav.sqlite
./home/navusr/navigator/instance/nav.sqlite
```
连接数据库备份并查看：
```
[root@centos82 instance]# sqlite3 nav.sqlite
SQLite version 3.26.0 2018-12-01 12:34:55
Enter ".help" for usage hints.
sqlite> .backup backup2310.db
sqlite> .quit
[root@centos82 instance]# ls -l
total 128
-rw-r--r-- 1 root   root   65536 Oct  7 23:23 backup2310.db
-rw-rw-r-- 1 navusr navusr 65536 Oct  7 23:02 nav.sqlite
```
当前数据库记录数量：
```
sqlite> SELECT count(*) FROM links;
188
```
2023年10月7日，一共188条，数据编号到了276，少了几十条，原因不明。
### 恢复数据库
今天发现数据库里面一条数据都没有了：
```
[navusr@centos82 instance]$ sqlite3 nav.sqlite
SQLite version 3.26.0 2018-12-01 12:34:55
Enter ".help" for usage hints.
sqlite> SELECT count(*) FROM links;
0
```
进行恢复，首先打开备份数据库，再进行恢复，示例：
```
[navusr@centos82 instance]$ sqlite3 backup2310.db
SQLite version 3.26.0 2018-12-01 12:34:55
Enter ".help" for usage hints.
sqlite> .restore nav.sqlite
Error: attempt to write a readonly database
```
修改以下权限：
```
[root@centos82 instance]# ls -l
total 128
-rw-r--r-- 1 root   root   65536 Oct  7 23:23 backup2310.db
-rw-rw-r-- 1 navusr navusr 65536 Dec 14 19:07 nav.sqlite
[root@centos82 instance]# chown navusr:navusr backup2310.db
[root@centos82 instance]# chmod 664 backup2310.db
[root@centos82 instance]# ls -l
total 128
-rw-rw-r-- 1 navusr navusr 65536 Oct  7 23:23 backup2310.db
-rw-rw-r-- 1 navusr navusr 65536 Dec 14 19:07 nav.sqlite
```
恢复失败，以下操作是搞反了，备份的数据也没有了，一定要慎重，不要搞反了：
```
[navusr@centos82 instance]$ sqlite3 backup2310.db
SQLite version 3.26.0 2018-12-01 12:34:55
Enter ".help" for usage hints.
sqlite> SELECT count(*) FROM links;
188
sqlite> .restore nav.sqlite
sqlite> .quit
[navusr@centos82 instance]$ sqlite3 nav.sqlite
SQLite version 3.26.0 2018-12-01 12:34:55
Enter ".help" for usage hints.
sqlite> SELECT count(*) FROM links;
0
sqlite> .quit
[navusr@centos82 instance]$ ls -l
total 128
-rw-rw-r-- 1 navusr navusr 65536 Feb 28 00:32 backup2310.db
-rw-rw-r-- 1 navusr navusr 65536 Dec 14 19:07 nav.sqlite
```
## 待补充
