# HashSet是什么

`HashSet`是Java集合框架中的一部分，它实现了`Set`接口。在Java中，`HashSet`是基于`HashMap`实现的。它不保证集合的迭代顺序；特别是，它不保证该顺序恒久不变。这个类允许存储的元素为`null`。

主要特点包括：

1. **不允许重复元素**：`HashSet`不允许包含重复的元素。这是因为`Set`接口的性质，它是集合的一种，不允许元素重复。

2. **无序集合**：`HashSet`不保证元素的顺序，元素在集合中的位置可能与添加进集合的顺序不同。这与`LinkedHashSet`（维护了元素插入顺序的`Set`实现）或`TreeSet`（根据元素的自然顺序或构造函数中指定的`Comparator`进行排序的`Set`实现）形成对比。

3. **为快速查找而优化**：`HashSet`背后使用的`HashMap`实例确保了元素查找的高效性。基本操作（如`add`、`remove`和`contains`）在理想情况下提供常数时间的性能。

4. **允许`null`值**：`HashSet`允许集合中有一个`null`元素。

   

由于`HashSet`是基于哈希表的，因此它的性能受到哈希函数的影响。良好的哈希函数可以将元素均匀分布在桶中，从而优化性能。

使用场景主要包括：

* 当需要快速查找、插入和删除集合中的元素时。
*  当不需要维护元素插入的顺序时。
*  当要确保集合中不出现重复元素时。

简单的使用示例：

```java

import java.util.HashSet;

public class HashSetExample {
  public static void main(String[] args) {
    HashSet<String> hset = new HashSet<>();

    // 添加元素到HashSet中
    hset.add("Apple");
    hset.add("Mango");
    hset.add("Grapes");
    hset.add("Orange");
    hset.add("Fig");
    // 重复的元素不会被添加
    hset.add("Apple");

  // 显示HashSet中的元素
    System.out.println(hset);
  // 输出可能是 [Mango, Fig, Apple, Orange, Grapes]
  // 注意，HashSet不保证元素的顺序。
  }
}
```

这个例子说明了`HashSet`的基本使用，包括如何添加元素以及如何显示集合中的所有元素。请注意，尽管添加了两次"Apple"，集合中只会包含一个"Apple"实例，因为`HashSet`不允许重复。



# HashSet的常用方法是什么

`HashSet`提供了一系列的方法，使得它在使用时非常灵活。这些方法允许你检查、修改集合，以及更多操作。以下是一些`HashSet`的常用方法：

1. **add(E e)**: 将指定的元素添加到此 `HashSet` 中（如果此元素尚未存在于集合中）。
2. **remove(Object o)**: 如果存在，则从该集合中移除指定元素。
3. **contains(Object o)**: 如果此集合包含指定元素，则返回 `true`。
4. **isEmpty()**: 如果此集合不包含任何元素，则返回 `true`。
5. **size()**: 返回此集合中的元素数（其容量）。
6. **clear()**: 移除此集合中的所有元素。
7. **iterator()**: 返回此集合中元素的迭代器。迭代器返回元素的顺序并不是确定的。
8. **toArray()**: 将此集合转换为一个数组。
9. **addAll(Collection<? extends E> c)**: 添加指定集合中的所有元素到此集合（可选操作）。
10. **removeAll(Collection<?> c)**: 从此集合中移除指定集合中包含的其所有元素（可选操作）。
11. **retainAll(Collection<?> c)**: 仅保留此集合中那些也包含在指定集合的元素（可选操作）。
12. **equals(Object o)**: 比较指定对象与此集合的相等性。
13. **hashCode()**: 返回此集合的哈希码值。



示例代码：

```java
import java.util.HashSet;
public class Main {
  public static void main(String[] args) {
    HashSet<String> fruits = new HashSet<>();
    fruits.add("Apple");
    fruits.add("Banana");
    fruits.add("Cherry");

    System.out.println("Fruits Set: " + fruits);
    System.out.println("Contains Apple? " + fruits.contains("Apple"));
    System.out.println("Set size: " + fruits.size());

    fruits.remove("Banana");
    System.out.println("After removing Banana: " + fruits);

    if (!fruits.isEmpty()) {
      System.out.println("Fruits Set is not empty.");
    }

    Object[] fruitsArray = fruits.toArray();
    System.out.println("Array: " + Arrays.toString(fruitsArray));

    fruits.clear();
    System.out.println("After clear method, Fruits Set: " + fruits);
  }
}
```

这段代码首先创建了一个`HashSet`实例，并通过使用`add`方法向其中添加了几个元素。然后它演示了如何使用`contains`、`size`、`remove`和`isEmpty`方法。此外，该示例还展示了如何将`HashSet`转换为数组，以及如何清空集合。

 

# HashSet的底层实现

`HashSet`在Java中是基于`HashMap`实现的。它实际上使用一个`HashMap`实例来存储其元素。每个插入到`HashSet`中的元素实际上作为`HashMap`的键存储，而这个`HashMap`的值则使用一个固定的、不变的对象来代表（通常是一个预定义的私有静态对象，不对外公开，仅用于内部标识）。这种设计允许`HashSet`利用`HashMap`的高效键查找能力。



## 底层数据结构

`HashMap`是一个数组和链表/红黑树的结合体。它使用数组来存储数据，每个数组的位置称为一个桶（Bucket）。每个桶的初始状态是空的，当冲突发生时（即两个元素的哈希码映射到同一个桶），`HashMap`会使用链表或红黑树来存储这些元素，以解决冲突。

## 哈希函数

`HashSet`的性能在很大程度上取决于哈希函数的质量和桶的数量。哈希函数将集合中的元素映射到桶。理想情况下，哈希函数应该均匀地分布元素，以减少冲突。当冲突发生时，查找效率可能会降低，因为需要在链表或红黑树中进一步查找。

## 添加元素

当向`HashSet`添加一个元素时，`HashSet`会使用元素的`hashCode()`方法来计算元素的哈希值，然后根据这个哈希值找到对应的桶位置。如果桶中没有其他元素，就直接存储；如果有，会进一步检查是否有相同的元素（通过`equals()`方法）。如果没有找到相同的元素，新元素会被添加到链表或红黑树中。

## 查找元素

查找元素时，也是先计算元素的哈希值，然后定位到对应的桶。如果桶中有多个元素（即发生了哈希冲突），则通过链表或红黑树遍历这些元素，使用`equals()`方法来查找匹配的元素。

## 扩容

当`HashMap`中的元素数量达到容量和加载因子的乘积时，`HashMap`会进行扩容操作，即创建一个新的元素数组，其容量是原数组的两倍，并将所有旧元素重新映射到新数组中。这个过程称为“重新哈希”或“扩容”，它可以保证即使在大量元素被添加到集合中时，`HashSet`的操作仍然能够保持相对较高的效率。

通过这种方式，`HashSet`利用`HashMap`的高效性来实现其自身的操作，同时隐藏了`HashMap`值的细节，只暴露与集合操作相关的接口。

 

 