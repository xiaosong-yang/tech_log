<<<<<<< HEAD
&ensp;&ensp;&ensp;&ensp;对于列表的forEach函数是我们经常用到的，函数中带入一个lambda表达式完成对整个列表的每个元素进行处理，功能和for、while等循环获取元素并进行处理相当。但我们在for或者while中都有break，continue函数,而forEach是没有continue和break的，哪怕使用return，返回的也是外部整个函数。
&ensp;&ensp;&ensp;&ensp;不过kotlin的forEach提供了return@forEach，但是需要注意的是这个命令跳出的是当前循环，进入下一循环，相当于continue的功能。切记不要当成了break使用，我一开始就把return@forEach当成了break，那如果我们要实现break怎么做呢，如下：
```kotlin
    val list = listOf("aaa","bbb","ccc")
    list.forEach {
        if(it == "aaa"){
            return@forEach
        }
        println(it)
    }
    println("over1111")

    run breaking@{
        list.forEach {
            if(it == "bbb"){
                return@breaking
            }
            println(it)
        }
    }
```
=======
&ensp;&ensp;&ensp;&ensp;对于列表的forEach函数是我们经常用到的，函数中带入一个lambda表达式完成对整个列表的每个元素进行处理。
>>>>>>> 691d97bf63b1e039468298be6710fe4fcfe27787
