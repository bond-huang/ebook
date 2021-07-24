# AIX_ksh小脚本
记录学习和使用过程中写的小脚本。
## 进程相关脚本
记录学习和使用过程中写的小脚本。
### 杀掉后台进程
例如杀掉下面的后台进程：
```
# (sleep 50;echo "test")&
[1]     5374152
# kill $!
# ps -ef |grep sleep
    root 12714022        1   0 11:39:27  pts/1  0:00 sleep 50
    root 13172928  6881490   0 11:39:57      -  0:00 /usr/bin/sleep 10
[1] + Terminated               (sleep 50;echo "test")&
```
&#8195;&#8195;直接kill后台运行的进程ID，可以看到后台程序Terminated，但是sleep 50还在运行。这个后台程序在系统中是当前会话作业（jobs），应该是有jobs命令查看，kill也使用杀掉jobs id方式：
```
# (sleep 50;echo "test")&
[1]     7340274
# jobs
[1] +  Running                 (sleep 50;echo "test")&
# kill -STOP %1 
[1] + Stopped (SIGSTOP)        (sleep 50;echo "test")&
# ps -ef |grep sleep
    root 11665562  7340274   0 12:08:08  pts/1  0:00 sleep 50
    root 13172936  6881490   0 12:08:46      -  0:00 /usr/bin/sleep 10
# jobs -l
[1] + 7340274   Stopped (SIGSTOP)        (sleep 50;echo "test")&
# kill %1
# ps -ef |grep sleep
    root 13172984  6881490   0 12:10:06      -  0:00 /usr/bin/sleep 10
[1] + Terminated               (sleep 50;echo "test")&
```
通过脚本自动杀掉：
```sh
(sleep 50;echo "test")&
pro_pid=$!
jobs_id=`jobs -l |awk '/'$pro_pid'/{print $1}'| tr -d "[]"`
kill %$jobs_id
```
通常杀掉是带有条件判断的，具体根据实际情况来使用。

其它相关命令：
- bg：在后台运行作业，可以将后台暂停了的作业在后台继续执行
- fg：在前台运行作业，可以将后台作业放到前台执行，暂停的也可以

## 待补充
