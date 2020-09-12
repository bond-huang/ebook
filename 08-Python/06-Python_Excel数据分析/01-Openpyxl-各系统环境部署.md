# Openpyxl-各系统环境部署
在不同的系统环境中安装Openpyxl过程。

### Openpyxl相关包下载
老版本openpyxl下载地址：[http://pypi.doubanio.com/simple/openpyxl](http://pypi.doubanio.com/simple/openpyxl)       
最新版openpyxl下载地址：[https://pypi.org/project/openpyxl/#files](https://pypi.org/project/openpyxl/#files)       
jdcal下载地址：[https://pypi.org/project/jdcal/#files](https://pypi.org/project/jdcal/#files)        
et_xmlfile下载地址：[https://pypi.org/project/et_xmlfile/#files](https://pypi.org/project/et_xmlfile/#files)

### windows下安装
系统版本：Windows 10 企业版，Python版本：Python 3.8.3              
首先升级pip（直接安装提示pip版本低），示例如下：
```
C:\Users\Bond>python -m pip install --upgrade pip
Collecting pip
  Downloading https://files.pythonhosted.org/packages/4e/5f/528232275f6509b1fff703c9280e58951a81abe24640905de621c9f81839/pip-20.2.3-py2.py3-none-any.whl (1.5MB)
     |████████████████████████████████| 1.5MB 8.2kB/s
Installing collected packages: pip
  Found existing installation: pip 19.2.3
    Uninstalling pip-19.2.3:
      Successfully uninstalled pip-19.2.3
Successfully installed pip-20.2.3
```
然后安装openpyxl：
```
C:\Users\Bond>pip install openpyxl
Collecting openpyxl
  Using cached openpyxl-3.0.5-py2.py3-none-any.whl (242 kB)
Collecting et-xmlfile
  Using cached et_xmlfile-1.0.1.tar.gz (8.4 kB)
Collecting jdcal
  Downloading jdcal-1.4.1-py2.py3-none-any.whl (9.5 kB)
Using legacy 'setup.py install' for et-xmlfile, since package 'wheel' is not installed.
Installing collected packages: et-xmlfile, jdcal, openpyxl
    Running setup.py install for et-xmlfile ... done
Successfully installed et-xmlfile-1.0.1 jdcal-1.4.1 openpyxl-3.0.5
```
成功后在IDLE中测试：
```
Python 3.8.3 (tags/v3.8.3:6f8c832, May 13 2020, 22:37:02) [MSC v.1924 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license()" for more information.
>>> from openpyxl import load_workbook
>>> from openpyxl import workbook
>>> 
```
### AIX下安装
系统版本：7100-04-03-1642，Python版本：Python 3.7.6         
将下面三个安装包拷贝到系统中：
- jdcal-1.4.1.tar.gz
- et_xmlfile-1.0.1.tar.gz
- openpyxl-3.0.5.tar.gz

安装jdcal步骤：
```
# gunzip jdcal-1.4.1.tar.gz
# tar -xvf jdcal-1.4.1.tar
# cd jdcal-1.4.1
# python3 setup.py install
```
安装et_xmlfile步骤：
```
# gunzip et_xmlfile-1.0.1.tar.gz
# tar -xvf et_xmlfile-1.0.1.tar
# cd et_xmlfile-1.0.1
# python3 setup.py install
```
安装openpyxl步骤：
```
# gunzip openpyxl-3.0.5.tar.gz
# tar -xvf openpyxl-3.0.5.tar
# cd openpyxl-3.0.5
# python3 setup.py install
```
安装成功后测试下：
```
# python3
Python 3.7.6 (default, Feb 28 2020, 04:49:11) 
[GCC 8.3.0] on aix6
Type "help", "copyright", "credits" or "license" for more information.
>>> from openpyxl import workbook
>>> 
```
### Linux下安装
#### 安装步骤
操作系统版本：RHEL8.0，Python版本：Python 3.6.8       
安装步骤基本和AIX一致（解压简单点）：
```
[root@redhat8 tmp]# tar -zxvf jdcal-1.4.1.tar.gz
[root@redhat8 tmp]# cd jdcal-1.4.1
[root@redhat8 jdcal-1.4.1]# chmod 755 setup.py
[root@redhat8 jdcal-1.4.1]# python3 setup.py install

[root@redhat8 tmp]# tar -zxvf et_xmlfile-1.0.1.tar.gz
[root@redhat8 tmp]# cd et_xmlfile-1.0.1
[root@redhat8 et_xmlfile-1.0.1]# python3 setup.py install

[root@redhat8 tmp]# tar -zvxf openpyxl-3.0.5.tar.gz
[root@redhat8 tmp]# cd openpyxl-3.0.5
[root@redhat8 openpyxl-3.0.5]# python3 setup.py install   
```
安装成功后测试下：
```
[root@redhat8 openpyxl-3.0.5]# python3
Python 3.6.8 (default, Jan 11 2019, 02:17:16) 
[GCC 8.2.1 20180905 (Red Hat 8.2.1-3)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from openpyxl import workbook
>>> 
```
#### 安装报错示例
在刚开始安装过程中有报错,无法安装，示例如下：
```
[root@redhat8 jdcal-1.4.1]# python3 setup.py install
running install
error: can't create or remove files in install directory

The following error occurred while trying to add or remove files in the
installation directory:

    [Errno 2] No such file or directory: '/usr/local/lib/python3.6/site-packages/test-easy-install-4302.write-test'

The installation directory you specified (via --install-dir, --prefix, or
the distutils default setting) was:

    /usr/local/lib/python3.6/site-packages/

This directory does not currently exist.  Please create it and try again, or
choose a different installation directory (using the -d or --install-dir
option).
```
#### 解决方法
首先在安装文件所在目录下运行如下命令：
```
[root@redhat8 jdcal-1.4.1]# python3 -m site
sys.path = [
    '/tmp/jdcal-1.4.1',
    '/usr/lib64/python36.zip',
    '/usr/lib64/python3.6',
    '/usr/lib64/python3.6/lib-dynload',
    '/usr/lib64/python3.6/site-packages',
    '/usr/lib/python3.6/site-packages',
]
USER_BASE: '/root/.local' (exists)
USER_SITE: '/root/.local/lib/python3.6/site-packages' (doesn't exist)
ENABLE_USER_SITE: True
```
&#8195;&#8195;看报错创建不了文件，可能是权限问题，但是我使用的是root用户，还是检查了/usr下lib和lib64目录的权限，没有写权限，还是修改了以下权限：
```
[root@redhat8 usr]# chmod -R 755 lib
[root@redhat8 usr]# chmod -R 755 lib64
```
&#8195;&#8195;尝试安装还是报错，参照如下文档将python3 -m site输出中USER_SITE提示的目录看来下确实not exist，然后创建了一个目录：
```
[root@redhat8 site-packages]# pwd
/root/.local/lib/python3.6/site-packages
```
参照文档:[Error “can't create or remove files in install directory” when installing by source code in Python](https://stackoverflow.com/questions/60727752/error-cant-create-or-remove-files-in-install-directory-when-installing-by-sou)

&#8195;&#8195;报错中还或说目录不存在，并且提示用户可以用命令行参数去指定目录，看了下目录确实不存在，根据提示新建了一个目录：
```
[root@redhat8 site-packages]# pwd
/usr/local/lib/python3.6/site-packages
```
再次运行在安装文件所在目录下`python3 -m site`命令：
```
[root@redhat8 python3.6]# python3 -m site
sys.path = [
    '/root/.local/lib/python3.6',
    '/usr/lib64/python36.zip',
    '/usr/lib64/python3.6',
    '/usr/lib64/python3.6/lib-dynload',
    '/root/.local/lib/python3.6/site-packages',
    '/usr/lib64/python3.6/site-packages',
    '/usr/lib/python3.6/site-packages',
]
USER_BASE: '/root/.local' (exists)
USER_SITE: '/root/.local/lib/python3.6/site-packages' (exists)
ENABLE_USER_SITE: True
```
然后安装可以正常进行了。
