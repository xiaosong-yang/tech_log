&ensp;&ensp;&ensp;&ensp;分布式锁，顾名思义即在多个进程间做一个拦截，让程序走到某一段代码时只能依次通过，不能同时执行，从而避免 一些问题。常用实现分布式锁的方式有zk，redis，数据库等这些多套服务会共同访问的服务来实现。上一章详细介绍了zk的各种特点和机制，所以这里基于这些特点和机制来进行一个分布式锁的应用。
# 临时节点+watch
&ensp;&ensp;&ensp;&ensp;zk的节点不可重复创建，所以我们便可以通过这个特性来进行加锁。
```mermaid
flowchat
st=>start: 开始
e=>end: 结束
create_node=>operation: 创建节点
if_creater_success=>condition: 是否创建成功
create_watch=>operation: 创建watch监听
block=>operation: 线程wait，等待监听结果：节点被删除
delete_watch=>operation: 存在watch，则删除watch

st->create_watch->create_node->if_creater_success
if_creater_success(no)->block
create_watch->create_node
block->create_node
if_creater_success(yes)->delete_watch
delete_watch->e
```
&ensp;&ensp;&ensp;&ensp;这里为什么是先加监听，后去创建节点呢，原因是，如果先创建节点，后加watch，那么可能你创建节点的时候，zk上节点还存在，你去监听节点的时候，节点已经没了，那你的这个锁就陷入死锁了，除非设置了超时时间，否则就再也拿不到监听了，也就没法唤醒阻塞了。
下面是代码：
```java
package zookeeper;

import org.I0Itec.zkclient.IZkDataListener;
import org.I0Itec.zkclient.ZkClient;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.LockSupport;

/*
分布式锁实现1：
原理：节点不可重复创建+watch
惊群效应，节点一删除，就会给一堆的客户端返回watch*/
public class ZKDistrubitionLock {


    private ZkClient client;
    private String lockPath;

    public ZKDistrubitionLock(String lockPath) {
        this.client = new ZkClient("localhost:2181");
        //这是设置序列化，和分布式锁没关系
        this.client.setZkSerializer(new MyZkSerializer());
        this.lockPath = lockPath;
    }


    public void lock() {
        Thread mainThread = Thread.currentThread();
        IZkDataListener iZkDataListener = new IZkDataListener() {
            @Override
            public void handleDataChange(String s, Object o) throws Exception {

            }

            @Override
            public void handleDataDeleted(String s) throws Exception {
                //如果数据被删除，唤醒
                LockSupport.unpark(mainThread);
            }
        };
        client.subscribeDataChanges(lockPath, iZkDataListener);
        while (!tryLock()) {
            //阻塞
            LockSupport.park();
        }
        client.unsubscribeDataChanges(lockPath,iZkDataListener);
    }


    public boolean tryLock() {
        //利用节点不可重名实现分布式锁
        try {
            client.createEphemeral(lockPath);
        } catch (RuntimeException e) {
            System.out.println(e.getMessage());
            return false;
        }
        return true;
    }

    public void unlock() {
        client.delete(lockPath);
    }

	//这是测试函数
    public static void main(String[] args) throws InterruptedException {
        TestInteger testInteger = new TestInteger();
        CountDownLatch countDownLatch = new CountDownLatch(2);
        ZKDistrubitionLock zkDistrubitionLock = new ZKDistrubitionLock("/lock");
        ZKDistrubitionLock zkDistrubitionLock2 = new ZKDistrubitionLock("/lock");
        Thread thread1 = new Thread(() -> {

            int index = 0;
            while (index < 100) {
            	//如果不设置分布式锁，就去除这行
                zkDistrubitionLock.lock();
                int a = testInteger.i;
                try {
                    Thread.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                int b = a + 1;
                testInteger.i = b;
                index++;
                //如果不设置分布式锁，就去除这行
                zkDistrubitionLock.unlock();
            }
            countDownLatch.countDown();
        });
        Thread thread2 = new Thread(() -> {

            int index = 0;
            while (index < 100) {
                //如果不设置分布式锁，就去除这行
                zkDistrubitionLock2.lock();
                int a = testInteger.i;
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                int b = a + 1;
                testInteger.i = b;
                index++;
                //如果不设置分布式锁，就去除这行
                zkDistrubitionLock2.unlock();
            }
            countDownLatch.countDown();
        });
        thread1.start();
        thread2.start();
        countDownLatch.await();
        System.out.println("i=" + testInteger.i);
    }
}

```

如果不加锁，跑出来的i会是各种乱七八糟的，而加了锁跑出来的永远是200，可见锁的效果有了。（代码中加了个小睡眠，原因是不加这个小睡眠的话，100次叠加，很难出现并发造成脏数据，所以加了个小的顺便，让两个线程执行时速度不一致，容易出现脏输出。）
&ensp;&ensp;&ensp;&ensp;这种利用zk节点不可重复的能力来实现分布式锁有时候会比较鸡肋，这里仅仅是有两个人在进行抢锁，如果是十个二十个，那每次删除节点时，zk都需要像所有服务器发送这条删除监听，不是特别理想，所以下面有第二种实现方式。


