### 拆分子节点&数据类型转换
&ensp;&ensp;&ensp;&ensp;我们知道mongo的某个元素是list，我们可以通过unwind把每个子元素拆解出来，然后再通过group可以很方便的对原本的list进行一个聚合操作，但是如果我们的元素的子节点是还是多个普通元素呢，就不能直接 通过unwind将每个子节点进行拆分。这个时候我们需要通过$objectToArray方法，将多个子节点转换成多个list元素，eg：
```javascript
db.finance_pay_daily.aggregate()
    .match({time:{$gte:ISODate("2021-01-01T00:00:00.000+08:00")}})
    .project({types:{$objectToArray: "$pay_type_data"}})
    .unwind("$types")
```
其中project({types:{\$objectToArray: "\$pay_type_data"}}),将原本的pay_type_data元素下的多个子节点，转换成了list表，并pay_type_data重新命名为types，然后就可以通过unwind方法，将list拆分成多个文档了:
```json
{
  "_id": "20210122",
  "types": {
    "k": "weixin_app",
    "v": {
      "cny": "0.14",
      "balance": 10000,
      "time": 5,
      "user_num": 1,
      "rate": 100
    }
  }
},
{
  "_id": "20210122",
  "types": {
    "k": "alipay_m",
    "v": {
      "cny": "0.03",
      "balance": 10000,
      "time": 3,
      "user_num": 1,
      "rate": 100
    }
  }
}
```
可以看到通过转变成list元素之后，原本节点名变成了k节点的值，原本节点的值变成了v节点的值。通过这样转变再拆分之后，我们就可以继续group进行聚合操作了，需要注意的是当我们进行聚合运算时，如果被聚合的字段是字符串就无法进行聚合，我们可以通过\$toDouble的方式转换成double类型，再进行聚合运算：
```javascript
db.finance_pay_daily.aggregate()
    .match({time:{$gte:ISODate("2021-01-01T00:00:00.000+08:00")}})
    .project({types:{$objectToArray: "$pay_type_data"}})
    .unwind("$types")
    .group({
          _id: "$types.k", 
          total_cny:{$sum: {$toDouble: "$types.v.cny"}},
          total_balance:{$sum:"$types.v.balance"}
    })
    .sort({_id:-1})
    .limit(100)
```

以上逻辑转换成kotlin的实现方式如下（java类似）:
```kotlin
        val typesAggregation = Aggregation.newAggregation(
            Aggregation.match(Criteria.where("time").gte(startTime).lt(timeQo.endTime)),
            //这里的$符号需要自己加，如果不加的话，spring-data-mongo的api自己不会加，然后就会有问题
            Aggregation.project().and(ObjectOperators.ObjectOperatorFactory("\$pay_type_data").toArray()).`as`("types"),
            Aggregation.unwind("types"),
            Aggregation.group("types.k")
                .sum(ConvertOperators.ConvertOperatorFactory("types.v.cny").convertToDouble()).`as`("total_cny")
                .sum("types.v.balance").`as`("total_balance"))
        adminMongo.aggregate(typesAggregation,"finance_pay_daily",Document::class.java).mappedResults.forEach {
            res["${it["_id"]}_cny"] = it["total_cny"]
            res["${it["_id"]}_balance"] = it["total_balance"]
        }
```

### 根节点替换
有时候我们需要将子节点替换为根节点进行聚合操作，可以通过\$replaceRoot进行替换
```javascript
db.finance_pay_daily.aggregate([
    {$match:{time:{$gte:ISODate("2021-01-01T00:00:00.000+08:00")}}},
    {$replaceRoot: { newRoot: "$pay_type_data" }},
    ])
```
但有时候被我们拿来的子节点有可能是为空的，这时候执行上面的语句是会报错的，这个时候我们可以使用\$replaceWith这个方法：
```javascript
db.finance_pay_daily.aggregate()
    .match({})
    .project({})
    .replaceWith({ $ifNull: [ "$pay_type_data.apple_sandbox", { _id: "1000", missingName: true} ] })
    .sort({time:-1})
    .limit(100)
```
这个方法同样是将pay_type_data.apple_sandbox这个节点提到根节点，区别是当pay_type_data.apple_sandbox不存在时，是可以拿{ _id: "1000"}这个进行代替的，当然_id这个可以替换成其他的名称，并且可以有多个元素：
```javascript
db.finance_pay_daily.aggregate()
    .match({})
    .project({})
    .replaceWith({ $ifNull: [ "$pay_type_data.apple_sandbox", { cny: "1000",balance:500, missingName: true} ] })
    .sort({time:-1})
    .limit(100)
```
甚至替代的元素也是可以拿文档中其他的数据：
这个方法同样是将pay_type_data.apple_sandbox这个节点提到根节点，区别是当pay_type_data.apple_sandbox不存在时，是可以拿{ _id: "1000"}这个进行代替的，当然_id这个可以替换成其他的名称，并且可以有多个元素：
```javascript
db.finance_pay_daily.aggregate()
    .match({})
    .project({})
    .replaceWith({ $ifNull: [ "$pay_type_data.apple_sandbox", { cny: "$total_cny",balance:"$total_balance", missingName: true} ] })
    .sort({time:-1})
    .limit(100)
```