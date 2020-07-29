线程生命周期

![image-20200719165516737](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200719165516737.png)

# 线程同步机制

线程同步机制的实现：锁，volatile 关键字， final 关键字，static 关键字，Object.wait()/Object.notify等

## 1.锁

锁机制：将多个线程对共享资源的并发访问转换为串行访问，即一个共享资源一次只能被一个线程访问，该线程访问结束后其他线程才能继续访问。换句话说，锁就是线程访问共享资源的许可证。

锁具有排他性。

|          | 内部锁           | 显式锁               |
| -------- | ---------------- | -------------------- |
| 锁的分类 | synchronized     | ReentrantLock        |
| 公平锁   | 非公平锁         | 支持非公平锁和公平锁 |
| 使用方式 | 修饰方法、代码块 | 修饰同步代码块       |

![image-20200718215135764](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200718215135764.png)

### (1). 锁的作用

* 原子性

  锁的互斥性保障原子性

* 可见性

  可见性的保障通过**写线程冲刷处理器高速缓存**和**读线程刷新处理器缓存实现**

  - 锁的释放隐含高速冲刷处理器缓存，执行临界区代码之后，当前线程对共享数据的更新推送到处理器的高速缓存中，使得更新结果对其他线程可见；
  - 锁的获得隐含刷新处理器缓存，执行临界区代码之前，可以将其他线程对共享数据的更新同步到该处理器的高速缓存中

* 有序性

  由于锁的可见性，共享数据对对读线程都是可见的；由于锁的原子性，写线程对共享数据的更新在读线程看来都是有序的。

注意：锁对原子性，可见性，有序性的保障的先决条件：

* 多线程访问同一组共享数据时，使用同一个锁
* 线程仅读取数据时，也需要持有相应的锁

### (2). 锁的几个问题

#### 可重入性

线程在持有一个锁时，还可以多次申请该锁，即是可重入锁

![image-20200718221128229](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200718221128229.png)

#### 锁的争用与调度

锁也算共享资源的一种，高并发就有可能导致锁的争用。

#### 锁的粒度

锁实例保护的共享数据的大小即是锁的粒度。

- 粒度过粗导致线程在申请锁的时候产生等待
- 粒度过细增加锁调度的开销，包括上下文切换、锁的释放与申请产生的开销（这些都是处理器时间）

#### 锁的开销及可能导致的问题

- 开销：上下文切换的开销、锁的释放与申请产生的开销（这些都是处理器时间）

- 问题：
  - 锁泄露：锁一直没被释放，导致其他线程无法获得该锁的情况
  - 线程的活性故障：死锁，锁死，活锁，线程饥饿

## 3. synchronized关键字

修饰方法或者同步代码块

```java
synchronized(锁句柄) {
	// 同步代码块
}
```

锁句柄的变量通常使用 final 修饰符

```java
private final Object lock = new Object();
```

synchronized 的申请和释放由 JVM 代为处理；内部锁不会导致锁泄露。

内部锁的调度：非公平调度。

JVM 会为每个内部锁维护一个入口集记录等待获得内部锁的线程。

申请失败的线程生命周期转变成 BLOCKED.

锁释放，唤醒任意一个线程，使其获得申请锁的机会，与其他为进入入口集且处于 RUNNABLE 状态的线程竞争该锁，即被唤醒的线程不一定能持有当前锁。

## 4.显式锁：Lock 接口

使用模板：

```java
private final Lock lock = ...;// 创建锁实例对象

lock.lock(); // 申请锁
try {
    // 同步代码块
} catch (Exception) {
    
} finally {
    // 避免锁泄露
    lock.unlock();
}
```

### 显式锁的调度

公平锁：增加线程暂停和唤醒的可能性，即增加了上下文切换的代价，使用的代价大于非公平锁。默认使用非公平策略。

公平锁适用于锁被持有时间相对较长，或者线程平均等待时间较长的情形。

### 显式锁与内部锁的比较

显式锁更灵活，比如支持在一个方法内申请锁，在另一个方法内释放锁；

内部锁更安全，不会导致锁泄露。

## 5. 读写锁

![image-20200718230410830](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200718230410830.png)

读写锁的使用方法与显式锁类似，也要注意锁泄露

```java
private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
private final Lock readLock = rwLock.readLock();
private final Lock writeLock = rwLock.writeLock();

private void reader() {
    readLock.lock();
    try {
        // 读取共享变量
    } catch {
        
    } finally {
        // 防止锁泄露
        readLock.unlock();
    }
}

private void writer() {
    writeLock.lock();
    try {
        // 写共享变量
    } catch {
        
    } finally {
        writeLock.unlock();
    }
}
```

