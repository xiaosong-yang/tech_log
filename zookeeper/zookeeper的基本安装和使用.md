## zk的下载和启动

 1. 下载地址：https://archive.apache.org/dist/zookeeper/（比如3.4.14版本），尽量不要下最新的，越新的越容易有问题。下的文件是.tar.gz的压缩包，是可以在windows和linux上都可以使用的。（不要感觉是.tar.gz包就觉得在windows上没法用）
 
 2. window下下载之后，用压缩软件连续解压两次，就能解压出来了，linux上直接使用命令解压。解压出来，如图：
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltYWdlLnB1YmxpYy55eWYyNTYudG9wL3RlY2huaWNhbC8yMDE5MTIwNi8xNTc1NjQzNTc4NDMy?x-oss-process=image/format,png)
 3. 这面包含了zk的源码，使用的话，我们只要关心bin目录和conf目录即可。进入conf目录，会看到一个zoo_sample.cfg文件，我们将其重命名为：zoo.cfg。这个文件是zk的配置文件。重命名之后，我们回到上一层，进入bin目录，我们会看到好多个cmd和sh文件，他们分别是用于linux系统和window系统上执行的。比如zkServer.cmd和zkCli.cmd，是用来在windows系统上启动一个zk服务端和客户端的。同理，zkServer.sh和zkCli.sh用于在linux系统上启动zk的服务端和客户端的。（如果我们前面没有将zoo_sample.cfg改成zoo.cfg，直接执行zkServer.cmd，那弹出来的命令窗就会一闪而过，代表启动失败。有时候遇到其他错误时，也会出现命令窗一闪而过，这种情况，我么你可以通过记事本打开zkServer.cmd脚本，然后在末尾加上pause，再执行，就不会一闪过，然后我们就可以在命令窗里看到错误原因了）

## zk的基本使用

 1. 我们先后执行zkServer.cmd和zkCli.cmd两个脚本，启动zk的服务端和客户端。zk的服务端启动后就不再需要动了，我们在客户端进行操作，实现对zk的使用。
 
 2. 如果我们什么命令都不知道，可以在客户端的命令框中敲入help，即会返回zk的基本使用命令。
 
 3. 我们要使用zk，首先要知道zk是什么，有什么功能。简单来说，zk的服务端提供了一个按节点记录数据的服务。所以最简单的使用即为在客户端利用
    create命令让zk服务端创建一个节点，并存入指定数据，eg：
    我们在/目录下创建zk3这么一个节点，并在节点中存入shabi这个字符：
   
>  命令：
>  create /zk3 shabi
>  结果： 
>  Created /zk3

我们通过ls /，可以看到/目录下多了zk3这么一个节点：

> 命令：
> s /
> 返回：
> [zk2, zk1, zk10000000001, zk20000000003, zk3, zookeeper]


我们可以通过get /zk3看到我们存入的数据

> 命令： 
> get /zk3 
> 返回：
>  shabi    //存储的值 cZxid = 0xb  //节点创建时的zxid 
> ctime = Fri Dec 06 23:09:14 CST 2019 //节点创建时间 
> mZxid = 0xb  //节点最后一次修改时的zxid 
> mtime = Fri Dec 06 23:09:14 CST 2019  //节点最后一次修改时的时间 
> pZxid = 0xb //该节点的子节点最后一次修改的zxid，可以理解为parent_zxid，子节点的数据修改不算 
> cversion = 0 //该节点的子节点的变更次数，可以理解为child_version，子节点的数据修改不算 
> dataVersion = 0 //该节点的数据被修改次数 aclVersion = 0  //该节点的ACL变更次数 
> ephemeralOwner = 0x0 //如果该节点为临时节点，那就是临时节点所有者的会话id，如果是持久节点那就是0，这里是0 
> dataLength = 5 //该节点存放的数据长度 numChildren = 0 //该节点的子节点数量


上面列了一堆的zk节点的信息，大部分还是好理解的，其中有两个概念，zxid和acl。zk的每一个操作都对应一个全局唯一的事务id即zxid，zxid是有序的，也就是如果zxid1小于zxid2，那么zxid1的事务必然发生在zxid2的事务之前。第二个是acl，它是只zk的权限管理，zk作为一个分布式协调框架，很多时候不是所有节点都可以让每个人来修改的，这样太危险了，所以acl是用来控制zk权限的。我们可以看一下/zk3这个节点的Acl：

> 命令： 
> getAcl /zk3
>  返回：
> 'world,'anyone 
> : cdrwa

即代表，所有人不用任何方式就有了所有权限。ACL权限的具体介绍，会在下一章说。
