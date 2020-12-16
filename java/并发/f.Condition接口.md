1. 传统上，我们可以通过synchronized关键字 + wait + notify、notifyAll 来实现多个线程之间的协调与通信，整个过程都是由JVM帮助我们实现的，开发者无需了解底层的实现细节。
2. 从JDK 5开始，并发包提供了Lock，Condition（await + signal/signalAll）来实现多个线程之间的协调与通信，整个过程都是由开发者来控制的，而且相比于传统方式，更加灵活，功能更加强大。
3. Thread.sleep与await（或者是Object的wait方法）的本质区别：sleep方法本质上不会释放锁，而await方法会释放锁，并且在signal后，还需要重新获得锁才能继续执行（该行为与Object的wait方法完全一致）。



接口定义:

```java
public interface Condition {

    /**
     * 进入等待，可中断
     */
    void await() throws InterruptedException;

    /**
     * 进入等待，不可中断
     */
    void awaitUninterruptibly();

    /**
     * 进入等待，时间限制纳秒，返回 nanosTimeout - 等待用的时间
     */
    long awaitNanos(long nanosTimeout) throws InterruptedException;

    /**
     * 进入等待，传入限制时间和单位
     */
    boolean await(long time, TimeUnit unit) throws InterruptedException;

    /**
     * 进入等待，知道某个未来的时间
     */
    boolean awaitUntil(Date deadline) throws InterruptedException;

    /**
     * 唤醒一个在该Condition等待的线程
     */
    void signal();

    /**
     * 唤醒所有在该Condition等待的线程
     */
    void signalAll();
}
```



程序示例：

```java
public class MyTeat2 {

    public static void main(String[] args) {
        BoundContainer boundContainer = new BoundContainer();

        IntStream.range(0,10).forEach( i -> new Thread( () -> {
            try {
                boundContainer.take();
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }).start());

        IntStream.range(0,10).forEach( i -> new Thread( () -> {
            try {
                boundContainer.put("hello");
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }).start());

    }

}

class BoundContainer{

    private String[] elements = new String[10];

    private Lock lock = new ReentrantLock();

    private Condition notEmptyCondition = lock.newCondition();

    private Condition notFullCondition = lock.newCondition();

    private int count;

    private int putIndex;

    private int takeIndex;

    public void put(String element) throws InterruptedException {
        lock.lock();
        try {
            while (this.count == elements.length){
                notFullCondition.await();
            }
            elements[putIndex] = element;
            if (++putIndex == elements.length){
                putIndex = 0;
            }
            count++;
            notEmptyCondition.signal();
            System.out.println("put method:" + Arrays.toString(elements));
        }finally {
            lock.unlock();
        }
    }

    public String take() throws InterruptedException {
        lock.lock();
        try {
            while (this.count == 0){
                notEmptyCondition.await();
            }
            String element = elements[takeIndex];
            elements[takeIndex] = null;
            if (++takeIndex == elements.length){
                takeIndex = 0;
            }
            count--;
            notFullCondition.signal();
            System.out.println("take method:" + Arrays.toString(elements));
            return element;
        }finally {
            lock.unlock();
        }
    }

}

```