适用场景：

- 只读操作比写操作频繁
- 读线程持有锁的时间长

其他：读写锁支持锁的降级，写锁持有期间可以获得相应的读锁，demo:

```java
private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
private final Lock readLock = rwLock.readLock();
private final Lock writeLock = rwLock.writeLock();

public void rwLockDowngrade() {
    boolean readLockAquired = false;
    writeLock.lock();
    try {
        // 更新共享数据
        readLock.lock();
        readLockAquired = true;
    } catch {
        
    } finally {
        writeLock.unlock();
    }
    
    if (readLockAquired) {
        try {
            // 读取共享数据
        } catch {
        	
        } finally {
        	read.unlock();
        }
    } else {
        //
    }
}
```

## 6. 线程适用场景

- check - then - act : 线程读取共享数据，并判断下一步如何操作
- read - modify - write : 线程读取共享数据并在基础上更新操作。
- 多线程对多个数据进行更新

## 7. volatile 关键字

轻量级锁，保证可见性和有序性。仅能保证写的原子性，不会引起上下文切换。除此之外还能保证 long / double 变量的读写原子性。

开销介于临界区变量和普通变量之间，volatile 变量每次都要从高速缓存或者内存中读取。

**使用场景**

* 使用 volatile 变量作为状态标志
* 使用 volatile 保证可见性
* 代替锁
* 实现简易版读写锁 

## 8. 实现线程安全的单例模式

```java
public class ThreadSafeSingleton {
    // 存放唯一实例
    private static volatile ThreadSafeSingleton instance;
    
    // 构造器
    public ThreadSafeSingleton() {
        
    }
    
    public static ThreadSafeSingleton getInstance() {
        if (instance == null) { // 第一期检查
            synchronized(ThreadSafeSingleton.class) {
                /*
                	第二次检查
                	使用 volatile 防止重排序
                	JVM 可能将临界区内指令进行重排序，导致 instance 实例化前没进行第二次检查
                */
                if (instance == null) {
                    instance = new ThreadSafeSingleton();
                }
            }
        }
        return instance;
    }
}
```

## 9. CAS 和原子变量

### (1) CAS (CompareAndSwap)

synchronized 是一种悲观锁，这种线程一旦得到锁，其他需要锁的线程就挂起的情况就是悲观锁，悲观地认为程序中并发情况严重

CAS 是一种乐观锁，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止，乐观的认为程序中病房情况不严重，让线程不断尝试

> CompareAndSwap(V, newValue, oldValue)
>
> V - 变量的存储地址
>
> newValue - 修改的新值
>
> oldValue - 旧的预期值

缺点：

- 保障变量的原子性，不保障代码的原子性，也不保证变量的可见性，可以考虑将变量定义为 volatile

- CPU 消耗较大，并发量较高，线程反复尝试更新变量，且不成功，导致 CPU 开销增大

- 无法解决 A-B-A 问题，Thead1 看到变量 V 变成 A 时，Thread2 将 V 更新成 B ，Thread1 执行 CAS 操作时，Thread3 又将 V 更新成 A，此时无法判断变量 V 是否被其他线程更新过。可以通过添加操作次数戳的方法，来查看 V 是否被改变过，[V, 操作次数戳]

  [A, 0] -> [B, 1] -> [A, 2]

### (2) 原子变量类

## 10. final 和 static 关键字

- static ：保证线程在没有其他同步机制下，仍能读到变量的初始值（非默认值），此可见性保障仅限于线程初次读取该变量。对于引用型静态变量，static 能保障线程读到该变量的初始值时，变量对应的对象已经初始化完成。
- final：对象被发布到线程时，final 字段都已初始化完成，即其他线程读取的 final 字段的时候都是相应字段的初始值。对于引用型 final 字段，进一步确保字段引用的对象已经初始化完成，即线程读取引用对象的所有变量都已经初始化完成。final 只能保证对象对外可见时，对象的 final 字段已经初始化完成。

# 线程间协作

## 1. wait(), notify()/notifyAll()

- wait()：使当前线程暂停( WAITING )
- notify：随机唤醒一个等待线程
- notifyAll：唤醒全部等待线程 

内部实现：为对象维护一个等待集，wait() 方法使当前线程暂停且释放相应的内部锁，当前线程就被添加到等待线程集中；notify() 会随机唤醒一个等待线程集中的线程，直到该线程获取了该对象的内部锁，wait() 方法会将该线程从等待集中移除，线程才会开始执行。

## 2. Condition



