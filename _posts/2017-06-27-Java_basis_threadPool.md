---
layout: post
title: 线程池源码分析
date: 2017-06-27 
tag: Java基础
---

### 1.四种线程池 ###

#### 1).newCachedThreadPool
	ExecutorService cachedThreadPool = Executors.newCachedThreadPool();

创建一个可缓存的线程池,如果线程池长度超过处理需要,可灵活回收空闲线程,若无可回收,则新建线程.
如果线程池空闲时,即线程池中没有可运行任务时,有效线程空闲了指定的时间,则该有效线程将自动终止.
终止后,你若又提交了新任务,则线程池重新创建一个有效线程.

	public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

#### 2).newFixedThreadPool ####
	ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3); 

创建一个定长线程池,可控制线程最大并发数,超出的线程会在队列中等待.

    public static ExecutorService newFixedThreadPool(int nThreads) {
               return new ThreadPoolExecutor(nThreads, nThreads,
                                         0L, TimeUnit.MILLISECONDS,
                                         new LinkedBlockingQueue<Runnable>());
    }

#### 3).newScheduledThreadPool ####

	ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5); 

创建一个定长线程池,支持定时及周期性任务执行.

    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
            return new ScheduledThreadPoolExecutor(corePoolSize);
        }

    public ScheduledThreadPoolExecutor(int corePoolSize) {
            super(corePoolSize, Integer.MAX_VALUE, 0, TimeUnit.NANOSECONDS,
                  new DelayedWorkQueue());
        }

#### 4).newSingleThreadExecutor ####

	ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();

创建一个单线程化的线程池,它只会用唯一的工作线程来执行任务,保证所有任务按指定顺序执行.

    public static ExecutorService newSingleThreadExecutor() {
            return new FinalizableDelegatedExecutorService
                (new ThreadPoolExecutor(1, 1,
                                        0L, TimeUnit.MILLISECONDS,
                                        new LinkedBlockingQueue<Runnable>()));
        }

通过分析发现上面四种线程池都调用了new ThreadPoolExecutor();方法,下面对ThreadPoolExecutor方法进行源码分析

### 2.ThreadPoolExecutor源码分析 ###

    public ThreadPoolExecutor(int corePoolSize,
                                  int maximumPoolSize,
                                  long keepAliveTime,
                                  TimeUnit unit,
                                  BlockingQueue<Runnable> workQueue,
                                  ThreadFactory threadFactory,
                                  RejectedExecutionHandler handler) {
            if (corePoolSize < 0 ||
                maximumPoolSize <= 0 ||
                maximumPoolSize < corePoolSize ||
                keepAliveTime < 0)
                throw new IllegalArgumentException();
            if (workQueue == null || threadFactory == null || handler == null)
                throw new NullPointerException();
            this.corePoolSize = corePoolSize;
            this.maximumPoolSize = maximumPoolSize;
            this.workQueue = workQueue;
            this.keepAliveTime = unit.toNanos(keepAliveTime);
            this.threadFactory = threadFactory;
            this.handler = handler;
        }


#### 1).corePoolSize ####
corePoolSize字段表示的是线程池中一直存活着的线程的最小数量,这些一直存活着的线程又被称为核心线程.
当提交一个任务到线程池时,线程池会创建一个新的线程来执行任务,即使核心线程能够处理执行新任务也会创建线程,
等到需要执行的任务数大于线程池的核心线程时就不再创建,如果调用了线程池的prestartAllCoreThreads(),线程池会
提前创建并启动所有核心线程.

#### 2).maximumPoolSize ####
maximumPoolSize字段表示线程池内能够容纳线程数量的最大值,当然线程数量最大值不能超过CAPACITY的大小,
CAPACITY的数值为(1 << 29-1),约等于5亿.

    private static final int COUNT_BITS = Integer.SIZE - 3;
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

