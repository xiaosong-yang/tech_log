&ensp;&ensp;&ensp;&ensp;今天使用gradle构建发布脚本，但是start.sh脚本上传到linux后执行报错，但同样的代码复制到vi编辑器重新保存执行，却没问题。最后发现window默认的shell脚本编码格式和linxu不一致，我们可以通过vi命令进入编辑界面，然后在命令框执行：set ff命令查看,当前编码格式，linux默认的是是：fileformat=unix，而window上传上来的是fileformat=dos，然后我们可以通过set ff=unix,在保存退出vi就可以修改文件的格式了。
&ensp;&ensp;&ensp;&ensp;但是上面这种修改方式不便于我们的脚本构建，所以找到了另一种方式，在shell命令行中执行：
```shell
sed -i 's/\r$//' <filename> 转化为unix格式
```
就可以把文件直改成unix格式了。最终gradle的构造脚本如下：
```gradle
tasks.create("deployTest") {
    dependsOn("bootJar")
    group = "deploy"
    doLast {
        exec {
            workingDir("./")
            commandLine("./gradlew.bat", "clean")
            commandLine("./gradlew.bat", "bootJar")
        }
        ssh.run(delegateClosureOf<RunHandler> {
            session(remotes["testServer"], delegateClosureOf<SessionHandler> {
                println("-----------------上传jar包--------------")
                put(hashMapOf(
                        "from" to "./build/libs/stock-ana-0.0.1-SNAPSHOT.jar",
                        "into" to "/users/xiaosong/data/JavaWorkspace/stock-ana/"
                ))
                println("-----------------上传启动文件--------------")
                put(hashMapOf(
                        "from" to "./restart.sh",
                        "into" to "/users/xiaosong/data/JavaWorkspace/stock-ana/"
                ))
                println("-----------------重启服务:${rootProject.name}-${version}.jar--------------")
                executeScript("""
                    #!/bin/sh
                    cd /users/xiaosong/data/JavaWorkspace/stock-ana
                    sed -i 's/\r${'$'}//' restart.sh
                    chmod +x restart.sh
                    ./restart.sh ${rootProject.name}-${version}.jar
                """.trimIndent())
                println("-----------------重启服务完成--------------")
            })
        })
    }
}
```
其中有一行sed -i 's/\r${'$'}//' restart.sh这个就是修改文件格式的，但是我们发现原本的$变成了${'$'}，这是因为kotlin的表达式造成的，不能直接输入$。