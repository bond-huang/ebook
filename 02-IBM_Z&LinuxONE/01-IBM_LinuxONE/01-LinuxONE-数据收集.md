# LinuxONE-数据收集
IBM LinuxONE目前关于硬件及其维护维护的资料都是内部的，未在knowledge center中,所以此处记笔记只会记录IBM公开的信息。对于用户层面的配置以及运维资料很多，都是公开的，knowledge center和红皮书都有很多介绍，主页LinuxONE简介中有提供相关链接。个人也是初步接触在学习中，
操作系统层面故障诊断和排除官方链接:[Linux on Z and LinuxONE](https://www.ibm.com/support/knowledgecenter/linuxonibm/liaaf/lnz_r_main.html)

### 操作系统数据收集
使用dbginfo.sh脚本收集基本的诊断信息。官方介绍：[dbginfo.sh脚本](https://www.ibm.com/support/knowledgecenter/linuxonibm/com.ibm.linux.z.lxsv/lxsv_ts_tool_dbginfo.html)

根据系统类型收集对应的信息：
- SUSE Linux Enterprise Server上运行`supportconfig`,官方介绍：[supportconfig](https://www.ibm.com/support/knowledgecenter/linuxonibm/com.ibm.linux.z.lxsv/lxsv_ts_tool_supportconfig.html)
- Red Hat Enterprise Linux上运行`sosreport`,官方介绍：[sosreport](https://www.ibm.com/support/knowledgecenter/linuxonibm/com.ibm.linux.z.lxsv/lxsv_ts_tool_sosreport.html)
- Ubuntu Server上运行`sosreport`
