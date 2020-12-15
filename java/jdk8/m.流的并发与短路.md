

##### 并发流 parallelStream()

```java
public static void main(String[] args) {
        List<String> list = new ArrayList<>(5000000);
        for (int i = 0; i < 5000000 ;i++){
            list.add(UUID.randomUUID().toString());
        }
        System.out.println("开始排序");
        long startTime = System.nanoTime();
        //串行流 2048
        //list.stream().sorted().count();
        //并行流 689
        list.parallelStream().sorted().count();
        long endTime = System.nanoTime();
        long millis = TimeUnit.NANOSECONDS.toMillis(endTime - startTime);
        System.out.println("排序时间" + millis);
    }
```

##### 流的短路

```java
public static void main(String[] args) {
        List<String> list = Arrays.asList("hello","world","hello world");
        //流的短路，符合条件退出
        list.stream().filter(item -> {
            System.out.println("循环的元素:" + item);
            return item.length() == 5;
        }).findFirst().ifPresent(System.out::println);
    }
    
循环的元素:hello
hello
```

##### 分组和分区

```java
public class StreamTest13 {
    public static void main(String[] args) {
        Student student1 = new Student("zhangsan" ,100);
        Student student2 = new Student("lisi" ,90);
        Student student3 = new Student("wangwu" ,90);
        Student student4 = new Student("zhangsan" ,80);
        List<Student> students = Arrays.asList(student1,student2,student3,student4);
        //根据姓名分组
        Map<String, List<Student>> map1 = 		           students.stream().collect(Collectors.groupingBy(Student::getName));
        System.out.println(map1);
        System.out.println("---------------------");
        //根据姓名分组,计算每个分组个数
        Map<String, Long> map2 = students.stream().collect(Collectors.groupingBy(Student::getName, Collectors.counting()));
        System.out.println(map2);
        System.out.println("---------------------");
        //根据姓名分组,计算每个分组分数平均数
        Map<String, Double> map3 = students.stream().collect(Collectors.groupingBy(Student::getName, Collectors.averagingDouble(Student::getScore)));
        System.out.println(map3);
        System.out.println("---------------------");
        //分区是特殊的分组，只能有两个分区，true和false
        Map<Boolean, List<Student>> map4 = students.stream().collect(Collectors.partitioningBy(student -> student.getScore() >= 90));
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





{lisi=[Student{name='lisi', score=90}], zhangsan=[Student{name='zhangsan', score=100}, Student{name='zhangsan', score=80}], wangwu=[Student{name='wangwu', score=90}]}
---------------------
{lisi=1, zhangsan=2, wangwu=1}
---------------------
{lisi=90.0, zhangsan=90.0, wangwu=90.0}
---------------------
{false=[Student{name='zhangsan', score=80}], true=[Student{name='zhangsan', score=100}, Student{name='lisi', score=90}, Student{name='wangwu', score=90}]}
```



