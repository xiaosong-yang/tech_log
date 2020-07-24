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
<<<<<<< HEAD
上面的函数变量的案例可以改写成Lambda表达式
=======
Lambda表达式中不要使用return，表达式的最后一行默认为表达式的返回值，如果使用return则返回的不是表达式，而是外部的整个函数。上面的函数变量的案例可以改写成Lambda表达式
>>>>>>> 92f9b39ea8d969d09454784399f6432e2cc7debe
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
&ensp;&ensp;&ensp;&ensp;内联函数是inline关键字修饰的函数，例如下面的let函数：
```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this)
}
```
所谓内联函数和普通函数的区别在于，内联函数在代码编译时，会把内联函数中的代码块拷贝到内联函数被调用的地方，也就是在代码真实执行过程中是没有内联函数的概念的，这也就意味着两点：
1. 由于代码执行的时候内联函数中的代码被复制到了调用函数里面去了，这样就节省了开辟新函数所需要的虚拟机栈，节省了开始和结束一个新函数的内存和时间消耗。同时由于赋值的原因会增加编译后源码的大小，所以是否使用内联函数需要进行权衡。
2. 前面有讲到Lambda表达式中使用return返回的不是表达式本身，而是外部函数。而由于内联函数编译时会复制到上级函数体中，所以内联函数中的Lambda表达式返回的是调用内联函数的外部函数eg：
```kotlin
fun main(args: Array<String>) {
    var a = User { a, b ->
        a + b
    }.let {
        it.fun_sum = { a, b ->
            a + b
        }
        it.fun_sum
        return
    }
    println("aaa")
}
```
上面的案例，最终不会打印"aaa"，因为表达式中return将整个main函数返回了。

###### 常用内联函数

