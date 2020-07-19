&ensp;&ensp;&ensp;&ensp;git作为如今我们最广泛使用的项目代码管理工具，这里详细整理一下使用基本流程,适用于基本理解git流程的，当做工具书用的。git分之间的基本操作如下图：
[![git仓库操作](..\drawo\git仓库操作.jpg)](https://www.processon.com/view/link/5f12f13f1e08537d50b25e1a)


#### git内容比较工具(git diff命令)
- git diff：是查看 workspace（工作区） 与 index（暂存区） 的差别的。
- git diff --cached：是查看 index（暂存区） 与 local repositorty（本地仓库） 的差别的。
- git diff HEAD：是查看 workspace 和 local repository 的差别的。（HEAD 指向的是 local repository 中最新提交的版本）

注：git diff 后跟两个参数，如果只写一个参数，表示默认跟 workspace中的代码作比较。git diff 显示的结果为 第二个参数所指的代码在第一个参数所指代码基础上的修改。如，git diff HEAD 表示 workspace 在 最新commit的基础上所做的修改



#### git配置ssh免密登录
1. 我们打开gitbash执行命令ssh-keygen，然后一路回车，就会在我们c盘的用户目录下生成公私钥，比如我的就是在这里：
![git_ssh_create_rsa](..\picture_back_up\git_ssh_create_rsa.png)
然后将上图中的id_rsa.pub中的内容拷贝到git或者gitlab或者github之类的ssh设置中，如下图。
![git_ssh_create_rsa](..\picture_back_up\github_ssh_set_rsa.png)