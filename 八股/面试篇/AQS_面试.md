# 面试题

## 什么是 AQS

![图 0](../../images/c9e0dd31077498bb61b9712bab99e663425dfe0e2d5801dbd873ab8ae423d84c.png)  

## 有哪些类是基于 AQS 实现的

- `ReentrantLock`
- 并发工具类

![图 1](../../images/c2f3bc326bab203f6f55db685869923ac73868ba54e4ba7d3ece5a4eee3b4e4e.png)  

## AQS 解决什么问题

![图 2](../../images/3b8c88a975764a37f8096373338788f374c99bae69ab9b4abebe2da13d53de28.png)  

- 提供通用框架，用于实现各种同步器
- 定义了资源获取和释放的通用流程

## AQS 同步队列的基本结构

![图 3](../../images/a6041e9f3b8f4f3fc3d255e1147fb4dbdc41a30beda2d12c3b7ba30c1c257e2f.png)  

### AQS 自身

#### state

```java
// 共享变量，使用volatile修饰保证线程可见性
private volatile int state;
```

- AQS 使用 int 成员变量 `state` 表示同步状态
- `state` 变量由 `volatile` 修饰，用于展示当前临界资源的获取情况
- 这个值是和 `monitor` 中的计数器有点像，支持可重入

![图 4](../../images/3f8e2a004ce7a4d722535df05d6f7d3992f9947ef9422a41aafdf1fbaf6e9fe4.png)  

![图 5](../../images/2477c583ac809c450ba3c93da616c53c28b49d7bd4ada7815de47476476fa593.png)  

#### AQS 的 CLH 队列

- AQS 中的 CLH 队列是 CLH 锁队列的变体
- 通过内置的 **FIFO 线程等待/等待队列** 来完成获取资源线程的排队工作。

##### 原版 CLH 锁队列

![图 6](../../images/46a1ba17d7fd2020eb3b2e852da57eb33a3e078ea2c12d67b6ffca7fe4326ee3.png)  

##### AQS 中的队列

- 变成了双向队列
- 不需要一直自旋，自旋获取锁失败了就阻塞
- 因为是双向队列，所以前驱节点对应的线程释放锁之后，可以主动唤醒后面的线程

![图 7](../../images/110171c3eaa59b5d38236f74885b7834e03d5af6923923adcf2b530c6b00415c.png)  

### 内部类 Node

#### 内部结构

![图 8](../../images/0fced20b8f1e79de3a09da3f09e5a6fdc62aa9b0652980bc596984904ac45613.png)  

#### waitStatus，用 volatile 修饰

![图 9](../../images/53fb3a6ee3829dd3dfee955ce2bbd1df948c238cabbf8953777335cef24f8d5d.png)  

- 何为等待队列何为同步队列？<https://blog.csdn.net/zx48822821/article/details/86768484>

## ReentrantLock 底层如何基于 AQS 实现

- Lock接口的实现类，基本都是通过聚合了一个队列同步器的子类完成线程访问控制的

![图 10](../../images/203282be264286b8d42fa0acf1bd029c31be63a21d0e7ec4a1bcd31457991b9f.png)  

### 创建锁

- 默认为非公平锁（效率高）

![图 11](../../images/0ba985545486a2b9169437c2ac772007a0ed982c4a76e35b972eaefe8245652a.png)  

### 加锁

- 公平锁需要判断是否需要排队

![图 14](../../images/5d97e83c49dc15b40e22d777e181f7be4beb9db7313ced70e0b2876dca950ec5.png)  

![图 12](../../images/aeeccd5970f1f2ceea5d5197889ab0fe5815248835fa64b6263d813c85fb8a05.png)  

![图 13](../../images/a7723d48dfc7f3b16efdc6949e805f2a9177d1de076ef8c75222965d476e410c.png)  

![图 16](../../images/fbc31df1412109c8bc7b7857a22b560b291d67e04f74900c6448a5e975f5fb7e.png)  

![图 17](../../images/59b83f960098367105c56e05f1fc5626dabb3e144e4af079ce34755a072e138c.png)  

![图 18](../../images/0de99bc68aa245303e0c79da5996db93b06b151e0a8cc70338eebfd35db9746c.png)  

#### lock()

![图 20](../../images/b18179455ac7698a11b96dcf1f1a1d04021592e26aa74cd56f13e870ec8d7f28.png)  

![图 19](../../images/e2f5e7730d8c25bbbc4de51d8262357b13d52b8027c5820f750a17d305386eba.png)  

#### acquire()

![图 21](../../images/258103e6c99e030b712e3319f7acd7cdd539c386b3e0e938cbaf2bdd660e00c3.png)  

- 调用 tryAcquire ，尝试获得锁

![图 22](../../images/f05eafad2cf29fc7060bc7ce04e35b8ac39c684da0dd3a58ec320b7851cd9c68.png)  

- 调用 addwaiter ，通过enq入队

![图 23](../../images/eb780c3d24f64e814b335efe08d14467804eea4be6a803dca9e6e5209c4552e9.png)  

- 调用 acquireQueued ，坐稳队列

```java
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {//死循环
                final Node p = node.predecessor();//获得该node的前置节点
                /**
                * 如果前置节点是head，表示之前的节点就是正在运行的线程，表示是第一个排队的
（一般讲队列中第一个是正在处理的，可以想象买票的过程，第一个人是正在买票(处理中)，第二个才是真正排队的人）；
那么再去tryAcquire尝试获取锁，如果获取成功，说明此时前置线程已经运行结束，则将head设置为当前节点返回
                *
                *
                **/
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC，将前置节点移出队列，这样就没有指针指向它，可以被gc回收
                    failed = false;
                    return interrupted;//返回false表示不能被打断，意思是没有被挂起，也就是获得到了锁
                }
                /**shouldParkAfterFailedAcquire将前置node设置为需要被挂起，
                    注意这里的waitStatus是针对当前节点来说的，
                    即是前置node的ws指的是下一个节点的状态**/
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())//挂起线程 park()
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);//如果失败取消尝试获取锁(从上面的代码看只有进入p == head && tryAcquire(arg)这个逻辑是才会触发，这个时候前置节点正好在当前节点入队的时候执行完，当前节点正好获得锁，具体的代码以后分析)
        }
    }
//看到因为是死循环，所以当执行到parkAndCheckInterrupt()时，当前线程被挂起，等到某一天被unpark继续执行，这个时候已经是对头的第二个节点了，那么就会进入if (p == head && tryAcquire(arg))逻辑获取到锁并结束循环
```

### 解锁

![图 27](../../images/6c2a86f967b54b44701ff42aad9efee0f96147e03659d7deb7a48a9eb79f8865.png)  

- 释放锁（修改资源的占有状态，即修改 `state` 的值）
- 唤醒头节点的后置节点（获取下一个节点，唤醒）
