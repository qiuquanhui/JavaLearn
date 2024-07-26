# ArrayList源码解析

ArrayList：是基于数组实现的集合列表，它被称为动态数组，因为相比与数组的固定大小，ArrayList 当添加元素时，添加的元素数量大于数组的容量的话就会触发扩容机制。每一次都扩大原来的 1.5 倍。



每一次添加元素的时候，都要确保添加元素后的个数是否大于数组容量。

## add() 方法源码

添加前都使用 ensureCapacityInternal()来确保数组容量。

```java
 public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

ensureCapacityInternal()方法源码

```java
   private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));//进一步判断是否有足够的容量
    }
```

calculateCapacityl()方法源码，判断当前数组最少需要多少容量

```java
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
```

ensureExplicitCapacity()方法源码

```java
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0) //当添加元素后的元素个数 大于 数组容量时
            grow(minCapacity);  //核心扩容方法
    }
```

## grow()方法源码（重点）

每次通过右移的方法扩容 1.5 倍，最后将旧数组的元素复制到新数组中。

```java
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);//每一次扩大原来的 1.5 倍
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);  //直接复制
    }
```

## get()方法源码

很简单，直接根据下标返回数组的元素

```java
public E get(int index) {
    rangeCheck(index);//确保下标的范围

    return elementData(index);
}
```

##  set()方法源码

很简单，直接根据下标设置数组的元素

```java
    public E set(int index, E element) {
        rangeCheck(index);//确保下标的范围

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```

## remove()方法源码

数组的移除是通过覆盖的方式完成，将该元素后面的值往前覆盖。

再将最后的值设置为 null，是为了让 GC 垃圾回收机制进行清理。

```java
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```



ArrayList 是线程不安全的，如果要实现线程安全，可以使用 CopyOnWriteArrayList，或者用 Collections 工具类封装。



> 大家好，我是小辉，持续分享。时间：20240611第一版，状态：应届找工作中