1. 应用场景

    大多数的商业计算中，一般采用java.math.BigDecimal类来进行精确计算。比如：货币

2. 构建BigDecimal

    BigDecimal BigDecimal(double d); //不允许使用,精度不能保证
    BigDecimal BigDecimal(String s); //常用,推荐使用
    static BigDecimal valueOf(double d); //常用,推荐使用

3. 方法

| 方法                 | 描叙                                       |
| -------------------- | ------------------------------------------ |
| add(BigDecimal)      | BigDecimal对象中的值相加，然后返回这个对象 |
| subtract(BigDecimal) | BigDecimal对象中的值相减，然后返回这个对象 |
| multiply(BigDecimal) | BigDecimal对象中的值相乘，然后返回这个对象 |
| divide(BigDecimal)   | BigDecimal对象中的值相除，然后返回这个对象 |
| toString()           | 将BigDecimal对象的数值转换成字符串         |
| doubleValue()        | 将BigDecimal对象中的值以双精度数返回       |
| floatValue()         | 将BigDecimal对象中的值以单精度数返回       |
| longValue()          | 将BigDecimal对象中的值以长整数返回         |
| intValue()           | 将BigDecimal对象中的值以整数返回。         |

4. 问题

    BigDecimal的divide方法进行除法时当不整除，出现无限循环小数时，就会抛异常的，异常如下：java.lang.ArithmeticException: Non-terminating decimal expansion; no exact representable decimal result. at java.math.BigDecimal.divide(Unknown Source)

    解决方法：给divide设置精确的小数点