对于ReentrantLock来说.其执行逻辑如下所示:

1. 尝试获取对象的锁，如果获取不到，那么它就会进入到AQS的阻塞队列当中。

2. 如果获取到，那么根据锁是公平锁还是非公平锁来进行不同的处理

   1. 如果是公平锁，那么线程会直接放置到AQS阻塞队列的末尾

   2. 如果是非公平锁，那么线程会首先尝试进行CAS计算，如果成功，则直接获取到锁;

      如果失败，则与公平锁的处理方式一致，被放到阻塞队列末尾

3. 当锁被释放时(调用了unlock方法)，那么底层会调用release方法对state成员变量值进行减一操作，如果减一后，state值不为0，那么release操作就执行完毕;

   如果减一操作后，state值为0，则调用LockSupport的unpark方法唤醒该线程后的等待队列中的第一个后继线程(pthreade_mutex_unlock)，将其唤醒，使之能够获取到对象的锁(release时，对于公平锁与非公平锁的处理逻辑是一致的);

   之所以调用release方法后。state值可能不为零，原因在于ReentrantLock是可里入锁，表示线程可以多次调用lock方法，导致每调用一次，state值都会加一。

4. 对于ReentrantLock来说，所谓的上锁.本质上就是对AQS中的state成员变量的操作:对该成员变量+1，表示上锁;对该成员变量-1,表示释放锁。



```java
public class ReentrantLock implements Lock, java.io.Serializable {
    
    private final Sync sync;
    
    abstract static class Sync extends AbstractQueuedSynchronizer {
        
        abstract void lock();
		//非公平锁尝试上锁
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
		//释放锁
        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
        //上锁调用acquire
        public final void acquire(int arg) {
            if (!tryAcquire(arg) &&
                acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
                selfInterrupt();
        }
    }
    //非公平锁内部类
    static final class NonfairSync extends Sync {
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
    //公平锁内部类
    static final class FairSync extends Sync {
        final void lock() {
            acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
    
    //默认非公平是
    public ReentrantLock() {
        sync = new NonfairSync();
    }
	//true公平锁，false非公平锁
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
    
    public void lock() {
        sync.lock();
    }
    
    public void unlock() {
        sync.release(1);
    }
    //获得Condition对象
    public Condition newCondition() {
        return sync.newCondition();
    }
    
}
```