&ensp;&ensp;&ensp;&ensp;fiddler用于抓http/https包的，如果你是其他协议的，fiddler无法抓到。整理一下fiddler如何抓苹果手机里app的包。
#### fiddler的配置修改
1. 配置fiddler可以拦截https请求(毕竟现在基本都是https协议)
![fiddler配置解密https](..\picture_back_up\fiddler配置解密https.webp)
2. 配置fiddler可以接受远程连接
![fiddler配置解密https](..\picture_back_up\fiddler配置远程连接.webp)

配置完以上两项之后，fiddler需要重启才会生效

#### 保持网络相同
&ensp;&ensp;&ensp;&ensp;让fiddler所在的PC与手机在同一个局域网内,比如手机开热点让电脑连，电脑开热点让手机连，让两个连第三方的同一个wifi都可以，反正在同一个局域网里即可。另外需要关闭PC的防火墙，否则可能造成无法访问。然后获取电脑的局域网ip，注意如果PC在多个局域网中，理论上每个ip都可以，确保万一的话，使用和手机相同局域网的那个ip，比如我这里就是192.168.137.1，因为我是pc开的wifi，然后手机去连，所以PC的ip最后一位一般就是1了。
![获取ip](..\picture_back_up\获取ip.png)



#### 配置代理
&ensp;&ensp;&ensp;&ensp;在wifi界面进入代理配置:
![fiddler修改wifi配置](..\picture_back_up\fiddler修改wifi配置.webp)
![fiddler配置代理](..\picture_back_up\fiddler配置代理.webp)


#### 证书下载、安装、添加信任
1. 下载，在手机浏览器中打开(上文的ip:8080)
![下载fiddler证书](..\picture_back_up\下载fiddler证书.webp)

2. 下载后去设置->通用->描述文件 进行安装
![安装证书1](..\picture_back_up\安装证书1.webp)
![fiddler安装证书2](..\picture_back_up\fiddler安装证书2.webp)
![fiddler安装证书3](..\picture_back_up\fiddler安装证书3.webp)
3. 下载后去设置->通用->关于本机->证书信任设置 进行安装
![证书添加信任](..\picture_back_up\证书添加信任.webp)
这一步比较重要，之前网上看到很多教程都缺这一步，造成的结果就是，所有都正常，就是app连不上网，怎么都访问不到，特坑。

以上就是所有的fiddler抓包配置，完成之后，打开app进行操作，应该就能抓到数据了。

