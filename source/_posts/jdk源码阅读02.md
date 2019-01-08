---
title: jdk源码阅读02
date: 2019-01-08 21:19:00
categories: jdk
---

> jdk 源码阅读简单记录。

<!-- more -->

### java.util

#### ArrayList

注释表明 ArrayList 允许 null 值，大致功能与 Vector 相同，只是是非线程安全的。

ArrayList 内部本身维护了一个 Object 数组来完成数据的存储。

需要注意的是，ArrayList 的 clone 方法并不能复制一个新对象，有如下代码：

```
ArrayList<SomeEntity> list = new ArrayList<>();
        list.add(new SomeEntity());
        ArrayList<SomeEntity> newList = (ArrayList) list.clone();
        System.out.println(list.equals(newList));
        System.out.println(list.hashCode() == newList.hashCode());
        SomeEntity se = list.get(0);
        se.setProperty(11);
        list.set(0, se);
        System.out.println(list.get(0).getProperty());
        System.out.println(newList.get(0).getProperty());
```

程序运行后输出如下

```
true
true
11
11
```

可见 clone() 方法不能复制一个 ArrayList。

#### LInkedList

注释表明内部维护了双向链表，同时允许 null 值作为元素，非线程安全。如果想要在多线程下使用，可以使用如下代码：

```
List list = Collections.synchronizedList(new ArrayList());
...
synchronized (list) {
    Iterator i = list.iterator(); // Must be in synchronized block
    while (i.hasNext())
    foo(i.next());
}
```

#### HashMap

简单摘录注释如下

```
permits `null` values and the `null` key. The <tt>HashMap</tt> class is roughly equivalent to <tt>Hashtable</tt>, except that it is unsynchronized and permits nulls. When the number of entries in the hash table exceeds the product of the load factor and the current capacity, the hash table is <i>rehashed</i> (that is, internal data structures are rebuilt) so that the hash table has approximately twice the number of buckets.
```

其中rehash的作用是扩容，在hashmap中已经更名为 `resize()` ，而hashtable中仍旧为 `rehash()`。简单对比了 hashmap 与 hashtale 后发现，hashmap的由于hash操作更加合理，冲突处理更加好，所以现在更加推荐使用hashmap；而并发情况下使用hashtable效率也不高，通常更推荐使用 concurrentHashmap。

hashmap的原理大致如下：hashmap维护了一个数组，当向hashmap中添加元素时，首先计算key值的hash，并通过计算转换为数组中的下标，如果对应的位置为空，则放入位置；否则在对应的位置处使用链表，将元素添加到链表的尾部；当链表长度大于设定的值时，将链表转换为查找速度更快的红黑树。

对于hashmap，最常用的方法就是put() 与get() 了。对于put() 方法，源码中只是直接调用了putval方法，所以摘录putVal() 方法如下，同时添加注释：

```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //对于数组，默认的初始化方法并不会初始化数组，所以这里当数组为空时对数组初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
        //hash对应位置为空直接存入
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                //如果key、key的hash都与数组中的一直，则key相同，在后面的if处更新value
                e = p;
            else if (p instanceof TreeNode)
            //如果该节点是红黑树节点使用树插入方法
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
            //链表则添加到尾部
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //当链表长度超出设定值时转换为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //设置value
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                //预留的callback    
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        //当hashmap中元素数量超过loadfactor*capacity则扩容
        if (++size > threshold)
            resize();
        //预留的callback
        afterNodeInsertion(evict);
        return null;
    }
```

get()方法同样是调用了getNode()方法，在获取到节点后返回节点的value。以下为getNode()方法:

```
final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        //判断数组非空
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            //直接按照hash取数组对应下标的节点的key
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            //节点key不符合，则存在链表或者红黑树
            if ((e = first.next) != null) {
            	//如果是树节点使用树的查找方法，红黑树的查找比较高效
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                //如果是链表直接遍历
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

对于hashmap，影响其容量大小的有两个成员变量：loadfactor与capacity。loadfactor是扩容系数，当当前hashmap的容量达到 loadfactor * capacity时，hashmap会进行扩容，容量翻倍。

#### HashSet

通过查看源码可以知道，java的HashSet是基于HashMap实现的。HashSet内部持有一个HashMap实例map，map的key作为set的元素，map的value被设置为new Object()。

#### LinkedHashMap 

增加了双向链表结构的HashMap，本身也是基于HashMap实现的，多用于LRU。

#### Vector 

相当于可变容量的数组，同时每个方法都有synchronized修饰，支持多线程同时并发效率也比较低。

#### TreeMap

TreeMap源码使用红黑树实现，代码比较复杂，可以参考[这篇文章](https://github.com/CarpenterLee/JCFInternals/blob/master/markdown/5-TreeSet%20and%20TreeMap.md)

#### Collections

主要为Collection下的类提供一些操作方法，下面是一些日常可能会用到的：

```
public static <T> int binarySearch(List<? extends T> list, T key, Comparator<? super T> c)//list的二分查找

private static void swap(Object[] arr, int i, int j)//交换数组元素

public static <T extends Object & Comparable<? super T>> T min(Collection<? extends T> coll) //取最小值

public static <T extends Object & Comparable<? super T>> T max(Collection<? extends T> coll) //取最大值

public static <T> boolean replaceAll(List<T> list, T oldVal, T newVal)//替换值

public static int indexOfSubList(List<?> source, List<?> target)//查找相同的sublist

public static int frequency(Collection<?> c, Object o) //查找指定对象o的出现次数

public static boolean disjoint(Collection<?> c1, Collection<?> c2)//查询两个集合中是否有相同元素，有则返回false
```

#### Arrays

提供数组的一些操作方法，例如排序、查找、填充等。其中有一个方法是 `asList(T... a)` ，返回一个List对象，但是这个对象的类是Arrays的子类并不是ArrayList。