&ensp;&ensp;&ensp;&ensp;自己手上有一个项目，想要分环境设置yml配置文件，比如数据库，密钥等，各个环境都配置的不一样。但是又有一些公共的配置，比如一些需要进行配置化的业务常量等。比如当下有开发和生产环境，需要有三个配置文件：application.yml,application-dev.yml,application-prod.yml。其中application.yml为公共配置文件，在其中配置：
```yml
spring:
  profiles:
    active: dev
```
这样开发环境启动的时候，默认加载application-dev.yml文件。而当要生产部署时，在生产部署的启动命令上加上：--spring.profiles.active=prod。比如：
```shell
java -jar -XX:+UseConcMarkSweepGC -Xmx256m  -XX:MaxMetaspaceSize=100m $1 --spring.profiles.active=prod  > stock.log  2>&1 &
```
这样启动的时候他就会加载公共配置文件和application-prod.yml这个生产配置文件了。