## 3.倒计时协调器(CountDownLatch)——Java多线程编程实战指南 清单5-6

某服务器在启动时需要启动若干比较耗时的服务。为了尽可能减少耗时，通常使用专门的工作者线程启动这些服务。但是服务器在所有服务启动操作结束后，需要判断这些服务的状态以检查服务器的启动是否成功，否则服务器的启动失败，此服务器自动终止。

### 内部实现

* **CountDownLatch**用来实现一个或者多个线程等待其他线程完成一组特定操作之后才继续运行，这组操作被称为先决操作。**CountDownLatch**内部维护一个用于表示未完成的先决操作的计数器 **count**。

* **CountDownLatch.countDown()** 执行一次，则计数器 **count--** 。**countDown()** 方法相当于一个通知方法，在计数器 **count == 0** 时唤醒所有等待线程。   

* **CountDownLatch.await()** 是一个受保护方法，当计数器 **count != 0**，则执行 **await()** 方法线程都会等待，直到 **count == 0**，这些线程被称为 **CountDownLatch**上的等待线程。

### 构造函数

```java
public CountDownLatch(int count)
```

参数count用于表示先决操作的的数量和需要被执行的次数。

### 一些使用说明

* 一个 **CountDownLatch** 实例只能实现一次等待和唤醒，即该实例的使用是一次性的。
* 工作线程执行 **countDown() ** 方法起到的作用仅仅是向 **CountDownLatch** 报告它完成了某个操作，其本身无法区分某个操作是否成功完成，一般来说使用下列代码段完成整个操作：
* 

```java
try {
    doSomeWork(); //一些具体操作
} catch (Exception e) {
    dealException(); //处理异常
} finally {
    latch.countDown(); //latch为CountDownLatch的实例
    doSomething(); //其他操作
}
```

* 如果 **CountDownLatch()** 内部计数器应为程序错误永远无法到达0，相应实例上的等待线程会一直处于 **WAITING** 状态，避免该问题有以下两种方法：

  1.保证所有的 **countDown()** 操作在正确的位置，例如上段代码所示，将 **countDown()** 放在 **finally** 代码块中，在具体操作出现异常时，程序仍然能继续运行；

  2.使用 **await()** 的另一种形式

  ```java
  /**
   * @param timeout the maximum time to wait
   * @param unit the time unit of the {@code timeout} argument
   * @return {@code true} if the count reached zero and {@code false}
   *         if the waiting time elapsed before the count reached zero
   * @throws InterruptedException if the current thread is interrupted
   *         while waiting
   */
  public boolean await(long timeout, TimeUnit unit) throws InterruptedException;
  ```

  该方法指定一个超时时间，在该时间内若相应实例计数器未达到0，所有执行await()方法的线程均被唤醒，该方法返回值可以用于确定是否超时。

## 栅栏(CyclicBarrier) ——清单5-8

程序清单5-8模拟士兵参与打靶射击，所有士兵被分为一个连队(company)，一个连队中有若干排。每次只有一排士兵进行射击，每排的士兵都需要同时开始射击，且他们必须等待同排的所有士兵射击完成后才能一起撤离。交替训练，直到射击时间结束。

使用 **CyclicBarrier** 实现等待的线程被称为**参与方**，通过执行方法 **await()** 实现等待。

### 基本实现逻辑

**CyclicBarrier** 内部维护一个显示锁，使其可以分辨出最后一个执行 **await()** 方法的线程，该线程被称为最后一个线程。除了最后一个线程外，任何参与方执行 **await()** 方法都会导致线程生命周期变为 **WAITING** 。最后一个线程执行 **await()** 方法会使相应实例的参与方被唤醒，且当前线程不会暂停。

与 **CountDownLatch** 不同的是，**CyclicBarrier** 的实例是可以重复使用的。

### 一些TIPS

* **CyclicBarrier** 内部是基于条件变量的，其开销与条件变量的开销类似，主要是上下文切换；
* 设  **cb** 是 **CyclicBarrier** 的实例，任意一个参与方在执行 **cb.await()** 前所有的操作都是对 **barrierAction.run()** 可见的、有序的。同样的 **barrierAction.run()** 中执行的任何操作对所有参与方在 **cb.await()** 调用成功返回之后的代码是可见的，有序的。

### 典型应用场景

* 迭代算法(Interative)算法并发化。在并发化的迭代算法中，迭代操作是由多个工作线程并发执行的。**CyclicBarrier** 可用来实现执行迭代操作的任何一个工作线程必须等待其他工作线程也完成当前迭代操作的情况下，才继续下一轮迭代操作，以使上一轮结果作为下一轮的输入。
* 在测试代码中模拟高并发。

