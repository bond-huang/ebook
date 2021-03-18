# SQLite-常见问题
记录一些使用过程中常见问题。
## 常见报错
### 创建表报错
创建时候发现报错：
```
  File "D:\navigator\nav\db.py", line 20, in init_db
    db.executescript(f.read().decode('utf8'))
sqlite3.OperationalError: near ")": syntax error
```
以为Python代码写错了，后来发现是在创建表的SQL语句中，最后一项加了逗号，去掉即可。
### UNIQUE问题
报错提示：
```
sqlite3.IntegrityError: UNIQUE constraint failed: links.urlname
```
这个错误是因为在建表的时候设定了UNIQUE这个键，当出现重复的时候就报错。

## locked锁表
### 示例一
报错如下：
```
sqlite3.OperationalError: database is locked
```
因为sqlite只支持单线程操作，导致锁表操作不详，重启应用即可。

## 数据异常
### 数据丢失
&#8195;&#8195;最近发现SQLite丢数据了。我将项目文件包括数据库文件拷贝到TXY CentOS宿主系统上，然后拷贝到容器nav，没有用nginx，使用waitress运行成功后添加了不少数据，大概了七十多条。后来想使用nginx，做了个镜像到navigator，nav就没动过了，然后在navigator里面调试nginx和uwsgi，成功后发现数据少了十几条，然后回到nav，发现数据少了二十多条，少的内容不一样。

一开始怀疑是我频繁使用`kill -9`导致数据库损坏，参考网上链接进行处理：
```
[root@cd414072bc2b instance]# sqlite3 nav.sqlite .dump > news.sqlite
[root@cd414072bc2b instance]# ls
nav.sqlite  news.sqlite
[root@cd414072bc2b instance]# cat news.sqlite
```
查看一下也没发现异常。参考链接：[https://blog.csdn.net/m0_37168878/article/details/89430040](https://blog.csdn.net/m0_37168878/article/details/89430040)

&#8195;&#8195;使用SQLite数据库查看软件也是少了数据，但是没发现什么异常，最后找到以前的数据表，和现有的合并一下。使用的DB Browser for SQLite软件，先把数据少的那个库里面表导出来，需要勾上“第一列列名”，导出后把表导入到数据多的那个库里面，执行语句：
```sql
insert into links select * from temp
Result: UNIQUE constraint failed: links.id
At line 1:
insert into links select * from temp
```
报错了，因为有重复的id的数据，使用下面方法：
```sql
insert or replace into links select * from temp
Result: 查询执行成功。耗时 0ms54 行数据受影响
At line 1:
insert or replace into links select * from temp
```
删掉不需要的表保存即可。

数据丢失的原因还没找到，通过此方法恢复了数据，也算是学到一点，技术很重要，备份也很重要。

## 待补充
