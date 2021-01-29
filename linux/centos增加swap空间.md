linux centos 如何设置swap大小？

swap的值都是安装系统的时候设置好的，一般设置为内存的两倍大小。使用过程中发现swap值过小只能添加。
用free -m 命令查看当前swap大小

1、使用下面的命令创建2G的空间

dd if=/dev/zero of=/var/swap bs=1024 count=2048000

if 表示infile，of表示outfile，bs=1024代表增加的模块大小，count=2048000代表2048000个模块，也就是2G空间


2、将目的文件设置为swap分区文件


mkswap /var/swap


3、激活swap，立即启用交换分区文件


mkswap -f /var/swap


free -m查看swap已经增加了，但这只是临时性的，如果机器重启会失效


4、vi /etc/fstab
最后一行添加
/var/swap swap swap defaults 0 0
重启或free -m测试 swap添加成功

下面是实战命令：


[root@host ~]# free -m


total used free shared buff/cache available
Mem: 1006 381 190 42 434 427
Swap: 259 94 165


[root@host ~]# dd if=/dev/zero of=/var/swap bs=1024 count=2048000
2048000+0 records in
2048000+0 records out
2097152000 bytes (2.1 GB) copied, 8.71197 s, 241 MB/s


[root@host ~]# mkswap /var/swap
Setting up swapspace version 1, size = 2047996 KiB
no label, UUID=941931fe-683b-4082-a6db-82bb741f77e5


[root@host ~]# mkswap -f /var/swap
mkswap: /var/swap: warning: wiping old swap signature.
Setting up swapspace version 1, size = 2047996 KiB
no label, UUID=17a9a21e-67d2-4343-b95d-d8958814c334


[root@host ~]# swapon /var/swap
swapon: /var/swap: insecure permissions 0644, 0600 suggested.


[root@host ~]# free -m
total used free shared buff/cache available
Mem: 1006 388 65 42 552 417
Swap: 2259 94 2165


[root@host ~]# cat /proc/swaps
Filename Type Size Used Priority
/swap file 266236 96512 -2
/var/swap file 2047996 0 -3
[root@host ~]# vim /etc/fstab


最后一行添加
/var/swap swap swap defaults 0 0


如果不再需要swap，可以清理该分区： 
[root@mysql01 var]# swapoff /var/swap

 

/var/log 下的messages日志可以看出内存不够杀死的进程。