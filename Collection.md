# Collection

标签： Java

---

## 概览

![Java Collection简略版](https://c4.staticflickr.com/6/5323/29981422043_bb2787fded_o.jpg)

![Java Collection详细版](https://c2.staticflickr.com/6/5493/30315303640_245e321a36_o.png)

![经常用的Colletions](http://zeroturnaround.com/wp-content/uploads/2016/04/Java-Collections-cheat-sheet.png)

![所有的Collection的比较](https://c2.staticflickr.com/6/5572/30875373285_bbe1f3228e_o.png)

## 常用集合对比

## List

排列有序的元素

- ArrayList
    + 非线程安全
    + 增删操作（除了末端）需要移动剩下的元素，所以慢    
    + 随机访问比LinkedList要快
    + ArrayList不需要申明大小，但是有个初始容量，最好声明大些。当ArrayList当initial capacity满了的时候，会copy数组，增加50%的大小
    + 适合随机访问多，增删操作不多的场景
- LinkedList
    + 非线程安全
    + 增删操作比ArrayList要快
    + 随机访问比较慢
    + 适合随机访问少，增删操作频繁的场景
- Vector
    + 同ArrayList功能相同
    + 线程安全，慢
    + 已经过时，多线程环境下可以用CopyOnWriteArrayList完全替代
- Stack
    + 继承自Vector，后进先出的栈
    + 线程安全
    + 已过时，可以使用Deque接口和实现ArrayDeque代替
- CopyOnWriteArrayList
    + COW原理，list的实现每一次更新都会产生一个新的隐含数组副本，所以这个操作成本很高
    + 线程安全
    + 用在只遍历几乎不更新的场景

## Queue和Deque

队列

- PriorityQueue
    + 基于优先级的队列，装入队列的元素需要实现Comparable的`  compareTo()`方法
    + `poll`/`peek`/`remove`会返回一个队列的最小值
    + 非线程安全
- ArrayDeque
    + 继承自Deque接口，Deque是基于有首尾指针的数组实现的
    + 只能取出首尾元素
    + 可以实现栈的功能
    + 非线程安全
- ArrayBlockingQueue
    + 一个基于数组结构的有界阻塞队列，此队列按FIFO(先进先出)原则对元素进行排序
    + 线程安全
- LinkedBlockingQueue
    + 一个基于链表结构的阻塞队列，此队列按FIFO(先进先出)排序元素，可以定义有界的或无界
    + 线程安全
- PriorityBlockingQueue
    + 根据优先权取出元素，放入其中的元素需要实现Comparable接口
    + 线程安全
- SynchronousQueue
    + 同步队列，来一个元素就及时处理一个
    + 线程安全
- LinkedBlockingDeque
    + 队列两头都可以加入也可以取出元素的队列
    + 线程安全
- DelayQueue
    + 其中的元素过期最多的首先被取出（头部），元素需要实现Delayed接口，如果没有元素过期，则poll返回null
    + 线程安全
        
### Map

- HashMap
    + 通用的Map实现，快速
    + 可以接受null键和null值
    + 非线程安全
- Hashtable
    + 不可以接受null键和null值
    + 线程安全，Hashtable存取时锁定整个Map，速度慢
    + 已经过时，多线程环境下可以完全用ConcurrentHashMap代替
- ConcurrentHashMap
    + 可以接受null键和null值
    + 线程安全，多线程下HashMap的完美替代
    + 性能优于Hashtable，采取分段锁技术
- TreeMap
    + 不接受null键和null值
    + 树结构，键值有序，键要实现Comparable的compareTo()方法
    + 非线程安全
- LinkedHashMap
    + 可以接受null键和null值
    + 非线程安全
    + 保存了插入时的顺序
- ConcurrentSkipListMap
    + 基于跳跃列表（Skip List）的ConcurrentNavigableMap实现
    + 键值有序
    + 线程安全，在多线程环境下可以代替TreeMap

## Set

唯一的元素

- HashSet
    + 默认的Set实现，快速
    + 内部实现使用HashMap
    + 无序
    + 非线程安全
- LinkedHashSet
    + 保存插入的顺序
    + 非线程安全
- TreeSet
    + 树结构，有序，放入其中的对象要实现Comparable的`  compareTo()`方法
    + 非线程安全
- CopyOnWriteArraySet
    + 使用CopyOnWriteArrayList来存储的线程安全的Set
    + 线程安全
    + 用在只遍历几乎不更新的场景
- ConcurrentSkipListSet
    + 使用 ConcurrentSkipListMap来存储的线程安全的Set
    + 有序
    + 线程安全

参考：[集合总览英文版](http://java-performance.info/java-collections-overview/)|[集合总览中文版](http://www.kawabangga.com/posts/926)

## Fail-fast iterator和Fail-safe iterator
- Fail-fast iterator
多线程环境下，如果一个线程已经在Iterator迭代中了，那么如果另一个线程要增删时，就会抛出ConcurrentModificationException。
不管是多线程还是单线程，如果已经创建Iterator了，只能使用Iterator的add和remove方法，如果使用集合自己的add或者remove方法，则会抛出ConcurrentModificationException。
典型的例子有ArrayList，HashMap，HashSet的Iterator。
- Fail-safe iterator
多线程环境下，不会抛出ConcurrentModificationException
因为clone了原集合。
典型的例子有：ConcurrentHashMap，CopyOnWriteArrayList的Iterator

## 数据结构复杂度
http://bigocheatsheet.com/

## Map
### HashMap

HashMap有一个叫做table、默认大小是16的Entry数组，table的索引在逻辑上叫做“桶”(bucket)，它存储了链表的第一个元素。数组大小可以更改，但要是2的指数（16,32,64...）：

```java
transient Entry[] table;
```
HashMap有个内部类Entry用来存储key-value对，Entry的结构如下：

```java
static class Entry<K ,V> implements Map.Entry<K ,V> {
    final K key;
    V value;
    Entry<K ,V> next;
    final int hash;
    ...
}
```
- 每个Entry都包含Key-Value键值对
- next指向下一个Entry，所以每个bucket中装的是一个单向链表
- hash存储Key的hashCode再rehash（下面的`hash()`）计算出的hash值，这么做是为了不用每次都要计算一次key的hash值。有时候`hashcode()`函数可能写的不好，所以JDK的设计者添加了另一个叫做`hash()`的方法，它接收刚才计算的hash值作为参数。

```java
// the "rehash" function in JAVA 7 that directly takes the key
static int hash(int h) {
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}

// the "rehash" function in JAVA 8 that directly takes the key
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

> **为什么table的大小要是2的指数？**
> 因为通过indexFor得到索引，只有length是2的指数（如16），(length-1)=15，也就是**1111**，这样`h&(1111)`才能均匀的分布在16个bucket中。
> ```java
// the function that returns the index from the rehashed hash
static int indexFor(int h, int length) {
    return h & (length-1);
}
> ```

![HashMap结构](https://c7.staticflickr.com/6/5622/30651041102_e663053e09_o.jpg)

#### **put(K key, V value)的工作原理**
```java
public V put(K key, V value) {
    if (key == null)
    return putForNullKey(value);
    int hash = hash(key.hashCode());
    int i = indexFor(hash, table.length);
    for (Entry<K , V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
 
    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```

1. 对key做null检查。如果key是null，会被存储到table[0]，因为null的hash值总是0。
2. key的hashcode()方法会被调用，然后计算hash值。hash值用来找到存储Entry对象的数组的索引。
3. indexFor(hash,table.length)用来计算在table数组中存储Entry对象的精确的索引。
4. 如果在刚才计算出来的索引位置没有元素，直接把Entry对象放在那个索引上。如果索引上有元素，然后会进行迭代，依次比较链表上的元素的key是否和Entry对象的key相同，一直到Entry->next是null。如果没有找到和key相同的元素，则将Entry对象变成链表的第一个结点，将原本的第一个结点变为它的next。
如果我们找到和Entry的key相同的元素呢（我们叫做碰撞collision）？它应该替换老的value。在迭代的过程中，会调用equals()方法来检查key的相等性(key.equals(k))，如果这个方法返回true，它就会用当前Entry的value来替换之前的value。

#### **get(Object key)的工作原理**

```java
public V get(Object key) {
    if (key == null)
    return getForNullKey();
    int hash = hash(key.hashCode());
    for (Entry<K , V> e = table[indexFor(hash, table.length)]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
            return e.value;
    }
    return null;
}
```
和put类似，通过key的hashCode找到bucket，通过key的equals()方法比较，从链表中遍历，直到找到该key，也就找到value值。

#### **resize**

初始化HashMap时，可以传入参数initial capacity(初始容量)和load factor(负载因子)：

```java
public HashMap(int initialCapacity, float loadFactor);
```
默认初始容量是16，负载因子默认为0.75，也就是HashMap中如果有超过12个key，将会创建原来HashMap大小的两倍的table数组，并将原来的Entry对象重新对key的hashCode进行rehash，从而放入新的bucket中。

> 如果初始知道HashMap中不同的key的数量为X时，可以设置
> initial capacity = 刚刚好大于(X/0.75)的最接近的2的指数。
> 这样就可以省去rehash的耗时。

#### **Java 8对HashMap的改动**

```java
 static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
...
}

static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
	final int hash; // inherited from Node<K,V>
	final K key; // inherited from Node<K,V>
	V value; // inherited from Node<K,V>
	Node<K,V> next; // inherited from Node<K,V>
	Entry<K,V> before, after;// inherited from LinkedHashMap.Entry<K,V>
	TreeNode<K,V> parent;
	TreeNode<K,V> left;
	TreeNode<K,V> right;
	TreeNode<K,V> prev;
	boolean red;
...
}
```
![Java 8的HashMap](https://c2.staticflickr.com/6/5646/30871214625_df50694ca4_o.jpg)

Java 8仍然保留了Node的结构，也就是可以实现链表存储。而Node扩展成了TreeNode，TreeNode是一个红黑树的数据结构，它可以存储更多的信息，这样我们可以在O(log(n))的复杂度下添加、删除或者获取一个元素。Java 8的改动会占用更多的空间，换取更快的存取时间。

- 对于bucket中的node的数目多于8个，那么链表就会被转换成红黑树。
- 对于bucket中的node的数目小于6个，那么红黑树就会被转换成链表。

### 线程安全

HashMap是非线程安全的，多线程的时候，resize会形成条件竞争(race condition)，甚至会形成[死循环](http://mailinator.blogspot.hk/2009/06/beautiful-race-condition.html)。
所以多线程的情况下，应该用下面的方法替代：

- Collection.synchronizedMap(Map)
- Hashtable
- ConcurrentHashMap。其中ConcurrentHashMap采用的分段锁技术，性能最好。

[参考1](http://coding-geek.com/how-does-a-hashmap-work-in-java/)|[参考1中文版](http://www.importnew.com/16599.html)|[参考2](http://howtodoinjava.com/core-java/collections/how-hashmap-works-in-java/)
