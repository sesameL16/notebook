自定义Collector

```java
public class MySetCollector<T> implements Collector<T, Set<T>,Set<T>> {
    @Override
    public Supplier<Set<T>> supplier() {
        System.out.println("supplier invoked");
        return HashSet::new;
    }

    @Override
    public BiConsumer<Set<T>, T> accumulator() {
        System.out.println("accumulator invoked");
        return Set::add;
    }

    @Override
    public BinaryOperator<Set<T>> combiner() {
        System.out.println("combiner invoked");
        return (set1,set2) -> {
            set1.addAll(set2);
            return set1;
        };
    }

    @Override
    public Function<Set<T>, Set<T>> finisher() {
        System.out.println("finisher invoked");
        return Function.identity();
    }

    @Override
    public Set<Characteristics> characteristics() {
        System.out.println("characteristics invoked");
        return Collections.unmodifiableSet(
            EnumSet.of(Characteristics.IDENTITY_FINISH ,Characteristics.UNORDERED));
    }

    public static void main(String[] args) {
        List<String> list = Arrays.asList("hello","world","welcome","hello");
        Set<String> collect = list.stream().collect(new MySetCollector<>());
        System.out.println(collect);
    }
}

输出：
supplier invoked
accumulator invoked
combiner invoked
characteristics invoked
characteristics invoked
[world, hello, welcome]

```

characteristics中的IDENTITY_FINISH属性表示返回combiner返回结果本身，会强制类型转换

源码中对supplier、accumulator、combiner各调用一次，调用characteristics两次，没有调用finisher。

```java
    public static <T, I> TerminalOp<T, I>
    makeRef(Collector<? super T, I, ?> collector) {
        Supplier<I> supplier = Objects.requireNonNull(collector).supplier();
        BiConsumer<I, ? super T> accumulator = collector.accumulator();
        BinaryOperator<I> combiner = collector.combiner();
        .....
        
        return new ReduceOp<T, I, ReducingSink>(StreamShape.REFERENCE) {
            @Override
            public int getOpFlags() {
                return collector.characteristics().
                    contains(Collector.Characteristics.UNORDERED)
                       ? StreamOpFlag.NOT_ORDERED
                       : 0;
            }
        };
    }

return collector.characteristics().contains(Collector.Characteristics.IDENTITY_FINISH)
               ? (R) container
               : collector.finisher().apply(container);
```

characteristics中的CONCURRENT属性表示并发的，

1. 在并发流中，不加该属性，多个线程会调用多次supplier方法，创建多个中间结果集合，每个线程操作各自的集合，然后调用combiner进行合并。
2. 在并发流中，添加该属性，多个线程会调用一次supplier方法，创建一个中间结果集合，多个线程同时操作一个中间结果，不调用combiner方法。
3. 在非并发流中，单个线程调用一次supplier方法，创建一个中间结果集合，操作一个中间结果，不调用combiner方法。

```java
public class MySetCollector<T> implements Collector<T, Set<T>,Map<T,T>> {
    @Override
    public Supplier<Set<T>> supplier() {
        System.out.println("supplier invoked");
        return () -> {
            System.out.println("调用supplier.get");
            return new HashSet<>();
        };
    }

    @Override
    public BiConsumer<Set<T>, T> accumulator() {
        System.out.println("accumulator invoked");
        return (set,item) ->{
            System.out.println("accumulator调用BiConsumer.accept:" + set + "线程" + Thread.currentThread().getName());
            set.add(item);
        };
    }

    @Override
    public BinaryOperator<Set<T>> combiner() {
        System.out.println("combiner invoked");
        return (set1,set2) -> {
            System.out.println("combiner调用BinaryOperator.apply合并元素");
            set1.addAll(set2);
            return set1;
        };
    }

    @Override
    public Function<Set<T>, Map<T, T>> finisher() {
        return set -> {
            HashMap<T,T> map = new HashMap<>();
            set.stream().forEach(item -> map.put(item,item ));
            return map;
        };
    }

    @Override
    public Set<Characteristics> characteristics() {
        System.out.println("characteristics invoked");
        //return Collections.unmodifiableSet(EnumSet.of(Characteristics.UNORDERED));
        return Collections.unmodifiableSet(EnumSet.of(Characteristics.UNORDERED,Characteristics.CONCURRENT));
    }

    public static void main(String[] args) {
        List<String> list = Arrays.asList("hello","world","welcome","hello","a","b","c","d","e","f");
        Map<String, String> collect = list.parallelStream().collect(new MySetCollector<>());
        System.out.println(collect);
    }
}
```

输出结果:

```java
单线程结果：
    
supplier invoked
accumulator invoked
combiner invoked
characteristics invoked
调用supplier.get
accumulator调用BiConsumer.accept:[]线程main
accumulator调用BiConsumer.accept:[hello]线程main
accumulator调用BiConsumer.accept:[world, hello]线程main
accumulator调用BiConsumer.accept:[world, hello, welcome]线程main
accumulator调用BiConsumer.accept:[world, hello, welcome]线程main
accumulator调用BiConsumer.accept:[a, world, hello, welcome]线程main
accumulator调用BiConsumer.accept:[a, b, world, hello, welcome]线程main
accumulator调用BiConsumer.accept:[a, b, world, c, hello, welcome]线程main
accumulator调用BiConsumer.accept:[a, b, world, c, d, hello, welcome]线程main
accumulator调用BiConsumer.accept:[a, b, world, c, d, e, hello, welcome]线程main
characteristics invoked
{a=a, b=b, world=world, c=c, d=d, e=e, f=f, hello=hello, welcome=welcome}


多线程不加CONCURRENT属性：
    
characteristics invoked
supplier invoked
accumulator invoked
combiner invoked
characteristics invoked
调用supplier.get
调用supplier.get
调用supplier.get
调用supplier.get
调用supplier.get
调用supplier.get
调用supplier.get
accumulator调用BiConsumer.accept:[]线程ForkJoinPool.commonPool-worker-15
accumulator调用BiConsumer.accept:[]线程main
accumulator调用BiConsumer.accept:[]线程ForkJoinPool.commonPool-worker-9
combiner调用BinaryOperator.apply合并元素
accumulator调用BiConsumer.accept:[]线程ForkJoinPool.commonPool-worker-4
调用supplier.get
调用supplier.get
accumulator调用BiConsumer.accept:[]线程ForkJoinPool.commonPool-worker-2
accumulator调用BiConsumer.accept:[]线程ForkJoinPool.commonPool-worker-6
调用supplier.get
accumulator调用BiConsumer.accept:[]线程ForkJoinPool.commonPool-worker-13
accumulator调用BiConsumer.accept:[]线程ForkJoinPool.commonPool-worker-8
accumulator调用BiConsumer.accept:[]线程ForkJoinPool.commonPool-worker-11
accumulator调用BiConsumer.accept:[]线程ForkJoinPool.commonPool-worker-1
combiner调用BinaryOperator.apply合并元素
combiner调用BinaryOperator.apply合并元素
combiner调用BinaryOperator.apply合并元素
combiner调用BinaryOperator.apply合并元素
combiner调用BinaryOperator.apply合并元素
combiner调用BinaryOperator.apply合并元素
combiner调用BinaryOperator.apply合并元素
combiner调用BinaryOperator.apply合并元素
characteristics invoked
{a=a, b=b, world=world, c=c, d=d, e=e, f=f, hello=hello, welcome=welcome}    


多线程添加CONCURRENT属性：
    
characteristics invoked
characteristics invoked
supplier invoked
调用supplier.get
accumulator invoked
accumulator调用BiConsumer.accept:[]线程main
accumulator调用BiConsumer.accept:[c]线程main
accumulator调用BiConsumer.accept:[b, c]线程main
accumulator调用BiConsumer.accept:[b, c, d]线程main
accumulator调用BiConsumer.accept:[b, c, d, f]线程ForkJoinPool.commonPool-worker-11
accumulator调用BiConsumer.accept:[b, c, d, f, hello]线程ForkJoinPool.commonPool-worker-11
accumulator调用BiConsumer.accept:[b, c, d, f, hello]线程main
accumulator调用BiConsumer.accept:[b, c, d, f, hello]线程ForkJoinPool.commonPool-worker-9
accumulator调用BiConsumer.accept:[b, c, world, d, f, hello, welcome]线程ForkJoinPool.commonPool-worker-2
accumulator调用BiConsumer.accept:[b, c, world, d, e, f, hello, welcome]线程ForkJoinPool.commonPool-worker-13
characteristics invoked
{a=a, b=b, c=c, world=world, d=d, e=e, f=f, hello=hello, welcome=welcome}
```

注意事项：

代码中accumulator方法中，多个线程对集合进行读写操作，有几率导致同时对集合进行遍历和写入。

抛出ConcurrentModificationException异常

```java
System.out.println("accumulator调用BiConsumer.accept:" + set + "线程" + Thread.currentThread().getName());

set.add(item);
```

```java
Exception in thread "main" java.util.ConcurrentModificationException: java.util.ConcurrentModificationException
	at java.util.stream.ForEachOps$ForEachOp.evaluateParallel(ForEachOps.java:160)
	at java.util.stream.ForEachOps$ForEachOp$OfRef.evaluateParallel(ForEachOps.java:174)
	at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:233)
	at java.util.stream.ReferencePipeline.forEach(ReferencePipeline.java:418)
	at java.util.stream.ReferencePipeline$Head.forEach(ReferencePipeline.java:583)
	at java.util.stream.ReferencePipeline.collect(ReferencePipeline.java:496)
	at cn.bejwt.jdk8.stream.MySetCollector.main(MySetCollector.java:57)
Caused by: java.util.ConcurrentModificationException
	at java.util.HashMap$HashIterator.nextNode(HashMap.java:1437)
	at java.util.HashMap$KeyIterator.next(HashMap.java:1461)
	at java.util.AbstractCollection.toString(AbstractCollection.java:461)
	at java.lang.String.valueOf(String.java:2994)
	at java.lang.StringBuilder.append(StringBuilder.java:131)
	at cn.bejwt.jdk8.stream.MySetCollector.lambda$accumulator$1(MySetCollector.java:24)
	at java.util.stream.ReferencePipeline.lambda$collect$1(ReferencePipeline.java:496)
	at java.util.stream.ForEachOps$ForEachOp$OfRef.accept(ForEachOps.java:184)
	at java.util.Spliterators$ArraySpliterator.forEachRemaining(Spliterators.java:948)
	at java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:481)
	at java.util.stream.ForEachOps$ForEachTask.compute(ForEachOps.java:291)
	at java.util.concurrent.CountedCompleter.exec(CountedCompleter.java:731)
	at java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java:289)
	at java.util.concurrent.ForkJoinPool$WorkQueue.runTask(ForkJoinPool.java:1056)
	at java.util.concurrent.ForkJoinPool.runWorker(ForkJoinPool.java:1692)
	at java.util.concurrent.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:157)
```