关于CyclicBarrier的滥用

# 多线程编程的硬件基础和 Java 内存模型

## 内存屏障( Memory Barrier, Fence )

Java 虚拟机底层借助内存屏障实现刷新处理器缓存和冲刷处理器缓存。

内存屏障是对一类针对内存读、写操作指令的比较底层的称呼（抽象）。内存屏障是被插入到两个指令之间进行使用的，其作用是禁止编译器、处理器重排序保障有序性，但是其副作用就是——刷新处理器缓存、冲刷处理器缓存，这两个操作比较费时。

| 可见性保障 | 加载屏障( Load Barrier ) ：刷新处理器缓存                    | 存储屏障( Store Barrier ) ：冲刷处理器缓存                   |
| :--------: | :----------------------------------------------------------- | ------------------------------------------------------------ |
| 可见性保障 | 获取屏障( Aquire Barrier ) : 禁止读操作与其后的任何写操作之间进行重排序 ，相当于进行后续操作要获取相应共享数据的所有权，即保证屏障之前的**读操作**先于屏障之后**读写操作** | 释放屏障( Release Barrier ) : 禁止写操作与之前的任何读操作之间进行重排序，相当于对在对相应共享数据操作结束后释放所有权，保证屏障之后的**写操作**晚于屏障之前的**读写操作** |

上述两种屏障与临界区的关系如图，Monitor Enter 包括读操作，Moniter Exit 包含写操作。

![image-20200718162450058](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200718162450058.png)

锁对有序性的保障是通过读写线程配对使用获取屏障和释放屏障实现的。临界区的任何读写操作都无法被重排序到临界区之外。

## Java 同步机制和内存屏障

Java 虚拟机对 **synchronized, volatile, final** 等关键字的语义实现就是借助内存屏障。

### 1. volatile 的实现

- volatile 的写操作
  - JVM 在 volatile 变量写操作之前插入**释放屏障**，使得**该屏障之前**的任何读写操作都先于这个 volatile 变量写操作之前被提交；
  - JVM 在 volatile 变量写操作之后插入 StoreLoad 屏障，除了防止重排序之外，还有以下两个作用：
    - 存储屏障。该屏障之前的所有写操作的结果到达高速缓存，使这些结果对其他处理器可见。
    - 加载屏障。使其他处理器对 volatile 变量所做的更新能够被同步到执行 volatile 变量读线程的处理器上。

![image-20200718172454567](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200718172454567.png)

- volatile 的读操作
  - 在 volatile 变量读操作之后插入**获取屏障**，使该 volatile 变量的读操作先于该屏障的任何读写操作被提交；
  - 在读操作之前插入加载屏障( LoadLoad 屏障 )，使得其后的读操作有机会读到其他处理器对 volatile 变量的更新。

![image-20200718172715662](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200718172715662.png)

写线程和读线程通过各自执行的释放屏障和获取屏障保障有序性，通过各自的存储和加载屏障保障变量的可见性。

![image-20200718173231066](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200718173231066.png)

### 2.synchronized 的实现

* 原子性保障

  JVM 会在 monitorenter (申请锁的指令码) 对应的指令之后和临界区开始之前插入获取屏障，在 临界区结束之后和 monitorexit(释放锁的指令码) 之前插入释放屏障。保障临界区之内的任何读写操作都无法被重排序到临界区之外，加上锁的排他性，保证临界区内的操作具有原子性。

* 有序性保障

  释放屏障使临界区内的任何读写操作都先于 monitorexit 指令码之前提交；获取线程使任何读线程必须在 monitorenter 之后才能执行临界区中的操作。

* 其他

  JVM 会在 monitorexit 操作之后插入一个 StoreLoad 屏障，其作用和 volatile 写操作之后的 StoreLoad 屏障类似。
  * 存储屏障，保证锁的持有线程在释放锁之前执行的结果能够到达高速缓存；加载屏障，消除存储分发的副作用，保证其他处理器对共享资源做的操作对当能同步到其他对共享资源进行读操作的处理器上。
  * 禁止 monitorexit 指令码与其他同步块的 monitorenter 指令码的重排序，使 synchronized 代码块的并列和嵌套可以实现。 

![image-20200718180924516](C:\Users\YarTang\AppData\Roaming\Typora\typora-user-images\image-20200718180924516.png)

### 3. final 的实现

JVM 保证 final 字段的初始化不会被重排序到其构造器之外（或者说，构造器结束之后），使实例化对象对外可见时，该对象的 final 字段以及 final 字段引用的对象已经初始化完毕。