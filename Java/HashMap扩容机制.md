# 扩容机制

触发条件：HashMap 的默认容量为 16 ，负载因子为 0.75，当集合中的元素超过容量与负载因子的乘积时，会引发扩容机制。

1. 判断旧容量是否已经最大值，是的话不再扩容

不是：

1. 先初始化一个原来两倍的数组。
2. 判断旧数组中的元素是数组还是链表还是红黑树
3. 如果桶中只有一个节点，直接计算这个节点在新数组中的位置并放置
4. 如果桶中结构是红黑树，则进行树分割处理

​			如果红黑树的值小于 6 ，就会退化成链表。

5. 如果是链表，将链表一分为二，通过e.hash & oldCap判断节点应该留在原位置还是移动到新的位置
6. 将链表重新连接到新数组的桶中，第一条在原位置，第二条在原位置 + 原容量

详细代码如下

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
            // 如果旧容量为0但阈值大于0，通常是在构造函数中设置的初始容量
            newCap = oldThr; // 新容量设置为原阈值
        } else {
            // 如果旧容量和阈值都为0，使用默认的初始容  // 如果旧容量和阈值都为0，使用默认的初始容量和负载因子
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
                     newTab[e.hash & (newCap - 1)] = e; //计算hash值通过 hash & n - 1，如果 n 为二次幂，那么hash & n -1 相当于 hash % n
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
                            // 通过e.hash & oldCap判断节点应该留在原位置还是移动到新的位置
                            if ((e.hash & oldCap) == 0) {
                                // 保留在原位置的节点
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            } else {
                                //  需要移动位置 ，调整到“原位置+旧容量”这个位置 
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

