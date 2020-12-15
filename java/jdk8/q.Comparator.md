Comparator接口从jdk1.2提供，jdk1.8对其进行增强，添加了很多默认方法和静态方法



```java
@FunctionalInterface
public interface Comparator<T> {

    /**
     * 排序抽象方法
     */
	int compare(T o1, T o2);
    
    /**
     * 反序
     */
    default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }
    
    /**
     * 添加排序，排序之后相同的元素进行二次排序
     * thenComparing，thenComparingInt,thenComparingLong,thenComparingDubbo
     */
    default Comparator<T> thenComparing(Comparator<? super T> other) {
        Objects.requireNonNull(other);
        return (Comparator<T> & Serializable) (c1, c2) -> {
            int res = compare(c1, c2);
            return (res != 0) ? res : other.compare(c1, c2);
        };
    }

    /**
     * 静态方法，生成反序比较器
     */
    public static <T extends Comparable<? super T>> Comparator<T> reverseOrder() {
        return Collections.reverseOrder();
    }
    
    /**
     * comparing,comparingInt,comparingLong,comparingDouble
     * 生成比较器，传入Function接口
     */
    public static <T, U extends Comparable<? super U>> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor){
        .........
    }
    
    public static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor) {
        .........
    }
```



程序示例

```java
public static void main(String[] args) {
        List<String> list = Arrays.asList("nihao","hello","world","welcome");

        //按照字符串长度排序,使用Collections.sort,实际调用list.sort
        Collections.sort(list,(item1,item2) -> item1.length() - item2.length());
        Collections.sort(list, Comparator.comparingInt((String item) -> item.length()));
        Collections.sort(list, Comparator.comparingInt(String::length));

        //按照字符串长度反序排序，list.sort，反序Comparator.reversed()，default方法
        list.sort(Comparator.comparingInt((String item) -> item.length()).reversed());
        list.sort(Comparator.comparingInt(String::length).reversed());

        //多重排序,先按长度排序，长度相等按字母顺序排序
        Collections.sort(list, Comparator.comparingInt(String::length).
                         thenComparing(String.CASE_INSENSITIVE_ORDER));

    }
```

