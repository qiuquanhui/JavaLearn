# reentrantLock

`ReentrantLock`是`java.util.concurrent.locks`包中提供的一个可重入的互斥锁，是`java.util.concurrent`包中对同步的一种强化。`ReentrantLock`提供了比内置`synchronized`关键字更丰富的功能，包括更灵活的锁定操作和对锁状态的查询能力。

### 可重入性
`ReentrantLock`是可重入的，这意味着同一个线程可以多次获得同一个锁而不会发生死锁，前提是该线程已经持有锁。每次锁重入时，持有计数将递增，当线程退出同步块时，持有计数将递减。只有当计数达到0时，锁才被释放。

### 锁的基本操作

- **`lock()`**：调用此方法的线程将获得锁，如果锁已被其他线程持有，则当前线程将被阻塞直到锁被释放。
- **`unlock()`**：调用此方法的线程将释放锁，如果当前线程持有计数降至0，则释放锁。
- **`tryLock()`**：尝试获得锁而不进行等待。如果锁立即可用，则返回true，并持有锁；如果锁被其他线程持有，则返回false。
- **`tryLock(long timeout, TimeUnit unit)`**：尝试在给定的时间内获得锁，如果在给定时间内获得了锁，或者当前线程在等待锁的过程中被中断，会返回相应的结果。
- **`lockInterruptibly()`**：如果当前线程未被中断，则获得锁，如果已被中断则抛出`InterruptedException`。

### 公平性和非公平性

`ReentrantLock`支持创建公平锁和非公平锁。

- **公平锁**：在构造`ReentrantLock`时传入`true`，意味着等待锁的线程会按照请求锁的时间顺序获得锁，这种方式减少了饥饿，但可能导致更大的总体开销，因为要维护一个有序队列。
- **非公平锁**：默认情况下（或传入`false`），锁采用非公平方式，即不保证等待锁的线程会按照请求锁的时间顺序获得锁。这通常有较高的吞吐量。

### 条件变量（Condition）

`ReentrantLock`提供了一个`Condition`类，每个`ReentrantLock`对象可以同时有多个`Condition`对象。

- **`await()`**：类似于`Object`类的`wait()`方法，它使当前线程等待，直到其他线程调用`Condition`的`signal()`或`signalAll()`方法唤醒线程。
- **`signal()`**：类似于`Object`类的`notify()`方法，它唤醒等待在`Condition`上的单个线程。
- **`signalAll()`**：类似于`Object`类的`notifyAll()`方法，它唤醒等待在`Condition`上的所有线程。

### 查询方法

- **`isHeldByCurrentThread()`**：查询当前线程是否持有此锁。
- **`isLocked()`**：查询此锁是否被任何线程持有。
- **`hasQueuedThreads()`**：查询是否有线程正在等待获取此锁。

### 性能

相较于`synchronized`，`ReentrantLock`在某些高竞争环境下可能提供更好的性能。此外，它提供了更加丰富和精细的锁控制，使得开发者能够构建更复杂的同步结构。

### 使用场景

`ReentrantLock`通常用于那些需要高度同步控制的场景，或者是`synchronized`无法满足需求的场景，如需要尝试锁、定时锁、可中断锁、公平锁或者需要多个条件变量。



`ReentrantLock`锁适用于以下几个场景：

1. **互斥同步**：`ReentrantLock`提供了互斥锁的功能，确保多个线程在更新同一个共享资源（在这个例子中是账户余额）时，同一时间只有一个线程能执行相关操作。

2. **尝试锁定与超时**：如果在一个更复杂的场景中需要尝试锁定资源，而不是无限期等待，那么`ReentrantLock`提供的`tryLock()`方法能够实现这一功能。这在上面的简单示例中没有展示，但在需要管理锁等待时间的场景下非常有用。

3. **可中断的锁获取操作**：使用`lockInterruptibly()`方法可以在等待锁的过程中响应中断。这在处理长时间等待的锁时提供了更好的控制，允许在必要时中断等待过程。

