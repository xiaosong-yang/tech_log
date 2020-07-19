### 获取linux的ip地址
&ensp;&ensp;&ensp;&ensp;如果是阿里云或者什么服务器，正常情况下空台上都会告诉你该服务器的公网ip，如果你要在linux中获取ip地址（就像window中用ipconfig获取一样）。linux中的命令如下：
```shell
ip a s   #这是命令简写版，其实等价于ip address show
```
![ip_a_s](http://image.public.yyf256.top//blog/technical/ip_a_s.png)

&ensp;&ensp;&ensp;&ensp;你可以通过命令ifconfig(和window中ipconfig差一个字母),但是这个命令是需要安装的，不是自带的，安装过程如下（很简单）。
##### ifconfig命令的安装
&ensp;&ensp;&ensp;&ensp;通过yum进行安装
![install_ifconfig](http://image.public.yyf256.top//blog/technical/install_ifconfig.png)
通过yum search 这个命令我们发现ifconfig这个命令是在net-tools.x86_64这个包里，接下来我们安装这个包就行了,命令：yum install net-tools.x86_64。安装完成后，再次使用ifconfig -a命令就可以查看到所有的网卡了。
![ifconfig_use](http://image.public.yyf256.top//blog/technical/ifconfig_use.png)
我们可以看到我们的linux的ip地址是192.168.1.101。


### 确认sshd服务
&ensp;&ensp;&ensp;&ensp;确认好ip之后，我们就需要确认linux上是否开启了linux的ssh服务。在linux通过命令：yum list installed | grep openssh-server，判断yum是否已经安装过了openssh-server。需要注意的是，如果你不是通过yum安装的，那这种确认方式不适合你。

&ensp;&ensp;&ensp;&ensp;已经安装之后我们需要检查sshd的配置，配置路径如下：/etc/ssh/sshd_config。我们通过vi进入，检查一下几项是否正确：
![密码配置](http://image.public.yyf256.top/ssh_passowrd_authentication_set.jpg)
![端口配置](http://image.public.yyf256.top/ssh_port_set.jpg)
![远程许可配置](http://image.public.yyf256.top/ssh_remote_allow_set.jpg)


&ensp;&ensp;&ensp;&ensp;确认和修改后，保存文本，并对sshd服务进行启动或者是重启,命令：sudo service sshd start。


### 确认许可权限
&ensp;&ensp;&ensp;&ensp;在/etc/目录下，有两个文件，白名单：hosts.allow和黑名单：hosts.allow。我们需要从这白名单中添加ip（也可以添加所有），从黑名单中去除ip。添加所有的方式是在hosts.allow文件中最后加一行（sshd: all）。
![允许所有配置](http://image.public.yyf256.top/hosts_allow_all.png)



### 虚拟机注意事项
&ensp;&ensp;&ensp;&ensp;如果是云服务器，那以上的问题解决就好了，如果是虚拟机则还要注意一点就是，虚拟机的网络配置需要选择桥连模式。下图是virtualbox中的配置。
![配置为桥连模式](http://image.public.yyf256.top/virtualbox_set_net.png)
配置后需要将虚拟机重启才会生效。