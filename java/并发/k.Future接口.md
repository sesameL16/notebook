

##### Future接口

```java
public interface Future<V> {
    /**
     * 终止操作
     */
    boolean cancel(boolean mayInterruptIfRunning);

    /**
     * 是够已经终止
     */
    boolean isCancelled();

    /**
     * 任务是否结束
     */
    boolean isDone();

    /**
     * 获得任务执行结果，阻塞直到获得结果
     */
    V get() throws InterruptedException, ExecutionException;

    /**
     * 获得任务执行结果，阻塞直到获得结果或者超时
     */
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```




##### FutureTask类

实现了Future接口，构造方法接收Callable或者Runnable，会将接收的Runnable转换为Callable

在run方法中调用Callable中的call()方法

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}

public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```



示例：

```java
public static void main(String[] args) throws InterruptedException, ExecutionException {
    Callable<Integer> callable = () -> {
        System.out.println("pre execution");
        Thread.sleep(5000);
        int num = new Random().nextInt(300);
        System.out.println("post execution");
        return num;
    };

    FutureTask<Integer> futureTask = new FutureTask<Integer>(callable);
    new Thread(futureTask).start();
    System.out.println("thread has started");

    Thread.sleep(2000);
    System.out.println(futureTask.get());
    System.out.println("main end");
}

结果：
thread has started
pre execution
post execution
276
main end
```



##### CompletableFuture类

CompletableFuture类提供了Future功能的扩展

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T>{
    
}
```



示例程序：

```java
public static void main(String[] args) {
        String result = CompletableFuture.supplyAsync(() -> "hello")
            .thenApply(value -> value + " world").join();
        System.out.println(result);

        System.out.println("============");
        CompletableFuture.supplyAsync(() -> "hello")
            .thenAccept(value -> System.out.println("welcome " + value));

        System.out.println("============");

        String result2 = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "hello";
        }).thenCombine(
                CompletableFuture.supplyAsync(() -> "hello"),
                (s1,s2) -> s1 + " " + s2).join();
        System.out.println(result2);
    }
```



```java
    public static void main(String[] args) {
        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
            try {
                TimeUnit.MILLISECONDS.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("task finished");
        });
		//当任务执行完后会进行回调
        completableFuture.whenComplete((t,action) -> System.out.println("执行完成"));
        System.out.println("主线程执行完毕");
    }
```





