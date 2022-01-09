# RedHat-常用软件安装
## 其它软件
### Cmatrix屏保
下载Cmatrix源码包：[https://github.com/abishekvashok/cmatrix/releases](https://github.com/abishekvashok/cmatrix/releases)      
解压压源码包：
```
[root@VM-0-6-centos tmp]# tar -zxvf cmatrix-2.0.tar.gz
[root@VM-0-6-centos tmp]# cd cmatrix-2.0
[root@VM-0-6-centos cmatrix-2.0]# ./configure
```
Make报错：
```
[root@VM-0-6-centos cmatrix]# make
CDPATH="${ZSH_VERSION+.}:" && cd . && /bin/sh /root/tmp/cmatrix/missing aclocal-1.16 
/root/tmp/cmatrix/missing: line 81: aclocal-1.16: command not found
WARNING: 'aclocal-1.16' is missing on your system.
         You should only need it if you modified 'acinclude.m4' or
         'configure.ac' or m4 files included by 'configure.ac'.
         The 'aclocal' program is part of the GNU Automake package:
         <https://www.gnu.org/software/automake>
         It also requires GNU Autoconf, GNU m4 and Perl in order to run:
         <https://www.gnu.org/software/autoconf>
         <https://www.gnu.org/software/m4/>
         <https://www.perl.org/>
make: *** [aclocal.m4] Error 127
```
automake下载地址：[http://ftp.gnu.org/gnu/automake/](http://ftp.gnu.org/gnu/automake/)
安装：
```
[root@VM-0-6-centos tmp]# tar -zxvf automake-1.16.tar.gz
[root@VM-0-6-centos tmp]# cd automake-1.16
[root@VM-0-6-centos automake-1.16]# ./configure
...
configure: error: Autoconf 2.65 or better is required.
...
```
yum安装Autoconf：
```
[root@VM-0-6-centos automake-1.16]# yum install autoconf
```
安装成功后继续安装automake：
```
[root@VM-0-6-centos automake-1.16]# make
...
help2man: can't get `--help' info from automake-1.16
Try `--no-discard-stderr' if option outputs to stderr
make: *** [doc/automake-1.16.1] Error 255
```
编辑Makefile源文件
```
[root@VM-0-6-centos automake-1.16]# vi Makefile.in
```
修改后如下：
```
doc/aclocal-$(APIVERSION).1: $(aclocal_script) lib/Automake/Config.pm
        $(update_mans) aclocal-$(APIVERSION)
doc/automake-$(APIVERSION).1: $(automake_script) lib/Automake/Config.pm
        $(update_mans) automake-$(APIVERSION) --no-discard-stderr
```
参考链接：[https://blog.csdn.net/developerof/article/details/88206384](https://blog.csdn.net/developerof/article/details/88206384)

再次make，成功：
```
[root@VM-0-6-centos automake-1.16]# make
 cd . && /bin/sh ./config.status Makefile 
config.status: creating Makefile
  GEN      bin/automake
  GEN      bin/aclocal
  GEN      bin/aclocal-1.16
  GEN      bin/automake-1.16
  GEN      t/ax/shell-no-trail-bslash
  GEN      t/ax/cc-no-c-o
  GEN      runtest
  GEN      lib/Automake/Config.pm
  GEN      doc/aclocal-1.16.1
  GEN      doc/automake-1.16.1
  GEN      t/ax/test-defs.sh
[root@VM-0-6-centos automake-1.16]# make install
```
回到Cmatrix目录，继续make，报错：
```
[root@VM-0-6-centos cmatrix]# make
 cd . && /bin/sh /root/tmp/cmatrix/missing automake-1.16 --gnu
"none" is not exported by the List::Util module
Can't continue after import errors at /usr/local/bin/automake-1.16 line 76.
BEGIN failed--compilation aborted at /usr/local/bin/automake-1.16 line 76.
make: *** [Makefile.in] Error 1
```
将提示中文件76行`use List::Util 'none';`改为`use List::Util;`：
```
[root@VM-0-6-centos bin]# vi /usr/local/bin/automake-1.16
```
继续安装Cmatrix：
```
[root@VM-0-6-centos cmatrix]# make
...
cmatrix.c:43:20: fatal error: curses.h: No such file or directory
 #include <curses.h>
                    ^
compilation terminated.
make[1]: *** [cmatrix.o] Error 1
make[1]: Leaving directory `/root/tmp/cmatrix'
```
需要安装libncurses5-dev：
```
[root@VM-0-6-centos cmatrix]# yum install ncurses-libs
[root@VM-0-6-centos cmatrix]# yum install ncurses-devel
```
参考链接：[https://zhidao.baidu.com/question/624814867542104324.html](https://zhidao.baidu.com/question/624814867542104324.html)

继续安装cmatrix：
```
[root@VM-0-6-centos cmatrix]# make
make  all-am
make[1]: Entering directory `/root/tmp/cmatrix'
gcc -DHAVE_CONFIG_H -I.     -g -O2 -MT cmatrix.o -MD -MP -MF .deps/cmatrix.Tpo -c -o c
matrix.o cmatrix.cmv -f .deps/cmatrix.Tpo .deps/cmatrix.Po
gcc  -g -O2   -o cmatrix cmatrix.o  -lncurses  -lncurses
make[1]: Leaving directory `/root/tmp/cmatrix'
[root@VM-0-6-centos cmatrix]# make install
make[1]: Entering directory `/root/tmp/cmatrix'
 /usr/bin/mkdir -p '/usr/local/bin'
  /usr/bin/install -c cmatrix '/usr/local/bin'
 Installing matrix fonts in /usr/lib/kbd/consolefonts...
 /usr/bin/mkdir -p '/usr/local/share/man/man1'
 /usr/bin/install -c -m 644 cmatrix.1 '/usr/local/share/man/man1'
make[1]: Leaving directory `/root/tmp/cmatrix'
[root@VM-0-6-centos cmatrix]# cmatrix
```
至此成功安装。
### 待补充