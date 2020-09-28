&ensp;&ensp;&ensp;&ensp;通过IDEA执行gradle的任务时，在终端的输出出现中文乱码。解决方法是Help->Edit Custom VM Options，然后再最后一行加上
```
-Dfile.encoding=UTF-8
```
重启IDEA即可解决，一定要重启！！！
