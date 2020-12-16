##### 锁的底层实现

1. JVM中的同步是基于进入与退出监视器对象(管程对象)(Monitor)来实现的，每个对象实例都会有一个Monitor对象，Monitor对象会和Java对象一同创建并销毁。Monitor对象是由C++来实现的。
2. 当多个线程同时访问一段同步代码时，这些线程会被放到一个EntryList集合中，处于阻塞状态的线程都会被放到该列表当中。Monitor是依赖于底层操作系统的mutex lock来实现互斥的，线程获取mutex成功，则会持有该mutex，这时其他线程就无法再获取到该mutex。
3. 如果线程调用了，wait方法，那么该线程就会释放掉所持有的mutex，并且该线程会进入到WaitSet集合(等待集合)中，等待下一次被其他线程调用notify/notifyAll唤醒。如果当前线程顺利执行完毕方法，那么它也会释放掉所持有的mutex。
4. 同步锁在这种实现方式当中，因为Monitor是依赖于底层的操作系统实现，这样就存在用户态与内核态之间的切换，所以会增加性能开销。通过对象互斥锁的概念来保证共享数据操作的完整性。每个对象都对应于一个可称为【互斥锁】的标记，这个标记用于保证在任何时刻，只能有一个线程访问该对象。

##### 互斥锁的属性

1. PTHREAD_MUTEX_TIMED_NP:这是缺省值，也就是普通锁。当一个线程加锁以后.其余请求锁的线程将会形成一个等待队列，并且在解锁后按照优先级获取到锁。这种策略可以确保资源分配的公平性。
2. THREAD_MUTEX_RECURSIVE NP:嵌套锁。允许一个线程对同一个锁成功获取多次，并通过unlock解锁。如果是不同线程请求，则在加锁线程解锁时重新进行竞争。
3. PTHREAD_MUTEX_ERRORCHECK_NP:检错锁。如果一个线程谓求同一个锁，则返回EDEADLX，否则与PTHREAD_MUTEX_TIMED_NP类型动作相同，这样 就保证了当不允件多次加锁时不会出现最简单情况下的死锁。
4. PTHREAD_MUTEX_ADAPTIVE_NP:适应锁，动作最简单的锁类型，仅仅等待解锁后重新竟争。

##### jdk实现

1. 在JDK 1.5之前，我们若想实现线程同步，只能通过synchronized关键字这一种方式来达成;底层，Java也是通synchronized关键字来做到数据的原子性维护的；synchronized关键字是JVM实现的一种内置锁，从底层角度来说，这种锁的获取与释放都是由JVM帮我们隐式实现的。

2. 从JDK 1.5开始，并发包引入了Lock锁，Lock同步锁是基于Java来实现的，因此锁的获取与释放都是通过Java代码来实现与控制的；然而，synchronized是基于底层操作系统的Mutex Lock来实现的，每次对锁的获取与释放动作都会带来用户态与内核态之间的切换，这种切换会极大地增加系统的负担;在并发量较高时，也就是说锁的竞争比较激烈时，synchronized锁在性能上的表现就非常差。

3. 从JDK 1.6开始，synchronized锁的实现发生了很大的变化；JVM引入了相应的优化手段来提升synchronized锁的性能，这种提升涉及到偏向锁、轻量级锁及重量级锁等，从而减少锁的竟争所带来的用户态与内核态之间的切换；这种锁的优化实际上是通过Java对象头中的一些标志位来去实现的；对于锁的访问与改变，实际上都与Java对象头息息相关。

###### JVM对synchronized支持

从JDK 1.6开始，对象实例在堆当中会被划分为三个组成部分:对象头、实例数据与对齐填充。对象头主要也是由3块内容来构成:

1. Mark Word
2. 指向类的指针
3. 数组长度

其中Mark Word(它记录了对象、锁及垃圾回收相关的信息，在64位的JVM中，其长度也是64bit)的位信息包括了如下组成部分:

1. 无锁标记
2. 偏向锁标记
3. 轻量级锁标记
4. 重量级锁标记
5. GC标记

