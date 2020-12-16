关于ReentrantReadWriteLock的操作逻辑:

读锁:
1.在获取读锁时，会尝试判断当前对象是否拥有了写锁，如果已经拥有，则直接失败。
2.如果没有写锁，就表示当前对象没有排他锁，则当前线程会尝试给对象加锁。
3.如果当前线怪已经持有了该对象的锁，那么直接将读锁数量加1。

写锁:
1.在获取写锁时，会尝试判断当前对象是否拥有了锁(读锁与写锁)，如果已经拥有且持有的线程并非当前线程，直接失败。
2.如果当前对象没有被加锁，那么写锁就会为为当前对象上锁，并且将写锁的个数加1。
3.将当前对象的排他锁线程持有者设为自己。



程序示例：

```java
public class MyTest2 {
    
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    
    public void readMethod() throws InterruptedException {
        try {
            readWriteLock.readLock().lock();
            Thread.sleep(200);
            System.out.println("method");
        }finally {
            readWriteLock.readLock().unlock();
        }
    }

    public void writeMethod() throws InterruptedException {
        try {
            readWriteLock.writeLock().lock();
            Thread.sleep(200);
            System.out.println("method");
        }finally {
            readWriteLock.writeLock().unlock();
        }
    }
}
```

ReentrantReadWriteLock源码：

```java
public class ReentrantReadWriteLock implements ReadWriteLock, java.io.Serializable {
        
    private final ReentrantReadWriteLock.ReadLock readerLock;
    
    private final ReentrantReadWriteLock.WriteLock writerLock; 
    
    final Sync sync;
    //默认非公平锁
    public ReentrantReadWriteLock() {
        this(false);
    }

    public ReentrantReadWriteLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
        readerLock = new ReadLock(this);
        writerLock = new WriteLock(this);
    }
    
    //把state拆为高16位表示读，低16位标识写
    abstract static class Sync extends AbstractQueuedSynchronizer {
        static final int SHARED_SHIFT   = 16;
        static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
        static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
        static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
        
        protected final boolean tryRelease(int releases) {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            int nextc = getState() - releases;
            boolean free = exclusiveCount(nextc) == 0;
            if (free)
                setExclusiveOwnerThread(null);
            setState(nextc);
            return free;
        }
        
        protected final boolean tryAcquire(int acquires) {
            /*
             * Walkthrough:
             * 1. If read count nonzero or write count nonzero
             *    and owner is a different thread, fail.
             * 2. If count would saturate, fail. (This can only
             *    happen if count is already nonzero.)
             * 3. Otherwise, this thread is eligible for lock if
             *    it is either a reentrant acquire or
             *    queue policy allows it. If so, update state
             *    and set owner.
             */
            Thread current = Thread.currentThread();
            int c = getState();
            int w = exclusiveCount(c);
            if (c != 0) {
                // (Note: if c != 0 and w == 0 then shared count != 0)
                if (w == 0 || current != getExclusiveOwnerThread())
                    return false;
                if (w + exclusiveCount(acquires) > MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                // Reentrant acquire
                setState(c + acquires);
                return true;
            }
            if (writerShouldBlock() ||
                !compareAndSetState(c, c + acquires))
                return false;
            setExclusiveOwnerThread(current);
            return true;
        }

        protected final boolean tryReleaseShared(int unused) {
            Thread current = Thread.currentThread();
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
                if (firstReaderHoldCount == 1)
                    firstReader = null;
                else
                    firstReaderHoldCount--;
            } else {
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                int count = rh.count;
                if (count <= 1) {
                    readHolds.remove();
                    if (count <= 0)
                        throw unmatchedUnlockException();
                }
                --rh.count;
            }
            for (;;) {
                int c = getState();
                int nextc = c - SHARED_UNIT;
                if (compareAndSetState(c, nextc))
                    // Releasing the read lock has no effect on readers,
                    // but it may allow waiting writers to proceed if
                    // both read and write locks are now free.
                    return nextc == 0;
            }
        }
        
        protected final int tryAcquireShared(int unused) {
            /*
             * Walkthrough:
             * 1. If write lock held by another thread, fail.
             * 2. Otherwise, this thread is eligible for
             *    lock wrt state, so ask if it should block
             *    because of queue policy. If not, try
             *    to grant by CASing state and updating count.
             *    Note that step does not check for reentrant
             *    acquires, which is postponed to full version
             *    to avoid having to check hold count in
             *    the more typical non-reentrant case.
             * 3. If step 2 fails either because thread
             *    apparently not eligible or CAS fails or count
             *    saturated, chain to version with full retry loop.
             */
            Thread current = Thread.currentThread();
            int c = getState();
            if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
                return -1;
            int r = sharedCount(c);
            if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) {
                if (r == 0) {
                    firstReader = current;
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {
                    firstReaderHoldCount++;
                } else {
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                return 1;
            }
            return fullTryAcquireShared(current);
        }
    }
    
    
    public static class ReadLock implements Lock, java.io.Serializable {
        private final Sync sync;

        public void lock() {
            sync.acquireShared(1);
        }
        
        public void unlock() {
            sync.releaseShared(1);
        }
        
        public Condition newCondition() {
            throw new UnsupportedOperationException();
        }
    }
    
    public static class WriteLock implements Lock, java.io.Serializable {
        private final Sync sync;

        public void lock() {
            sync.acquire(1);
        }
        
        public void unlock() {
            sync.release(1);
        }
        
        public Condition newCondition() {
            return sync.newCondition();
        }
        
    }

}
```

