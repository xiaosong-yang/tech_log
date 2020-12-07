&ensp;&ensp;&ensp;&ensp;java8中引入了Lambda表达式，其中带来的一个好处就是，我们实现匿名类时变得更加简洁：
java8以前：
```java
        Thread thread1 = new Thread() {
            @Override
            public void run() {
                System.out.println("hahah");
            }
        };
        thread1.start();
```
java8以后：
```java
        Thread thread2 = new Thread(()-> System.out.println("hahah"));
        thread2.start();
```
