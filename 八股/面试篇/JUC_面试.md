# 面试题

## 并发工具类

### Semaphore

#### 是什么

信号量，是一种共享锁，用来控制同时访问特定资源的线程数量

#### 有什么用

![图 0](../../images/04cbeb1103a6e80a8de5f48664e7219124aa45bcb18bee01515ca64534808501.png)  


#### 底层是如何基于 AQS 实现的

##### 构造器是如何定义的

![图 1](../../images/8d1182e208d87c3babe5ec35923a76b49dae6aa5ed77a6bbd8e0a48e88ead55e.png)  

##### acquire

1. 获取许可证
2. 获取成功，CAS修改state
3. 获取失败，加入阻塞队列，挂起线程

![图 2](../../images/7b0812300d60eef97f29e701d06eda7f680016e17aeb152de51a271f08f288c5.png)  

##### release

1. 释放许可证
2. 释放成功，CAS修改state，唤醒下一个线程

![图 3](../../images/ff8d06f956f6d512980f6ecd5c216bedd34f28650d340bbb753a4c4eab1ea797.png)  

### CountDownLatch

#### 是什么

#### 有什么用

#### 底层是如何基于 AQS 实现的

##### 构造器如何定义

##### await

##### countdown
