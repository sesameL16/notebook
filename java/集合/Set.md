#### HashSet

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
	private transient HashMap<E,Object> map;
    
    private static final Object PRESENT = new Object();
    
    /**
     * default initial capacity (16) and load factor (0.75).
     */
    public HashSet() {
        map = new HashMap<>();
    }
    
    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }
    
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
    
    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }

}
```



1. HashSet底层实际上是一个HashMap实例，依靠HashMap的key即为HashSet的值，value为同一个Object对象。
2. HashSet是非同步的集合，线程不安全。



#### TreeSet

```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
{
    
    private static final Object PRESENT = new Object();
    
	public TreeSet() {
        this(new TreeMap<E,Object>());
    }
    
    public boolean add(E e) {
        return m.put(e, PRESENT)==null;
    }
}
```



1. TreeSet底层实际上是一个TreeMap实例，依靠TreeMap的key即为TreeSet的值，value为同一个Object对象。
2. TreeSet是非同步的集合，线程不安全。



#### LinkedHashSet

```java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
    
    public LinkedHashSet() {
        super(16, .75f, true);
    }
}
```



1. LinkedHashSet继承自HashSet，提供了4个构造方法，通过传递一个标识参数，调用父类构造方法底层构建出一个LinkedHashMap来实现，其他相关操作直接调用父类方法实现。