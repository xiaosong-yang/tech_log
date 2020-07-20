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
Lambda表达式中不要使用return，表达式的最后一行默认为表达式的返回值，如果使用return则返回的不是表达式，而是外部的整个函数。上面的函数变量的案例可以改写成Lambda表达式
```kotlin
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
&ensp;&ensp;&ensp;&ensp;当使用Lambda表示是作为入参时，如果该参数在形参列表的最后一位，可以将Lambda表达式直接提取到形参表达式以外
```kotlin
data class User(val id: Int, val fun_sum: (Int,Int)->Int) {
}

fun main(args: Array<String>) {
    var user = User(10){a,b->
        a+b
    }
    print(user.fun_sum(1,2))
}
```
我们在创建user对象时，由于第二个参数是Lambda表达式，所以直接提取到外部。

###### Lambda表达式提出并省略形参括号
&ensp;&ensp;&ensp;&ensp;当使用Lambda表示是作为入参时，如果形参列表只有一个Lambda表达式作为参数，可以将Lambda表达式提取到形参表达式以外并省略形参列表的()
```kotlin
data class User(val fun_sum: (Int,Int)->Int) {
}

fun main(args: Array<String>) {
    var user = User{a,b->
        a+b
    }
    print(user.fun_sum(1,2))
}
```

### 内联函数
&ensp;&ensp;&ensp;&ensp;内联函数是inline
```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this)
}
```