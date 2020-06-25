### Queue
![Queue类图](http://img-blog.csdn.net/20150821100053315)
队列继承集合接口
```
public interface Queue<E> extends Collection<E> {
```


```
//向队列添加元素，如果是有界队列，超出队列最大数量抛出异常
boolean add(E e);

//向队列添加元素，如果是有界队列，超出队列最大数量则添加失败，返回false
 boolean offer(E e);
 
 //移除队列元素，如果队列为空抛出异常
 E remove(); 
 
 //移除队列元素，如果队列为空返回null
 E poll();
 
 //返回队列头元素，如果队列为空抛出异常
 E element();
 
  //返回队列头元素，如果队列为空返回null
  E peek();
  
```

### AbstractQueue
抽象队列类,集成抽象AbstractCollention类，实现队列接口Queue

```
public abstract class AbstractQueue<E>
    extends AbstractCollection<E>
    implements Queue<E> {
    }
```


```
protected AbstractQueue() {
    }
    
    
    //add方法调用offer方法，返回false则抛出异常
    public boolean add(E e) {
        if (offer(e))
            return true;
        else
            throw new IllegalStateException("Queue full");
    }
    
    //remove调用poll方法，返回null抛异常
    public E remove() {
        E x = poll();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }
    
    //element方法调用peek方法，返回null则抛异常
    public E element() {
        E x = peek();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }
    
    //clear方法，循环调用poll方法
    public void clear() {
        while (poll() != null)
            ;
    }
    
    //addAll方法循环调用add方法
    public boolean addAll(Collection<? extends E> c) {
        if (c == null)
            throw new NullPointerException();
        if (c == this)
            throw new IllegalArgumentException();
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
    
```


### BlokingQueue
java.util.concurrent 并发包下阻塞队列

```
public interface BlockingQueue<E> extends Queue<E> {
}
    //向队列添加元素，超出队列长度限制抛出异常
   boolean add(E e);

    //向队列添加元素，如果违反容量限制则返回false
      boolean offer(E e);

    //向队列添加元素，如果队列已满，等待可用后插入
     void put(E e) throws InterruptedException;

    //在一定等待时间内插入元素，如果队列已满等待一段时间若无可用空间则返回false
      boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException;

    //在一定时间内弹出元素，如果队列为空，等待一段时间若无可用元素则返回null
    E poll(long timeout, TimeUnit unit)
        throws InterruptedException;

    //返回队列剩容量，如果是无界队列则返回Integer.MAX_VALUE
     int remainingCapacity();

 boolean remove(Object o);

```

### Deque
无界队列，继承Queue，双端队列，提供对队列头和队列尾部元素进行操作方法

```
public interface Deque<E> extends Queue<E> {
    
    //向队列头添加元素
    void addFirst(E e);

    //向队列尾添加元素
    void addLast(E e);
    
    //相对列头部添加元素，如果超出队列容量选择则返回false
    boolean offerFirst(E e);
    
   //相对列尾部添加元素，如果超出队列容量选择则返回false
    boolean offerLast(E e);

    //移除队列头部元素，如果伪劣为空抛出异常
   E removeFirst();

    //移除队列尾部元素，如果伪劣为空抛出异常
   E removeLast();

    //弹出队列头部元素，如果队列为空则返回null
    E pollFirst();

    //弹出队列尾部元素，如果队列为空则返回null
    E pollLast();

    //获取队列头部元素，如果队列为空则抛出异常
    E getFirst();

    //获取队列尾部元素，如果队列为空则抛出异常
    E getLast();

    //获取队列头部元素，如果队列为空则返回null
    E peekFirst();

    //获取队列尾部元素，如果队列为空则返回null
    E peekLast();

    //删除指定元素第一次出现在该列表中(遍历从头部到尾部列表时)。如果列表中不包含该元素，它是不变的。
     boolean removeFirstOccurrence(Object o);

    //删除指定元素最后一次出现在该列表中(遍历从头部到尾部列表时)。如果列表中不包含该元素，它是不变的。
    boolean removeLastOccurrence(Object o);
}
```

### ArrayDeque
ArrayDeque继承了AbstractCollection抽象类，实现了Deque无界队列接口

```
public class ArrayDeque<E> extends AbstractCollection<E>
                           implements Deque<E>, Cloneable, Serializable{

//内部由数组实现，对数组进行操作
transient Object[] elements; // non-private to simplify nested class access

    //队列头索引
  transient int head;

    //队列尾索引
    /**
     * The index at which the next element would be added to the tail
     * of the deque (via addLast(E), add(E), or push(E)).
     */
    transient int tail;


/**
    *数组初始化容量，必须为2的平方
     * The minimum capacity that we'll use for a newly created deque.
     * Must be a power of 2.
     */
    private static final int MIN_INITIAL_CAPACITY = 8;
            
    //分配元素大小，Allocates empty array to hold the given number of elements.获取最近给定大小的2的平方
    private void allocateElements(int numElements) {
        int initialCapacity = MIN_INITIAL_CAPACITY;
        // Find the best power of two to hold elements.
        // Tests "<=" because arrays aren't kept full.
        if (numElements >= initialCapacity) {
            initialCapacity = numElements;
            initialCapacity |= (initialCapacity >>>  1);
            initialCapacity |= (initialCapacity >>>  2);
            initialCapacity |= (initialCapacity >>>  4);
            initialCapacity |= (initialCapacity >>>  8);
            initialCapacity |= (initialCapacity >>> 16);
            initialCapacity++;

            if (initialCapacity < 0)   // Too many elements, must back off
                initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 elements
        }
        elements = new Object[initialCapacity];
    }        
            
    //扩容，2倍容量。数据有界，Deque无界。有界实现无界，同步扩容来实现
     private void doubleCapacity() {
        assert head == tail;
        int p = head;
        int n = elements.length;
        int r = n - p; // number of elements to the right of p
        int newCapacity = n << 1;
        if (newCapacity < 0)
            throw new IllegalStateException("Sorry, deque too big");
        Object[] a = new Object[newCapacity];
        System.arraycopy(elements, p, a, 0, r);
        System.arraycopy(elements, 0, a, r, p);
        elements = a;
        head = 0;
        tail = n;
    }
    
    
    //拷贝数组元素，通过底层调用System.arraycopy实现
     private <T> T[] copyElements(T[] a) {
        if (head < tail) {
            System.arraycopy(elements, head, a, 0, size());
        } else if (head > tail) {
            int headPortionLen = elements.length - head;
            System.arraycopy(elements, head, a, 0, headPortionLen);
            System.arraycopy(elements, 0, a, headPortionLen, tail);
        }
        return a;
    }
                               
}

```

##  7个阻塞队列BlockingQueue
- ArrayBlockingQueue
- DelayQueue
- DelayedWorkQueue
- LinkedBlockingDeque
- LinkedTransferQueue
- PriorityBlockingQueue
- SynchronousQueue


### ArrayBlockingQueue
ArrayBlockingQueue 继承抽象队列，实现阻塞队列
```
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {
            
            /**
     * Serialization ID. This class relies on default serialization
     * even for the items array, which is default-serialized, even if
     * it is empty. Otherwise it could not be declared final, which is
     * necessary here.
     */
    private static final long serialVersionUID = -817911632652898426L;

    /** The queued items 队列元素，数组集合*/
    final Object[] items;

    /** items index for next take, poll, peek or remove */
    int takeIndex;

    /** items index for next put, offer, or add */
    int putIndex;

    /** Number of elements in the queue */
    int count;

    /*
     * Concurrency control uses the classic two-condition algorithm
     * found in any textbook.
     */

    /** Main lock guarding all access */
    final ReentrantLock lock;

    /** Condition for waiting takes */
    private final Condition notEmpty;

    /** Condition for waiting puts */
    private final Condition notFull;

    /**
     * Shared state for currently active iterators, or null if there
     * are known not to be any.  Allows queue operations to update
     * iterator state.
     */
    transient Itrs itrs = null;
            
            
            
        }
```


