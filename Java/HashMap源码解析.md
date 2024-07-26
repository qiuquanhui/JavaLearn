# HashMap 源码解析

HashMap 用于存储键值对 ， 基于数组+链表/红黑树，根据 hash 值来计算元素在数组的下标位置，也有一个相应的扩容机制。



##  如何计算hash值？

hashmap 一般是根据 hashcode &（n-1）来计算该值在数组中的索引位置。hashcode & （n-1）等价于 hashCode % n，因为 HashMap 的容量是 2 的幂次方。



因此 n - 1 就是二进制低位全是1，跟 hashCode 相与的话会将左边的哈希值的高位全抹掉，剩下的就是余数了，因此相当于 HashCode % n， 两者的结果是一样的，但使用 HashCode & （n - 1）是位运算，执行在硬件层，效率比直接求余要高。



## 底层数据结构

HashMap 基层数据结构是 数组 + 链表 + 红黑树

在代码中维护一个 Node 节点。

Node 节点内有 hash ，key ，value，next

```java
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

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }

```



## 构造方法

第一种：传入初始化容量与负载因子

第二种：传入初始化容量，使用默认的负载因子

第三种：未传参数，会在第一次 put 的时候进行扩容

第四种：传入map，进行复制

```java
public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and the default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

    /**
     * Constructs a new <tt>HashMap</tt> with the same mappings as the
     * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
     * default load factor (0.75) and an initial capacity sufficient to
     * hold the mappings in the specified <tt>Map</tt>.
     *
     * @param   m the map whose mappings are to be placed in this map
     * @throws  NullPointerException if the specified map is null
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
```

## put 方法（重点）

如何加入元素的，put过程：

1. 计算当前元素的 hash 值，如果该索引位置上没有元素直接添加。
2. 如果该索引位置有元素，则与该元素则通过 equal() 或 == 比较值是否相同。
3. 如果相同则不添加。
4. 如果不相同，新元素连接在数组的后面，形成链表。
5. 如果链表的长度大于等于 8 ，并且 hashMap 的容量大于等于 64,就会发生树化。



> 树化的过程，两个条件：1.hashmap 的容量要大于等于 64 并且 链表的长度要大于等于 8 ，才能实现树化。



```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

// 第四个参数 onlyIfAbsent 如果是 true，那么只有在不存在该 key 时才会进行 put 操作
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;   // 第一次 put 值的时候，如果数组为空，启动扩容机制,执行resize()方法
   
    if ((p = tab[i = (n - 1) & hash]) == null)//计算该元素在数组中的下标，如果此位置为空
        tab[i] = newNode(hash, key, value, null);//那么初始化一个Node 并加入。

    else {// 数组该位置有数据
        Node<K,V> e; K k;
        //  判断该位置的第一个数据和我们要插入的数据，key 是不是"相等"，如果是，就直接赋值。
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)        // 如果该节点是代表红黑树的节点，调用红黑树的插值方法
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 到这里，说明数组该位置上是一个链表
            for (int binCount = 0; ; ++binCount) {
                // 插入到链表的最后面
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // TREEIFY_THRESHOLD 为 8，所以，如果新插入的值是链表中的第 8 个
                    // 会触发下面的 treeifyBin，也就是将链表转换为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 如果在该链表中找到了"相等"的 key(== 或 equals)，那么就不插入，直接返回
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 此时 break，那么 e 为链表中[与要插入的新值的 key "相等"]的 node
                    break;
                p = e;
            }
        }
        // e!=null 说明存在旧值的key与要插入的key"相等"
        // 对于我们分析的put操作，下面这个 if 其实就是进行 "值覆盖"，然后返回旧值
        if (e != null) {
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 如果 HashMap 由于新插入这个值导致 size 已经超过了阈值，需要进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

## 扩容机制方法（重点）

扩容机制：HashMap 的默认容量为 16 ，负载因子为 0.75，当集合中的元素超过容量与负载因子的乘积时，会引发扩容机制。比如 16 * 0.75 = 12，当元素的数量大于等于 12 时就会发生扩容机制，每次就扩容为原来的两倍。将旧数组的元素复制到新数组中。



旧数组的元素是如何复制到新数组中的？

1. 判断就数组上的元素是数组还是链表还是红黑树。
2. 数组的话直接就计算该值的 hashcode 在新数组位置，就直接插入。
3. 链表的话，根据 hash & n 来一分为二，第一条链表插入在原位置，第二条链表插入在原位置+原容量。
4. 红黑树的话，采用树分割来处理。



```java
 final Node<K, V>[] resize() {
        Node<K, V>[] oldTab = table; // 当前使用的数组
        int oldCap = (oldTab == null) ? 0 : oldTab.length; // 当前数组容量
        int oldThr = threshold; // 当前扩容阈值
        int newCap, newThr = 0; // 新的容量和阈值，初始化新阈值为0

        // 判断旧容量
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                // 如果当前容量已达到最大容量限制，就设置阈值为最大整数值，不再扩容
                threshold = Integer.MAX_VALUE;
                return oldTab;
            } else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY) {
                // 如果新容量没有超过最大容量，且旧容量大于等于默认初始容量
                // 新容量是旧容量的两倍，新阈值也是旧阈值的两倍
                newThr = oldThr << 1;
            }
        } else if (oldThr > 0) {
            // 如果旧容量为 0 但阈值大于 0，通常是在构造函数中设置的初始容量
            newCap = oldThr; // 新容量设置为原阈值
        } else {
           // 如果旧容量和阈值都为 0，使用默认的初始容量和负载因子
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }

        // 如果新阈值在上面的条件中没有被设置，根据新容量和负载因子计算新阈值
        if (newThr == 0) {
            float ft = (float) newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ?
                    (int) ft : Integer.MAX_VALUE);
        }

        // 设置新的扩容阈值
        threshold = newThr;
        // 创建新的数组，用于存放扩容后的数据
        @SuppressWarnings({"rawtypes", "unchecked"})
        Node<K, V>[] newTab = (Node<K, V>[]) new Node[newCap];
        table = newTab;

        // 如果旧数组不为空，将旧数组中的数据迁移到新数组中
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K, V> e;
                // 遍历旧数组
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null; // 帮助垃圾收集器回收
                    // 如果桶中只有一个节点，直接计算这个节点在新数组中的位置并放置
                    if (e.next == null)
                     newTab[e.hash & (newCap - 1)] = e; //通过 hash & n - 1 求助元素的数组下标
                    else if (e instanceof TreeNode)
                        // 如果桶中结构是红黑树，则进行树分割处理
                        ((TreeNode<K, V>) e).split(this, newTab, j, oldCap);
                    else {
                        // 如果桶中结构是链表
                        Node<K, V> loHead = null, loTail = null;
                        Node<K, V> hiHead = null, hiTail = null;
                        Node<K, V> next;
                        do {
                            next = e.next;
                            // 通过 e.hash & oldCap 判断节点应该留在原位置还是移动到新的位置
                            if ((e.hash & oldCap) == 0) {
                                // 保留在原位置的节点
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            } else {
                                //  需要移动位置 
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);

                        // 将链表重新连接到新数组的桶中
                        // 第一条链表
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            // 第二条链表的新的位置是 j + oldCap，这个很好理解
                            // 注意这里加上了原数组的长度
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        // 返回新的数组
        return newTab;
    }

```

