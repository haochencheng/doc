####   java1.8 HashMap新加方法

```
default V getOrDefault(Object key, V defaultValue)
```

 存在返回对应key的值，否则返回默认值。  
 使用场景，map用作统计计数的时候(key，count) 
 循环数组，判断key是否存在，不存在count设为1,存在则put(key,count+1)       
 
 源代码：
 
```
int[] nums=new int[]{1,1,2,3,4,4,5,5,5,5};         
    Map<Integer,Integer> map1=new        HashMap<>();            
    for (int j = 0; j < length1; j++) {     
        int v=map1.getOrDefault(nums[j],1 );
        map.put(key,v+1);
    }
```



HashMap内部是 Entry数组（键值对数组）      

```
transient Node<K,V>[] table;
```

Node<K,V>实现了Map.Entry<K,V>接口

```
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
}
```

#####   默认容量

```
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

#####   最大容量

```
static final int MAXIMUM_CAPACITY = 1 << 30;
```

#####   默认加载因子

```
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

#####   树化阈值

```
static final int TREEIFY_THRESHOLD = 8;
```
#####   树到链表阈值

```
static final int UNTREEIFY_THRESHOLD = 6;
```
#####   树最小容量

```
static final int MIN_TREEIFY_CAPACITY = 64;
```
#####   key的hash算法
null的hash为0，所以HashMap的key可以为null

```
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
异或的操作如下：0 ^ 0=0，1 ^ 1 =0，0 ^ 1=1，1 ^ 0=1，也就是相同时返回0，不同时返回1。

#####    entrySet

```
/**
     * Holds cached entrySet(). Note that AbstractMap fields are used
     * for keySet() and values().
     */
    transient Set<Map.Entry<K,V>> entrySet;
```
#####    HashMap 元素数量

```
/**
     * The number of key-value mappings contained in this map.
     */
    transient int size;
```
#####    修改数量 

```
transient int modCount;
```
#####   原理图
![image](https://javadoop.com/blogimages/map/4.png)

？会达到这种状态吗，会不会扩容

#####   put方法
1.如果table数组为空则初始化table

```
if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
```

2.如果对应hash位置节点为null，则赋值

```
if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
```
3.如果对应hash位置节点不为null，并且节点hash和传入hash相等则替换新值

```
if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
```
4.如果对应hash位置节点为TreeNode，则将值放入树中
```
     else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
```
5.如果对应hash位置节点为链表，遍历将值插入链表尾部，然后判断链表大小大于树化阈值，则树化。

```
for (int binCount = 0; ; ++binCount) {
    if ((e = p.next) == null) {
        p.next = newNode(hash, key, value, null);
        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
            treeifyBin(tab, hash);
        break;
    }
    if (e.hash == hash &&
        ((k = e.key) == key || (key != null && key.equals(k))))
        break;
    p = e;
}
```

####    get方法
同伙hash(key) key的hash和key 获取对应节点内容
```
public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
```

1.先检查tab[-1&hash]是否有值
2.检查第一个节点是否是所找的key
3.判断下一个节点是否不为null，如果是null返回   
    3.1如果节点是树，在树种查找     
    3.2如果是链表循环链表       
```
final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

###     Map类图
![image](http://img.my.csdn.net/uploads/201108/28/0_1314521559nHPq.gif)

####    LinkedhashMap
继承了HashMap，实现了Map接口

```
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
{
}
```

Entry增加了前一个几点和后一个节点的引用
```
static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

通过修改afterNodeAccess方法维护双向链表，插入顺序

```
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

####    WeakHashMap
WeekHashMap 的这个特点特别适用于需要缓存的场景。在缓存场景下，由于内存是有限的，不能缓存所有对象；对象缓存命中可以提高系统效率，但缓存MISS也不会造成错误，因为可以通过计算重新得到

弱引用（WeakReference）。我们都知道Java中内存是通过GC自动管理的，GC会在程序运行过程中自动判断哪些对象是可以被回收的，并在合适的时机进行内存释放。GC判断某个对象是否可被回收的依据是，是否有有效的引用指向该对象。如果没有有效引用指向该对象（基本意味着不存在访问该对象的方式），那么该对象就是可回收的。这里的“有效引用”并不包括弱引用。也就是说，虽然弱引用可以用来访问对象，但进行垃圾回收时弱引用并不会被考虑在内，仅有弱引用指向的对象仍然会被GC回收。

####    ConcurrentHashMap

#####   put方法
```
final V putVal(K key, V value, boolean onlyIfAbsent) {
}
```
######  1.计算key的hash

```
if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
```
######  2.hash方法

```
static final int spread(int h) {
        return (h ^ (h >>> 16)) & HASH_BITS;
    }
```
######  3.自旋

```
for (Node<K,V>[] tab = table;;) {
}
```
######  4.如果table为空，初始化

```
Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
```

######  4.如果table对应hash位置上为null，cas设置值，失败进入轮训。无锁
```
else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
    if (casTabAt(tab, i, null,
                 new Node<K,V>(hash, key, value, null)))
        break;                   // no lock when adding to empty bin
}
```
 
```
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}

static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
```
##### 5.如果在扩容，当前线程帮助扩容

```
else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
```
##### 6.同步segment 分段锁 进行赋值操作

```
synchronized (f) {
    if (tabAt(tab, i) == f) {
        if (fh >= 0) {
            binCount = 1;
            for (Node<K,V> e = f;; ++binCount) {
                K ek;
                if (e.hash == hash &&
                    ((ek = e.key) == key ||
                     (ek != null && key.equals(ek)))) {
                    oldVal = e.val;
                    if (!onlyIfAbsent)
                        e.val = value;
                    break;
                }
                Node<K,V> pred = e;
                if ((e = e.next) == null) {
                    pred.next = new Node<K,V>(hash, key,
                                              value, null);
                    break;
                }
            }
        }
        else if (f instanceof TreeBin) {
            Node<K,V> p;
            binCount = 2;
            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                           value)) != null) {
                oldVal = p.val;
                if (!onlyIfAbsent)
                    p.val = value;
            }
        }
    }
}
```














    
    