---
title: Java线程池
toc: true
date: 2024-01-16 10:52:06
tags:
categories: Java
---

## 一、为什么要使用线程池？

new Thread(Runnable task).start();的方式创建线程执行任务存在弊端：线程不能复用；而重复创建和销毁线程耗时耗资源；

**线程池的优势：**

* 降低资源消耗：通过重复利用已创建的线程降低线程创建和销毁造成的销毁 (线程池里没有销毁的线程处于什么状态，不会占用资源吗？为什么不会占用？)
* 提高响应速度：当有任务时，任务可以不需要等到线程创建就能立即执行（暂不理解）
* 提高线程的可管理性：线程池可以进行统一的分配，调优和监控（如何管理？）

## 二、什么是线程池？

线程池(ThreadPool)是一种基于池化思想管理线程的工具。当没有线程池时，我们会创建一个线程，将任务传递给线程，并且一个线程只能执行一个任务，如果还有任务，就只能再创建一个线程去执行它，当任务执行完时，线程就销毁了，重复创建和销毁线程是一件很耗时耗资源的事，如果能重复利用，就可以减少不必要的消耗，于是线程池就应运而生了。

事先将线程创建好，当有任务需要执行时，提交给线程池，线程池分配线程去执行，有再多的任务也不怕，线程池中的线程能复用，执行完一个任务，再接着执行其他任务，当所有任务都执行完时，我们可以选择关闭线程池，也可以选择等待接收任务。 

## 三、怎么用线程池？

在JAVA中主要是使用ThreadPoolExecutor类来创建线程池，并且JDK中也提供了Executors工厂类来创建线程池（不推荐使用）。创建线程的方式一共有8种，都是基于原生创建线程池的方式。

### 3.1 为什么不推荐用Executors工厂类来创建线程池，推荐原生方式？

《阿里巴巴Java开发手册》的第7章第4小节中写到：

线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

说明：Executors 返回的线程池对象的弊端如下：

1. **FixedThreadPool 和 SingleThreadPool:**

允许的请求队列长度为Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM。

2. **CachedThreadPool:**

允许的创建线程数量为Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM。

### 3.2 如何使用原生方式创建线程池？

Java中线程池的核心实现类是ThreadPoolExecutor，可以通过该类地构造方法来构造一个线程池

#### 3.2.1 ThreadPoolExecutor的构造组成

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) 
```

**corePoolSize：**核心线程数

只要线程池不关闭，核心线程就不会被销毁

**maximumPoolSize：**最大线程数

表示线程池中最多允许存在的线程数量，在线程池中，除去核心线程之外的线程是非核心线程，非核心线程如果没有执行任务的话会被清理，在被清理之前能够存活多久取决于后面两个参数。

**keepAliveTime：**空闲线程存活时间

**unit：**时间单位

**workQueue：**任务队列，存放已提交任务的队列

* ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列（数组结构可配合指针实现一个环形队列）。

* LinkedBlockingQueue： 一个由链表结构组成的有界阻塞队列，在未指明容量时，容量默认为 Integer.MAX_VALUE。

* PriorityBlockingQueue： 一个支持优先级排序的无界阻塞队列，对元素没有要求，可以实现 Comparable 接口也可以提供 Comparator 来对队列中的元素进行比较。跟时间没有任何关系，仅仅是按照优先级取任务。

* DelayQueue：类似于PriorityBlockingQueue，是二叉堆实现的无界优先级阻塞队列。要求元素都实现 Delayed 接口，通过执行时延从队列中提取任务，时间没到任务取不出来。

* SynchronousQueue： 一个不存储元素的阻塞队列，消费者线程调用 take() 方法的时候就会发生阻塞，直到有一个生产者线程生产了一个元素，消费者线程就可以拿到这个元素并返回；生产者线程调用 put() 方法的时候也会发生阻塞，直到有一个消费者线程消费了一个元素，生产者才会返回。

* LinkedBlockingDeque： 使用双向队列实现的有界双端阻塞队列。双端意味着可以像普通队列一样 FIFO（先进先出），也可以像栈一样 FILO（先进后出）。

* LinkedTransferQueue： 它是ConcurrentLinkedQueue、LinkedBlockingQueue 和 SynchronousQueue 的结合体，但是把它用在 ThreadPoolExecutor 中，和 LinkedBlockingQueue 行为一致，但是是无界的阻塞队列。

注意有界队列和无界队列的区别：如果使用有界队列，当队列饱和时并超过最大线程数时就会执行拒绝策略；而如果使用无界队列，因为任务队列永远都可以添加任务，所以设置 maximumPoolSize 没有任何意义。

**threadFactory：**线程工厂

线程工厂指定创建线程的方式，需要实现 ThreadFactory 接口，并实现 newThread(Runnable r) 方法。

```java
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.atomic.AtomicInteger;

public class MyThreadFactory implements ThreadFactory {

    private final String name;
    private final AtomicInteger i = new AtomicInteger(1);

    public MyThreadFactory(String name) {
        this.name = name;
    }

