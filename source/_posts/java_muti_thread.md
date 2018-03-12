---
title: Java线程
date: 2017-07-14 20:04:20
author: wúzguó
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
categories: 后台
tags: 
	- Thread
	- 多线程
	- Runable

keywords: Thread，多线程，Runable
photos:
	- /blog/images/201707/1.png
description: Java线程类（Thread）的中主要方法的使用和注意事项
---


#### Java线程

##### 一、进程和线程的基本概念

1. 进程

   进程是操作系统结构的基础，是一个程序及其数据在处理机上顺序执行时所发生的活动，是程序在一个数据集合上运行的过程，**是系统进行资源分配和调度的一个独立单位**。

   ![](/blog/images/201707/1.png)


2. 线程

   线程是进程中独立运行的子任务，一个进程至少包含一个线程，线程是CPU资源分配的基本单位。

   - 多线程的优点：

     充分利用CPU空闲的计算能力，提高程序运行的效率。

   ![](/blog/images/201707/2.png)

   如上图所示，CPU完全可以在任务1和任务2之间来回切换，使任务2不必等到10秒后再运行，这样程序的运行效率得到大大提升。

   ​

##### 二、线程的实现

1. 线程的实现方式
   - 继承Thread抽象类
     - 缺点：不支持多继承，Java语言只支持单继承。
   - 实现runable接口
   - 实现callable接口（`TestCallable.java`）
     - 与runable的区别：可以有返回值，并且可以抛出异常。

```java
// callable的基本使用
public class TestCallable {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        for (int i = 0; i < 10; i++) {
            FutureTask<Result> task = new FutureTask<Result>(new Call());
            new Thread(task).start();
            // 获取运行结果
            Result result = task.get();
            System.out.println("result: " + result);
        }
    }
}

class Call implements Callable<Result> {
    public Result call() throws Exception {
        System.out.println("hello world...");
        return new Result(new Random(10).nextInt(10), true, "hello world");
    }
}
```
执行结果：

```java
hello world...
result: Result{id=3, success=true, message='hello world', data=null}
hello world...
result: Result{id=3, success=true, message='hello world', data=null}
```


2. 实例变量和线程安全（`TestShareData.java`）

   自定义线程类中的实例变量针对其他线程可以有共享与不共享之分，这在多个线程之间交互时是一个需要特别注意的地方。

   - 不共享数据的情况

   - 共享数据的情况

     **共享数据的情况就是多个线程可以访问同一个变量**，比如在实现投票功能的程序中，多个线程可以同时处理同一个人的票数。

3. 线程状态切换

   Java语言定义了5种线程状态，在任意一个时间点，一个线程只能处于一种状态，这5种状态分别如下：

   - 新建（New）：创建后尚未启动的线程处于这种状态。

   - 运行（Runable）：Runable包括了操作系统线程状态的running和ready，也就是处于此状态的线程有可能正在执行，也有可能正在等着CPU分配时间片。

   - 无限期等待（Waiting）：处于这种状态的线程不会被CPU分配时间，它们需要等待被其他线程显式的唤醒，以下方法会让线程进入无限期的等待状态：

     - 没有设置timeout参数的`Object.wait()`方法。
     - 没有设置timeout参数的`Thread.join()`方法。
     - LockSupport.park()方法。

   - 有限期等待（Timed Waiting）：处于这种状态的线程也不会被分配CPU执行时间，不过无需等待被其他线程显示的唤醒，在一定时间之后它们会被系统自动唤醒。以下方法会让线程进入限期等待状态：

     - Thread.sleep()方法。

     - 设置了timeout参数的`Object.wait()`方法。
     - 设置了timeout参数的`Thread.join()`方法。
     - LockSupport.parkNamos()方法。
     - LockSupport.parkUntil()方法。

   - 阻塞（Blocked）：线程被阻塞了，线程阻塞状态和等待状态的区别在于：

     - 阻塞状态在等待获取到一个排他锁，这个事件在另外一个线程放弃这个锁的时候发生。
     - 等待状态则在等待一段时间，或者唤醒动作的时候发生。在程序等待进入同步区域的时候，线程进入这种状态。

   - 结束（Terminated）：已终止线程的线程状态，线程已结束执行。

   ![](/blog/images/201707/3.png)

