主线程进入等待，子线程执行完毕或者到了超时时间，主线程继续执行；

1. 如果调用countDown方法次数少于构造方法传入子线程个数count，会导致主线程永远不会执行
2. await方法设置超时时间结束后，主线程会退出等待，继续执行；
3. CountDownLatch对象不可重用，计数器count最小为0；



```java
public class CountDownLatch {
    /**
     * Synchronization control For CountDownLatch.
     * Uses AQS state to represent count.
     */
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;

        Sync(int count) {
            setState(count);
        }

        int getCount() {
            return getState();
        }

        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }

        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
    }

    private final Sync sync;

    /**
     * 构造方法，传入计数个数
     */
    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }

    /**
     * 主线程进入等待
     */
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }

    /**
     * 主线程进入等待，设置超时时间
     */
    public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }

    /**
     * 计数减一，最小为0
     */
    public void countDown() {
        sync.releaseShared(1);
    }

    /**
     * Returns the current count.
     */
    public long getCount() {
        return sync.getCount();
    }


    public String toString() {
        return super.toString() + "[Count = " + sync.getCount() + "]";
    }
}
```



程序示例：

```java
public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(3);

        IntStream.range(0,3).forEach(i -> new Thread(()-> {
            try {
                    Thread.sleep(2000);
                    System.out.println("hello");
                } catch (InterruptedException e) {

                }finally {
                countDownLatch.countDown();
                }
        }).start());

        System.out.println("启动子线程完毕");

        countDownLatch.await();

        System.out.println("主线程执行完毕");
}

启动子线程完毕
hello
hello
hello
主线程执行完毕
```

