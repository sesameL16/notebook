1. Collector是一个接口，作为stream.collect()方法的参数。它是一个可变的汇聚操作，将输入元素累积到一个可变的结果容器中；它会在所有元素处理完毕之后，将累积的结果转换为一个最终的表示（这是一个可选操作）;支持串行与并行两种方式执行。

```java
<R, A> R collect(Collector<? super T, A, R> collector); //收集器
```

2. Collectors本身提供了关于Collector的常见汇聚实现，Collectors本身实际是一个工厂。
3. Collector依赖4个函数得到最终结果：

```java
/**
  * T 流中每个元素的类型
  * A 中间结果类型
  * R 结果类型
  */
public interface Collector<T, A, R> {
    /**
     * A function that creates and returns a new mutable result container.
     * 创建一个结果容器
     */
    Supplier<A> supplier();

    /**
     * A function that folds a value into a mutable result container.
     * 将每个结果折叠到容器中
     */
    BiConsumer<A, T> accumulator();

    /**
     * A function that accepts two partial results and merges them.  The
     * combiner function may fold state from one argument into the other and
     * return that, or may return a new result container.
     * 合并部分结果到一个结果容器（并发使用）
     */
    BinaryOperator<A> combiner();

    /**
     * Perform the final transformation from the intermediate accumulation type
     * {@code A} to the final result type {@code R}.
     * 将中间结果转换为最终结果
     */
    Function<A, R> finisher();
}
```

4. 为了确保串行与并行操作结果的等价性，Collector函数需要满足两个条件：identity（同一性）与associativity（结合性）。

5. identity表示对于任何一个中间结果与空容器合并产生结果与合并前等价，

   即 a = combiner.apply(a, supplier.get())

6. associativity表示串行执行与并行执行再合并结果相同，注重两个结果合并结果相同 , r1 = r2

```java
A a1 = supplier.get();
accumulator.accept(a1, t1);
accumulator.accept(a1, t2);
R r1 = finisher.apply(a1);  // result without splitting

A a2 = supplier.get();
accumulator.accept(a2, t1);
A a3 = supplier.get();
accumulator.accept(a3, t2);
R r2 = finisher.apply(combiner.apply(a2, a3));  // result with splitting
```



程序示例：

```java
public class StresmTest1 {
    public static void main(String[] args) {
        Student student1 = new Student("zhangsan",80);
        Student student2 = new Student("lisi",90);
        Student student3 = new Student("wangwu",100);
        Student student4 = new Student("zhaoliu",90);
        Student student5 = new Student("zhaoliu",90);
        List<Student> students = Arrays.asList(student1,student2,student3,student4,student5);
        //转换为list
        List<Student> students1 = students.stream().collect(Collectors.toList());
        System.out.println(students1);
        System.out.println("-------------");
        //求总数
        System.out.println("count : " + students.stream().collect(Collectors.counting()));
        System.out.println("count : " + students.stream().count());
        //最小值、最大值、平均值、总数、算数集合
        students.stream().collect(Collectors.minBy(Comparator.comparingInt(Student::getScore))).ifPresent(System.out::println);
        students.stream().collect(Collectors.maxBy(Comparator.comparingInt(Student::getScore))).ifPresent(System.out::println);
        System.out.println(students.stream().collect(Collectors.averagingInt(Student::getScore)));
        System.out.println(students.stream().collect(Collectors.summingInt(Student::getScore)));
        IntSummaryStatistics summaryStatistics = students.stream().collect(Collectors.summarizingInt(Student::getScore));
        System.out.println(summaryStatistics);
        System.out.println("-------------");
        //拼接“，”分割、拼接“，”分割添加头和尾
        System.out.println(students.stream().map(student -> student.getName()).collect(Collectors.joining(",")));
        System.out.println(students.stream().map(student -> student.getName()).collect(Collectors.joining(",", "<start>","<end>")));

        //多重分组，先按分数分组，再按分数分组
        Map<Integer, Map<String, List<Student>>> map = students.stream().collect(
                Collectors.groupingBy(Student::getScore, Collectors.groupingBy(Student::getName)));
        System.out.println(map);
        //分区
        Map<Boolean, List<Student>> map1 = students.stream().collect(
                Collectors.partitioningBy(student -> student.getScore() > 80));
        System.out.println(map1);
        //多重分区，先按是否大于80分区，再按是否大于90分区
        Map<Boolean, Map<Boolean, List<Student>>> map2 = students.stream().collect(
                Collectors.partitioningBy(student -> student.getScore() > 80,
                        Collectors.partitioningBy(student -> student.getScore() > 90)));
        System.out.println(map2);
        //按大于80分区计数
        Map<Boolean, Long> map3 = students.stream().collect(
                Collectors.partitioningBy(student -> student.getScore() > 80, Collectors.counting()));
        System.out.println(map3);
        //按姓名分组，找出分数最低的学生，然后从Optional中取出
        Map<String, Student> map4 = students.stream().collect(
                Collectors.groupingBy(Student::getName,
                Collectors.collectingAndThen(
                        Collectors.minBy(Comparator.comparingInt(Student::getScore))
                        , Optional::get)));
        System.out.println(map4);
    }
}

class Student{

    private String name;

    private int score;

    public Student(String name, int score) {
        this.name = name;
        this.score = score;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getScore() {
        return score;
    }

    public void setScore(int score) {
        this.score = score;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", score=" + score +
                '}';
    }
}
```

