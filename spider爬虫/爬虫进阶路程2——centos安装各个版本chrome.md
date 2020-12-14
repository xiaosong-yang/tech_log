&ensp;&ensp;&ensp;&ensp;接《爬虫进阶路程1——开篇》，里面讲到使用selenium进行实现高级别的爬虫，能够绕过那些绞尽脑汁是js复杂化的反爬方式，而selenium是需要配合浏览器来搭配使用的，这里就来讲一下如何在linux安装无头浏览器，window上怎么装就不讲了，直接百度很容易就装上了，但是如果正儿八经做爬虫的肯定不会止步于在自己PC上来爬数据，最终一定是走linux服务器的。
#### 安装
&ensp;&ensp;&ensp;&ensp;这里主要通过yum本地安装rpm包来完成的chrome浏览器安装的，chrome安装包各版本下载地址如下：https://www.chromedownloads.net/chrome64linux/。从上面下载我们需要版本的安装包，解压安装包压缩文件，里面有一个以rpm为后缀的文件，将该文件上传至linux。上传之后通过rpm命令安装，比如安装包名为：google-chrome-stable_current_x86_64-64_84.0.4147.105.rpm，安装命令如下：
```shell
$ yum install google-chrome-stable_current_x86_64-64_84.0.4147.105.rpm
```
等安装完之后，我们可以做一个软链，因为chrome安装之后命令默认为google-chrome-stable，我们可以将google-chrome-stable软链到chrome，通过chrome直接执行命令，软链就类似window的快捷方式一样。
```shell
which google-chrome-stable
```
可以得到google-chrome-stable的执行路径，正常情况下应该是/usr/bin/google-chrome-stable,然后我们创建软链
```shell
$ ln -s /usr/bin/google-chrome-stable /bin/chrome
```
最后我们通过查看chrome版本号查看是否安装成功
```shell
$ chrome -version
Google Chrome 87.0.4280.66
```
可以看到我们安装的谷歌浏览器版本是87.0.4280.66

#### 重装
如果我们想要卸载原有的浏览器，安装其他版本，我们的可以通过yum来进行卸载。通过以下命令查看我们安装的chrome
```
$ yum list installed|grep chrome
google-chrome-stable.x86_64      87.0.4280.66-1               installed
```
然后我们就可以通过yum对原有的chrome进行卸载
```shell 
$ yum remove google-chrome-stable.x86_64
```
之后再重复上面的安装步骤安装其他的包