    @Override
    public Thread newThread(Runnable task) {
        Thread thread = new Thread(task);
        thread.setName(name + i.getAndIncrement());
        return thread;
    }
}
```



**handler：**任务拒绝策略

当线程池的线程数达到最大线程数时，需要执行拒绝策略。拒绝策略需要实现 RejectedExecutionHandler 接口，并实现 rejectedExecution(Runnable r, ThreadPoolExecutor executor) 方法。不过 Executors 框架已经为我们实现了 4 种拒绝策略：

* AbortPolicy（默认）：丢弃任务并抛出 RejectedExecutionException 异常。

* CallerRunsPolicy：由调用线程处理该任务。

* DiscardPolicy：丢弃任务，但是不抛出异常。可以配合这种模式进行自定义的处理方式。

* DiscardOldestPolicy：丢弃处于任务队列头部的任务，添加被拒绝的任务

#### 3.2.2 实践，创建一个线程池

```java
public class Task implements Runnable{

    private final String taskName;

    public Task(String taskName) {
        this.taskName = taskName;
    }

    @Override
    public void run() {
        System.out.println("线程名："+ Thread.currentThread().getName()+ ";" + taskName + " 已完成");
    }
}
```

```java
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.atomic.AtomicInteger;

public class MyThreadFactory implements ThreadFactory {

    private final String name;
    private final AtomicInteger i = new AtomicInteger(1);

    public MyThreadFactory(String name) {
        this.name = name;
    }

    @Override
    public Thread newThread(Runnable task) {
        Thread thread = new Thread(task);
        thread.setName(name + i.getAndIncrement());
        return thread;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Task task1 = new Task("任务一");
        Task task2 = new Task("任务二");
        Task task3 = new Task("任务三");

        ThreadPoolExecutor threadPool = new ThreadPoolExecutor(10, 25, 10L,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(),
                new MyThreadFactory("自定义线程"),
                new ThreadPoolExecutor.AbortPolicy());

        threadPool.execute(task1);
        threadPool.execute(task2);
        threadPool.execute(task3);

        threadPool.shutdown();
    }
}
```

**输出结果：**  

线程名：自定义线程3;任务三 已完成
线程名：自定义线程1;任务一 已完成
线程名：自定义线程2;任务二 已完成

### 3.3提交任务的方式

1. execute：

用于向线程池提交Runnable任务，无返回值

1. submit：

用于向线程池提交Callable和Runnable任务，有返回值。

submit有以下三种方法：

| 方法名              | 返回值类型 | 描述                           |
| ------------------- | ---------- | ------------------------------ |
| submit(Runnable)    | Future<?>  | 提交Runnable任务               |
| submit(Runnable)    | Future<T>  | 提交Runnable任务并指定执行结果 |
| submit(Callable<T>) | Future<T>  | 提交Callable任务               |

### 3.4 shutdown与shutdownNow

* shutdown

  1. 不再接收新的任务

   shutdown方法一旦调用，线程池就会被关闭，假如池中还有任务正在执行，不会中断，假如此时提交新的任务，线程池不会接受并会根据设置的拒绝策略拒绝它。

  **程序演示**

  ```java
  public class Main {
      public static void main(String[] args) {
  
  
          ThreadPoolExecutor poolExecutor = new ThreadPoolExecutor(1, 1, 0L, TimeUnit.SECONDS,
                  new LinkedBlockingQueue<>(1),
                  new ThreadPoolExecutor.AbortPolicy());
  
          poolExecutor.execute(new Task("1"));
  
          poolExecutor.shutdown();
  
          poolExecutor.execute(new Task("2"));
  
      }
  }
  
  ```

  **运行结果**

  Exception in thread "main" java.util.concurrent.RejectedExecutionException: Task Task@2812cbfa rejected from java.util.concurrent.ThreadPoolExecutor@2acf57e3[Shutting down, pool size = 1, active threads = 1, queued tasks = 0, completed tasks = 0]
  	at java.base/java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2055)
  	at java.base/java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:825)
  	at java.base/java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1355)
  	at Main.main(Main.java:34)
  线程名：pool-1-thread-1;1 已完成

  

  2. 继续执行完任务队列中的任务

  调用shutdown后，假如线程池队列中还有任务没有执行，线程池会继续执行完它，直到线程池队列中所有的任务都执行完，线程池才会彻底得关闭。

  **程序演示**

  ```java
  public class Main {
      public static void main(String[] args) {
  
  
          ThreadPoolExecutor poolExecutor = new ThreadPoolExecutor(1, 1, 0L, TimeUnit.SECONDS,
                  new LinkedBlockingQueue<>(3),
                  new ThreadPoolExecutor.AbortPolicy());
  
          poolExecutor.execute(new Task("1"));
          poolExecutor.execute(new Task("3"));
          poolExecutor.execute(new Task("4"));
          poolExecutor.shutdown();
  
          poolExecutor.execute(new Task("2"));
  
      }
  }
  ```

  **运行结果**

  Exception in thread "main" java.util.concurrent.RejectedExecutionException: Task Task@2812cbfa rejected from java.util.concurrent.ThreadPoolExecutor@2acf57e3[Shutting down, pool size = 1, active threads = 0, queued tasks = 2, completed tasks = 0]
  	at java.base/java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2055)
  	at java.base/java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:825)
  	at java.base/java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1355)
  	at Main.main(Main.java:35)
  线程名：pool-1-thread-1;1 已完成
  线程名：pool-1-thread-1;3 已完成
  线程名：pool-1-thread-1;4 已完成

  Process finished with exit code 1

  

* shutdownNow





## 参考资料
> - 【看动画，学Java线程池教程】 https://www.bilibili.com/video/BV1wh411e7nd/?share_source=copy_web&vd_source=657251adc83ac8ca83b61a76eec4be46
