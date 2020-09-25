&ensp;&ensp;&ensp;&ensp;前一章讲的标记清除，标记整理，复制这些都是算法层面的，而真正进行垃圾回收的都是各种垃圾收集器，他们是对各种垃圾回收算法的具体实现。垃圾收集器按大类可分为串行垃圾收集器和并行垃圾收集器，串行有：Serial和Serial Old。并行有：ParNew、Parallel Scavenge、Parallel Old、CMS,G1。下面我们从发展流程和各自利弊来介绍一下每一种收集器。


### Serial与Serial Old收集器
Serial：（-Serial GC -XX:+UseSerialGC）
Serial Old：（-Serial Old -XX:+UseSerialOldGC）
Serial是新生代收集器，采用复制法，Serial Old是Serial的老年代版本，采用了标记整理法，在很长一段时间，他们都是java最主要的收集器，但是由于串行问题，当进行垃圾收集的时候，程序必须停下来(stop the world)等待垃圾收集结束。虽然Serial和Serial Old作为串行收集器历史悠久，但是现在仍然有应用场景，在单核CPU下使用串行收集器，可以避免上下文的切换的损耗，一般为客户端开发的场景。  



### ParNew GC收集器
-ParNew GC -XX:+UseParNewGC
ParNew其实就是Serial的多线程版本，可以控制线程数量，参数：-XX:ParallelGCThreads。但是在很长一段时间，ParNew却是在服务端的不二之选，原因是jdk1.5出现划时代的CMS收集器，但这个收集器是一款老年代收集器，而能配合CMS在新生代使用的收集器除了Serial只有ParNew。

### Parallel和Parallel Old收集器
Parallel：（-Parallel GC -XX:+UseParallelGC）
Parallel Old：（-Parallel Old GC -XX:+UseParallelOldGC）
Parallel看上去和ParNew没什么区别，同样是多线程+复制法。但是Parallel有自己的关注点，他是吞吐量优先的垃圾收集器，什么是吞吐量优先呢，就是用户可以控制垃圾回收使用的时间，从而把更多的时间花在用户程序上，这种对于需要快速响应的场合是能起到很好的效果的，但是控制垃圾回收的时间，并不代表垃圾回收会更快，这是通过牺牲回收空间的大小来提高速度的，比如原本回收1GB需要10ms，现在你要求1ms完成垃圾回收，那垃圾收集器就只回收0.1GB的内存，也就是如果你频繁的出现内存不足时，其实吞吐量也降下来了，因为需要频繁的进行垃圾回收，那总的回收时间并没有减少，甚至增加。所以Parallel Scavenge在CPU资源敏感而内存充足的情况下希望提高吞吐量是不错的选择，但是在内存敏感的情况下并不是一个很好的选择。
相关参数：
- -XX:ParallelGCThreads：设置用来垃圾回收的线程数，通常设置为和CPU个数相同，充分利用CPU又避免上下文切换
- -XX:MaxGCPauseMills：设置最大垃圾收集停顿时间。他的值是一个大于0的整数。
- -XX:GCTimeRatio：设置吞吐量大小，他的值是一个0到100之间的整数。
- -XX:UseAdaptiveSizePolicy：打开自适应GC策略。以达到堆大小，吞吐量和停顿时间之间的平衡点。



### CMS收集器
-CMS -XX:+UseConcMarkSweepGC
CMS是老年代收集器，他采用的算法是标记清除(Mark-Sweep)，设计目的是尽可能减少停顿时间,CMS运行流程如下：
![cms流程图](..\picture_back_up\cms_process.png)
CMS回收流程分为四部分：
- 第一步初始标记，这个步骤会stop the world，但是他只标记与GC Root相关联的节点，所以非常快。
- 第二步并发标记，他和用户线程一起运行，标记那些除了直接与GC ROOT关联的点，这一步不会stop the world。
- 第三步重新标记，由于第二步没有stop the world，就会造成一些准备清除的内存又重新被使用，没有被标记的内存又可以使用了的情况。所以第三步重新标记，就是对第二步标记过的内存进行检查，这一步会stop the world。由于不像第二步那样对除直接与GC ROOT关联的所有点进行排查，所以也很快。
- 第四步并发清理，就是对那些确认要清理的对象进行清理。

