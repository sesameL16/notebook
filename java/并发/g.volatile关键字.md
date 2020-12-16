主要作用：

1. 实现long/double类型变量的原子操作；
2. 防止指令重排序；
3. 实现变量的可见性，当使用volatile修饰变量时，应用就不会从寄存器中获取该变量的值，而是从内存（高速缓存）中获取；



与锁的相似之处：

1. 防止指令重排序；

2. 实现变量的可见性



与锁的不同之处：

1. volatile可以确保对变量写操作的原子性，但不具备排他性；
2. 使用锁可能会导致线程上下文切换（内核态与用户态之间的切换），但使用volatile不会出现这种情况；



示例：

```java
//如果要实现volatile写操作的原子性，那么就在等号右侧的赋值变量中不能出现被多线程所共享的变量，变量是volatile也不行

//合法是使用
volatile int a = 1;
private volatile int count;
//非法是使用
volatile int a = a + b;
volatile int a = a++;
volatile Date date = new Date();
```



防止指令冲排序与实现变量可见性都是通过一种手段来实现的：内存屏障（memory barrier）

1. 写入操作：

   ​	Release Barrier：防止volatile与上面所有操作的指令冲排序；

   ​	Store Barrier：刷新处理器缓存，结果是可以确保该存储屏障之前的一切操作所生成的结果对于其他处理器来说都可见；

```java
int a = 1;
String s = "hello";
//内存屏障（Release Barrier，释放屏障）
volatile boolean v = false;
//内存屏障（Store Barrier,存储屏障）    
```

2. 读取操作

   ​	Load Barrier：可以刷新处理器缓存，同步其他处理器对该volatile变量的修改结果；

   ​	Acquire Barrier：可以防止volatile读取操作与下面的所有操作语句的指令冲排序；

```java
//内存屏障（Load Barrier，加载屏障）
volatile boolean v1 = v;
//内存屏障（Acquire Barrier，获取屏障）
int a = 1;
String s = "hello";
```



锁中内存屏障的应用：

```java
monitorenter
//内存屏障（Acquire Barrier，获取屏障）
...................
//内存屏障（Release Barrier，释放屏障）
monitorexit
```



