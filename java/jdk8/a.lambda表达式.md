##### lambda表达式

lambda表达式概要：

1. java Lambda表达式是一种匿名函数；它是没有声明的方法，即没有访问修饰符、返回值和名字



lambda表达式作用：

1. 传递行为，而不仅仅是值
2. 提升抽象层次
3. API重用性好
4. 更加灵活



lambda表达式基本结构：

```java
（param1，param2,param3） -> {

}
```

代码示例：

```java
public class SwingTest {
    public static void main(String[] args) {
        JFrame jFrame = new JFrame("MyJFrame");
        JButton jButton = new JButton("MyJButton");

        //匿名内部类
        jButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("点击按钮。。。。。");
            }
        });

        //lambda表达式
        jButton.addActionListener(event -> {
            System.out.println("点击按钮。。。。。");
        });

        jFrame.add(jButton);
        jFrame.pack();
        jFrame.setVisible(true);
        jFrame.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
    }
}
```



关于函数式接口： `@FunctionalInterface`

1. 如果一个接口只有一个抽象方法（复写Object类中的方法不算），那么该接口就是一个函数式接口。
2. 如果我们在某个接口上声明了`@FunctionalInterface`注解，那么编译器就会按照函数式接口的定义来要求该接口。
3. 如果某个接口只有一个抽象方法，但我们并没有给该接口声明`@FunctionalInterface`注解，那么编译器依旧会将该接口看作是函数式接口。



函数式接口：

```java
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

使用函数式接口：

```java
public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8);
        //forEach（）方法，Iterable的default方法
        //内部类实现函数式接口
        list.forEach(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) {
                System.out.println(integer);
            }
        });
        System.out.println("------------------");
        //lambda表达式
        list.forEach(value -> System.out.println(value));
    }
```



Lambda表达式作用：

1. lambda表达式为java添加了缺失的函数式编程特性，使我们能将函数当做一等公民看待。
2. 在将函数作为一等公民的语言中，lambda表达式的类型是函数。但在java中，lambda表达式是对象，他们必须依附于一类特别的对象类型--------函数式接口。
3. lambda表达式的对象类型由上下文推断。

示例：

```java
interface MyInterface{

    void myMethod();

}

public class Test2 {

    public static void main(String[] args) {
        //lambda表达式是对象
        MyInterface myInterface = () -> System.out.println("hello lambda");
        System.out.println(myInterface.getClass());
        System.out.println(myInterface.getClass().getSuperclass());
        System.out.println(myInterface.getClass().getInterfaces()[0]);
    }
}
```



Lambda表达式与流

```java
public static void main(String[] args) {
        List<String> list = Arrays.asList("hello","world","hello world");

        //lambda表达式
        List<String> list2 = new ArrayList<>();
        list.forEach(value -> list2.add(value.toUpperCase()));
        list2.forEach(value -> System.out.println(value));

        //流 + lambda
        list.stream().map(value -> value.toUpperCase()).forEach(value -> System.out.println(value));
        //流 + 方法引用
        list.stream().map(String::toUpperCase).forEach(value -> System.out.println(value));
        //方法引用 类+普通方法，第一个参数是调用的对象
        Function<String,String> function = String::toUpperCase;
        System.out.println(function.getClass().getInterfaces()[0]);
    }
```



lambda表达式不会开辟新的作用域

```java
public class LambdaTest {
    //this表示LambdaTest类对象cn.synway.LambdaTest@332ba300
    Runnable r1 = () -> System.out.println(this);

    //this匿名内部类对象cn.synway.LambdaTest$1@20917dd4
    Runnable r2 = new Runnable() {
        @Override
        public void run() {
            System.out.println(this);
        }
    };

    public static void main(String[] args) {
        LambdaTest test = new LambdaTest();

        new Thread(test.r1).start();
        new Thread(test.r2).start();
    }
}
```

