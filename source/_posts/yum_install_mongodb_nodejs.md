title: '用yum安装mongodb, nodejs'
id: 19
categories:
  - Uncategorized
date: 2014-01-19 16:56:57
tags:
---

最近装了个vps，想尽可能用yum来管理软件包。

Centos里yum默认的源里面的包数量有限，版本更新不及时。

1\. mongodb

我找到了mongodb官方的源，以下是安装过程：
<pre>vi /etc/yum.repos.d/10gen.repo</pre>
里面写入内容
<pre>[10gen] 
name=10gen Repository 
baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64 
gpgcheck=0</pre>
之后运行
<pre>yum install mongo-10gen-server mongo-10gen</pre>
启动mongodb
<pre>service mongod start</pre>
2\. nodejs

我找到了另外一个源，这个源是由 Fedora 社区打造，为 RHEL 及衍生发行版如 CentOS、Scientific Linux 等提供高质量软件包的项目。我首先就应该用这个来安装的，只怪自己孤陋寡闻。
<pre>wget http://download-i2.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
sudo rpm -ivh epel-release-6-8.noarch.rpm
yum install nodejs.x86_64 --enablerepo=epel
yum install npm.noarch</pre>