##### cBiFunction接口

接口定义：

```java
public interface BiFunction<T, U, R> {

    /**
     * 接受两个参数返回一个结果
     *
     * @param t the first function argument
     * @param u the second function argument
     * @return the function result
     */
    R apply(T t, U u);

    /**
     * 先执行BiFunction.apply,返回结果作为参数执行Function.apply
     */
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t, U u) -> after.apply(apply(t, u));
    }
}
```

使用案例：

```java
public class BiFunctionTest {

    public static void main(String[] args) {
        BiFunctionTest test = new BiFunctionTest();
        //使用BiFunction.apply
        System.out.println(test.compute1(2,3,(value1,value2) -> value1 + value2));
        //使用BiFunction.andThen
        System.out.println(test.compute2(2,3,(value1,value2) -> value1 + value2 , value -> value * value));
    }

    public int compute1(int a, int b, BiFunction<Integer,Integer,Integer> biFunction){
        return biFunction.apply(a,b);
    }

    public int compute2(int a, int b, BiFunction<Integer,Integer,Integer> biFunction,
                        Function<Integer,Integer> function){
        return biFunction.andThen(function).apply(a,b);
    }
}
```

