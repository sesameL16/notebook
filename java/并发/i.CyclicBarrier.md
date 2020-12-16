CyclicBarrier的底层执行流程：

1. 初始化CyclicBarrier中的各种成员变量，包括parties、count、以及Runnable（可选）

2. 当调用await方法时，底层会先检查计数器是否归零，如果是的话，那么就首先执行可选的Runnable，接下来开始下一个Generation
3. 在下一个分代中，将会重置count为parties，并创建新的Generation实例
4. 调用Condition的signalAll方法唤醒所有在屏障前等待的线程，让其开始继续执行
5. 如果计数器没有归零，那么当前的调用线程将会通过Condition的await方法，在屏障前进行等待
6. 以上所有执行流程均在lock锁的控制范围内，不会出现并发情况



```java
public class CyclicBarrier {
    /**
     * 内部类，每个Generation表示一个阶段
     */
    private static class Generation {
        boolean broken = false;
    }
	
    private final ReentrantLock lock = new ReentrantLock();
    
    private final Condition trip = lock.newCondition();

    private final int parties;

    private final Runnable barrierCommand;

    private Generation generation = new Generation();

    private int count;
    
    /**
     * 构造方法，子线程数，达成条件执行的操作
     */
    public CyclicBarrier(int parties, Runnable barrierAction) {
        if (parties <= 0) throw new IllegalArgumentException();
        this.parties = parties;
        this.count = parties;
        this.barrierCommand = barrierAction;
    }

    /**
     * 构造方法，子线程数
     */
    public CyclicBarrier(int parties) {
        this(parties, null);
    }


    /**
     * 进入等待
     */
    public int await() throws InterruptedException, BrokenBarrierException {
        try {
            return dowait(false, 0L);
        } catch (TimeoutException toe) {
            throw new Error(toe); // cannot happen
        }
    }

    /**
     * 进入等待，超时会抛出异常
     */
    public int await(long timeout, TimeUnit unit)
        throws InterruptedException,
               BrokenBarrierException,
               TimeoutException {
        return dowait(true, unit.toNanos(timeout));
    }
}
```



程序示例：

```java
public static void main(String[] args) {
    CyclicBarrier cyclicBarrier = new CyclicBarrier(3);

    for (int i = 0; i < 3; i++){
        new Thread(() -> {
            try {
                Thread.sleep((long)(Math.random() * 2000));

                int randomInt = new Random().nextInt(500);

                System.out.println("hello" + randomInt);

                cyclicBarrier.await();

                System.out.println("world" + randomInt);

            } catch (Exception e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

