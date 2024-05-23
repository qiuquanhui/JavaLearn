# JUC的基础概念

## 什么是JUC

JUC 是 Java.utils.concurrent 包内的类，是为了开发者可以在多线程的情况下减少竞争条件和防止死锁而出现的。

## 进程与线程

进程：一个进程包含多个线程。

线程：系统最小的调度单位。

进程是系统的**资源分配**的基本单元，一个进程拥有大量的空间与资源，线程是**资源分配**的最小单位。

Java 默认的线程：Main 线程，GC 垃圾回收线程



## 并行与并发

并发：CPU只有一个核，但是有多个进程同时运行，CPU 快速切换进程运行，所以出现类似并行的情况。

并行：CPU多个核，一个核对应一个进程，一起运行。

## 线程的五种状态

新建状态：线程刚创建，还没有启动。
运行状态：线程正在运行。
堵塞状态：线程在等待某个条件，条件满足后，线程恢复运行。
等待状态：线程在运行，但是由于某些原因（比如：等待IO）暂停了。
死亡状态：线程已经结束。

```java
 public enum State {
        /**
           线程还没开始启动的状态
         * 新建状态.
         */
        NEW,

        /**
         * T运行状态.
         */
        RUNNABLE,

        /**
         * 堵塞状态.
         */
        BLOCKED,

        /**
         * 等待状态.
         */
        WAITING,

        /**
         * 定时等待
         */
        TIMED_WAITING,

        /**
         * 摧毁状态.
         */
        TERMINATED;
    }
```

## JUC的架构

![img](https://cdn.nlark.com/yuque/0/2024/jpeg/33747484/1714893271476-bc17a9d6-b65a-49d4-8c23-6dbc884957af.jpeg)

tools：提供同步的辅助类，如 **CountDownLatch**（闭锁），**CylicBarrier**（栅栏）和**Semaphore**（信号量），用于线程间的协调与同步。

executor：线程池的顶级接口，**ExecutorService** 是其的子接口，常用的实现类有：**ThreadPoolExecutor**（常用），SingleThreadExecutor（单个），CachedThreadPool（缓存）。

atomic：提供原子操作，比如 **AtomicBoolean**，**AtomicInterger**，一般用于多线程的环境下进行原子操作，保证操作原子性。

Locks：存放一些锁，功能更强大，比如 **ReentrantLock** 可重入锁，**ReentrantReadWriteLock**

Collections：并发集合类，比如 **copyonwirteArrayList**，**concurrentHashMap**



