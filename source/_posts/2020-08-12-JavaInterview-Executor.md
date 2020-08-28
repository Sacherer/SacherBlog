---
title: 【JAVA】面试总结——线程池篇
date: 2020-08-12 17:22:15
tags:
  - Java
  - 面试
top: true
cover: true
img: /images/timg.jpg
categories: Java面试
---
#### 本文目的
总结面试线程池常见问题

#### 线程池（ThreadPoolExecutor）

###### 参数解析

- **corePoolSize**：核心线程池的大小。默认情况下，创建线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中。如果调用prestartAllCoreThreads()、prestartCoreThread()方法是预先创建线程。

- **maximumPoolSize**：线程池最大线程数，线程池中最多能创建多少个线程。

- **keepAliveTime**：线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程小于corePoolSize。但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；

- **unit**：参数keepAliveTime的时间单位，有7种取值。

  ```java
  TimeUnit.DAYS;         //天
  TimeUnit.HOURS;        //小时
  TimeUnit.MINUTES;      //分钟
  TimeUnit.SECONDS;      //秒
  TimeUnit.MILLISECONDS; //毫秒
  TimeUnit.MICROSECONDS; //微妙
  TimeUnit.NANOSECONDS;  //纳秒
  ```

- **workQueue**：阻塞队列，用来存储等待执行的任务。常见阻塞队列有：有界队列、无界队列、同步移交

  ```java
  ArrayBlockingQueue    // 数组实现的阻塞队列，数组不支持自动扩容。所以当阻塞队列已满
                        // 线程池会根据handler参数中指定的拒绝任务的策略决定如何处理后面加入的任务
  
  LinkedBlockingQueue   // 链表实现的阻塞队列，默认容量Integer.MAX_VALUE(不限容)，
                        // 当然也可以通过构造方法限制容量
  
  SynchronousQueue      // 零容量的同步阻塞队列，添加任务直到有线程接受该任务才返回
                        // 用于实现生产者与消费者的同步，所以被叫做同步队列
  
  PriorityBlockingQueue // 二叉堆实现的优先级阻塞队列
  
  DelayQueue          // 延时阻塞队列，该队列中的元素需要实现Delayed接口
                      // 底层使用PriorityQueue的二叉堆对Delayed元素排序
                      // ScheduledThreadPoolExecutor底层就用了DelayQueue的变体"DelayWorkQueue"
                      // 队列中所有的任务都会封装成ScheduledFutureTask对象(该类已实现Delayed接口)
  ```

- **threadFactory**：线程工厂，主要用来创建线程。

- **handler**：表示当拒绝处理任务时的策略，有以下四种取值：

  ```java
  ThreadPoolExecutor.AbortPolicy   //丢弃任务并抛出RejectedExecutionException异常。 
  ThreadPoolExecutor.DiscardPolicy //也是丢弃任务，但是不抛出异常。 
  ThreadPoolExecutor.DiscardOldestPolicy //丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
  ThreadPoolExecutor.CallerRunsPolicy //由调用线程处理该任务 
  ```

  > 可通过实现RejectedExecutionHandler接口来自定义任务拒绝后的处理策略

###### 常见线程池

1. **newFixedThreadPool** 固定线程数量的线程池

   ```java
   public static ExecutorService newFixedThreadPool(int nThreads) {
           return new ThreadPoolExecutor(nThreads, nThreads,
                                         0L, TimeUnit.MILLISECONDS,
                                         new LinkedBlockingQueue<Runnable>());
   }
   ```

2. **newSingleThreadExecutor** 单个线程的线程池(本质上就是容量为1的FixedThreadPool)

   ```java
   public static ExecutorService newSingleThreadExecutor() {
           return new FinalizableDelegatedExecutorService
               (new ThreadPoolExecutor(1, 1,
                                       0L, TimeUnit.MILLISECONDS,
                                       new LinkedBlockingQueue<Runnable>()));
   }
   ```

3. **newCachedThreadPool** 无数量限制可自动增减线程的线程池

   ```java
   public static ExecutorService newCachedThreadPool() {
           return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                         60L, TimeUnit.SECONDS,
                                         new SynchronousQueue<Runnable>());
   }
   ```

4. **newScheduledThreadPool** 任务延时执行线程池

   ```java
   public ScheduledThreadPoolExecutor(int corePoolSize) {
           super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
                 new DelayedWorkQueue());
   }
   ```

###### 线程池关闭

- **shutdown()**：不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务。
- **shutdownNow()**：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务。

###### 线程池数量

- IO密集型应用，则线程池大小设置为**2N+1；**
- CPU密集型应用，则线程池大小设置为**N+1；**