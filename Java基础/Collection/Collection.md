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
+ 
+ HashMap:
+ TreeMap:
+ 



