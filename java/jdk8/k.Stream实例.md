##### Stream流转化为集合，字符串拼接

```java
public static void main(String[] args) {
        //stream转化为linkedList
        Stream<String> stream = Stream.of("hello","world","hello world");
        LinkedList<String> linkedList = stream.collect(Collectors.toCollection(LinkedList::new));
        System.out.println(linkedList);
        System.out.println(linkedList.getClass());
        System.out.println("-------------------");
        //stream转化为treeSet
        Stream<String> stream1 = Stream.of("hello","world","hello world");
        TreeSet<String> treeSet = stream1.collect(Collectors.toCollection(TreeSet::new));
        System.out.println(treeSet);
        System.out.println(treeSet.getClass());
        System.out.println("-------------------");
        //stream转化为String拼接
        Stream<String> stream2 = Stream.of("hello","world","hello world");
        String collect = stream2.collect(Collectors.joining());
        System.out.println(collect);
    }
```



##### map方法与flatmap方法

```java
    public static void main(String[] args) {
        //集合中的元素转化为大写
        Stream<String> stream = Stream.of("hello","world","hello world","test");
        List<String> list = stream.map(String::toUpperCase).collect(Collectors.toList());
        System.out.println(list);
        System.out.println("------------------");
        //集合中数字转换为平方
        List<Integer> integerList = Arrays.asList(1,2,3,4,5,6);
        List<Integer> collect = integerList.stream().map(item -> item * item).collect(Collectors.toList());
        System.out.println(collect);
        System.out.println("------------------");
        //Stream.flatMap方法，把多个list转化为一个list
        Stream<List<Integer>> listStream = Stream.of(Arrays.asList(1),
                Arrays.asList(2,3),Arrays.asList(4,5,6),Arrays.asList(7,8,9));
        List<Integer> collect1 = listStream.flatMap(theList -> theList.stream()).map(item -> item * item).collect(Collectors.toList());
        System.out.println(collect1);
        
    }
```



##### Stream.generate()和Stream.iterate()

```java
public static void main(String[] args) {
        //Stream.generate（(Supplier<T> s），接收一个Supplier，调用get方法创建流
        Stream<String> stream = Stream.generate(UUID.randomUUID()::toString);
        stream.findFirst().ifPresent(System.out::println);
        //Stream.iterate(final T seed, final UnaryOperator<T> f)
        //接收一个种子和一个函数，生成一个无限流，使用limit做个数限制
        //seed为第一个元素，f(seed)为第二个元素，f(f(seed))为第三个元素，以此类推
        Stream.iterate(1,item -> item + 2).limit(6).forEach(System.out::println);
    }
```



filter过滤出大于2的元素，mapToInt将元素值乘以2

skip跳过前两个元素，limit取出两个元素，sum求和

min()方法找出最小元素返回Optional<T>

max()方法找出最大元素返回Optional<T>

```java
Stream<Integer> integerStream = Stream.iterate(1, item -> item + 2).limit(6);
int sum = integerStream.filter(item -> item > 2).mapToInt(item -> item * 2).
    skip(2).limit(2).sum();
System.out.println(sum);
```



##### IntStream的summaryStatistics()方法返回IntSummaryStatistics对象，其中包含最大值、最小值、平均值等信息

```java
        Stream<Integer> integerStream1 = Stream.iterate(1, item -> item + 2).limit(6);
        IntSummaryStatistics intSummaryStatistics = integerStream1.mapToInt(
            item -> item * 2).summaryStatistics();
        System.out.println(intSummaryStatistics.getMax());
        System.out.println(intSummaryStatistics.getSum());
        System.out.println(intSummaryStatistics.getMin());
        System.out.println(intSummaryStatistics.getAverage());
```



