&ensp;&ensp;&ensp;&ensp;最近我们项目中增加了附近的人功能，如果单从实现上考虑其实没有什么难度，采集用户的经纬度坐标，然后用户调用接口去查询附近的人，根据用户的当前坐标去db里一个一个算距离，如果我们用户数就几十几百个，可能每次调接口的时候去算，也能够接受，但是如果你有几百万甚至上亿用户，用户每次来获取附近的人，你就把所有的人的坐标和当前距离算一遍，再排个序，肯定炸了。经过研究发现，现有的一些app，比如微信，陌陌等都是有附近的人这个功能的，而这种空间距离计算，其实都是基于geohash这个算法。
&ensp;&ensp;&ensp;&ensp;我们都知道在一位数组里，我们能够通过hash散列对列表的每个值进行映射，从而达到在不需要遍历整个数组的情况下就能拿到某个元素的值的效果。而geoHash其实和hash一样，只不过是种地址编码方法，他能够把二维的空间经纬度数据编码成一个字符串。具体原理大致描述一下：
### GeoHash原理
&ensp;&ensp;&ensp;&ensp;我们知道，经度范围是东经180到西经180，纬度范围是南纬90到北纬90，我们设定西经为负，南纬为负，所以地球上的经度范围就是[-180， 180]，纬度范围就是[-90，90]。如果以本初子午线、赤道为界，地球可以分成4个部分。如果纬度范围[-90°, 0°)用二进制0代表，（0°, 90°]用二进制1代表，经度范围[-180°, 0°)用二进制0代表，（0°, 180°]用二进制1代表，那么地球可以分成如下4个部分:
![geo_hash_1](..\picture_back_up\geo_hash_1.webp)
那我们对四个格子再次这样划分呢，就可以划分成如下：
![geo_hash2](..\picture_back_up\geo_hash2.webp)
我们看到每一个小区域都可以用一个个二进制编码进行表示，如果我们进行划分，二进制编码会更长，对应的每个区域的精度也就更精密了。
&ensp;&ensp;&ensp;&ensp;如此划分之后，对我们计算附近的人有何帮助呢，我们首先可以根据自己的坐标快速定位出自己对应的二进制编码，然后只需要计算和自己同一块区域的人的坐标和自己的距离，就可以得出结果了。从而避免最前说的需要对所有用户都进行一轮距离运算才能知道附近的人。
&ensp;&ensp;&ensp;&ensp;当然如果仅仅计算当前区域的用户可能并不精准，因为可能你的位置出在该块去区域的边角上，相对于当前区域中对角线的点，可能旁边区域里的点更近一些。为此如果你需要更精准的附近的人时，可能你需要计算当前区域以及周围八块区域里的点。不过相较于将世界上所有的点都和你进行距离运算并排序的而言，已经大大降低了数据量。


### Mongo与Redis的地理位置索引
&ensp;&ensp;&ensp;&ensp;上面简单阐述了原理，而到了具体实现，我们到不需要自己再去实现这些算法了，Redis和Mongo都有相关现成的功能，而他们的这些功能其实就是源于GeoHash这个算法。虽然redis的性能上更有优势，不过考虑到需求上需要很多维度上去进行举例筛选，所以最好还是决定使用了mongo来实现附近的人。mongo中有两种地理位置索引，2d和2dsphere，区别在于前者是2d的，2dsphere其实是三维的，会考虑地球的曲率。所以我们这里使用了2dsphere。我们对users表中的position字段添加地理位置索引：
```javascript
db.users.createIndex(
    {
        position: "2dsphere"
    },
    {
        name: "position_2dsphere",
        background: true,
        "2dsphereIndexVersion": 3
    }
)
```
mongo查询附近的人的命令：
```
db.users.aggregate({
        "$geoNear": {
			"query": {
				"_id": {
					"$in": [……]
				},
				"position": {
					"$exists": true
				},
				"sex": 1
			},
			"distanceMultiplier": 6378.137,
			"near": [121.3700992160942, 31.17387023300054],
			"spherical": true,
			"distanceField": "distance"
		}
    })
```
$geoNear命令中有多个参数，其中"query"是出距离查询以外的其它查询条件，"distanceMultiplier"是球形半径，所以这里是地球半径6378.137km，为什么需要这个参数呢，原因是mongo，默认返回的是弧度，如果大家记得话数学公式：弧长=弧度*半径，而弧长其实就是我们的所说的距离，因为地球半径的我们用的km作为单位，根据弧长公式，求出来的距离也是km为单位。"near"表示当前的位置，"distanceField"表示返回的距离的那个字段：
```json
{
	"nickname" : "呆瓜(ˇ_ˇ)",
	"sex" : 1,
	"position" : [
		121.54309,
		31.22014
	],
	"online" : false,
	"ips" : [
		"10.1.1.2"
	],
	"distance" : 17.258984441839168
}
```
返回的distance就是实际的距离这里是距离17.26km。

### SpringDataMongo进行实现
```kotlin
    /**
     * 获取附近的在线用户
     */
    fun getNearlyOnlineUser(sex: Int?, userId: Int, position: Position, page: PageReq): PageDto<NearlyUserInfo> {
        val query = Query.query(
                    Criteria.where("online").`is`(true))
        
        if (sex != null) {
            query.addCriteria(Criteria.where("sex").`is`(sex))
        } else {
            query.addCriteria(Criteria.where("sex").`in`(listOf(1, 2)))
        }

        val total = coreMongo.count(query, NearlyUserInfo::class.java)

        if (total == 0L) {
            return PageDto(emptyList(), total, page)
        }

        val nearAggregation = TypedAggregation.newAggregation(Aggregation.geoNear(NearQuery.near(Point(position.longitude, position.latitude)).query(query).spherical(true).distanceMultiplier(6378.137)
                .skip((page.qPage * page.size).toLong()).limit(page.size.toLong()), "distance"))
        val res = coreMongo.aggregate(nearAggregation, "users",
                NearlyUserInfo::class.java).mappedResults
        return PageDto(res, total, page)
    }
```