wait方法：

1. 它会导致当前线程进入等待，将当前线程放入这个对象的等待集合中（wait set）。

2. 当前线程必须拥有被调用对象的锁，当调用了wait()方法，当前线程会释放掉这个锁的拥有权，进入等待状态，直到另一个线程通知在这个对象上等待的所有线程，唤醒的方式是调用这个对象的notify()或者notifyAll()方法。

3. 一旦这个线程被其他线程唤醒，该线程与其他线程一同竞争这个对象的锁（公平竞争）；只有当该线程获得到这个对象的锁后，线程才会继续执行。

4. 对于无参数的版本来说，中断或者虚假的唤醒是有可能发生的，这个方法应该使用在循环当中。为确保线程在调用wait方法时拥有当前对象的锁，wait方法需要放到synchronized方法或者synchronized块中。

   ```java
   		synchronized (obj) {
               while (<condition does not hold>)
                   obj.wait();
               ... // Perform action appropriate to condition
           }
   ```

5. 在调用Thread类的sleep方法时，线程是不会释放掉对象的锁的

6. 在某一时刻，只有唯一一个线程可以拥有对象的锁。

7. static方法的锁属于这个类的Class对象，非static方法的锁属于当前对象



notify方法：

1. 它会唤醒等待这个对象锁的单个线程，如果有多个线程都在等待，会随机唤醒其中的一个。
2. 被唤醒的线程不会执行，直到当前线程放弃这个对象的锁，被唤醒的线程会与其他线程进行平等竞争这个对象的锁。
3. 本方法应该被持有此对象锁的线程调用，一个线程有三种方式拥有这个对象的锁
   * 执行这个对象被标记为synchronized的方法
   * 执行这个对象被标记为synchronized的语句块
   * 对于是Class类型的对象，执行这个类的被标记为synchronized  static的方法



notifyAll方法：

1. 它会唤醒等待这个对象锁的所有线程。
2. 被唤醒的线程不会执行，直到当前线程放弃这个对象的锁，被唤醒的线程会与其他线程进行平等竞争这个对象的锁。
3. 本方法应该被持有此对象锁的线程调用





程序示例：

1. 存在一个对象，该对象有一个int类型的成员变量counter，该成员变量的初始值为0，
2. 创建多个线程，其中一半线程对该对象成员变量counter加1，另一半线程对该对象成员变量counter减1.
3. 输出该成员变量counter每次变化后的值。
4. 最终输出结果为：101010101010101......

```java
public class MyTest1 {
    public static void main(String[] args) throws InterruptedException {
        Num num = new Num();
        AddThread addThread = new AddThread(num);
        MinusThread minusThread = new MinusThread(num);
        AddThread addThread1 = new AddThread(num);
        MinusThread minusThread1 = new MinusThread(num);
        addThread.start();
        minusThread.start();
        addThread1.start();
        minusThread1.start();
    }
}

class AddThread extends Thread{

    private Num num;

    public AddThread(Num num) {
        this.num = num;
    }

    @Override
    public void run() {
        for (int i = 0; i < 30; i++) {
            try {
                Thread.sleep((long) (Math.random() * 1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            num.increase();
        }
    }
}

class MinusThread extends Thread{

    private Num num;

    public MinusThread(Num num){
        this.num = num;
    }

    @Override
    public void run() {
        for (int i = 0; i < 30; i++){
            try {
                Thread.sleep((long) (Math.random() * 1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            num.decrease();
        }
    }
}

class Num{

    private int counter;

    public synchronized void increase(){
        while (counter != 0){
            try {
                wait();
            } catch (InterruptedException e) {
            }
        }
        counter ++;
        System.out.println(counter);
        notify();
    }

    public synchronized void decrease() {
        while (counter == 0) {
            try {
                wait();
            } catch (InterruptedException e) {
            }
        }
        counter--;
        System.out.println(counter);
        notify();
    }

}
```

