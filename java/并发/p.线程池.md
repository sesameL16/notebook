Executor接口：执行器的顶层接口

```java
public interface Executor {
    void execute(Runnable command);
}
```

ExecutorService接口

```Java
public interface ExecutorService extends Executor {
	void shutdown();
    List<Runnable> shutdownNow();
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit)throws InterruptedException;
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);
}
```

抽象类AbstractExecutorService

```java
public abstract class AbstractExecutorService implements ExecutorService {
    
}
```

类ThreadPoolExecutor

```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    //三位代表线程池本身的状态，其他29位记录线程个数
	private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
    private static final int COUNT_BITS = Integer.SIZE - 3;
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
    
    //最全的构造方法
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
    
    //拒绝策略，提交线程执行
    public static class CallerRunsPolicy implements RejectedExecutionHandler {
        public CallerRunsPolicy() { }
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                r.run();
            }
        }
    }
    //默认拒绝策略，抛异常
    public static class AbortPolicy implements RejectedExecutionHandler {
        public AbortPolicy() { }
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            throw new RejectedExecutionException("Task " + r.toString() +
                                                 " rejected from " +
                                                 e.toString());
        }
    }
    //拒绝策略，丢弃
    public static class DiscardPolicy implements RejectedExecutionHandler {
        public DiscardPolicy() { }
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        }
    }
    //拒绝策略，丢弃队头，尝试放到队尾
    public static class DiscardOldestPolicy implements RejectedExecutionHandler {
        public DiscardOldestPolicy() { }
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                e.getQueue().poll();
                e.execute(r);
            }
        }
    }
    
}
```

工厂类Executors

```java
public class Executors {
    //固定线程数量的线程池
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
    //单个线程数量的线程池
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
    //根据实际情况进行缓存的线程池
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
    
    //默认的线程工厂，普通用户线程，优先级是5
    static class DefaultThreadFactory implements ThreadFactory {
        private static final AtomicInteger poolNumber = new AtomicInteger(1);
        private final ThreadGroup group;
        private final AtomicInteger threadNumber = new AtomicInteger(1);
        private final String namePrefix;

        DefaultThreadFactory() {
            SecurityManager s = System.getSecurityManager();
            group = (s != null) ? s.getThreadGroup() :
                                  Thread.currentThread().getThreadGroup();
            namePrefix = "pool-" +
                          poolNumber.getAndIncrement() +
                         "-thread-";
        }

        public Thread newThread(Runnable r) {
            Thread t = new Thread(group, r,
                                  namePrefix + threadNumber.getAndIncrement(),
                                  0);
            if (t.isDaemon())
                t.setDaemon(false);
            if (t.getPriority() != Thread.NORM_PRIORITY)
                t.setPriority(Thread.NORM_PRIORITY);
            return t;
        }
    }
}
```



##### ThreadPoolExecutor构造方法参数

1. int corePoolSize:线程池当中所一直维护的线程数量，如果线程池处于任务空闲期间，那么该线程也并不会被回收掉
2. int maximumPoolSize:线程池中所维护的线程数的最大数量
3. long keepAliveTime:超过了corePoolSize的线程在经过keepAliveTime时间后如果一直处于空闲状态，那么超过的这部分线程将会被回收掉
4. TimeUnit unit:指的是keepAliveTime的时间单位
5. BlockingQueue<Runnable> workQueue:向线程池所提交的任务位于的阻塞队列，它的实现有多种方式
6. ThreadFactory threadFactory:线程工厂，用于创建新的线程并被线程池所管理，默认线程工厂所创建的线程都是用户线程且优先级为正常优先级
7. RejectedExecutionhandler handler:表示当线程池中的线程都在忙于执行任务且阻塞队列也已经满了的情况下，新到来的任务该如何被对待和处理。

##### 默认实现的拒绝策略

1. AbortPolicy:直接抛出一个运行期异常。
2. DiscardPolicy:默默地丢弃掉提交的任务，什么都不做且不抛出任何异常
3. DiscardOldestPolicy:丢弃掉阻塞队列中存放时间最久的任务(队头元素)，并且为当前所提交的任务留出一个队列中的空闲空间,以便将其放进到队列中
4. CallerRunsPolicy:直接由提交任务的线程来运行这个提交的任务

##### 线程提交

对于线程池来说，其提供了execute与submit两种方式来向线程池提交任务
总体来说，submit方法是可以取代execute方法的，因为它既可以接收Callable任务，也可以接收Runnable任务。

#####  关于线程池的总体执行策略:

1. 如果线程池中正在执行的线程数<corePoolSize，那么线程池就会优先选择创建新的线程而非将提交的任务加到阻塞队列中。
2. 如果线程池中正在执行的线程数>=corePoolSize，那么线程池就会优先选择对提交的任务进行阻塞排队而非创建新的线程。
3. 如果提交的任务无法加入到阻塞队列当中，那么线程池就会创建新的线程;如果创建的线程数超过了maximumPoolSize，那么拒绝策略就会起作用。

```java
public static void main(String[] args) {

    ExecutorService executorService = new ThreadPoolExecutor(3, 5, 0, TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(3),new ThreadPoolExecutor.CallerRunsPolicy());

    IntStream.range(0,9 ).forEach(i -> {
        executorService.submit(()-> {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName());
        });
    });
    executorService.shutdown();
}
```

##### 线程池状态

对于线程池来说，存在两个状态需要维护:

1. 线程池本身的状态:ctl的高3位来表示
2. 线程池中所运行着的线程的数量:ctl的其余29位来表示

线程池一共存在5种状态:
1.  RUNNING线程池可以接收新的任务提交，并且还可以正常处理阻塞队列中的任务。
2. SHUTDOWN:不再接收新的任务提交，不过线程池可以继续处理阻塞队列中的任务。
3. STOP:不再接收新的任务，同时还会丢弃阻塞队列中的既有任务;此外，它还会中断正在处理中的任务。
4.  TIDYING:所有的任务都执行完毕后(同时也涵盖了阻塞队列中的任务)，当前线程池中的活动的线程数降为0，将会调用terminated方法
5.  TERMINATED:线程池的终止状态，当terminated方法执行完毕后，线程池将会处于该状态之下。

线程池状态变化:

1. RUNNING->SHUTDOWN:当调用了线程池的shutdown方法时，或者当finalize方法被隐式调用后(该方法内部会调用shutdown方法)
2. RUNNING,SHUTDOWN->STOP:当调用了线程池的shutdownNow方法时
3. SHUTDOWN->TIDYING:在线程池与阻塞队列均变为空时
4. STOP->TIDYING:在线程池变为空时
5. TIDYING->TERMINATED:在terminated方法被执行完毕时

