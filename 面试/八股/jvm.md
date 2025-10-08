# JVM

## 内存结构

java 1.8 之前，线程共享的有方法区，方法区由永久代实现永久代在堆中，运行时常量池在方法区中，运行时常量池中有字符串常量池。直接内存在本地内存中

java1.8之后，方法区移到本地内存中，变为元空间，在本地内存中，但是当中的字符串常量池储存在堆中

线程隔离的有程序计数器、本地方法栈、虚拟机栈

程序计数器：存的是下一条指令的地址

本地方法栈/虚拟机栈：当中存一条条栈帧，栈帧存储方法的局部变量表、操作数栈、动态链接、方法返回地址

- 局部变量表：方法内的局部变量
- 操作数栈：存储中间计算结果
- 动态链接：连接（方法）字节码和运行时常量池中的具体地址
- 方法返回地址：调用者方法的执行位置

## 双亲委派机制打破（JDBC）

- JDBC的driver接口定义在JDK中，但是它的实现类是放在classpath下的（比如MySQL）只能由应用程序类加载器（Application ClassLoader）加载；。

- DriverManager类会加载每个Driver接口的实现类并管理它们，但是DriverManager类自身是 jre/lib/rt.jar 里的类，是由bootstrap classloader加载的

- 按双亲委派，DriverManager（Bootstrap 加载）要加载驱动类时，会先委托父加载器（但 Bootstrap 没有父加载器），自己又无法访问classpath，导致驱动加载失败。

- 因此只能在DriverManager里强行指定下层classloader来加载Driver实现类，而这就会打破双亲委派模型

实现方式：通过线程的上下文类加载器默认被设置为Application ClassLoader，DriverManager通过Thread.currentThread().getContextLoader()拿到Application ClassLoader，绕过双亲委派，通过Application ClassLoader来加载驱动类