对于synchronized锁来说，锁的升级主要都是通过Mark Word中的锁标志位与是否是偏向锁标志位来达成的synchronized关键字所对应的锁都是先从偏向锁开始，随着锁竞争的不断升级，逐步演化至轻量级锁，最后则变成了重量级锁。

对于锁的演化来说，它会经历如下阶段:
		无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁



###### 偏向锁:
针对于一个线程来说的，它的主要作用就是优化同一个线程多次获取一个锁的情况；如果一个synchronized方法被一个线程访问，那么这个方法所在的对象就会在其Mark Word中将偏向锁进行标记，同时还会有一个字段来存储该线程的ID；当这个线程再次访问同一个synchronized方法时，它会检查这个对象的Mark Word的偏向锁标记以及是否指向了其线程ID。，如果是的话，那么该线程就无需再去进入管程（Monitor)了，而是直接进入到该方法体中。

如果是另外一个线程访问这个synchronized方法，那么实际情况会如何呢?

偏向锁会被取消掉。



#####  轻量级锁:
若第一个线程已经获取到了当前对象的锁，这时第二个线程又开始尝试争抢该对象的锁，由于该对象的锁已经被第一个线程获取到，因此它是偏向锁，而第二个线程在争抢时，会发现该对象头中的Mark Word已经是偏向锁，但里面存储的线程ID并不是自己(是第一个线程)，那么它会进行CAS(Compare and Swap），从而获取到锁，这里面存在两种情况:

1. 获取锁成功:那么它会直接将Mark Word中的线程ID由第一个线程变成自己(偏向锁标记位保持不变)，这样该对象依然会保持偏向锁的状态。
2. 获取锁失败:则表示这时可能会有多个线程同时在尝试争抢该对象的锁，那么这时偏向锁就会进行升级，升级为轻里级锁



##### 自旋锁：
1. 自旋锁是轻量级锁的一种实现，若自旋失败(依然无法获取到锁)，那么锁就会转化为重量级锁，在这种情况下，无法获取到锁的线程都会进入到Monitor(即内核态)，自旋最大的一个特点就是避免了线程从用户态进入到内核态。
2. 那些处于EntryList与WaitSet中的线程均处于阻塞状态，阻塞操作是由操作系统来完成的，在linux下是通过pthread_mutex_lock函数实现的。线程被阻塞后便会进入到内核调度状态，这会导致系统在用户态与内核态之间来回切换，严重影响锁的性能。
3. 解决上述问题的办法便是自旋(Spin)。其原理是:当发生对Monitor的争用时，若Owner能够在很短的时间内释放掉锁，则那些正在争用的线程就可以稍微等待一下(即所谓的自旋)，在Owner线程释放锁之后，争用线程可能会立刻获取到锁，从而避免了系统阻塞。不过，当Owner运行的时间超过了临界值后，争用线程自旋一段时间后依然无法获取到锁，这时争用线程则会停止自旋而进入到阻塞状态。所以总体的思想是:先自旋，不成功再进行阻塞，尽最降低阻塞的可能性，这对那些执行时间很短的代码块来说有极大的性能提升。显然,自旋在多处理器(多核心)上才有怠义。

##### 重量级锁：
线程最终从用户态进入到了内核态。



##### 锁消除技术

JIT编译器(Just In Time编译器)可以在动态编译同步代码时，使用一种叫做逃逸分析的技术，来通过该项技术判别程序中所使用的锁对象是否只被一个线程所使用，而没有散布到其他线程当中；如果情况就是这样的话，那么JIT编辑器在编译这个同步代码时就不会生成synchronized关键字所标识的锁的申请与释放机器码，从而消除了锁的便用流程。

```java
public class MyTest4 {
    
    public void method(){
        Object object = new Object();
        
        synchronized (object){
            System.out.println("hello world");
        }
    }
}
```

##### 锁粗化

JIT编译器在执行动态编译时，若发现前后相邻的synchronized块使用的是同一个锁对象，那么它就会把这几个synchronized块给合并为一个较大的同步块，这样做的好处在于线程在执行这些代码时，就无需频繁申请与释放锁了，从而达到申请与释放锁一次，就可以执行完全部的同步代码块，从而提升了性能。

```java
public class MyTest5 {
    
    private Object object = new Object();
    
    public void method(){
        
        synchronized (object){
            System.out.println("hello world");
        }

        synchronized (object){
            System.out.println("welcome");
        }

        synchronized (object){
            System.out.println("person");
        }
    }
}
```



##### 死锁

死锁:线程1等待线程2互斥持有的资源，而线程2也在等待线程1互斥持有的资源，两个线程都无法继续执行
活锁:线程持续重试一个总是失败的操作，导致无法继续执行
饿死:线程一直被调度器延迟访问其赖以执行的资源，也许是调度器先于低优先级的线程而执行高优先级的线程，同时总是     		会有一个高优先级的线程可以执行，饿死也叫做无限延迟

代码示例：

```java
public class MyTest6 {

    private Object lock1 = new Object();
    private Object lock2 = new Object();

    public void myMethod1(){
        synchronized (lock1){
            synchronized (lock2){
                System.out.println("myMethod1 invoked");
            }
        }
    }

    public void myMethod2(){
        synchronized (lock2){
            synchronized (lock1){
                System.out.println("myMethod2 invoked");
            }
        }
    }

    public static void main(String[] args) {
        MyTest6 test = new MyTest6();

        Runnable runnable1 = () -> {
            while (true){
                test.myMethod1();
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };

        Runnable runnable2 = () -> {
            while (true){
                test.myMethod2();
                try {
                    Thread.sleep(150);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };

        Thread thread1 = new Thread(runnable1);
        Thread thread2 = new Thread(runnable2);
        thread1.start();
        thread2.start();
    }
}
```

##### 死锁检测工具

jvisualvm:

生产环境tomcat的配置,远程连接后可显示

```java
export JAVA_OPTS="-Xms256m -Xmx512m -Xss256m -XX:PermSize=512m -XX:MaxPermSize=1024m  
-Djava.rmi.server.hostname=136.64.45.24 -Dcom.sun.management.jmxremote.port=9315 
-Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"
```

死锁信息：

双击应用，点击线程，线程Dump，即可查看死锁信息

```java
Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x0000000026bff5e8 (object 0x0000000715b05bb8, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x0000000026c00be8 (object 0x0000000715b05bc8, a java.lang.Object),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
        at cn.bejwt.jdk8.concurrency.MyTest6.myMethod2(MyTest6.java:19)
        - waiting to lock <0x0000000715b05bb8> (a java.lang.Object)
        - locked <0x0000000715b05bc8> (a java.lang.Object)
        at cn.bejwt.jdk8.concurrency.MyTest6.lambda$main$1(MyTest6.java:40)
        at cn.bejwt.jdk8.concurrency.MyTest6$$Lambda$2/1096979270.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"Thread-0":
        at cn.bejwt.jdk8.concurrency.MyTest6.myMethod1(MyTest6.java:11)
        - waiting to lock <0x0000000715b05bc8> (a java.lang.Object)
        - locked <0x0000000715b05bb8> (a java.lang.Object)
        at cn.bejwt.jdk8.concurrency.MyTest6.lambda$main$0(MyTest6.java:29)
        at cn.bejwt.jdk8.concurrency.MyTest6$$Lambda$1/1324119927.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.


```



jps + jstack

在cmd控制台执行`jps -l`,然后执行`jstack 448 `，会出现使用上述工具相同信息

```
C:\Users\sesame>jps -l
448 cn.bejwt.jdk8.concurrency.MyTest6
5872 org.jetbrains.idea.maven.server.RemoteMavenServer
11204
11556 org/netbeans/Main
10312 org.jetbrains.jps.cmdline.Launcher
10780 sun.tools.jps.Jps

C:\Users\sesame>jstack 448
2020-08-16 14:34:22
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.131-b11 mixed mode):

"RMI TCP Connection(3)-192.168.115.1" #29 daemon prio=5 os_prio=0 tid=0x000000002b47a000 nid=0x33b4 runnable [0x000000000145e000]
   java.lang.Thread.State: RUNNABLE
   
   。。。。。。。。。。。。。。。。。
```



