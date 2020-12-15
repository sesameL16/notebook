jdk8中，接口可以提供默认的方法实现，使用default关键字修饰

```java
interface MyInterface1{
    
    default void myMethod(){
        System.out.println("MyInterface1");
    }
    
}
```

如果一个类实现两个接口，两个接口中存在同样的默认方法，实现不同，编译器会报错，需要类中覆盖此方法

```java
interface MyInterface1{
    default void myMethod(){
        System.out.println("MyInterface1");
    }

}

interface MyInterface2{
    default void myMethod(){
        System.out.println("MyInterface2");
    }
}

public class MyClass implements MyInterface1 ,MyInterface2{

    @Override
    public void myMethod() {
        MyInterface1.super.myMethod();
    }

    public static void main(String[] args) {
        MyClass myClass = new MyClass();
        myClass.myMethod();
    }
}
```

如果一个类实现两个接口，两个接口中一个有默认实现，一个没有实现，编译器会报错，需要类中覆盖此方法

```java
interface MyInterface1{

    default void myMethod(){
        System.out.println("MyInterface1");
    }

}

interface MyInterface2{

    void myMethod();
}

public class MyClass implements MyInterface1 ,MyInterface2{

    @Override
    public void myMethod() {
        MyInterface1.super.myMethod();
    }

    public static void main(String[] args) {
        MyClass myClass = new MyClass();
        myClass.myMethod();
    }
}
```

如果一个类继承一个类，实现一个接口，且接口的默认方法与继承的类中有相同的方法，会默认取继承的类的方法实现

```java
interface MyInterface1{

    default void myMethod(){
        System.out.println("MyInterface1");
    }

}

class MyclassImpl{

    public void myMethod(){
        System.out.println("MyclassImpl");
    }
}

public class MyClass extends MyclassImpl implements MyInterface1{
    
    public static void main(String[] args) {
        MyClass myClass = new MyClass();
        myClass.myMethod();
    }
}
```

