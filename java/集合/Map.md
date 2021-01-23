#### MapHashMap定义

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    
    //默认容量 16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
    //最大容量 2的30次方
    static final int MAXIMUM_CAPACITY = 1 << 30;
    //默认加载因子0.75
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    //链表转换红黑树，链表节点个数>=8
    static final int TREEIFY_THRESHOLD = 8;
    //红黑树转换链表，链表节点个数<=6
    static final int UNTREEIFY_THRESHOLD = 6;
    //链表转换红黑树时，bucket < 64 会扩容
    static final int MIN_TREEIFY_CAPACITY = 64;
    
    //静态内部类
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
    }
    
    //table数组
    transient Node<K,V>[] table;
    //Map.Entry<K,V>的set集合
    transient Set<Map.Entry<K,V>> entrySet;
    //元素个数
    transient int size;
    //记录修改次数，迭代时被其他线程修改会抛出异常，快速失败
    transient int modCount;
    //capacity * load_factor
    int threshold;
}
```



1. 初始化时，不传参数默认容量为16，传入参数时会使用大于等于该值的最小2幂指数作为容量值。
2. HashMap由（数组+链表/红黑树）实现，当链表长度达到8，且数组长度大于等于64时，链表会转化为红黑树。红黑树节点数降低到6，红黑树会重新转换为链表。
3. 当容器中元素个数大于 容量 * 加载因子（capacity*loadfactor） 时，数组会进行2倍扩容（resize方法）。并重新计算旧数组中节点的储存位置。节点在新数组中的位置只有两种，原下标位置或原下标+旧数组的大小。
4. JDK1.8中，hash是通过hashCode()的高16位异或低16位实现的，保证了对象的hashCode的32位值，只要有一位发生改变，整个hash()返回值就会改变，尽可能减少碰撞。
5. 如果map中不存在新添加的数据，则执行插入，JDK1.7采用头插法，JDK1.8采用尾插法。
6. HashMap是线程不安全的，允许一个键为null，允许多个值为null。



HashMap之所以选择红黑树是为了解决二叉查找树的缺陷，二叉查找树在特殊情况下会变成一条线性结构，遍历查找会非常慢。而红黑树在插入数据后可能需要通过左旋、右旋、变色这些操作来保持平衡，引入红黑树就是为了查找数据快，解决链表查询深度的问题，我们知道红黑树属于平衡二叉树。为了平衡需要付出代价，但是该代价所消耗的资源要比遍历线性表要少，所以长度大于8的时候会使用红黑树。



根据泊松分布，在负载因子默认为0.75的时候，单个hash槽内元素个数为8的概率小于百万分之一，将7作为一个分水岭，等于7的时候不转换，大于等于8的时候进行转换，小于等于6的时候转换为链表。



##### 快速失败（fail-fast）

是java集合中的一种机制，在用迭代器遍历一个集合对象时，如果遍历过程中对集合内容进行了修改，则会抛出ConcurrentModificationException。

java.util包下的集合类都是快速失败的，不能再多线程机制下发生并发修（迭代过程被修改）改算是一种保护机制。

快速失败原理： 

1. 迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个modCount变量，
2. 迭代器被遍历期间如果内容发生变化，就会改变modCount的值。
3. 每当迭代器使用hashNext()/next()遍历下一个元素之前，都会检测modCount变量是否为expectedmodCount值，是的话就遍历，否则抛出异常，终止遍历。

Tip:这里异常的抛出条件是检测到modCount ！= expectedmodCount，如果集合发生了变化时修改modCount 值刚好又设置为expectedmodCount，则不会抛出异常，因此不能依赖于该异常是否抛出而进行并发操作的编程。





#### HashTable

```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {
    
    private transient Entry<?,?>[] table;
    
    public Hashtable() {
        this(11, 0.75f);
    }
    
    public synchronized V get(Object key) {
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                return (V)e.value;
            }
        }
        return null;
    }
    
}
```



1. HashTable是线程安全的，使用synchronized关键字枷锁，效率比不上HashMap。
2. HashTable不允许键和值为null。
3. HashTable默认初始化数组为11，扩容2倍+1。
4. HashTable直接使用对象的hashCode。

Hashtable与ConcurrentHashMap是支持并发的，如果使用null值，就会使得其无法判断其对应的key是不存在还是为空，再调用一次contain（key）进行判断，会使你此次读取到的数据不一定是最新的数据。



##### 安全失败（fail-safe）：

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。

原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所做的修改并不能被迭代器检测到，所以不会触发ConcurrentModificationException。

缺点:迭代器不能及时访问到修改后的内容

java.util.concurrent包下的容器都是安全失败的，可以在多线程下并发使用，并发修改。



#### TreeMap

```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
{
	private final Comparator<? super K> comparator;

    private transient Entry<K,V> root;
    
    public TreeMap() {
        comparator = null;
    }
    
    public TreeMap(Comparator<? super K> comparator) {
        this.comparator = comparator;
    }
}
```

1. TreeMap基于红黑树实现，没有调优选项，因为该树总处于平衡状态。
2. 根据其键的自然顺序进行排序，或者根据传入的Comparator进行排序。
3. TreeMap是线程不安全的。



#### LinkedHashMap

```java
public class LinkedHashMap<K,V>extends HashMap<K,V>
    implements Map<K,V>{
	
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
    
    transient LinkedHashMap.Entry<K,V> head;
    
    transient LinkedHashMap.Entry<K,V> tail;
    
}
```

1. 继承自HashMap，保存了插入顺序，遍历时会以插入顺序输出。
2. 构造方法调用父类构造方法，默认使用插入顺序
3. 没有重写put方法，HashMap的put方法调用了LinkedHashMap的newNode方法
4. get方法添加了判断是否为顺序访问的逻辑



#### ConcurrentHashMap

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
    
    //简化版
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        for (Node<K,V>[] tab = table;;) {
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
    
    public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        int h = spread(key.hashCode());
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
    
}
```

##### 1.8中具体实现

抛弃了原有的Segment分段锁，采用`CAS + synchronized`来保证并发安全性。

put方法：

1. 根据key计算出hashcode。
2. 判断是否需要就行初始化
3. 根据当前的key定位出Node，如果为空表示当前位置可以写入数据，利用CAS尝试写入，失败则自旋保证成功
4. 如果当前位置的 hashcode == MOVED == -1则需要进行扩容
5. 如果都不满足，则利用synchronized锁写入数据
6. 如果链表数量大于TREEIFY_THRESHOLD则要转换为红黑树

get方法：

1. 根据计算出来的hashcode寻址，如果就在桶上就直接返回值。
2. 如果是红黑树就按红黑树方式获取值
3. 如果不满足就按照链表的方式遍历获取值





#### Collections.synchronizedMap(Map)

```java
private static class SynchronizedMap<K,V>
    implements Map<K,V>, Serializable {
    
    private final Map<K,V> m;     // Backing Map
    final Object mutex;
    
    SynchronizedMap(Map<K,V> m) {
        this.m = Objects.requireNonNull(m);
        mutex = this;
    }

    SynchronizedMap(Map<K,V> m, Object mutex) {
        this.m = m;
        this.mutex = mutex;
    }
    
    public V get(Object key) {
        synchronized (mutex) {return m.get(key);}
    }

    public V put(K key, V value) {
        synchronized (mutex) {return m.put(key, value);}
    }
    public V remove(Object key) {
        synchronized (mutex) {return m.remove(key);}
    }
    
}
```

1. 内部维护了一个普通map对象和一个Object对象用来当做互斥锁对象
2. 构造方法传入一个map，其他方法使用synchronized关键字给Object对象上锁，实现线程互斥。

