e接口定义：

```java
@FunctionalInterface
public interface Supplier<T> {

    /**
     * 不接收参数，返回一个结果
     */
    T get();
}
```

示例：

```java
public class SupplierTest {
    public static void main(String[] args) {
        //简单示例
        Supplier<String> stringSupplier = () -> "hello world";
        System.out.println(stringSupplier.get());
        //lambda表达式返回Student
        Supplier<Student> supplier = () -> new Student();
        System.out.println(supplier.get());
        System.out.println("---------------------------");
        //方法引用---构造方法引用
        Supplier<Student> supplier2 = Student::new;
        System.out.println(supplier2.get());
    }
}

class Student{
    private String name = "zhangsan";
    private int age = 20;

    public Student() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

