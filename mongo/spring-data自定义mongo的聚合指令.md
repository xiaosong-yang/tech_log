&ensp;&ensp;&ensp;&ensp;最近使用基于spring-data的api对mongo进行操作时发现一些局限，比如如下命令：
```javascript
    .match({time:{$gte:ISODate("2020-08-01T00:00:00.000+08:00"),$lt:ISODate("2020-09-01T00:00:00.000+08:00")}})
    .unwind("$userList")
    .group({
          _id: "$qd",
          "regUsers":{$addToSet: "$userList"}
    })
    .addFields({ "time" : ISODate("2020-08-01T00:00:00.000+08:00")})
    .project({"_id":{$concat: ["2020-08_","$_id"]},"qd":"$_id","user_list":"$regUsers","reg_user":{$size: "$regUsers"},"time":"$time"})
```
看到最后的addFields和project，对于project，spring还是有支持的，基于Aggregation.project()能对project进行一些基本操作，但是如果我想在输出中凭空补充一列呢，比如我想加一列"time"为当前系统时间，这时利用Aggregation.project()自带的功能，怎么都实现不了，然后我们想到了利用addFields来进行增加字段，结果发现spring-data直接不支持addFields命令，但是对于这种超出正常API场景的，spring-data提供了扩展，比如我们自定义一个addFields：
```kotlin
        val addFields = AggregationOperation {
            val document = Document(
                "time",
                beginTime
            )
            Document("\$addFields", document)
        }
```
自定义一个project
```kotlin
        val project = AggregationOperation {
            Document(
                "\$project", mapOf(
                    "reg_user" to "\$reg_user",
                    "qd" to "\$_id",
                    "_id" to mapOf("\$concat" to arrayOf("${beginTime.toString("yyyy-MM")}_", "\$_id")),
                    "time" to "\$time"
                )
            )
        }
```
会发现其实就是将手动写入一份map结构的mongo命令。