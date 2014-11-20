title: 服务器python升级
id: 38
categories:
  - Uncategorized
tags:
---

# 服务器python升级

服务器的操作系统是CentOS 5.4，python 默认版本是2.4.3，有些老了，需要升级到2.7

`[root@dev wanghz]# lsb_release -a
LSB Version:    :core-3.1-amd64:core-3.1-ia32:core-3.1-noarch:graphics-3.1-amd64:graphics-3.1-ia32:graphics-3.1-noarch
Distributor ID: CentOS
Description:    CentOS release 5.4 (Final)
Release:    5.4
Codename:   Final`

以下是安装步骤：

`wget http://legacy.python.org/ftp//python/2.7.6/Python-2.7.6.tgz
tar zxvf Python-2.7.6.tgz
mkdir /usr/local/python2.7.6
cd Python-2.7.6
./configure --prefix=/usr/local/python2.7.6
make
make install`

此时2.7.6版的python已经安装好了，让我们看一下python的版本

`[root@dev wanghz]# python -V
Python 2.4.3`

这说明目前系统默认的python版本依然是2.4.3，现在开始设置系统默认的python到2.7.6.

首先查看python命令所在目录

`[root@dev wanghz]# whereis python
python: /usr/bin/python /usr/bin/python2.4 /usr/lib/python2.4 /usr/include/python2.4 /usr/share/man/man1/python.1.gz`

将/usr/bin/python 的软连接修改为python2.4.3

`[root@dev wanghz]# mv /usr/bin/python /usr/bin/python2.4.3
[root@dev wanghz]# python -V
bash: /usr/bin/python: No such file or directory`

python命令找不到，这时只需要将版本python2.7.6命令加入环境变量即可！

方法1.
修改/etc/profile加入如下两行：

`PATH=$PATH:/usr/local/python2.7.6/bin
export PATH`

然后

`[root@dev wanghz]# source /etc/profile
[root@dev wanghz]# python -V
Python 2.7.6`

方法2.
当然也可以创建2.7.6版本的python的软连接

`[root@dev wanghz]# ln -s /usr/local/python2.7.6/bin/python /usr/bin/python`

ok，python升级完成

到现在为止，还有最后一件事需要做，那就是yum与python的兼容问题：

```
[root@localhost pipe]# vi /usr/bin/yum

# !/usr/bin/python

```

第一行修改为

```

# !/usr/bin/python2.4.3（即原始的python变更后的名字）

```

* * *

# 安装Trac

### 安装pip

`wget https://raw.github.com/pypa/pip/master/contrib/get-pip.py
python get-pip.py`

有可能会遇到如下错误

> 1.  `zipimport.ZipImportError: can't decompress data; zlib not available`
> 
>       安装zlib

`wget http://zlib.net/zlib-1.2.8.tar.gz
tar xvf zlib-1.2.8.tar.gz
cd zlib-1.2.8
./configure
make 
make install`

> > 这个错误：`ImportError: cannot import name HTTPSHandler`
> > 
> >     yum安装openssl和openssl-devel。然后重新编译python。

### 安装trac，并新建项目

`pip install trac
trac-admin /path/to/myproject initenv`

> 有可能遇到如下错误

`Creating and Initializing Project
Initenv for '/home/whzhyh/project1' failed. 
Failed to create environment.
Cannot load Python bindings for SQLite`

解决方案：

`yum install sqlite-devel.x86_64 --enablerepo=epel
pip install pysqlite`

部署trac

http://trac.edgewall.org/wiki/TracModWSGI

/home/whzhyh/proj/trac.wsgi

```
import os

os.environ['TRAC_ENV_PARENT_DIR'] = '/home/whzhyh/proj'
os.environ['PYTHON_EGG_CACHE'] = '/home/whzhyh/proj/eggs'

import trac.web.main
application = trac.web.main.dispatch_request

import site
site.addsitedir('/usr/local/lib/python2.7/site-packages')
`/etc/httpd/conf/httpd.conf`
<VirtualHost 127.0.0.1:80>
WSGIScriptAlias /trac /home/whzhyh/proj/trac.wsgi

<Directory /home/whzhyh/proj>
    WSGIApplicationGroup %{GLOBAL}
    Order deny,allow
    Allow from all
</Directory>
</VirtualHost>
```

