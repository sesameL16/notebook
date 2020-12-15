接口定义：

```java
/**
 * BiFunction接口的特殊形式，两个参数和返回值都是同一类型
 */
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {
    /**
     * 根据传入的比较器，返回较小的值
     */
    public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
    }
    /**
     * 根据传入的比较器，返回较大的值
     */
    public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
    }
}
```

接口示例：

```java
public class BinaryOperatorTest {
    public static void main(String[] args) {
        BinaryOperatorTest test = new BinaryOperatorTest();
        //BinaryOperator.apply(a,b)方法，继承自BiFunction接口
        System.out.println(test.operator(1,2,(a,b) -> a + b));
        System.out.println(test.operator(1,2,(a,b) -> a - b));
        System.out.println("-------------------------------------");
        //BinaryOperator.minBy（）方法示例
        System.out.println(test.getShort("hello123","world",(a,b) -> a.length() - b.length()));
        System.out.println(test.getShort("hello123","world",(a,b) -> a.charAt(0) - b.charAt(0)));
    }

    public Integer operator(Integer a,Integer b,BinaryOperator<Integer> binaryOperator){
        return binaryOperator.apply(a,b);
    }

    public String getShort(String a, String b, Comparator<String> comparator){
        return BinaryOperator.minBy(comparator).apply(a,b);
    }
}
```

