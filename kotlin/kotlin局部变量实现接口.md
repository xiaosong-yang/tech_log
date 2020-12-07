&ensp;&ensp;&ensp;&ensp;java8中引入了Lambda表达式，其中带来的一个好处就是，我们实现匿名类时变得更加简洁：
java8以前：
```java
        //对于类的方法覆盖
        Thread thread1 = new Thread() {
            @Override
            public void run() {
                System.out.println("hahah");
            }
        };
        thread1.start();

        //对于接口实现
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("lallal");
            }
        };
        runnable.run();        
```
java8以后：
```java
        //对于类的方法覆盖
        Thread thread2 = new Thread(()-> System.out.println("hahah"));
        thread2.start();
        //对于接口的实现
        Runnable runnable2 = ()->{
            System.out.println("lallal");
        };
        runnable2.run();
```


kotlin中实现：
```kotlin
    val runnable = Runnable {
        print("hahaha")
    }
    runnable.run()
    val thread = Thread{
        print("hahaha")
    }
    thread.start()
```
而对于以下这种复杂的接口：
```kotlin
interface Content{
    val t1:String

    val t2:Int

    val t3:Int

    fun test()
}
```
实现如下：
```kotlin
    val content = object :Content{
        override val t1: String
            get() = "xxx"
        override val t2: Int
            get() = 5485
        override val t3: Int
            get() = 2323

        override fun test() {
            println("${t1}${t2}${t3}")
        }
    }
    content.test()
```