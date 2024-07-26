# 锁

## Sychronized锁入门实例

```java
public class SychronizedDemo {
    public static void main(String[] args) {
        SychronizedDemo sychronizedDemo = new SychronizedDemo();
        sychronizedDemo.deposit(1000);
        sychronizedDemo.syncmethod();
    }

    //同步方法
    public synchronized void syncmethod(){
        System.out.println(Thread.currentThread().getName() + " 执行同步方法 ");
    }

    //同步代码快
    public void deposit(long money){
        long balance = 1000;

        synchronized (this){
           try {
               Thread.sleep(1000);
           }catch (Exception e){
               System.out.println("同步失败");
               e.printStackTrace();
           }
        }
        balance+=money;
        System.out.println(Thread.currentThread().getName()+"存钱成功，余额为"+balance);
    }
}
```

注意事项：

锁的粒度：尽量要减少锁的粒度就是同步的代码的数量。

锁的对象：**sychronized** 锁，锁的是当前的对象。

可重入性：**sychronized** 锁，可以让一个线程多次获取锁。

死锁：当使用不恰当会出现死锁。



同步代码块时，sychronized 锁的是当前调用的实例对象

同步方法时，sychronized 锁的是当前调用的实例对象

同步类的静态方法时，sychronized 锁的是当前 Class 类

可以搭配Object 类的 wirte ，notify等方法灵活使用锁



## Lock锁实例

```java
public class LockDemo {

    public static long banlance = 10000;

    public static void main(String[] args) {
        LockDemo lockDemo = new LockDemo();
        lockDemo.lockMethod(1000);
        System.out.println("账户余额为："+banlance);
    }

    //使用lock锁的同步方法
    public void lockMethod(long money){
        ReentrantLock lock = new ReentrantLock();

        lock.lock();
        try {
            System.out.println("线程"+Thread.currentThread().getName()+"开始存款");
            banlance+=money;
        }catch (Exception e){
            System.out.println("线程"+Thread.currentThread().getName()+"存款失败");
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}
```

注意事项：

1. 一定要在finally中释放锁。
2. 可以搭配 condition 对象的 awirte，signal，signalAll 来灵活使用锁



## Sychronized 关键字 和 Lock锁的区别

1. Sychronized 是关键字， Lock 是接口，接口下有 ReentrantLock 和 ReentrantReaderLock，ReentrantWirteLock。
2. sychronized 自动释放锁， lock手动释放锁。
3. sychronized 线程1（获取锁），线程2 等待(阻塞)，lock 锁的话不一定会出现这种情况(可以采取中断的方式结束等待)
4. sychronized 是可重入非公平锁，lock 也是可重入非公平锁（不过Lock可以调整）
5. sychronized 适合少量的代码，lock 适合多量的代码(有try-cacth)



备注：公平锁就是先来先执行，一个一个的排队。后来的放进队列中，当先来的执行完成后再去队列中唤醒后来的。

非公平锁：不一定按照先来先执行的现象。