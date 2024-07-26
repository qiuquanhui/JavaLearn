# 什么是HashMap

HashMap 是 Java 中一种常用的基于散列的映射数据结构，它实现了 Map 接口。HashMap 可以存储键值对（key-value pairs），其中**每个键都是唯一**的，用于映射到特定的值。HashMap **允许使用 null 值和 null 键**。HashMap 它不是线程安全的。如果多个线程同时访问一个 HashMap，并且至少有一个线程从结构上修改了映射，它必须保持外部同步。



## 主要特点

1. **无序**: HashMap 不保证映射的顺序；该顺序可能随时间变化。

2. **键值对**: 可以存储键和值，其中键是唯一的。

3. **允许一个 null 键和多个 null 值**: 可以将 null 用作键和/或值。

4. **非同步**: HashMap 不是线程安全的。线程安全可以通过工具类 Collections 的 synchronizedMap 方法来包装 HashMap，或者使用CopyOnWriteHashMap 

## 基本结构

HashMap 的底层实现：数组+链表/红黑树。

- **数组（Table）**: 最核心的部分是一个数组，数组的每个槽位称为一个“桶”（Bucket），每个桶可以存储一个或多个键值对。
- **链表**: 当多个键值对的键具有相同的哈希值（散列冲突）时，这些键值对会以链表的形式存储在同一个桶中。
-  **红黑树**: 从 Java 8 开始，如果一个桶中的键值对数量超过了某个阈值（默认是 TREEIFY_THRESHOLD，值为 8），链表就会转换成红黑树，以减少搜索时间。当红黑树小于等于 6 时会转换回链表。



### 工作原理

1. **散列**: 当向 HashMap 添加一个键值对时（put 时），HashMap 会使用键的 `hashCode()` 方法计算哈希值，根据哈希值找到对应的数组索引。
2. **键值对存储**: 根据计算得到的索引，键值对被存储在对应的桶中。如果该位置已经有元素存在（发生哈希冲突），判断新元素与旧元素是否相等，相等的话不添加，不相等则转化为链表。
3. **哈希冲突处理**:
   \- **链表**: 在冲突时，新的键值对节点会被添加到链表的末尾。如果需要查找一个键，HashMap 将遍历该链表直至找到匹配的键。
   \- **红黑树**: 如果链表中的节点数量超过阈值，链表会被转换为红黑树，以减少查找时间。
4. **扩容**: 当 HashMap 中的元素数量超过数组大小（capacity）和负载系数（load factor，默认值 0.75）的乘积时，默认为 16 ，负载因子为 0.75，则 16 X 0.75 = 12，当元素超出 12 时（默认情况下），HashMap 会进行扩容，即创建一个新的数组，其大小是原数组的两倍，并将所有的旧数据重新散列到新数组中。
5. **哈希函数**: 为了尽可能均匀地分布键值对，减少冲突，散列函数在设计时会尽量使得任意两个不同的键产生的索引不同。

## 性能考量

\- **时间复杂度**: 在最佳情况下（无或少量冲突），HashMap 的 `get` 和 `put` 操作的时间复杂度为 O(1)。但在最坏的情况下（如所有元素都映射到同一个桶中），时间复杂度会退化到 O(n)。通过将链表转换为红黑树，Java 8 之后的版本在冲突多的情况下改善了性能，将最坏情况下的时间复杂度降低到 O(log n)。

\- **负载因子和容量**: 负载因子与容量的选择直接影响到 HashMap 的性能和空间使用。较高的负载因子会减少空间开销但增加查找成本，相反，较低的负载因子会提高数据的分布均匀度，从而提高操作的性能，但同时也增加了空间消耗。



HashMap 的设计精妙地平衡了速度和空间使用，通过动态扩容和链表转红黑树等机制，有效地解决了散列冲突，从而保证了高效的数据访问和存储。



# Hashmap的常见方法

HashMap 提供了一系列丰富的方法来操作存储在其中的键值对。以下是一些最常见和最有用的方法及其简要说明：

1. **`void clear()`**
     \- 作用：移除所有的键值对。
     \- 示例：`map.clear();`
2. **`boolean containsKey(Object key)`**
     \- 作用：检查是否含有指定的键。
     \- 示例：`boolean hasKey = map.containsKey(1); // 如果键1存在，返回true`
3. **`boolean containsValue(Object value)`**
     \- 作用：检查是否含有指定的值。
     \- 示例：`boolean hasValue = map.containsValue("Java"); // 如果值"Java"存在，返回true`
4. **`Set<Map.Entry<K,V>> entrySet()`**
     \- 作用：返回此映射中包含的映射关系的 Set 视图。
     \- 示例：`Set<Map.Entry<Integer, String>> entries = map.entrySet();`
5. **`V get(Object key)`**
     \- 作用：获取指定键映射的值。
     \- 示例：`String value = map.get(1); // 如果键1存在，返回其对应的值`
6. **`boolean isEmpty()`**
     \- 作用：检查映射是否为空。
     \- 示例：`boolean isEmpty = map.isEmpty();`
7. **`Set<K> keySet()`**
     \- 作用：返回此映射中包含的键的 Set 视图。
     \- 示例：`Set<Integer> keys = map.keySet();`
8. **`V put(K key, V value)`**
     \- 作用：将指定的值与此映射中的指定键关联。
     \- 示例：`map.put(4, "JavaScript");`
9. **`void putAll(Map<? extends K,? extends V> m)`**
     \- 作用：将指定映射的所有映射关系复制到此映射中。
     \- 示例：`map.putAll(anotherMap);`
10. **`V remove(Object key)`**
     \- 作用：如果存在一个键的映射关系，则将其从映射中移除。
     \- 示例：`map.remove(2); // 移除键为2的项`
11. **`int size()`**
      \- 作用：返回映射中的键值对数量。
      \- 示例：`int size = map.size();`
12. **`Collection<V> values()`**
      \- 作用：返回此映射中包含的值的 Collection 视图。
      \- 示例：`Collection<String> values = map.values();`



 

 