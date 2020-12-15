##### Function接口

接口定义：

```java
@FunctionalInterface
public interface Function<T, R> {

    /**
     * 接收一个参数，返回一个结果
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
    
    /**
     * Returns a composed function that first applies the {@code before}
     * function to its input, and then applies this function to the result.
     * If evaluation of either function throws an exception, it is relayed to
     * the caller of the composed function.
     * 先执行before函数，返回结果作为参数再执行此函数
     */
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    
    /**
     * Returns a composed function that first applies this function to
     * its input, and then applies the {@code after} function to the result.
     * If evaluation of either function throws an exception, it is relayed to
     * the caller of the composed function.
     * 先执行本函数，返回结果作为参数再执行after函数
     */
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
    
    /**
     * Returns a function that always returns its input argument.
     * 返回一个函数，函数实现是返回参数自己
     */
    static <T> Function<T, T> identity() {
        return t -> t;
    }
    
}
```



高阶函数：如果一个函数接收一个函数作为参数，或者返回一个函数作为返回值，那么该函数就叫做高阶函数。



apply（）方法例子：

```java
public class FunctionTest {
    public static void main(String[] args) {
        FunctionTest functionTest = new FunctionTest();
        //lambda表达式传递行为
        System.out.println(functionTest.compute(1,value -> value * 2));
        System.out.println(functionTest.compute(1,value -> value + 2));

        //先定义行为
        Function<Integer,Integer> function = value -> value + 432;
        System.out.println(functionTest.compute(32,function));
        
        Function<String,String> stringFunction = String::toUpperCase;
        System.out.println(stringFunction.getClass().getInterfaces()[0]);

    }

    public int compute(int a ,Function<Integer,Integer> function){
        return function.apply(a);
    }
}
```



compose（）方法和andThen（）方法：

```java
public class FunctionTest2 {
    public static void main(String[] args) {
        FunctionTest2 test = new FunctionTest2();
        System.out.println(test.compute(2,value -> value * 3,value -> value * value));
        System.out.println(test.compute1(2,value -> value * 3,value -> value * value));
        //结果
        //12
        //36
    }

    public int compute(int a, Function<Integer,Integer> function1,
                       Function<Integer,Integer> function2){
        return function1.compose(function2).apply(a);
    }

    public int compute1(int a, Function<Integer,Integer> function1,
                        Function<Integer,Integer> function2){
        return function1.andThen(function2).apply(a);
    }
}
```

