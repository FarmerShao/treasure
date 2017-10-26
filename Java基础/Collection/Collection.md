# Collection
标签（空格分隔）： Java collection 

---

## Collection
### List：
+ List的实现类都是有序的
+ 插入的元素可以重复，可以插入`多个` null
+ 各实现类的差异：

| List实现类|线程安全|实现方式|特点|扩容方式
|:---|:---:|:---:|:---:|:---:|
|ArrayList|不安全|数组|查询快，有序插入快(即从末尾插入)|将原数组元素复制到一个大小为原大小1.5倍 (newSize = oldSize + oldSize >> 1) 的新数组中。
|LinkedList|不安全|链表|查询较慢，无序插入快，有序插入还是比ArrayList慢|没有ArrayList的容量大小的概念来限制元素的插入。
|Vector|安全|数组|因为所有方法都是同步的，所以查询和插入都很慢|同ArrayList类似，但是扩容后大小为默认为原来的2倍，或者可以指定增长大小(构造方法中传入增量)
|Stack(继承于Vector)|安全|数组|同Vector,查询慢，插入慢，但有堆特有的pop,peek,push操作（这3个操作也是同步的）|同Vector。

### Set:
+ Set的子类是无序的，除TreeSet外，TreeSet是以指定的Comparator排序，或者是默认的排序方式排序。
+ 插入的元素不可重复，可以插入`一个` null。
+ Set 实际上也是Map，只不过Set的value都是同一个对"PRESENT"。
```
// TreeSet 的JDK 源码
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable 
{
    private transient NavigableMap<E,Object> m;
    private static final Object PRESENT = new Object();
    public boolean add(E e) {
        return m.put(e, PRESENT)==null;
    }
}
```
| Set实现类|线程安全|如何判断元素的重复|
|:---|:---:|:---:|
|HashSet|不安全|通过计算对象的 `hashCode` 值来判断是否一致
|LinkedHashSet(HashSet的子类)|不安全|通过计算对象的 `hashCode` 值来判断是否一致
|TreeSet|不安全| 通过`Comparator` 计算元素是否重复

## Map：
+ 以kye-value的形式存储数据；元数据为Map.Entry<K,V>；
+ key不可以重复，且可以为null，但只会存一个 key == null 的键值对。
  注：TreeMap 不传入 Comparator 时，key不能为null。
+ value可以重复，且可以为null。

### HashMap:
+ 使用 Node<K,V>[] table 存储所有key-vaule的（当第一次使用HashMap时会初始化这个数组的大小 2的指数）

```
   /**
     * The table, initialized on first use, and resized as
     * necessary. When allocated, length is always a power of two.
     * (We also tolerate length zero in some operations to allow
     * bootstrapping mechanics that are currently not needed.)
     */
    transient Node<K,V>[] table;
```
+ 因为HashMap的元数据为Node, 所以在table[i] 中存放的其实是一个链表，且是单向；换句话说就是当key的hash值冲突时，HashMap把这些冲突的数据放在一个链表里。
+ put() 的流程：
    1. 判断table 是否为null，或table 的大小为0;若为0，resize table的大小 n。
    2. 判断key 的hash 值是否冲突： table[ i = (n-1) & hash(key)] == null? 若table对应位置不存在数据，将新的Entry 存入table[i = (n-1) & hash(key)] 。
    3. 若对应位置已经存在数据，判断key是否相等；若key 相等，则将新的Node 替换oldNode，并return oldNode
    4. 若key 不相等，hash又冲突，则遍历 table[i] 的链表，查看是否已经有key相等的存在，若存在，则替换链表中的oldNode，并return oldNode；若不存在，则在链表尾部插入Node
+ LinkedHashMap 是HashMap的子类，存储元数据为 TreeNode<K,V>。
    
### TreeMap:
+ 元数据为 Entry<K,V>，可以从这个类的属性中看出来，TreeMap是通过“红黑二叉树”来进行插入/查询数据的

    ```
    static final class Entry<K,V> implements Map.Entry<K,V> {
        K key;
        V value;
        Entry<K,V> left;     
        Entry<K,V> right;    
        Entry<K,V> parent;   
        boolean color = BLACK;  
        ...
    }
    
    ```
+ put 流程：
    1. 判断 root节店是否存在，若不存在，则当前节点为root节点。
    2. 若root 节点存在，判断是否有 Comparator， 若有则用 Comparator 比较 key ；若不存在，则用key的 compareTo()方法进行比较，所以当 Comparator 不存在时，key不能为null，否则会报空指针异常。
    3. 使用二叉树算法插入数据，若中间遇到key相等（即compare 返回值为0时）则直接覆盖改节点的value。

## Collections常用方法：
+ sort(List<T> list, Comparator<? extends T> c) : List排序
+ reverse(List<?> list)： List反转
+ min(Collection<?> c, Comparator<T> comp): 求集合中最小值
+ max(Collection<?> c, Comparator<T> comp): 求集合中最大值
+ Set<T> synchronizedSet(Set<T> s)：转换为同步Set
+ List<T> synchronizedList(List<T> list): 转换为同步List
+ Map<K,V> synchronizedMap(Map<K,V> m)： 转换为同步Map

## Arrays常用方法：
+ List<T> asList(T... a)： 将多个对象转换为List
+ sort(int[] a): 快速为数组排序（参数还可以是其他基本类型的数组）
+ boolean equals(int[] a, int[] a2)：判断2个数组是否相等（参数还可以是其他基本类型的数组）
+ fill(int[] a, int val)： 使用 val 填充整个 a 数组（参数还可以是其他基本类型的数组）
+ toString(int[] a)： 将数组转换为String ，（参数还可以是其他基本类型的数组\Objec[]）
+ int[] copyOfRange(int[] original, int from, int to): 复制from..to 的数组部分

## 强引用、软引用、弱引用和虚引用
+ **强引用**： 只要引用存在，垃圾回收器永远不会回收，即使内存不足，也不会回收，JVM 会抛出OutOfMemoryError错误。
例 Object o = new Object(); 只要这个 o 变量不显示的 o = null; 则在 o 变量的所处的代码块任然被执行，则 o 不会被GC所回收。
+ **软引用**： 非必须引用，内存溢出之前进行回收。
```
    Object obj = new Object();
    SoftReference<Object> sf = new SoftReference<Object>(obj);
    sf.get();  //获取obj，可能获取null，因被GC回收
```
+ **弱引用**： 第二次垃圾回收时回收，即第一次GC时，对象呗标记为弱引用，在下次GC时被回收。
```
    Object obj = new Object();
    WeakReference<Object> wf = new WeakReference<Object>(obj);
    wf.get();  //获取obj，可能获取null，因被GC回收
    wf.isEnQueued();//返回是否被垃圾回收器标记为即将回收的垃圾
```
+ **虚引用**: 虚引用如同没有引用，会再第一次GC时就被回收，且无法通过引用取到对象值。
```
    Object obj = new Object();
    PhantomReference<Object> pf = new PhantomReference<Object>(obj);
    pf.get();//永远返回null
    pf.isEnQueued();//返回是否从内存中已经删除
```

