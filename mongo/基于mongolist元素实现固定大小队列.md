&ensp;&ensp;&ensp;&ensp;最近手上有一个需求，需要记录任意两人的会话，并且需要记录最近N条聊天记录。我们首先可以想到一个表记录所有人的聊天记录：发送方，接收方，发送内容，时间。然后一个表记录会话，用户1，用户2，最新的会话时间。然后根据会话表查询聊天记录表获取两人之间的按时间倒序的聊天内容。但是如果最近的N条聊天记录要同会话列表有一同展示就有点头秃了。如果查到一个会话列表，然后遍历去查每个会话的前N条聊天记录。那假设每查一次200ms（因为聊天记录表是庞大的，就算加上了索引，也在几百毫秒的性能损耗），那如果一个列表50条数据，这个接口就得至少等10s才会有返回。这性能，炸裂了。
&ensp;&ensp;&ensp;&ensp;基于上面的性能考虑，决定在每次插入会话表的时候，实现一个固定大小的队列来保存会话的最近N条记录。好在mongo可以原子级的实现这个功能，mongo实现：
```javascript
db.test_col.update({ _id: 1 }, { $push: { "list": { $each: [{ "list.name": "eee", "list.time": 3 }], $slice: 3, $position: 0 } } }, { upsert: true })
```
在test_col这个集合中，对于id为1的文档，将其list元素，增加一条记录：{ "list.name": "eee", "list.time": 3 }, \$slice限制list的总大小，\$position设置插入位置在最前面。这样就能实现一个固定大小队列了，当元素超过队列大小时，会自动剔除队列底部的内容,来保证队列的大小不变。最主要的是整套操作是原子的。

&ensp;&ensp;&ensp;&ensp;那从java/kotlin基于spring-data-mongo怎么实现呢，如下：
```java
    fun save(id: String, user1: ChatSessionLog.UserInfo, user2: ChatSessionLog.UserInfo, latestChat: ChatSessionLog.ChatContent) {
        val query = Query.query(Criteria.where("_id").`is`(id))
        val update = Update()
                .set("user1", user1)
                .set("user2", user2)
                .set("update_time", LocalDateTime.now())
                .set("read_flag", 0)
                .push("latest_chat_content")).atPosition(0).slice(4).each(latestChat)
        ttlLogMongo.upsert(query, update, ChatSessionLog::class.java)
    }
```
