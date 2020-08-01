&ensp;&ensp;&ensp;&ensp;为了方便像linux上以上使用shell，在idea的终端里引入了git bash，这样就可以在window下使用shell了，但是配置之后却出现一个问题，使用git的过程中老会出现乱码问题。解决方法如下：
1. 在git bash命令行中依次输入以下命令：
```shell
$ git config --global core.quotepath false  		# 显示 status 编码
$ git config --global gui.encoding utf-8			# 图形界面编码
$ git config --global i18n.commit.encoding utf-8	# 提交信息编码
$ git config --global i18n.logoutputencoding utf-8	# 输出 log 编码
$ export LESSCHARSET=utf-8
# 最后一条命令是因为 git log 默认使用 less 分页，所以需要 bash 对 less 命令进行 utf-8 编码
```

2. 找到IDEA在本地的安装路径，找到idea.exe.vmoptions文件和idea64.exe.vmoptions文件，在其最后一行添加
```text
-Dfile.encoding=UTF-8
```

3. 修改git安装路径下的bash.bashrc文件，在文件最后添加
```text
export LANG="zh_CN.UTF-8"
export LC_ALL="zh_CN.UTF-8"
```