##### 三、线程的使用

1. currentThread()方法

   currentThread()方法可返回代码段正在被调用的线程信息。（`TestCurrentThread.java`）

2. isAlive()方法

   判断当前线程是否处于活动状态（指已启动并且尚未终止）。（`TestIsAlive.java`）

   另外在使用`isAlive()`方法时，如果将线程对象以构造方法的方式出递给Thread对象进行`start()`启动时，运行的结果和前面示例是有差异的。造成这样的差异的原因来自于`Thread.currentThread()`和`this`的差异。

3. sleep()方法

   让当前“当前正在执行”的线程休眠（暂停执行）指定的毫秒数，这个”正在执行“的线程指`this.currentThread()`返回的线程。（`TestSleep.java`）

   ​

4. yield()方法

   yield方法表示是当前线程放弃CPU资源，给其他线程和当前线程同台竞争的机会，至于谁将获取CPU资源还得看缘分。（`TestYield.java`）

   ​

5. join()方法

   join()方法的作用是让调用`join()`方法的线程执行完成之后再执行当前线程。（`TestJoin.java`  `TestJoinWithParam.java`）

   在join过程中，如果当前线程对象被打断，则当前线程出现异常。（`TestJoinException.java`）

   `join()`方法和`sleep()`方法的区别

   - `join()`方法内部使用`wait(long)`方法实现，所以`join()`方法具有释放锁的特点。（`TestJoinSleep.java`）
   - `sleep()`不释放锁。

   ```java
   public final synchronized void join(long millis) throws InterruptedException {
       long base = System.currentTimeMillis();
       long now = 0;

       if (millis < 0) {
           throw new IllegalArgumentException("timeout value is negative");
       }

       if (millis == 0) {
           while (isAlive()) {
               wait(0);
           }
       } else {
           while (isAlive()) {
               long delay = millis - now;
               if (delay <= 0) {
                   break;
               }
               wait(delay);
               now = System.currentTimeMillis() - base;
           }
       }
   }
   ```

6. wait()方法

   `wait()`方法是使当前执行代码的线程进行等待。在调用`wait()`方法之前，线程必须获得该对象的对象级别锁（`TestWaitNotSync.java`），即只能在同步方法或同步块中调用`wait()`方法，如果调用时没有持有适当的锁，会抛出`IllegalMonitorStateException`异常。调用`wait()`方法后，线程持有的锁会主动释放。（`TestWaitReleaseLock.java`）

   ```java
   public class TestWaitNotSync {
       public static void main(String[] args) throws InterruptedException {
           String  str = new String();
           str.wait();
       }
   }
   ```
   执行结果如下：
   ```java
   Exception in thread "main" java.lang.IllegalMonitorStateException
   at java.lang.Object.wait(Native Method)
   at java.lang.Object.wait(Object.java:503)
   at com.github.wuzguo.thread.TestWaitNotSync.main(TestWaitNotSync.java:14)
   ```

7. notify()/notifyAll()方法

   唤醒一个或所有呈wait状态的线程。`notify()/notifyAll()`方法也要在同步方法或同步块中调用，如果调用时没有持有适当的锁，也会抛出`IllegalMonitorStateException`异常。（`TestWaitNotify.java`）`notify()/notifyAll()`方法不释放当前持有的锁。（`TestNotifyHoldLock.java`）