### 安装mod_wsgi

wget https://modwsgi.googlecode.com/files/mod_wsgi-3.4.tar.gz

./configure报错

```
checking for apxs2... no
checking for apxs... no
checking Apache version... ./configure: line 1704: apxs: command not found
./configure: line 1704: apxs: command not found
./configure: line 1705: apxs: command not found
./configure: line 1708: /: is a directory

checking for python... /usr/bin/python
./configure: line 1877: apxs: command not found
```

yum install httpd-devel --enablerepo=epel

make报错

```
/usr/lib64/apr-1/build/libtool --silent --mode=link gcc -o mod_wsgi.la  -rpath /usr/lib64/httpd/modules -module -avoid-version    mod_wsgi.lo -L/usr/local/berkeleydb/lib -L/usr/local/lib -L/usr/local/lib/python2.7/config -lpython2.7 -lpthread -ldl -lutil -lm

/usr/bin/ld: /usr/local/lib/libpython2.7.a(abstract.o): relocation R_X86_64_32 against `.rodata.str1.8' can not be used when making a shared object; recompile with -fPIC

/usr/local/lib/libpython2.7.a: could not read symbols: Bad value
collect2: ld returned 1 exit status
apxs:Error: Command failed with rc=65536
.
make: *** [mod_wsgi.la] 错误 1
```
这是32位和64位版本搞混的问题

参考http://code.google.com/p/modwsgi/wiki/InstallationIssues

需要重新编译python

`./configure --enable-shared
make 
make install`

`vi /etc/httpd/conf/httpd.conf`, 并在其中加入

`LoadModule wsgi_module modules/mod_wsgi.so`

然后`service httpd restart`, 报如下错误

`Starting httpd: httpd: Syntax error on line 202 of /etc/httpd/conf/httpd.conf: Cannot load /etc/httpd/modules/mod_wsgi.so into server: libpython2.7.so.1.0: cannot open shared object file: No such file or directory`

解决方案参考：http://hi.baidu.com/sebastianwade/item/f3f05d038d0ba4cb2f4c6b66

`echo " /usr/local/lib" &gt;&gt; /etc/ld.so.conf
ldconfig`

`service httpd restart` 成功启动

遇到一个恶心的问题，trac.wsgi这个文件放在/home/下，就无法运行，最终移到/var/www/下就可以了，诡异，浪费了我很长时间，也不知道是哪的限制。

这个链接说的好像是一个问题：http://stackoverflow.com/questions/4807176/apache-mod-wsgi-error-forbidden-you-dont-have-permission-to-access-on-this-s

```
[Wed Mar 05 23:25:16 2014] [error] [client 10.211.55.2] mod_wsgi (pid=10260): Exception occurred processing WSGI script '/var/www/trac.wsgi'.
[Wed Mar 05 23:25:16 2014] [error] [client 10.211.55.2] Traceback (most recent call last):
[Wed Mar 05 23:25:16 2014] [error] [client 10.211.55.2]   File "/var/www/trac.wsgi", line 6, in <module>
[Wed Mar 05 23:25:16 2014] [error] [client 10.211.55.2]     import trac.web.main
[Wed Mar 05 23:25:16 2014] [error] [client 10.211.55.2]   File "/usr/local/lib/python2.7/site-packages/trac/**init**.py", line 14, in <module>
[Wed Mar 05 23:25:16 2014] [error] [client 10.211.55.2]     from pkg_resources import DistributionNotFound, get_distribution
[Wed Mar 05 23:25:16 2014] [error] [client 10.211.55.2] ImportError: No module named pkg_resources

```
http://stackoverflow.com/questions/7446187/no-module-named-pkg-resources

又一个权限问题
`OSError: [Errno 13] Permission denied: '/var/trac/proj'`