4. **公平锁**：虽然在提供的例子中没有使用，但`ReentrantLock`允许创建公平锁，这意味着等待时间最长的线程会优先获得锁。这有助于减少线程饿死的可能性，提供线程获取锁的公平性。

5. **条件变量**：`ReentrantLock`配合`Condition`对象，可以让线程在某些条件不满足时暂停执行（`await()`），并在条件可能已变为满足时被唤醒（`signal()`或`signalAll()`）。这比Object类的`wait()`和`notify()`方法提供了更细粒度的线程协作机制。

6. **锁的状态查询**：`ReentrantLock`提供了方法来查询锁的状态，如是否被持有、是否被当前线程持有、是否有线程在等待这个锁，这样的信息在`synchronized`中是不可获取的。

总结来说，`ReentrantLock`在上面的场景中提供了比`synchronized`关键字更高级的操作，能够更细粒度地控制锁的持有与释放，以及线程的等待与唤醒，从而使得同步更加灵活，更适用于复杂的同步需求。



总之，`ReentrantLock`是一个强大的同步机制，它为复杂的同步问题提供了更灵活的解决方案。然而，它也比`synchronized`更加复杂，需要更谨慎的错误处理（特别是确保在finally块中释放锁），因此在使用时需要权衡利弊。





下面是一个使用`ReentrantLock`的简单示例，演示了如何在一个模拟的银行账户转账场景中使用它来确保线程安全。

这个例子中，我们有一个`BankAccount`类，用于表示银行账户。我们将使用`ReentrantLock`来确保在执行转账操作时，账户余额的读写操作是线程安全的。

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class BankAccount {
    private double balance; // 账户余额
    private final Lock lock = new ReentrantLock(); // 创建一个ReentrantLock实例

    public BankAccount(double initialBalance) {
        this.balance = initialBalance;
    }

    // 存款
    public void deposit(double amount) {
        lock.lock(); // 获取锁
        try {
            if (amount > 0) {
                balance += amount;
                System.out.println("存款成功: " + amount + "，当前余额: " + balance);
            }
        } finally {
            lock.unlock(); // 释放锁
        }
    }

    // 取款
    public void withdraw(double amount) {
        lock.lock(); // 获取锁
        try {
            if (amount > 0 && amount <= balance) {
                balance -= amount;
                System.out.println("取款成功: " + amount + "，当前余额: " + balance);
            }
        } finally {
            lock.unlock(); // 释放锁
        }
    }

    // 获取当前余额
    public double getBalance() {
        lock.lock(); // 获取锁
        try {
            return balance;
        } finally {
            lock.unlock(); // 释放锁
        }
    }
}

public class BankAccountDemo {
    public static void main(String[] args) {
        BankAccount account = new BankAccount(1000); // 初始余额1000

        // 创建并启动两个线程进行存取款操作
        Thread t1 = new Thread(() -> {
            account.deposit(300);
            account.withdraw(50);
        });

        Thread t2 = new Thread(() -> {
            account.deposit(100);
            account.withdraw(200);
        });

        t1.start();
        t2.start();

        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("最终余额: " + account.getBalance());
    }
}
```

在这个示例中：

- `BankAccount`类有一个`balance`属性来存储账户余额，以及一个`ReentrantLock`实例`lock`用于控制对`balance`的访问。
- 我们定义了`deposit(double amount)`和`withdraw(double amount)`方法来执行存款和取款操作。在这两个方法内部，我们首先调用`lock.lock()`来获取锁，保证同一时间只有一个线程可以执行这段代码。在`try`块内执行实际的存取款操作，在`finally`块内释放锁，确保锁一定会被释放。
- `getBalance()`方法同样使用锁来确保读取余额的操作是线程安全的。

这个例子展示了如何使用`ReentrantLock`来保证多线程环境下对共享资源的安全访问。