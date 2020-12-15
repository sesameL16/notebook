###### 如下代码，是对stringStream的重复使用，stringStream.findFirst()实际返回了一个新的流

```java
Stream<String> stringStream = Stream.of("hello","world","hello world","test");
System.out.println(stringStream.filter(item -> item.length() > 2));
System.out.println(stringStream.distinct());
```

抛出异常

```java
Optional[hello]
Exception in thread "main" java.lang.IllegalStateException: stream has already been operated upon or closed
	at java.util.stream.AbstractPipeline.<init>(AbstractPipeline.java:203)
	at java.util.stream.ReferencePipeline.<init>(ReferencePipeline.java:94)
	at java.util.stream.ReferencePipeline$StatefulOp.<init>(ReferencePipeline.java:647)
	at java.util.stream.DistinctOps$1.<init>(DistinctOps.java:56)
	at java.util.stream.DistinctOps.makeRef(DistinctOps.java:55)
	at java.util.stream.ReferencePipeline.distinct(ReferencePipeline.java:384)
	at cn.synway.StreamTest6.main(StreamTest6.java:39)
```

多步操作分开正确使用(不建议使用)

```java
Stream<String> stringStream = Stream.of("hello","world","hello world","test");
Stream<String> stream1 = stringStream.filter(item -> item.length() > 2);
Stream<String> distinct = stream1.distinct();
```





###### 不调用终止操作，中间操作不会执行

```java
public static void main(String[] args) {
        List<String> list = Arrays.asList("hello","world","hello world","test");
        //运行不会有任何输出
        Stream<String> stream = list.stream().map(item -> {
            String result = item.substring(0, 1).toUpperCase() + item.substring(1);
            System.out.println("test");
            return result;
        });
        //此时才会执行打印test
        stream.forEach(System.out::println);
    }
```



###### 注意流中方法调用的顺序

```java
public static void main(String[] args) {
        //会打印出0,1但程序不会停止，产生无限流，distinct去重后元素一直不足6个，一直执行
        IntStream.iterate(0,i -> (i + 1) % 2).distinct().limit(6).
        forEach(System.out::println);
    }
```

