&ensp;&ensp;&ensp;&ensp;需要注意的是我的docker是安装在centos7.0上的，如果是其他的操作系统，方法可能不一样。修改docker镜像配置文件：/etc/docker/daemon.json：
```shell
{
  "registry-mirrors": ["https://3of2xqw2.mirror.aliyuncs.com"]
}
```
如果没有配置过的话，该文件初始应该是这样的:
```shell
{}
```
配置好之后，依次执行两个shell命令：
```shell
systemctl daemon-reload
systemctl restart docker
```
之后等docker重启来之后，就完成了redis的镜像仓库配置。由于我原本是在虚拟机上装的docker，再加上家里宽带不好，原本从docker国外仓库下载简直要了老命，换成阿里云之后是有质的变化。