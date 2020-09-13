&ensp;&ensp;&ensp;&ensp;docker 启动容器报错：Error response from daemon: oci runtime error: container_linux.go:247: starting container process caused "write parent: broken pipe"

其实原因还是，linux与docker版本的兼容性问题

第一步：通过uname -r命令查看你当前的内核版本
```shell
uname -r
```
第二步：使用 root 权限登录 Centos。确保 yum 包更新到最新。
```shell
yum update
```
第三步：卸载旧版本(如果安装过旧版本的话)
```shell
yum remove docker  docker-common docker-selinux dockesr-engine
```
第四步：安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的
```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```
第五步：设置yum源
```shell
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

第六步：可以查看所有仓库中所有docker版本，并选择特定版本安装
```shell
yum list docker-ce
```
第七步：安装docker
```shell
yum install docker-ce
```
第八步：启动并加入开机启动
```shell
systemctl start docker
systemctl enable docker
```