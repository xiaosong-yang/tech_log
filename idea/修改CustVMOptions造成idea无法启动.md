&ensp;&ensp;&ensp;&ensp;今天遇到一个问题，一大早过来发现IDEA无法启动了，昨晚还是好好的。于是我把IDEA安装目录下的/bin/idea.bat文件最后一行加上了pause暂停，然后手动执行，报错内容如下：
```shell
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
Error opening zip file or JAR manifest missing : lombok.jar
Error occurred during initialization of VM
agent library failed to init: instrument
```
让自己想起了昨晚对Help->Edit Cust VM Options的操作，该操作修改了idea64.exe.vmoptions这样一个文件，我在文件的最底下一行加了如下代码
```
-javaagent:lombok.jar
```
结果今天启动的时候就报错找不到lombok.jar，表象就是双加IDEA图标没反应。
&ensp;&ensp;&ensp;&ensp;找到原因后想着只需要找到idea64.exe.vmoptions，然后把那行代码去掉就解决了，但是问题出在找不到这个文件，从网上搜到的都是安装目录下的一个idea64.exe.vmoptions，但那时对于整个IDEA的配置，不属于用户的个性配置，该文件应该在C盘的用户目录下，但是茫茫文件实在是找不到，而且搜也搜不到。
&ensp;&ensp;&ensp;&ensp;最后换了一个思路，既然说是找不到lombok.jar这个jar包，那我就给你一个jar包呗，于是在/bin/idea.bat同级目录下放了个lombok.jar，然后idea.bat脚本，果然没报错，之后再把前面在idea.bat脚本里加的pause去掉，启动成功。
&ensp;&ensp;&ensp;&ensp;最后自己在IDEA里面通过Help->Edit Cust VM Options打开idea64.exe.vmoptions，然后去掉"-javaagent:lombok.jar"这段代码，顺便看了一下文件目录：C:\Users\Administrator\AppData\Roaming\JetBrains\IntelliJIdea2020.1。果不其然在C盘，就是藏的太深了。
