概念：

1. Collection提供了新的stream（）方发
2. 流不储存值，通过管道的方式获取值
3. 本质是函数式的，对流的操作会生成一个结果，不过并不会修改底层的数据源，集合可以作为流的底层数据源
4. 延迟查找，很多流操作（过滤、映射、排序等）都可以延迟实现

流由3部分构成：

1. 源
2. 零或多个中间操作
3. 终止操作

流操作的分类：

1. 惰性求值（对应中间操作）
2. 及早求值（对应终止操作）

流与集合

1. 集合关注的是数据与数据存储本身;
2. 流关注的则是对数据的计算。
3. 流与迭代器类似的一点是:流是无法重复使用和消费的。

流的创建：

```java
public static void main(String[] args) {
        //Stream.of()方发
        Stream<String> stream1 = Stream.of("hello","world","nihao");
        //Stream.of()方发
        String[] array = new String[]{"hello","world","nihao"};
        Stream<String> stream2 = Stream.of(array);
        //Arrays.stream(array);
        Stream<String> stream3 = Arrays.stream(array);
        //集合创建list.stream();
        List<String> list = Arrays.asList(array);
        Stream<String> stream4 = list.stream();
    }
```

简单示例：

```java
public static void main(String[] args) {
        //用数组构建IntStream并打印
        IntStream.of(new int[]{5,6,7}).forEach(System.out::println);
        System.out.println("-----------------------");
        //构造值为3-7，不包含8的IntStream
        IntStream.range(3,8).forEach(System.out::println);
        IntStream.rangeClosed(3,7).forEach(System.out::println);
        System.out.println("-----------------------");
    
    	List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8);
        System.out.println(list.stream().map(i -> 2 * i ).reduce(0,Integer::sum));
    }
```

stream转为array和list

```java
public static void main(String[] args) {
        Stream<String> stream = Stream.of("hello","world","hello world");
        //stream转为数组
        String[] strings = stream.toArray(length -> new String[length]);
        String[] strings1 = stream.toArray(String[]::new);
        //stream转为list
        List<String> collect = stream.collect(Collectors.toList());
        //lambda表达式转换
        List<String> collect1 = stream.collect(() -> new ArrayList<>(),(list1,item) -> list1.add(item),
                (theList1,theList2) -> theList1.addAll(theList2));
        //方法引用转换
        List<String> collect2 = stream.collect(LinkedList::new,LinkedList::add,LinkedList::addAll);

    }
```

转换array接受参数IntFunction<A[]> generator，实现类最终调用如下方发，将length作为参数传入，执行generator.apply(c.size()）

```java
public T[] asArray(IntFunction<T[]> generator) {
     return c.toArray(generator.apply(c.size()));
}
```

lambda表达式调用方法

```java
    <R> R collect(Supplier<R> supplier,
                  BiConsumer<R, ? super T> accumulator,
                  BiConsumer<R, R> combiner);
```

stream.collect(Collectors.toList())是对上述collect方法一种默认实现，以下是Collectors.toList()方法

```java
public static <T>
    Collector<T, ?, List<T>> toList() {
        return new CollectorImpl<>((Supplier<List<T>>) ArrayList::new, List::add,
                                   (left, right) -> { left.addAll(right); return left; },
                                   CH_ID);
    }
```





~~~java
public static void main(String[] args) { 
        //按空格分隔单词，去重 
        //flatMap（）接受一个参数，返回一个流,tiem -> Stream.of(tiem) 
        List<String> list = Arrays.asList("hello welcome", 
                    "hello world","world hello","hello world hello"); 
        List<String> collect = list.stream().map(item -> item.split(" ")). 
            flatMap(Arrays::stream).distinct().collect(Collectors.toList()); 
        System.out.println(collect); 
    } 
 
[hello, welcome, world]

~~~



~~~java
public static void main(String[] args) { 
        List<String> list1 = Arrays.asList("Hi","Hello","你好"); 
        List<String> list2 = Arrays.asList("zhangsan","lisi","wangwu","zhaoliu"); 
        //对每个人以不同的方式打招呼 
        List<String> result = list1.stream().flatMap(item ->  
             list2.stream().map(item2 -> item + " " + item2)).collect(Collectors.toList()); 
        System.out.println(result); 
    } 
 
[Hi zhangsan, Hi lisi, Hi wangwu, Hi zhaoliu,  
Hello zhangsan, Hello lisi, Hello wangwu, Hello zhaoliu, 
你好 zhangsan, 你好 lisi, 你好 wangwu, 你好 zhaoliu]

~~~



