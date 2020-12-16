##### synchronized关键字:

1. 当一个类中有多个synchronized关键字标记的普通方法，在某一时刻只能有一个线程拿到这个对象的锁，执行synchronized方法
2. synchronized关键字标记的static方法，线程执行拿到的锁是 `类名.class` 对象的锁



程序示例一:

```java
public class MyThreadTest {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        Thread thread1 = new Thread(myThread);
        Thread thread2 = new Thread(myThread);
        thread1.start();
        thread2.start();
    }

}

class MyThread implements Runnable{

    int x;

    @Override
    public void run() {
        x = 0;
        while (true){
            System.out.println("result:" + x++);
            try {
                Thread.sleep((long) (Math.random() * 1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (x == 30){
                break;
            }
        }
    }
}
```

程序示例二:

```java
public class MyThreadTest1 {
    public static void main(String[] args) throws InterruptedException {
        MyClass myClass = new MyClass();
        Thread1 thread1 = new Thread1(myClass);
        Thread2 thread2 = new Thread2(myClass);
        thread1.start();
        Thread.sleep(700);
        thread2.start();
    }
}

class MyClass{

    public synchronized void hello(){
        try {
            Thread.sleep(4000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("hello");
    }

    public synchronized void world(){
        System.out.println("world");
    }
}

class Thread1 extends Thread{

    private MyClass myClass;

    public Thread1(MyClass myClass){
        this.myClass = myClass;
    }

    @Override
    public void run() {
        myClass.hello();
    }
}

class Thread2 extends Thread{

    private MyClass myClass;

    public Thread2(MyClass myClass){
        this.myClass = myClass;
    }

    @Override
    public void run() {
        myClass.world();
    }
}
```

##### synchronized修饰代码快

1. 当使用synchronized关键字修饰代码块时，字节码层面是通过monitorenter 和monitorexit 指令来实现锁的获取与释放动作。
2. 当线程进入到monitorenter 指令后，线程会持有Monitor对象，对出monitorenter 指令后，线程将会释放Monitor对象。

```java
public class MyTest1 {

    private Object object = new Object();

    public void method(){
        synchronized (object){
            System.out.println("hello world");
        }
    }
}
```

使用`javap -c MyTest1.class`命令进行反编译得到结果：

两个monitorexit是因为无论正常执行还是抛出异常都会释放锁

```java
  public void method();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: getfield      #3    // Field object:Ljava/lang/Object;
         4: dup
         5: astore_1
         6: monitorenter
         7: getstatic     #4    // Field java/lang/System.out:Ljava/io/PrintStream;
        10: ldc           #5    // String hello world
        12: invokevirtual #6    // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        15: aload_1
        16: monitorexit
        17: goto          25
        20: astore_2
        21: aload_1
        22: monitorexit
        23: aload_2
        24: athrow
        25: return


```

##### synchronized修饰方法

1. 对于synchronized关键字修饰方法来说，并没有出现monitorenter与monitorexit指令，而是出现了一个ACC_SYNCHRONIZED标志.
2. JVM使用了ACC_SYNCHRONIZED访问标志来区分一个方法是否为同步方法;当方法被调用时，调用指令会检查该方法是否拥有ACC_SYNCHRONIZED标志。
3. 如果有，那么执行线程将会先持有方法所在对象的Monitor对象（静态方方法为该类对应的Class对象），然后再去执行方法体;在该方法执行期间，其他任何线程均无法再获取到这个Monitor对象，当线程执行完方法后，它将释放掉这个Monitor对象。

```java
public class MyTest2 {

    public synchronized void method(){
        System.out.println("hello world");
    }
}

public class MyTest3 {

    public static synchronized void method(){
        System.out.println("hello world");
    }
}
```

```java
//非静态方法 
public synchronized void method();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2     // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3     // String hello world
         5: invokevirtual #4     // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return

//静态方法             
public static synchronized void method();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=0, args_size=0
         0: getstatic     #2   // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3   // String hello world
         5: invokevirtual #4   // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return

```