# 临时有序节点+watch
&ensp;&ensp;&ensp;&ensp;这种实现有点类似于我们去银行或者医院排队的模型，我们每次去银行办理业务的时候，都需要取个号，然后等，直到你前面一个号被处理完了，那下一个就是你了。所以同理这个模型也是如此：
```mermaid
flowchat
st=>start: 开始
e=>end: 结束
create_node=>operation: 创建顺序节点
get_all_node=>operation: 获取所有节点
sort_node=>operation: 节点排序
is_first=>condition: 当前节点是否为第一个
create_watch_before=>operation: 创建watch，监听前一个节点
get_all_node_2=>operation: 获取所有节点
sort_node_2=>operation: 节点排序
is_first_2=>condition: 当前节点是否为第一个
block=>operation: 线程wait，等待监听结果：节点被删除
delete_watch=>operation: 存在watch，则删除watch

st->create_node->get_all_node->sort_node
sort_node->is_first
is_first(yes, right)->delete_watch
is_first(no)->create_watch_before->get_all_node_2->sort_node_2->is_first_2
is_first_2(yes)->delete_watch
is_first_2(no)->block
block->delete_watch
delete_watch->e

```
&ensp;&ensp;&ensp;&ensp;在流程图中，获取所有节点，节点排序，然后判断是否为第一个在创建对前一个节点的监听之后，又走了一遍，原因和前面一种实现一样，要看一下前一个节点是否还存在，确保你创建的监听将来能够监听到变化。代码如下：

```java
package zookeeper;

import org.I0Itec.zkclient.IZkDataListener;
import org.I0Itec.zkclient.ZkClient;

import java.text.SimpleDateFormat;
import java.util.Collections;
import java.util.Date;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.locks.LockSupport;

public class ZKDistrubitionLock2 {


    private ZkClient client;

    /**
     * 父节点
     */
    private String lockPath;


    /**
     * 当前节点
     */
    private ThreadLocal<String> currentPath = new ThreadLocal<>();

    /**
     * 前一个节点
     */
    private ThreadLocal<String> beforePath = new ThreadLocal<>();


    public ZKDistrubitionLock2(String lockPath) {
        this.client = new ZkClient("localhost:2181");
        this.client.setZkSerializer(new MyZkSerializer());
        this.lockPath = lockPath;
    }


    public void lock() {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
        String today = sdf.format(new Date());
        //这个node是完整路径
        String name = client.createEphemeralSequential(lockPath + "/" + today, "lock");
        currentPath.set(name);
        //而这个list中的子节点，仅仅是一个node名字
        List<String> list = client.getChildren(lockPath);
        Collections.sort(list);
        if (name.equals(lockPath + "/" + list.get(0))) {
            return;
        }
        int index = list.indexOf(currentPath.get().substring(lockPath.length() + 1));
        String beforePath = lockPath + "/" + list.get(index - 1);
        Thread mainThread = Thread.currentThread();
        IZkDataListener iZkDataListener = new IZkDataListener() {
            @Override
            public void handleDataChange(String s, Object o) throws Exception {

            }

            @Override
            public void handleDataDeleted(String s) throws Exception {
                LockSupport.unpark(mainThread);
            }
        };
        client.subscribeDataChanges(beforePath, iZkDataListener);


        list = client.getChildren(lockPath);
        Collections.sort(list);
        if (name.equals(lockPath + "/" + list.get(0))) {
            client.unsubscribeDataChanges(beforePath, iZkDataListener);
            return;
        }
        LockSupport.park();
        client.unsubscribeDataChanges(beforePath, iZkDataListener);

    }


    public void unlock() {
        client.delete(currentPath.get());
    }


    public static void main(String[] args) throws InterruptedException {
        TestInteger testInteger = new TestInteger();
        CountDownLatch countDownLatch = new CountDownLatch(2);
        ZKDistrubitionLock2 zkDistrubitionLock = new ZKDistrubitionLock2("/lock");
        ZKDistrubitionLock2 zkDistrubitionLock2 = new ZKDistrubitionLock2("/lock");
        Thread thread1 = new Thread(() -> {

            int index = 0;
            while (index < 100) {
                zkDistrubitionLock.lock();
                int a = testInteger.i;
                try {
                    Thread.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                int b = a + 1;
                testInteger.i = b;
                index++;
                zkDistrubitionLock.unlock();
            }
            countDownLatch.countDown();
        });
        Thread thread2 = new Thread(() -> {

            int index = 0;
            while (index < 100) {
                zkDistrubitionLock2.lock();
                int a = testInteger.i;
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                int b = a + 1;
                testInteger.i = b;
                index++;
                zkDistrubitionLock2.unlock();
            }
            countDownLatch.countDown();
        });
        thread1.start();
        thread2.start();
        countDownLatch.await();
        System.out.println("i=" + testInteger.i);
    }

}

```

