Optional可以理解为一个容器，推荐作为返回值，不作为参数

类详解：

```java
public final class Optional<T> {
    
    private static final Optional<?> EMPTY = new Optional<>();

    private final T value;

    /**
     * 默认构造方法，值为空，private
     */
    private Optional() {
        this.value = null;
    }

    /**
     * 静态工厂方法，值为空
     */
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }

    /**
     * 构造方法值不能为空
     */
    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }

    /**
     * 静态工厂方法，值不能为空
     */
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }

    /**
     * 静态工厂方法，值可以为空
     */
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }

    /**
     * 获取值，值为空抛异常
     */
    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }

    /**
     * 判断值是否为空
     */
    public boolean isPresent() {
        return value != null;
    }

    /**
     * 如果值不为空，执行Consumer行为
     */
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }

    /**
     * 值存在，且Predicate.test(calue)为true，返回当前Optional，否则返回空值Optional
     */
    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }

    /**
     * 如果值不为空，执行Function.apply(value),返回值构造新的Optional，值为空返回空Optional
     */
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }

    /**
     * 当前value执行Function.apply(value)
     */
    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
    }

    /**
     * 如果值为空，设置值为other
     */
    public T orElse(T other) {
        return value != null ? value : other;
    }

    /**
     * 如果值为空，设置值为Supplier.get()的返回值
     */
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }

    /**
     * Return the contained value, if present, otherwise throw an exception
     * to be created by the provided supplier.
     *
     * @apiNote A method reference to the exception constructor with an empty
     * argument list can be used as the supplier. For example,
     * {@code IllegalStateException::new}
     *
     * @param <X> Type of the exception to be thrown
     * @param exceptionSupplier The supplier which will return the exception to
     * be thrown
     * @return the present value
     * @throws X if there is no value present
     * @throws NullPointerException if no value is present and
     * {@code exceptionSupplier} is null
     */
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }

```

简单使用：

```java
public class OptionalTest {
    public static void main(String[] args) {
        Optional<Integer> optional = Optional.ofNullable(123);
        //常规方式
        if (optional.isPresent()){
            System.out.println(optional.get());
        }
        //推荐方式
        optional.ifPresent(item -> System.out.println(item));

        System.out.println(optional.orElse(10));
        System.out.println(optional.orElseGet(() -> 20));
        //map方法
        System.out.println(optional.map(value -> value + "hello").orElse("hello"));
    }
}
```

