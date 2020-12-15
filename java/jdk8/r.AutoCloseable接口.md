实现了AutoCloseable接口的类使用try-with-resources代码块时，close方法会自动被调用

```java
public class AutoCloseableTest implements AutoCloseable {
    @Override
    public void close() throws Exception {
        System.out.println("close invoked!");
    }

    public void doSomething(){
        System.out.println("doSomething invoked!");
    }

    public static void main(String[] args) throws Exception {
        try (AutoCloseableTest autoCloseableTest = new AutoCloseableTest()){
            autoCloseableTest.doSomething();
        }
    }
}
```

