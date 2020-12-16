Lock定义：

```java
public interface Lock {

    /**
     * 获得锁，如果获取不到，线程进行睡眠，直到获得到为止
     * Lock l = ...;
 	 * l.lock();
 	 * try {
 	 *   // access the resource protected by this lock
 	 * } finally {
 	 *   l.unlock();
 	 * }
     */
    void lock();

    /**
     * 获得锁，如果获取不到，线程进行睡眠，直到获得到锁或者被其他线程阻断
     */
    void lockInterruptibly() throws InterruptedException;

    /**
     * 尝试获得锁，无论释放获得到锁都立马返回
     * Lock lock = ...;
     * if (lock.tryLock()) {
     *   try {
     *     // manipulate protected state
     *   } finally {
     *     lock.unlock();
     *   }
     * } else {
     *   // perform alternative actions
     * }
     */
    boolean tryLock();

    /**
     * 尝试获得锁，在指定的时间内获得到则返回true，或得不到返回false
     * 被中断返回并抛出异常
     */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    /**
     * 释放锁
     */
    void unlock();

    /**
     * 
     */
    Condition newCondition();
```



示例代码：

如果myMethod1的`lock.unlock()`注释掉,运行结果只会输出  *myMethod1 incoked* ，因为线程1中不会释放锁，线程2获取不到锁，不会执行。而该`Lock`是可重入锁，线程1在持有锁的情况下还能继续获得锁执行myMethod1方法。

```java
public class MyTest1 {

    //可重入锁
    private Lock lock = new ReentrantLock();

    public void myMethod1(){
        try {
            lock.lock();
            System.out.println("myMethod1 incoked");
        }finally {
            lock.unlock();
            //lock.unlock();
        }
    }

    public void myMethod2(){
        try {
            lock.lock();
            System.out.println("myMethod2 incoked");
        }finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        MyTest1 test = new MyTest1();

        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 10; i++){
                test.myMethod1();
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < 10; i++){
                test.myMethod2();
                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        thread1.start();
        thread2.start();
    }
}
```

##### Lock与synchronized关键字区别

1. 锁的获取方式:前者是通过程序代码的方式由开发者手工获取，后者是通过JVM来获取。
2. 具体实现方式:前者是通过Java代码的方式来实现，后者是通过JVM底层来实现。
3. 锁的释放方式:前者务必通过unlock (）方法在finally块中手工释放，后者是通过JVM来释放。
4. 锁的具体类型:前者提供了多种，如公平锁、非公平锁，后者与前者均提供了可重入锁。