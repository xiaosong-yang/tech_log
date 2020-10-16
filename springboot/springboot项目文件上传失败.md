&ensp;&ensp;&ensp;&ensp;之前给博客做了一个后台，具备图片上传功能，后台是基于springboot实现的，服务器也是利用的springboot自带的tomcat，一开始做出来的时候还可以，但过了一段时间使用上传功能发现上传失败了，报错如下：

> 2019-09-16 17:27:10.277 [http-nio-8083-exec-2] ERROR org.apache.catalina.core.ContainerBase.[Tomcat].[localhost].[/blogback].[dispatcherServlet] - Servlet.service() for servlet [dispatcherServlet] in context with path [/blogback] threw exce
ption [Request processing failed; nested exception is org.springframework.web.multipart.MultipartException: Failed to parse multipart servlet request; nested exception is java.io.IOException: The temporary upload location [/tmp/tomcat.84655
30737475240398.8083/work/Tomcat/localhost/blogback] is not valid] with root cause
java.io.IOException: The temporary upload location [/tmp/tomcat.8465530737475240398.8083/work/Tomcat/localhost/blogback] is not valid
        at org.apache.catalina.connector.Request.parseParts(Request.java:2821) ~[tomcat-embed-core-9.0.12.jar!/:9.0.12]
        at org.apache.catalina.connector.Request.parseParameters(Request.java:3185) ~[tomcat-embed-core-9.0.12.jar!/:9.0.12]

&ensp;&ensp;&ensp;&ensp;看报错大体是缺少一个文件路径，文件路径：/tmp/tomcat.8465530737475240398.8083/work/Tomcat/localhost/blogback。原因就很明朗了，/tmp目录是linux的临时目录，相当于windows下的回收站，这个目录下的文件一段时间不用，会被自动清理掉，这就是为什么一开始能够上传文件，但是过了一段时间不可以了。 解决方案两种：
1. 手动去tmp目录下再把这个目录重新创建一下。
2. 在项目中加入如下配置：

```java
server:
  tomcat:
    basedir: /home/saysky/temp
```
通过自定义这个路径来实现不被linux自动删除掉。

