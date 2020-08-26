先上代码
```kotlin
    @Transactional
    fun bindParentQd(childQdName: String, parentQdId: String) {
        try {
            //插入子渠道表
            qdDao.insertChildQd(childQdName, parentQdId)
        } catch (e: DuplicateKeyException) {
            //插入冲突，说明有人插入过了，进行更新
            val cnt = qdDao.bindParentQd(childQdName, parentQdId)
            if (cnt == 0) {
                throw RuntimeException("更新失败，0件")
            }
        }
        //更新父渠道表的子渠道数量
        if (qdDao.incParentQdChildCnt(1, parentQdId) == 0) {
            throw RuntimeException( "子渠道【${childQdName}】绑定的父渠道【${parentQdId}】不存在")
        }
    }
```
运行后报错：
```
org.springframework.data.mongodb.MongoTransactionException: Command failed with error 251 (NoSuchTransaction): 'Transaction 1 has been aborted.' on server 127.0.0.1:30000. The full response is {"errorLabels": ["TransientTransactionError"], "operationTime": {"$timestamp": {"t": 1598437778, "i": 2}}, "ok": 0.0, "errmsg": "Transaction 1 has been aborted.", "code": 251, "codeName": "NoSuchTransaction", "$clusterTime": {"clusterTime": {"$timestamp": {"t": 1598437778, "i": 2}}, "signature": {"hash": {"$binary": {"base64": "AAAAAAAAAAAAAAAAAAAAAAAAAAA=", "subType": "00"}}, "keyId": 0}}}; nested exception is com.mongodb.MongoCommandException: Command failed with error 251 (NoSuchTransaction): 'Transaction 1 has been aborted.' on server 192.168.31.226:30000. The full response is {"errorLabels": ["TransientTransactionError"], "operationTime": {"$timestamp": {"t": 1598437778, "i": 2}}, "ok": 0.0, "errmsg": "Transaction 1 has been aborted.", "code": 251, "codeName": "NoSuchTransaction", "$clusterTime": {"clusterTime": {"$timestamp": {"t": 1598437778, "i": 2}}, "signature": {"hash": {"$binary": {"base64": "AAAAAAAAAAAAAAAAAAAAAAAAAAA=", "subType": "00"}}, "keyId": 0}}}

	at org.springframework.data.mongodb.core.MongoExceptionTranslator.translateExceptionIfPossible(MongoExceptionTranslator.java:130)
	at org.springframework.data.mongodb.core.MongoTemplate.potentiallyConvertRuntimeException(MongoTemplate.java:2874)
	at org.springframework.data.mongodb.core.MongoTemplate.execute(MongoTemplate.java:568)
	at org.springframework.data.mongodb.core.MongoTemplate.doUpdate(MongoTemplate.java:1625)
	at org.springframework.data.mongodb.core.MongoTemplate.updateFirst(MongoTemplate.java:1548)
	at yyf256.top.springboottest.dao.QdDao.bindParentQd(QdDao.kt:37)
	at yyf256.top.springboottest.dao.QdDao$$FastClassBySpringCGLIB$$3c385660.invoke(<generated>)
```
报错原因是走入了catch了得DuplicateKeyException里逻辑，当执行qdDao.bindParentQd(childQdName, parentQdId)更新操作时报错了。先说一下这一段逻辑，有两张表，子渠道表和父渠道表，子渠道可以绑定父渠道，父渠道表里会记录所属子渠道的数量。bindParentQd这个函数就是执行子渠道绑定父渠道的操作，如果子渠道不存在，顺便要落到子渠道表里。所以这里的逻辑就是先执行插入子渠道，如果渠道id冲突了，说明子渠道已经存在，则取更新子渠道表中的父渠道id，最后再去父渠道表里将所属子渠道数量进行+1，但是如果说所属父渠道不存在，则需要将先前的更新或者插入的操作一起回滚了。整个逻辑就是这样，但是当子渠道已经存在，走到qdDao.bindParentQd(childQdName, parentQdId)时就报上面的错了。
&ensp;&ensp;&ensp;&ensp;造成这个问题更深层次的原因其实没有找到，以后弄明白了在补上，这里往下说解决方案。将这个事务拆成两个事务：
```kotlin
    @Transactional
    fun insertBindParentQd(childQdName: String, parentQdId: String) {
        //插入子渠道表
        qdDao.insertChildQd(childQdName, parentQdId)
        //更新父渠道表的子渠道数量
        if (qdDao.incParentQdChildCnt(1, parentQdId) == 0) {
            throw BizCodeException(BizCodes.C_5911, "子渠道【${childQdName}】绑定的父渠道【${parentQdId}】不存在")
        }
    }


    @Transactional
    fun updateBindParentId(childQdName: String, parentQdId: String) {
        val cnt = qdDao.bindParentQd(childQdName, parentQdId)
        if (cnt == 0) {
            throw BizCodeException(BizCodes.C_5901, "子渠道【${childQdName}】已关联其他父渠道")
        }
        //更新父渠道表的子渠道数量
        if (qdDao.incParentQdChildCnt(1, parentQdId) == 0) {
            throw BizCodeException(BizCodes.C_5911, "子渠道【${childQdName}】绑定的父渠道【${parentQdId}】不存在】")
        }
    }
```
然后将DuplicateKeyException这个异常放到事务外面去捕获：
```kotlin
    try {
        qdHelper.insertBindParentQd(it, bindParentQdRequest.parentQdName)
    } catch (e: DuplicateKeyException) {
        qdHelper.updateBindParentId(it, bindParentQdRequest.parentQdName)
    }
```