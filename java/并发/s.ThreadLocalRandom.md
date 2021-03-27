对于一个随机数生成器来说，有两个要素需要考量：
1.随机数生成器的种子。
2.具体的随机数生成算法(函数)。

对于ThreadLocalRandom来说，其随机数生成器的种子是存放在每个线程的ThreadLocal中的。



```java
public static void main(String[] args) {
    Random random = new Random();
    IntStream.range(0,10 ).forEach(i -> {
        System.out.println(random.nextInt(10));
    });

    System.out.println("=================");

    ThreadLocalRandom threadLocalRandom = ThreadLocalRandom.current();
    IntStream.range(0,10 ).forEach(i -> {
        System.out.println(threadLocalRandom.nextInt(10));
    });
}
```