8. 线程优先级

   在操作系统中线程可以划分优先级，优先级较高的线程得到的CPU执行时间片就越多（不是优先级越高执行执行时间就越长）。也就是CPU优先执行优先级较高的线程对象中的任务。

   设置优先级有助于帮“线程调度器（）”确定下一次选择哪一个线程来优先执行。

   ```java
    /**
     * The minimum priority that a thread can have.
     */
    public final static int MIN_PRIORITY = 1;

   /**
     * The default priority that is assigned to a thread.
     */
    public final static int NORM_PRIORITY = 5;

    /**
     * The maximum priority that a thread can have.
     */
    public final static int MAX_PRIORITY = 10;
   ```

   优先级具有继承性（A线程启动B线程，则B线程的优先级与A的优先级一样）。（`TestPriority.java`）

   需要注意的是Java线程优先级并不是和操作系统的线程优先级一一对应，原因在于Java的线程通过映射到系统的原生线程上实现，所以线程的调度最后还取决于操作系统，Solaris操作系统中有2147483678（2$32$）种优先级，但是windows操作系统中只有七种，如下：

   | Java线程优先级        | Windows线程优先级                 |
   | ---------------- | ---------------------------- |
   | 1（MIN_PRIORITY）  | THREAD_PRIORITY_LOWEST       |
   | 2                | THREAD_PRIORITY_LOWEST       |
   | 3                | THREAD_PRIORITY_BELOW_NORMAL |
   | 4                | THREAD_PRIORITY_BELOW_NORMAL |
   | 5（NORM_PRIORITY） | THREAD_PRIORITY_NORMAL       |
   | 6                | THREAD_PRIORITY_ABOVE_NORMAL |
   | 7                | THREAD_PRIORITY_ABOVE_NORMAL |
   | 8                | THREAD_PRIORITY_HIGHEST      |
   | 9                | THREAD_PRIORITY_HIGHEST      |
   | 10（MAX_PRIORITY） | THREAD_PRIORITY_CRITICAL     |




8. 暂停线程

   暂停线程意味着此线程还可以恢复运行，在Java多线程中，可以使用`suspend()`  （`deprecated`）方法暂停线程，使用`resume()` （`deprecated`）方法恢复线程的执行。（TestSuspendResume.java）

   使用`suspend()`方法和`resume()`方法的缺点：

   - 使用不当，容易造成公共同步对象的独占，其他线程线程无法访问公共同步的对象。（`TestSuspendResumeDealLock.java`）

   - 容易出现线程的暂停而导致数据不同步的问题。（`TestSuspendResumeNoSameValue.java`）

     ​

9. 停止线程

   停止一个线程意味着在线程处理完任务之前停掉正在做的操作，停止一个线程可以使用`Thread.stop()`方法，他不是安全的（`unsafe`）而且已经被作废（`deprecated`）。大多数停止一个线程的操作都是使用`Thread.interrupt()`方法，**但是这个方法并不会中止正在运行的线程，还需要加入一个判断条件才能停止线程。** （`TestInterrupt.java`）

   在Java中可以用以下三种方法中止正在运行的线程：

   - 使用退出标志，是线程正常退出，也就是当`run()`方法完成后线程终止。
   - 使用`stop()`方法强行中止线程，但是不推荐使用，使用它们可能产生不可预料的结果。
   - 使用`interrupt()`方法。

   判断线程不是不是停止状态：

   - this.interrupted()：测试当前（运行`this.interrupted()`方法的线程）线程是否已经中断。

   - this.isInterrupted()：测试线程是否已经中断。 （`TestInterruptStatus.java`）

     ```java
     /**
      * Tests whether the current thread has been interrupted.  The
      * <i>interrupted status</i> of the thread is cleared by this method.  In
      * other words, if this method were to be called twice in succession, the
      * second call would return false (unless the current thread were
      * interrupted again, after the first call had cleared its interrupted
      * status and before the second call had examined it).
      *
      * <p>A thread interruption ignored because a thread was not alive
      * at the time of the interrupt will be reflected by this method
      * returning false.
      *
      * @return  <code>true</code> if the current thread has been interrupted;
      *          <code>false</code> otherwise.
      * @see #isInterrupted()
      * @revised 6.0
      */
     public static boolean interrupted() {
         return currentThread().isInterrupted(true);
     }

     /**
      * Tests whether this thread has been interrupted.  The <i>interrupted
      * status</i> of the thread is unaffected by this method.
      *
      * <p>A thread interruption ignored because a thread was not alive
      * at the time of the interrupt will be reflected by this method
      * returning false.
      *
      * @return  <code>true</code> if this thread has been interrupted;
      *          <code>false</code> otherwise.
      * @see     #interrupted()
      * @revised 6.0
      */
     public boolean isInterrupted() {
         return isInterrupted(false);
     }
     ```

#####  四、源代码地址

博客所使用的源码地址：[thread-examples](https://github.com/wzguo/thread-examples)