#### 3).keepAliveTime ####
keepAliveTime字段表示空闲线程处于等待状态的超时时间,也就是等待时间的上限值,超过该时间后该线程会停止工作.当总线程数大于corePoolSize(核心线程数),并且allowCoreThreadTimeOut为false时,这些多出来的非核心线程一旦进入到空闲等待状态,就开始计算各自的等待时间,并且这里设定的keepAliveTime的数值作为他们的超时时间,一旦某个线程的等待时间超出了这里设定的keepAliveTime,该线程就会停止工作,而核心线程在这种情况下却不会受超时机制的制约,核心线程会继续处于空闲等待状态而不会停止工作.如果allowCoreThreadTimeOut为true,不管核心或给核心线程都会受到超时机制的制约.如果将keepAliveTime和unit这两个参数分别设置为Long.MAX_VALUE和TimeUnit.NANOSECONDS(纳秒),这可让空闲线程一直处于存活状态.

**小结:**

 如果要执行的任务相对较多，并且每个任务执行的时间比较短，那么可以为该参数设置一个相对较大的数值，以提高线程的利用率。如果执行的任务相对较少, 线程池使用率相对较低, 那么可以先将该参数设置为一个较小的数值, 通过超时停止的机制来降低系统线程资源的开销, 后续如果发现线程池的使用率逐渐增高以后, 线程池会根据当前提交的任务数自动创建新的线程, 当然, 我们也可以自己手动调用 setKeepAliveTime(long, TimeUnit)方法来重新设定 keepAliveTime 字段的值

#### 4).workQueue ####
workQueue是BlockingQueue(阻塞队列)的实例,传入的泛型是Runnable.
阻塞队列是一个类似于生产者消费者模型的队列,当队列已满时如果继续向队列中插入元素,该插入操作将被阻塞一直处于等待状态,直到队列中有元素被移除产生空位置后,才能执行这次插入操作;当队列为空时,如果继续执行元素的删除或者获取操作,该操作同样会被阻塞而进入等待状态,直到队列中又有该元素后才能执行该操作.workQueue是一个用于保存等待执行任务的阻塞队列, 当提交一个新的任务到线程池以后, 线程池会根据当前池子中正在运行着的线程的数量, 指定出对该任务相应的处理方式, 主要有以下几种处理方式: 

1.如果线程池中正在运行的线程数少于核心线程数,那么线程池总是倾向于创建一个新线程来执行该任务,而不是将该任务
提交到该队列的workQueue中进行等待.

2.如果线程池中正在运行的线程数大于核心线程数,那么线程池总是倾向于将该任务先提交到队列workQueue中先让其等待
而不是创建一个新线程来执行该任务.

3.如果线程池中正在运行的线程数大于核心线程数,并且线程池中的阻塞队列也满了致使该任务入队失败,那么线程池会去判断
当前池子中运行的线程数是否已经等于了该线程池允许运行的最大线程数maximumPoolSize,如果发现已经等于,说明池子满了
无法再继续创建新的线程了,那么就会拒绝执行该任务,如果发现运行的线程数小于池子允许的最大线程数,那么就会创建
一个新的线程(不是核心线程)来执行该任务.

一般来说,队列对新提交的任务有三种常见的处理策略:

**(1) 直接切换.** 常用的队列是 SynchronousQueue (同步队列). 这种队列内部不会存储元素. 每一次插入操作都会先进入阻塞状态, 一直等到另一个线程执行了删除操作, 然后该插入操作才会执行. 同样地, 每一次删除操作也都会先进入阻塞状态, 一直等到另一个线程执行了插入操作, 然后该删除操作才会执行. 当提交一个任务到包含这种 SynchronousQueue 队列的线程池以后, 线程池会去检测是否有可用的空闲线程来执行该任务, 如果没有就直接新建一个线程来执行该任务而不是将该任务先暂存在队列中. “直接切换”的意思就是, 处理方式由”将任务暂时存入队列”直接切换为”新建一个线程来处理该任务”. 这种策略适合用来处理多个有相互依赖关系的任务, 因为该策略可以避免这些任务因一个没有及时处理而导致依赖于该任务的其他任务也不能及时处理而造成的锁定效果. 因为这种策略的目的是要让几乎每一个新提交的任务都能得到立即处理, 所以这种策略通常要求最大线程数 maximumPoolSizes 是无界的(即: Integer.MAX_VALUE). 静态工厂方法 Executors.newCachedThreadPool() 使用了这个队列。 

