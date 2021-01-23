#### ArrayList

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
	private static final int DEFAULT_CAPACITY = 10;
    
    transient Object[] elementData;
    
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

}
```



1. ArrayList底层为Object数组，为空的构造参数创建一个Object类型长度为0的数组，调用添加方法初始化数组大小为10。
2. 在调用add方法时，会进行数组容量检查，扩容机制是每次增加0.5倍（如果有小数，向下取整），也就是扩充到原来的1.5倍，扩容时要对数组进行拷贝。
3. ArrayList是线程不安全的集合框架。
4. 中间插入和删除元素涉及到数组拷贝，效率较低。
5. ArrayList适合使用for循环遍历，不适合迭代器。



#### LinkedList

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;

    transient Node<E> first;

    transient Node<E> last;
    
    public LinkedList() {
    }
    
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
}
```



1. LinkedList使用双向链表数据结构，通过内部类Node存储数据。
2. 添加和删除效率高，随机访问涉及遍历，效率较低。
3. ArrayList是线程不安全的集合框架。
4. ArrayList适合使用迭代器遍历，不适合for循环。



#### Vector

```java
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    public Vector() {
        this(10);
    }
    
    public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }
    
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
}
```



1. Vector的所有方法有synchronized修饰，是线程同步的，安全性高，效率较低。
2. Vector的无参构造方法会创建一个初始容量为10的Object类型数组。
3. 在调用add方法时，会进行数组容量检查，扩容机制是每次增加1倍，也就是扩充到原来的2倍，扩容时要对数组进行拷贝。