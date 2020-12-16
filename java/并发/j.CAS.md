悲观锁与乐观锁：

1. synchronized关键字与Lock等锁机制都是悲观锁：无论何种操作，首先需要上锁，接下来再去执行后续操作，从而接下来的所有操作都是由当前这个线程完成的。
2. 乐观锁：线程在操作之前不会做任何处理，而是直接去执行，当在最后执行变量更新的时候，当前线程需要有一种机制来确保当前被操作的变量是没有被其他线程修改的；CAS是乐观锁的一种极为重要的实现方式。



CAS ( Compare And  Swap )

比较与交换：这是一个不断循环的过程，一直到变量修改成功为止。CAS本身由硬件指令来提供支持的，硬件中是通过原子指令来实现比较和交换的；因此，CAS可以确保变量操作的原子性。



```java
public class MyTest {

    private int count;

    public int getCount(){
        return count;
    }

    /**
     * 读取 -> 修改 -> 写入 ： 不是原子操作
     */
    public void increseCount(){
        ++this.count;
    }
}
```



对于CAS来说，其主要涉及如下操作：

1. 需要被操作的内存值V
2. 需要进行比较的值A
3. 需要进行写入的值B

只有当 V == A 的时候，CAS才会通过原子操作的手段来将V的值更新为B



关于CAS的限制和问题：

1. 循环开销问题：并发量大的情况下会导致线程一直自旋
2. 只能保证一个变量的原子操作：可以通过AtomicReference来实现多个变量的原子操作
3. ABA问题：可以通过AtomicStampedReference解决ABA问题



原子类：

jdk提供了很多原子操作类，在`java.util.concurrent.atomic`包下，如AtomicInteger、AtomicLong等；

类中维护着`private static final Unsafe unsafe = Unsafe.getUnsafe();`成员变量用来完成原子操作；

Unsafe中使用do-while循环实现CAS操作

```java
public final long getAndSetLong(Object var1, long var2, long var4) {
    long var6;
    do {
        var6 = this.getLongVolatile(var1, var2);
    } while(!this.compareAndSwapLong(var1, var2, var6, var4));

    return var6;
}
```



