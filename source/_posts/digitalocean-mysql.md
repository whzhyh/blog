title: 'DigitalOcean mysql无法启动'
tags:
  - mysql
categories:
  - database
date: 2014-01-19 22:34:09
---

看到DigitalOcean的vps价格不贵，想装来试试，如果效果好的话，就把godaddy上的博客迁移过来。但是装好mysql之后，无法启动，让我很是郁闷。

下面我们来看下错误日志吧，大概就是这个样子的：
<pre>
131005 14:11:41 [Note] Plugin 'FEDERATED' is disabled.
131005 14:11:41 InnoDB: The InnoDB memory heap is disabled
131005 14:11:41 InnoDB: Mutexes and rw_locks use GCC atomic builtins
131005 14:11:41 InnoDB: Compressed tables use zlib 1.2.3.4
131005 14:11:41 InnoDB: Initializing buffer pool, size = 128.0M
InnoDB: mmap(135987200 bytes) failed; errno 12
131005 14:11:41 InnoDB: Completed initialization of buffer pool
131005 14:11:41 InnoDB: Fatal error: cannot allocate memory for the buffer pool
131005 14:11:41 [ERROR] Plugin 'InnoDB' init function returned error.
131005 14:11:41 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
131005 14:11:41 [ERROR] Unknown/unsupported storage engine: InnoDB
</pre>

开始总以为是mysql配置的问题，后来经过研究，居然是因为没有开启 Swap Space。同样的问题其他人也遇到过。不得不说，这应该算是DigitalOcean的bug吧。

开启Swap Space的过程略复杂，我就偷下懒，把别人的方法copy过来了。

## Check for Swap Space

* * *

Before we proceed to set up a swap file, we need to check if any swap files have been enabled on the VPS by looking at the summary of swap usage.
<pre>sudo swapon -s</pre>
An empty list will confirm that you have no swap files enabled:
<pre>Filename				Type		Size	Used	Priority</pre>

## Check the File System

* * *

After we know that we do not have a swap file enabled on the virtual server, we can check how much space we have on the server with the `df`command. The swap file will take 512MB— since we are only using up about 8% of the /dev/sda, we can proceed.
<pre>df
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda        20907056 1437188  18421292   8% /
udev              121588       4    121584   1% /dev
tmpfs              49752     208     49544   1% /run
none                5120       0      5120   0% /run/lock
none              124372       0    124372   0% /run/shm</pre>

## Create and Enable the Swap File

* * *

Now it’s time to create the swap file itself using the dd command :
<pre>sudo dd if=/dev/zero of=/swapfile bs=1024 count=512k</pre>
“of=/swapfile” designates the file’s name. In this case the name is swapfile.

Subsequently we are going to prepare the swap file by creating a linux swap area:
<pre>sudo mkswap /swapfile</pre>
The results display:
<pre>Setting up swapspace version 1, size = 262140 KiB
no label, UUID=103c4545-5fc5-47f3-a8b3-dfbdb64fd7eb</pre>
Finish up by activating the swap file:
<pre>sudo swapon /swapfile</pre>
You will then be able to see the new swap file when you view the swap summary.
<pre>swapon -s
Filename				Type		Size	Used	Priority
/swapfile                               file		262140	0	-1</pre>
This file will last on the virtual private server until the machine reboots. You can ensure that the swap is permanent by adding it to the fstab file.

Open up the file:
<pre>sudo nano /etc/fstab</pre>
Paste in the following line:
<pre> /swapfile       none    swap    sw      0       0</pre>
Swappiness in the file should be set to 0\. Skipping this step may cause both poor performance, whereas setting it to 0 will cause swap to act as an emergency buffer, preventing out-of-memory crashes.

You can do this with the following commands:
<pre>echo 0 | sudo tee /proc/sys/vm/swappiness
echo vm.swappiness = 0 | sudo tee -a /etc/sysctl.conf</pre>
To prevent the file from being world-readable, you should set up the correct permissions on the swap file:
<pre>sudo chown root:root /swapfile
sudo chmod 0600 /swapfile</pre>

参考资料：[https://www.digitalocean.com/community/articles/how-to-add-swap-on-ubuntu-12-04](https://www.digitalocean.com/community/articles/how-to-add-swap-on-ubuntu-12-04)
