### 函数变量
&ensp;&ensp;&ensp;&ensp;在kotlin中一个函数可以被当做一个参数一样，由入参->返参决定类型:
```kotlin
    var sum: (Int, Int) -> Int
    sum = fun(a: Int, b: Int): Int {
        return a + b
    }
    print(sum(1, 2))
```
定义一个函数变量sum，类型为(Int, Int) -> Int，表示两个入参为Int，一个出参也为Int。
&ensp;&ensp;&ensp;&ensp;如果入参为空，则函数类型可以定义为()->Int,如果出参为空可以定义为(Int,Int)->Unit，如果出参和入参都为空可以定义为()->Unit


### Lambda表达式
&ensp;&ensp;&ensp;&ensp;Lanbda表达式是一个灵活的代码块，减少代码量，Lambda表达式格式如下：
```kotlin
{参数 : 参数类型，其他参数…… ->
    逻辑块
}
```
上面的函数变量的案例可以改写成Lambda表达式
```
    var sum: (Int, Int) -> Int
    sum = { a: Int, b: Int ->
        a + b
    }
    print(sum(1, 2))
```

###### 形参类型省略
&ensp;&ensp;&ensp;&ensp;由于提前对sum指定了类型，所以kotlin可以进行类型推断，所以代码可以简写成如下：
```kotlin
    var sum: (Int, Int) -> Int
    sum = { a, b ->
        a + b
    }
    print(sum(1, 2))
```

###### 形参省略
&ensp;&ensp;&ensp;&ensp;如果入参只有一个话，形参也可以省略，使用it进行替代
```kotlin
    var isDouble: (Int) -> Boolean = {
        it % 2 == 0
    }
    print(isDouble(10))
```


###### Lambda表达式参数提出
&ensp;&ensp;&ensp;&ensp;如果入参只有一个话，形参也可以省略，使用it进行替代
```kotlin
    var isDouble: (Int) -> Boolean = {
        it % 2 == 0
    }
    print(isDouble(10))
```
