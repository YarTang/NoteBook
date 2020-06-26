# 倒计时协调器(CountDownLatch)——Java多线程编程实战指南 清单5-6
某服务器在启动时需要启动若干比较耗时的服务。为了尽可能减少耗时，通常使用专门的工作者线程启动这些服务。但是服务器在所有服务启动操作结束后，需要判断这些服务的状态以检查服务器的启动是否成功，否则服务器的启动失败，此服务器自动终止。

## CountDownLatch

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