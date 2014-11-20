title: 'vsftp 配置'
id: 24
categories:
  - linux
date: 2014-01-20 02:23:41
tags:
---

在vps上装了个了wordpress，想把博客上的内容转过来，需要装一个Wordpress Importer插件。居然还要ftp权限。好吧，我来装一个。

系统是sentos，vsftp可以直接由yum来安装, `yum install vsftpd`

启动一下，看看是不是正常安装了。

`service vsftpd start`

启动成功，而且匿名用户访问成功。

接下来我们要添加用户，总不能只用匿名用户吧，而且权限也不够，这个vsftp的用户系统似乎是跟linux的用户系统关联的。
```sh
useradd hongzheng #添加用户
passwd hongzheng #修改密码
```

之后要修改vsftpd.conf, `vi /etc/vsftpd/vsftpd.conf`
```
anonymous_enable=NO  #禁止匿名访问
userlist_deny=NO #(这条需手动添加到最后)使用FTP用户表，表里没有的用户需要添加才能登录
```

`vi /etc/vsftpd/user_list` #将里面其它初始用户全部删除，加入刚刚我们新建的hongzheng用户。

再测试一下，成功登录！哈哈，但是默认的目录是哪个呢，我是要改wordpress，所以要把目录设置在/var/www/html。`vi /etc/vsftpd/vsftpd.conf`
```sh
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list #被列入此文件的用户，在登录后将不能切换到自己目录以外的其他目录从而有利于FTP服务器的安全管理和隐私保护。此文件需自己建立
user_config_dir=/etc/vsftpd/userconf
```

`vi /etc/vsftpd/chroot_list` 创建chroot_list这个文件，似乎这个文件只要置为空就可以了，那就先这样吧。
`mkdir /etc/vsftpd/userconf` 这个文件夹里，要放入每个用户的配置文件。
`vi /etc/vsftpd/userconf/hongzheng` 新建以用户名同名的文件

在里面指定
```sh
local_root=/var/www/html
```

重新启动，能成功登录了，目录也是对的，还以为一切搞定呢，似乎没有写的权限，估计是hongzheng这个用户对/var/www没有写权限导致的。

`chmod 777 -R /var/html` 这样做可能安全性不好，就先这样吧。

现在一切都正常了。

参考资料：

[http://jingyan.baidu.com/article/adc815133476bdf723bf7393.html](http://jingyan.baidu.com/article/adc815133476bdf723bf7393.html)

[http://www.cnblogs.com/alex-blog/articles/2652337.html](http://www.cnblogs.com/alex-blog/articles/2652337.html)

[http://blog.163.com/xiaohui_1123@126/blog/static/398052402010101995025953/](http://blog.163.com/xiaohui_1123@126/blog/static/398052402010101995025953/)
