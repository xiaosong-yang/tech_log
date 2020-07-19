&ensp;&ensp;&ensp;&ensp;今天docker通过-p给容器追加端口时报错:WARNING: IPv4 forwarding is disabled. Networking will not work。下面记录一下结局方法：
1. 在etc目录下有一个sysctl.conf的配置文件。
```
vim /etc/sysctl.conf
```
在文件的最后一行添加一句配置：net.ipv4.ip_forward=1，然后保存。

2. 重启网络服务
```
systemctl restart network
或者
service network restart
```
3. 删除之前创建的容器，然后重新创建该容器。