&ensp;&ensp;&ensp;&ensp;为了便于做复杂的聚合查询，mongo提供了aggregate指令，基于管道运算进行对数据的处理，常见的\$sort,\$project,\$group,\$match就不在赘述了,下面整理一些不常见的但是有时候很有用的指令：
#### $unwind
我们插入mongo数据的时候，肯定会遇到插入数组或者列表的场景，但是对于列表数据的读取怎么操作呢，比如我想把多条文档中的列表数据重新整合去重该怎么操作，这是就需要用到unwind，它可以将某一个列表打散拆成多条文档:
```
{"id":1,list:[1,2,3,4]}
{"id":2,list:[3,100,90,4]}
```
对这条文档进行unwind操作之后，会变成
```
{"id":1,list:1}
{"id":1,list:2}
{"id":1,list:3}
{"id":1,list:4}
{"id":2,list:3}
{"id":2,list:100}
{"id":2,list:90}
{"id":2,list:4}
```
然后对拆解之后的所有文档进行addToSet就能很容易完成去重操作了。



#### $out
这个命令可以聚合统计的数据写入到一张新表中，一般在整个聚合操作流程的最后一步：
```javascript
db.users.aggregate()
    .match({time:{$gte:ISODate("2020-07-01T10:00:00.000+08:00"),$lt:ISODate("2020-08-01T10:00:00.000+08:00")},qd:"Channel_001"})
    .group({
          _id: null,
          "sum":{$sum:1}
    })
    .project({})
    .sort({_id:-1})
    .limit(100)
    .out("output-collection")
```
但是这个命令有一个坑人的地方就是，每次重新执行$out命令时，这张表都会被覆盖掉，不能进行追加数据。所以该命令适合做单次聚合结果导出的临时表。



#### $merge
这个命令和$out类似，是将聚合结果导出到某一张表的命令，区别在于merge不会覆盖原有数据，会进行合并：
```javascript
db.users.aggregate()
    .match({time:{$gte:ISODate("2020-07-01T10:00:00.000+08:00"),$lt:ISODate("2020-08-01T10:00:00.000+08:00")},qd:"Channel_001"})
    .group({
          _id: null,
          "sum":{$sum:1}
    })
    .project({})
    .sort({_id:-1})
    .limit(100)
    .merge({into: { db: "db", coll: "new_col" }, on: "_id",  whenMatched: "replace", whenNotMatched: "insert"})
```
可以看到merge还有子指令，into指明导出目标表，db为库名，coll为具体的表名，on是合并时的依据字段，比如这里以id为基准进行合并，如果id重复了，就会replace新数据覆盖老数据，如果id没有重复，就会直接插入新数据。所以这个命令比out强大，但是这个命令是到了mongo4.2以后才出来的，如果mongo版本太老了，就无法使用了。