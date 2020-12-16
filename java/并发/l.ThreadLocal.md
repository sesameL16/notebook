本质上，ThreadLocal是通过空间换取时间，实现每个线程当中都会有一个变量的副本，这样每个线程就会操作该副本，从而完全规避了多线程的并发问题。

```java
public class ThreadLocal<T> {
    //构造方法
	public ThreadLocal() {
    }
    
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
    
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
    
    public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
    
    //静态内部类，用于存储数据
    static class ThreadLocalMap {

        //Entry数据
        static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }

        private Entry[] table;
    }


}
```



1. Thread类中有个成员变量`ThreadLocal.ThreadLocalMap threadLocals = null`每个Thread对象都有一个ThreadLocalMap 来存储ThreadLocal的数据。
2. ThreadLocalMap 中用`Entry[] table`数组存储数据，一个类中有多个ThreadLocal对象，多个值存到一个线程的ThreadLocalMap 中。
3. ThreadLocalMap中使用ThreadLocal对象当key，ThreadLocal的值为value。
4. Entry继承WeakReference是为了解决内存溢出问题，保证Entry所引用的ThreadLocal对象可以被垃圾回收
5. ThreadLocal的get，set，remove方法都有移除Entry[] table中key为空的Entry逻辑。
6. 在使用ThreadLocal后要调用它的remove方法。



程序示例：

```java
public class MyTest7 {

    public static void main(String[] args) {
        MyRunnable runnable = new MyRunnable();
        Thread thread1 = new Thread(runnable);
        Thread thread2 = new Thread(runnable);
        thread1.start();
        thread2.start();
    }

}

class MyRunnable implements Runnable{

    private ThreadLocal<Integer> threadLocal = new ThreadLocal<>();

    @Override
    public void run() {
        threadLocal.set(new Random().nextInt());
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("threadLocal.value:" + threadLocal.get());
    }
}
```

