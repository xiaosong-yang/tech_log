&ensp;&ensp;&ensp;&ensp;在《爬虫进阶路程1——开篇》中说道过，自己本以为使用了selenium就万事大吉了，结果发现使用selenium之后还是死了的，似乎别人的代码能够识别出自己使用了selenium，查资料下来确实如此，反爬手段其实也简单，就是去获取你当前浏览器的一些基本信息，如果包含了selenium打开浏览器的一些特征，就认为你是selenium，而不是正常的浏览器。知道他反爬的原理，其实就知道怎么解决了，无非两种：
1. 在他进行特征判断之前进行篡改，如果你是客户端判断，就要修改源代码，如果是服务端判断，就要修改请求入参。
2. 让自己的selenium打开的浏览器不携带任何特征。

### mitproxy代理篡改脚本
mitproxy是对应第一种串改js脚本的策略，方法肯定是可行的，但是我没有使用这种方式，需要另外搭建一个代理服务器，对于大型企业，可能更合适一点，我这里就不再赘述，不过这种方式有这种方式的好处，在基于ip反爬的部分会讲到。


### selenium控制已有的浏览器
这种方式对应第二种让浏览器不携带selenium特种的策略，由于是我手动打开的一个浏览器，所以不会带着selenium的特征。在window上我们找到谷歌浏览器的安装路径，然后里面会有一个chrome.exe执行文件，我们在cmd中通过执行
```cmd
chrome.exe --remote-debugging-port=9222 
```
来启动一个谷歌浏览器，并设置连接端口9222，建议在执行命令之前关掉所有已打开的谷歌浏览器进程，否则可能会有冲突。然后我们在代码中配置selenium的链接端口：
```python
// python
chrome_options.add_experimental_option("debuggerAddress", "127.0.0.1:9222")
```
```kotlin
//kotlin
val options = ChromeOptions()
options.setExperimentalOption("debuggerAddress", "127.0.0.1:9222")
val webDriver: WebDriver = ChromeDriver(options)
```
因为我是用kotlin实现的，所以提供一下kotlin版本的代码。

&ensp;&ensp;&ensp;&ensp;在前一篇中说了如何在linux中安装chrome，所以这里就有用武之地了，在linux中通过如下命令启动一个谷歌浏览器：
```shell
/usr/bin/chrome --headless  --disable-gpu --no-sandbox   --remote-debugging-port=9022
```
然后我们就可以根据ip:9022来远程连接这个开启了的浏览器了。

##### 解决chrome不可远程访问问题
&ensp;&ensp;&ensp;&ensp;一开始以为可以直接本地连上服务器上的chrome了，这种就可以不用每次都本地再启动一个浏览器，结果发现怎么也连接不上，一开始以为是阿里云安全组问题，但是配了端口访问权限后还是不行，后来通过查看端口才知道，该端口仅对本地可访问：
```shell
$ netstat -nltp|grep 9022
tcp        0      0 127.0.0.1:9022          0.0.0.0:*               LISTEN      15722/chrome
```
可以看到仅可以127.0.0.1可访问，并且不可修改，于是只能曲线救国，利用ssh打开一个代理通道，命令如下：
```
$ ssh -NTf -L 0.0.0.0:9023:localhost:9022 localhost
```
然后会让输入密码，输入当前登录账号的密码即可。
其中：
- N参数：表示只连接远程主机，不打开远程shell；
- T参数：表示不为这个连接分配TTY；
- f参数：表示连接成功后，转入后台运行；
- L参数表示正向代理，可以换成R：反向代理，D:socks5代理，三者区别自行学习吧

把可供外界访问的9023端口指向仅供本地访问的9022端口，然后阿里云打开安全组里的9023端口，即可实现通过9023端口，对linux上的chrome浏览器使用。


##### 注意点
1. 在使用selenium控制浏览器时，需要使用对应的版本的chromedriver，通过https://npm.taobao.org/mirrors/chromedriver，即可进行下载各个chrome版本对应的驱动文件了，linux和window的驱动文件也不一样
2. linux版本的驱动包下载之后，放到linux上需要给予文件可执行权限，否则没法云行



### 使用V63以前的谷歌浏览器和对应驱动
据说这种方式也可行，可以一试，不知道效果，仅供参考。