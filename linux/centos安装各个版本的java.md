&ensp;&ensp;&ensp;&ensp;centos中安装各个版本的java，一般两种方式：使用yum或者软件压缩包，java这里我们采用软件压缩包来安装。如果之前已经用rpm包安装过其他jdk版本，可以通过以下方式进行卸载。
#### 卸载rpm方式安装的jdk
1. 查到到对应的软件
```shell
rpm -qa | grep jdk
```
2. 使用rpm -e --nodeps 命令删除上面查找到的内容
```shell
rpm -e --nodeps java-1.6.0-openjdk-devel-1.6.0.38-1.13.10.0.el6_7.x86_64
```

3. 全部删除完之后，可以用第一步的命令再检查一遍

#### 通过下载安装包安装jdk
1. 下载安装包
&ensp;&ensp;&ensp;&ensp;jdk全版本下载地址：https://www.oracle.com/cn/java/technologies/oracle-java-archive-downloads.html
&ensp;&ensp;&ensp;&ensp;根据具体要哪个版本下载对应的包，一般选择jdk而不是jre，因为jre是运行环境，只安装jre，各种调优的指令就没有了。所以报名一般为是：jdk.......tar.gz。下载之后上传到linux上
2. 解压压缩文件
```shell
tar -zxvf jdk-8u151-linux-x64.tar.gz
```
3. 配置环境变量
编辑配置文件
```shell
vi /etc/profile
```
将以下内容放到profile文件中
```shell
JAVA_HOME=/usr/java/jdk1.8.0_161        
JRE_HOME=/usr/java/jdk1.8.0_161/jre     
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```

其中/usr/java/jdk1.8.0_161 是djk压缩包解压之后的目录，不能配错了，否则就不生效了。

通过以下命令使环境变量生效
```shell
source /etc/profile
```

最后通过java -version来验证是否配置陈公公


#### 安装多个版本
&ensp;&ensp;&ensp;&ensp;有时候我们需要换jdk版本，我们可以通过上面的下载安装步骤，把其他版本的jdk压缩版在一个新的目录下进行解压，然后修改环境变量的路径。但这样不够，你会发现环境变量之后，通过java -version还是原来的版本，这是因为需要将原本版本的jdk解压包给删除掉，然后重新source /etc/profile，这样java -version才会是新的版本。