CMS通过并发和多次标记减少了停顿时间，但同时也出现一些问题，一下几点需要注意：
- 浮动垃圾。由于CMS是一边清理，用户线程一边跑，所以当CMS工作时，又会产生新的垃圾，这些垃圾需要等待下一次CMS工作才会回收，也叫做浮动垃圾。这就意味着CMS不能等到老年代已经无内存可用了才进行回收，否则用户线程无多余的内存可使用，所以CMS默认当老年代内存使用了68%时就开始进行垃圾回收，同时CMS提供了一个参数：-XX:CMSInitiatingOccupancyFraction可以修改这个开始回收的阈值。如果这个阈值设置过低，就会增加CMS的工作次数，而阈值设置过高，则可能也会造成在CMS工作过程中无更多内存可以分配，这就会触发CMS的Concurrent ModeFailure报错，然后jvm会启用备用回收器（Serial Old），而Serial Old大家都知道是单线程标记整理回收器，这就会导致较长时间的系统停顿。
- 适用于多CPU场景。默认情况下CMS开启的线程数为（CPU+3）/4个，所以在CPU数量较少的情况下，不适合使用CMS收集器，因为会抢占很多CPU资源。
- 自定义内存整理。由于CMS使用的是标记-清除算法，系统运行久了就会造成大量的碎片化内存，所以CMS也可以进行内存整理，有两种方式进行开启：
- +UseCMSCompactAtFullCollection:这参数可以指定每次在Full GC时，开启内存碎片的整理工作，默认是开启的。
- -XX:CMSFullGCsBeforeCompaction:用于指定，多少次不整理碎片的FullGC后，整理一次碎片。
需要注意的是，当CMS进行碎片整理的时候，是不能并发的，也就意味着stop the world，所以上面第一种优化方式是不建议的，会造成频繁的较长时间stop the world，但也不能把-XX:CMSFullGCsBeforeCompaction设置过大，会平白增加碎片化的困扰。
最后确定一件事，CMS GC不是Full GC，而上面的Concurrent ModeFailure才是Full GC，所以上面这两个参数都是作用于退化成Serial Old的，上面我们知道Serial Old使用的标记整理法，而-XX:CMSFullGCsBeforeCompaction可以使Concurrent ModeFailure发生时采用标记清理法(至于使用标记清理法时还是不是Serial Old这个收集器就不知道了,如果有人知道，麻烦留言告诉下我)。





### G1收集器
-G1 -XX:UseG1GC
G1是JDK9之后的默认选型，目标是替代CMS,他是针对大堆内存的收集器，之前说的那些收集器和算法，当堆过大时，效率都会下降，因为需要扫描的内存变大了。G1同时是新生代老年的收集器，在G1中新生代和老年代不需要像之前那样有一个明显的边界，G1中整个对被划分成很多个小模块（region），其中一些region是新生代，一些是老年代，G1会通过一个Set集合对他们进行维护。
![G1的模型](..\picture_back_up\G1_model.jpg)
如图这样，新生代老年代混杂在一起，每一个小块就是一个region。G1的优越性在于，他会记录每一region能够回收的到的内存，和回收时需要的时间，从而知道每个region的回收效率，然后根据回收效率做一个排序，当需要进行回收时，他会优先回收回收效率高的region。
G1是局部的复制法，整体的标记清理法，就跟图中一样，会有一些碎块话的region，而比如新生代，从一个Eden块和S1块到S2块的过程则是复制法。



### 垃圾收集器的组合使用方式
![垃圾收集器组合](..\picture_back_up\gc_group.jpg)
实际使用的时候，在启动项后面加上对应的收集器参数就好了。下面是一些关于GC日志打印的jvm参数，调试时可能会用到：
```java
-XX:+PrintGC 输出GC日志
-XX:+PrintGCDetails 输出GC的详细日志
-XX:+PrintGCTimeStamps 输出GC的时间戳（以基准时间的形式）
-XX:+PrintGCDateStamps 输出GC的时间戳（以日期的形式，如 2013-05-04T21:53:59.234+0800）
-XX:+PrintHeapAtGC 在进行GC的前后打印出堆的信息
-Xloggc:../logs/gc.log 日志文件的输出路径
```
