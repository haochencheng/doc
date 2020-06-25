### Set接口
继承了集合接口
不允许重复的接口，只允许一个NULL


####    SortedSet接口
SortedSet接口继承了Set接口

```
Comparator<? super E> comparator();

    //返回2个元素中间的字集合
    SortedSet<E> subSet(E fromElement, E toElement);
     
    //返回小于此元素的子集合 
    SortedSet<E> headSet(E toElement);
    
    //返回大于此元素的子集合
    SortedSet<E> tailSet(E fromElement);

    //返回第一个集合
    E first();

    //返回最后一个集合
    /**
     * Returns the last (highest) element currently in this set.
     *
     * @return the last (highest) element currently in this set
     * @throws NoSuchElementException if this set is empty
     */
    E last();
    

```

####    HashSet
HashSet 集成抽象AbstractSet接口，通过hashMap实现Set接口

```
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable{
        
    }
    
    private transient HashMap<E,Object> map;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();

    
```


####    LinkedHashSet
继承hashSet，通过hashSet中的构造器，初始化HashMap为LinkedHashMap实现，使用LinkedHashMap实现


```
public LinkedHashSet(int initialCapacity) {
        super(initialCapacity, .75f, true);
    }
```

hashset中构造方法

```
//hashset 中
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
```

### CopyOnWriteArraySet
继承抽象set类，通过CopyOnWriteArrayList实现

```
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
        implements java.io.Serializable {
}        
```

```
private final CopyOnWriteArrayList<E> al;

    /**
     * Creates an empty set.
     */
    public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }
```




