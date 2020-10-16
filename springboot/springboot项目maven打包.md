&ensp;&ensp;&ensp;&ensp;普通的maven项目直接通过执行maven命令package打包即可，如果springboot项目则需要依赖springboot的打包插件在执行package命令进行打包，但是对于聚合项目来说，直接打包子模块是不行的，可以通过直接package整个聚合项目进行打包，但是如果子模块中存在springboot项目，则这样打包出来的springboot项目无法通过java -jar命令进行启动运行，需要对springboot项目的pom文件中的springboot打包插件进行配置如下：
```xml
<build>

    <plugins>

        <plugin>

            <groupId>org.springframework.boot</groupId>

            <artifactId>spring-boot-maven-plugin</artifactId>

            <configuration>

                <mainClass>top.yyf256.zhulu.lottery.web.Application</mainClass>

            </configuration>

            <executions>

                <execution>

                    <goals>

                        <goal>repackage</goal>

                    </goals>

                </execution>

            </executions>

        </plugin>

    </plugins>

</build>
```

可以看到比平时的springboot打包插件多了一个入口函数配置，以及repackage配置，这样我们对整个聚合项目进行打包时，在对springboot项目进行maven打包结束之后，会再次对maven打包好的jar再次打包转换成可以通过java -jar启动的springboot项目的jar包。