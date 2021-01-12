&ensp;&ensp;&ensp;&ensp;与mysql的insert不同，mongo在数据插入方面有多种方式，这里进行统一的总结一下：

### insert
&ensp;&ensp;&ensp;&ensp;和mysql一样，mongo也支持insert这种直接插入，当插入出现唯一键冲突时则会失败，抛出异常:
```javascript
db.test.insert({
    "name":"1223"
})
```


### save
&ensp;&ensp;&ensp;&ensp;save是基于主键的文档替换，如果主键存在则其他字段完全替换成新的字段，如果老的字段，新的文档中不存在，则删除了，如果主键原本不存在，这插入一条新的文档。可能会觉得这种方式挺好的，不会出现唯一键冲突，有就替换，没有就插入。在基于整个文档更新的情况下，这种方式是很好用的，但有些场景下会有两种局限性：
1. 当需要对文档局部字段进行修改时，就会有问题，可能我并不想删除原本的字段
2. 在用spring-data-mongo进行操作mongo时，是无法获得save到底是插入，还是更新，还是什么都没有修改，无法对后续的操作提供信息。
```javascript
db.test.save({
    "_id":ObjectId("5ffd1706dc20733ffce6bb03"),
    "xixi" : "1223"
})
```

### upsert
&ensp;&ensp;&ensp;&ensp;upsert和save的执行模式相同，有则更新，没有则插入。区别在于upsert可以自定义条件，不一定要基于id来做判断原有的文档是否存在，其次upsert是更新不是替换，所以需要指明要更新的每个字段如何更新,另一方面基于spring-data-mongo时可以获取到执行结果，是否匹配到，是否更新到，从而有利于后续程序的判断,算是弥补了save的两点不足。
```javascript
db.test.update({_id:"1"},{$set: {name:"xixix"}}, {upsert: true})
```
可以看到upsert属于update的一个属性进行控制，所以本职upsert其实就是update的一个特例，当原有文档不存在的情况下执行插入操作。


### upsert & $setOnInsert
&ensp;&ensp;&ensp;&ensp;对于save和upsert下基本逻辑是：有责更新，无则插入。但对于特殊场景下，我就希望只有为空的场景下操作，不为空的情况下不修改老数据。可能会说我们直接捕获唯一性异常就可以了，但是如果在事务中，抛出异常会造成事务失败，进行回滚，无法做到继续执行，所以在upsert基础上有了$setOnInsert。
```javascript
db.test.update({_id:"1"},{$setOnInsert: {name:"hahaha"}}, {upsert: true})
```
可以看到name属性只有是插入时，操作赋值"hahaha",并且最后会返回修改和匹配结果方便后续逻辑操作。



### findAndModify
&ensp;&ensp;&ensp;&ensp;上面upsert中说道了当前语句执行结果对后续逻辑的执行影响问题，在有些场景下，我们不仅需要知道当前语句是否修改成功或者匹配成功，甚至需要得到执行结果，比如我们执行inc计数累计时，我们需要知道累计后的结果，如果我们先查在更新，然后自己计算更新后的结果，这样就存在非原子性问题，造成数据不准确，所以findAndModify可以用于应对这种场景：
```javascript
db.test.findAndModify({query: {_id:ObjectId("5ffd1706dc20733ffce6bb03")}, update: {$set: {"aaa":"bbb"}}, new: true})
```
其中new:true，表示更新后返回的文档是更新过后的。其中findAndModify可以通过如下方式实现文档替换：
```javascript
db.test.findAndModify({query: {_id:ObjectId("5ffd1706dc20733ffce6bb03")}, update: {"aaa":"bbb"}, new: true})
```
也可以通过实现setOnInsert：
```javascript
db.test.findAndModify({query: {_id:ObjectId("5ffd1706dc20733ffce6bb03")}, update: {$setOnInsert: {"aaa":"bbb"}}, new: true})
```
默认呢情况下findAndModify就是upsert，如果 要只更新的话可以如下：
```javascript
db.test.findAndModify({query: {_id:"54548"}, update: {"111":"213232"},upsert:false, new: true})
```