**(2) 使用无界队列** (也就是不预设队列的容量, 队列将使用 Integer.MAX_VALUE 作为其默认容量, 例如: 基于链表的阻塞队列 LinkedBlockingQueue). 使用无界队列将使得线程池中能够创建的最大线程数就等于核心线程数 corePoolSize, 这样线程池的 maximumPoolSize 的数值起不到任何作用. 如果向这种线程池中提交一个新任务时发现所有核心线程都处于运行状态, 那么该任务将被放入无界队列中等待处理. 当要处理的多个任务之间没有任何相互依赖关系时, 就适合使用这种队列策略来处理这些任务. 静态工厂方法 Executors.newFixedThreadPool() 使用了这个队列。 

**(3) 使用有界队列** (例如: 基于数组的阻塞队列 ArrayBlockingQueue). 当要求线程池的最大线程数 maximumPoolSizes 要限定在某个值以内时, 线程池使用有界队列能够降低资源的消耗, 但这也使得线程池对线程的调控变得更加困难. 因为队列容量和线程池容量都是有限的值, 要想使线程处理任务的吞吐量能够在一个相对合理的范围内, 同时又能使线程调配的难度相对较低, 并且又尽可能节省系统资源的消耗, 那么就需要合理地调配这两个数值. 通常来说, 设置较大的队列容量和较小的线程池容量, 能够降低系统资源的消耗(包括CPU的使用率, 操作系统资源的消耗, 上下文环境切换的开销等), 但却会降低线程处理任务的吞吐量. 如果发现提交的任务经常频繁地发生阻塞的情况, 那么你就可以考虑增大线程池的容量, 可以通过调用 setMaximumPoolSize() 方法来重新设定线程池的容量. 而设置较小的队列容量时, 通常需要将线程池的容量设置大一点, 这种情况下, CPU的使用率会相对较高, 当然如果线程池的容量设置过大的话, 可能会有非常非常多的线程来同时处理提交来的多个任务, 并发数过大时, 线程之间的调度将会是个非常严峻的问题, 这反而有可能降低任务处理的吞吐量, 出现过犹不及的局面.

#### 5).threadFactory ####
threadFactory线程工厂,用户创建线程,如果我们创建线程池的时候未指定该threadFactory参数,线程池则会使用 Executors.defaultThreadFactory() 方法创建默认的线程工厂.
我们也可以自定义threadFactory.

#### 6).handle ####
以下两个条件,满足其中任何一个的时候,如果继续向该线程池中提交新的任务,那么线程池将会调用他内部的RejectedExecutionHandler 对象的rejectdExecution()方法,表示拒绝执行这些新提交的任务.

1.当线程池处于shutDown关闭状态时(不论线程池和阻塞队列是否都已满)

2.当线程池中的所有线程都处于运行状态并且线程池中的阻塞队列已满时.


### 3.线程池的主要处理流程 ###
情况1: 如果线程池内的有效线程数少于核心线程数 corePoolSize, 那么就创建并启动一个线程(核心线程)来执行新提交的任务.
 
情况2: 如果线程池内的有效线程数达到了核心线程数 corePoolSize, 并且线程池内的阻塞队列未满, 那么就将新提交的任务加入到该阻塞队列中.
 
情况3: 如果线程池内的有效线程数达到了核心线程数 corePoolSize 但却小于最大线程数 maximumPoolSize, 并且线程池内的阻塞队列已满, 那么就创建并启动一个线程(非核心线程)来执行新提交的任务. 

情况4: 如果线程池内的有效线程数达到了最大线程数 maximumPoolSize, 并且线程池内的阻塞队列已满, 那么就让 RejectedExecutionHandler 根据它的拒绝策略来处理该任务, 默认的处理方式是直接抛异常.



例子:http://blog.csdn.net/zwk626542417/article/details/47084925
