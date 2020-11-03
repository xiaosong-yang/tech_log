&ensp;&ensp;&ensp;&ensp;由于mongo是一个树状结构数据存储，所以我们往往需要基于子文档去查询某个东西，比如数据如下：
```json
{
	"_id" : "97d73f8dd62a4c659c37335e321b8cc8",
	"end_time" : ISODate("2020-12-01T00:00:00.000+08:00"),
	"start_time" : ISODate("2020-11-01T14:00:00.000+08:00"),
	"week_cfg" : [
		NumberInt(0),
		NumberInt(1),
		NumberInt(2),
		NumberInt(3),
		NumberInt(4),
		NumberInt(5)
	],
	"packages" : [
		{
			"price" : NumberLong(3000000),
			"awards" : [
				{
					"_id" : NumberInt(0),
					"type" : "COIN",
					"desc" : "分贝",
					"num" : NumberInt(10)
				},
				{
					"_id" : NumberInt(1),
					"type" : "GIFT",
					"desc" : "彩虹糖",
					"num" : NumberInt(10)
				},
				{
					"_id" : NumberInt(3),
					"type" : "GIFT",
					"desc" : "尴尬",
					"num" : NumberInt(10)
				},
				{
					"_id" : NumberInt(0),
					"type" : "EXP",
					"desc" : "经验",
					"num" : NumberInt(10)
				},
				{
					"_id" : NumberInt(1),
					"type" : "CAR",
					"desc" : "周星测试",
					"num" : NumberInt(10)
				},
				{
					"_id" : NumberInt(1),
					"type" : "ROOM_SPEECH_BUBBLE",
					"desc" : "满天星",
					"num" : NumberInt(10)
				},
				{
					"_id" : NumberInt(1),
					"type" : "HOME_FLOAT",
					"desc" : "流星",
					"num" : NumberInt(10)
				}
			]
		},
		{
			"price" : NumberLong(5000000),
			"awards" : [
				{
					"_id" : NumberInt(0),
					"type" : "COIN",
					"desc" : "分贝",
					"num" : NumberInt(10)
				},
				{
					"_id" : NumberInt(327),
					"type" : "GIFT",
					"desc" : "小火苗",
					"num" : NumberInt(10)
				},
				{
					"_id" : NumberInt(1),
					"type" : "FRAME",
					"desc" : "来吼鸭",
					"num" : NumberInt(10)
				},
				{
					"_id" : NumberInt(0),
					"type" : "EXP",
					"desc" : "经验",
					"num" : NumberInt(10)
				},
				{
					"_id" : NumberInt(1),
					"type" : "CAR",
					"desc" : "周星测试",
					"num" : NumberInt(10)
				},
				{
					"_id" : NumberInt(1),
					"type" : "ROOM_SPEECH_BUBBLE",
					"desc" : "满天星",
					"num" : NumberInt(10)
				},
				{
					"_id" : NumberInt(1),
					"type" : "HOME_FLOAT",
					"desc" : "流星",
					"num" : NumberInt(10)
				}
			]
		}
	],
	"online" : true,
	"time" : ISODate("2020-11-02T10:51:34.866+08:00")
}
{
	"_id" : "de0a1b4df7e84f288eab663ea28b0aa7",
	"end_time" : ISODate("2001-11-30T00:00:00.000+08:00"),
	"start_time" : ISODate("2001-11-02T00:00:00.000+08:00"),
	"week_cfg" : [
		NumberInt(2)
	],
	"packages" : [
		{
			"price" : NumberLong(9000000),
			"awards" : [
				{
					"_id" : NumberInt(0),
					"type" : "COIN",
					"num" : NumberInt(100)
				},
				{
					"_id" : NumberInt(1),
					"type" : "GIFT",
					"num" : NumberInt(1)
				},
				{
					"_id" : NumberInt(0),
					"type" : "EXP",
					"num" : NumberInt(100)
				},
				{
					"_id" : NumberInt(1),
					"type" : "FRAME",
					"num" : NumberInt(100)
				},
				{
					"_id" : NumberInt(1),
					"type" : "CAR",
					"num" : NumberInt(100)
				},
				{
					"_id" : NumberInt(1),
					"type" : "ROOM_SPEECH_BUBBLE",
					"num" : NumberInt(100)
				},
				{
					"_id" : NumberInt(1),
					"type" : "HOME_FLOAT",
					"num" : NumberInt(100)
				}
			]
		},
		{
			"price" : NumberLong(1000000),
			"awards" : [
				{
					"_id" : NumberInt(0),
					"type" : "COIN",
					"num" : NumberInt(100)
				},
				{
					"_id" : NumberInt(1),
					"type" : "GIFT",
					"num" : NumberInt(100)
				},
				{
					"_id" : NumberInt(0),
					"type" : "EXP",
					"num" : NumberInt(100)
				},
				{
					"_id" : NumberInt(1),
					"type" : "FRAME",
					"num" : NumberInt(100)
				},
				{
					"_id" : NumberInt(1),
					"type" : "CAR",
					"num" : NumberInt(100)
				},
				{
					"_id" : NumberInt(1),
					"type" : "ROOM_SPEECH_BUBBLE",
					"num" : NumberInt(100)
				},
				{
					"_id" : NumberInt(1),
					"type" : "HOME_FLOAT",
					"num" : NumberInt(100)
				}
			]
		}
	],
	"online" : false,
	"time" : ISODate("2020-10-31T14:51:56.874+08:00")
}
```
如果我们希望查询week_cfg这个列表中有0的文档:
```javascript
db.exchange_package_cfg.find({"week_cfg":0})
   .projection({})
   .sort({_id:-1})
   .limit(100)
```
如果希望查询week_cfg这个类表存在0,1,2某一个的文档：
```javascript
db.exchange_package_cfg.find({"week_cfg":{$in:[0,1,2]}})
   .projection({})
   .sort({_id:-1})
   .limit(100)
```
如果我们需要查询packages下的awards列表中，type字段为COIN的文档：
```javascript
db.exchange_package_cfg.find({"packages.awards.type":"COIN"})
   .projection({})
   .sort({_id:-1})
   .limit(100)
```
如果我们需要查询packages下的awards列表中，type字段为COIN或者GIFT的文档：
```javascript
db.exchange_package_cfg.find({"packages.awards.type":{$in:["COIN","GIFT"]}})
   .projection({})
   .sort({_id:-1})
   .limit(100)
```
