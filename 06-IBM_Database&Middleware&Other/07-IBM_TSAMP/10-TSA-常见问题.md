# TSA-常见问题
## 启停问题
### 停止一直Pending
停止TSA资源组时候，某个应用一直在Pending状态：
- 应用停止脚本运行异常，没能停掉相关应用，手动停止即可
- 有其他进程占用了TSA的相关资源，导致无法停止，查找相关进程

运行命令查看应用相关启停脚本：
```sh
lsrsrc -l IBM.Application
```
## 待补充