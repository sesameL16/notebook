方法引用：method reference

​		方法引用实际是一种lambda表达式的语法糖

​		我们可以将方法引用看作是一个【函数指针】，function pointer

```java
public static void main(String[] args) {
        List<String> list = Arrays.asList("hello","wrold","nihao");
		//lambda表达式与方法引用
        list.forEach(item -> System.out.println(item));
        list.forEach(System.out::println);
}
```

使用场景：

​		当lambda表达式实现恰好有一个方法可以代替，此时可以用方法引用

方法引用分为4类：

1. 类名：：静态方法名
2. 对象名：：实例方法名
3. 类名：：实例方法名
4. 类名：：new（构造方法引用）



方法引用示例：

```java
public class MethodReferenceTest {

    public String getString(Supplier<String> supplier){
        return supplier.get() + "test";
    }

    public String getStringStr(String str, Function<String,String> function){
        return function.apply(str);
    }

    public static void main(String[] args) {
        Student student1 = new Student("zhangsan",10);
        Student student2 = new Student("lisi",90);
        Student student3 = new Student("wangwu",50);
        Student student4 = new Student("zhaoliu",40);
        List<Student> students = Arrays.asList(student1, student2, student3, student4);

        //lambda表达式排序
        students.sort((studentParam1,studentParam2) ->
                Student.compareStudentByScore(studentParam1,studentParam2));
        students.forEach(student -> System.out.println(student.getScore()));
        System.out.println("---------------------");
        //1.类名：：静态方法名
        students.sort(Student::compareStudentByScore);
        students.forEach(student -> System.out.println(student.getScore()));

        //2.对象名：：实例方法名
        StudentComparator studentComparator = new StudentComparator();
        students.sort(studentComparator::compareStudentByScore);
        students.forEach(student -> System.out.println(student.getScore()));

        //3.类名：：实例方法名(调用者是lambda表达式的第一个参数，后续参数作为方法参数传入）
        students.sort(Student::compareByScore);
        students.forEach(student -> System.out.println(student.getScore()));

        //类名：：new（构造方法引用）(自动适配有几个参数的构造方法)
        MethodReferenceTest methodReferenceTest = new MethodReferenceTest();
        System.out.println(methodReferenceTest.getString(String::new));
        System.out.println(methodReferenceTest.getStringStr("hello",String::new));
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

    public int compareByScore(Student student){
        return this.getScore() - student.getScore();
    }

    public static int compareStudentByScore(Student student1,Student student2){
        return student1.getScore() - student2.getScore();
    }

}

class StudentComparator{

    public int compareStudentByScore(Student student1,Student student2){
        return student1.getScore() - student2.getScore();
    }
}
```

