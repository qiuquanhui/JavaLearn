# String 介绍

在Java中，`String`是一个代表字符串的类。字符串本质上是一个不可变的字符序列，这意呀着一旦一个`String`对象被创建，它所包含的字符序列是不能被改变的。Java中的`String`类提供了大量的方法来进行字符串的操作，如字符串的查找、字符串的比较、子字符串的提取、字符串的分割等。



`String`对象可以通过以下几种方式创建：

1. 字符串字面量：这是最常见的方式，例如`String str = "Hello, World!";`。在Java中，每当你使用字符串字面量**，JVM会首先检查字符串常量池中是否已经存在相同内容的字符串。如果存在，就会返回该字符串的引用；如果不存在，就会在字符串常量池中创建一个新的字符串，然后返回这个新字符串的引用。**

2. 使用`new`关键字：例如`String str = new String("Hello, World!");`。通过这种方式创建的字符串对象不会检查字符串常量池，**直接在堆内存中创建一个新的对象。**

由于`String`对象是不可变的，任何对其进行修改的操作实际上都会生成一个新的`String`对象，而原始的字符串对象内容不会改变。这种设计有助于提高字符串操作的效率并减少内存的使用，因为相同内容的字符串可以在程序中共享使用。



Java中还有两个与`String`类似的类：`StringBuilder`和`StringBuffer`，它们代表的是可变的字符序列。`StringBuilder`和`StringBuffer`的主要区别在于`StringBuffer`的方法是线程安全的，而`StringBuilder`的方法不是线程安全的，因此在单线程环境下`StringBuilder`的性能稍高一些。



# String的方法

`String`类提供了丰富的方法用于操作字符串，以下是一些常用的方法：

1. **length()**: 返回字符串的长度。

   ```java
   String str = "Hello";
   int length = str.length(); // 5
   ```

2. **charAt(int index)**: 返回指定索引处的字符。

 ```java
 char ch = str.charAt(2); // 'l'
 ```

3. **substring(int beginIndex, int endIndex)**: 返回字符串的一个子字符串，从`beginIndex`**（包括）**开始到`endIndex`**（不包括）**。

     ```java
     String sub = str.substring(1, 3); // "el"
     ```

4. **concat(String str)**: 将指定字符串连接到此字符串的末尾。

     ```java
     String concatenated = str.concat(" World"); // "Hello World"
     ```

5. **indexOf(String str)**: 返回指定子字符串第一次出现的字符串内的索引。

     ```java
     int index = str.indexOf("l"); // 2
     ```

6. **lastIndexOf(String str)**: 返回指定子字符串最后一次出现的字符串内的索引。

     ```java
     int lastIndex = str.lastIndexOf("l"); // 3
     ```

7. **startsWith(String prefix)**: 判断字符串是否以指定的前缀开始。

     ```java
     boolean starts = str.startsWith("He"); // true
     ```

8. **endsWith(String suffix)**: 判断字符串是否以指定的后缀结束。

     ```java
       boolean ends = str.endsWith("lo"); // true  
     ```

9. **toLowerCase()**: 返回字符串的小写形式。

     ```java
     String lower = str.toLowerCase(); // "hello"
     ```

10. **toUpperCase()**: 返回字符串的大写形式。

      ```java
      String upper = str.toUpperCase(); // "HELLO"
      ```

11. **trim()**: 返回去除前导和尾随空格的字符串的副本。

      ```java
       String trimmed = "  Hello World!  ".trim(); // "Hello World!"
      ```

12. **replace(char oldChar, char newChar)**: 返回一个新的字符串，它是通过用`newChar`替换此字符串中出现的所有`oldChar`得到的。

      ```java
      String replaced = str.replace('l', 'p'); // "Heppo"
      ```

13. **split(String regex)**: 根据匹配给定的正则表达式来拆分字符串。

      ```java
        String[] parts = "apple-banana-cherry".split("-");
        // parts = ["apple", "banana", "cherry"]
      ```

14. **equals(Object anObject)**: 比较此字符串与指定对象。结果是true当且仅当参数不是null，并且是表示相同的字符序列的String对象。

      ```java
       boolean equals = str.equals("Hello"); // true
      ```

15. **equalsIgnoreCase(String anotherString)**: 将此String与另一个String比较，不考虑大小写。

      ```java
       boolean equalsIgnoreCase = str.equalsIgnoreCase("HELLO"); // true
      ```

这些方法是`String`类中最常用的一部分方法，通过这些方法你可以执行大多数字符串操作任务。



#StringBuild 不同于String的方法有哪些



`StringBuilder`类在Java中用于创建可变的字符序列，与`String`类不同，`StringBuilder`类的对象可以被修改。以下是`StringBuilder`类中一些与`String`不同的方法：

1. **append(各种类型)**: 用于追加数据到当前`StringBuilder`对象的末尾。`append`方法是重载的，它可以接受几乎所有类型的参数，包括`char`、`char[]`、`boolean`、`int`、`long`、`float`、`double`、`String`等。

  ```java
    StringBuilder sb = new StringBuilder("Hello");
    sb.append(" World"); // "Hello World"
  ```

2. **insert(各种类型)**: 将指定数据插入到`StringBuilder`对象中的指定位置。

  ```java
    StringBuilder sb = new StringBuilder("Hello World");
    sb.insert(6, "Java "); // "Hello Java World"
  ```

3. **delete(int start, int end)**: 移除序列的子字符串。

  ```java
    StringBuilder sb = new StringBuilder("Hello Java World");
    sb.delete(5, 10); // "Hello World"
  ```

4. **deleteCharAt(int index)**: 移除指定位置的字符。

  ```java
    StringBuilder sb = new StringBuilder("Hello World");
    sb.deleteCharAt(5); // "HelloWorld"
  ```

5. **reverse()**: 将序列中的字符反转。

  ```java
    StringBuilder sb = new StringBuilder("Hello World");
    sb.reverse(); // "dlroW olleH"
  ```

6. **replace(int start, int end, String str)**: 将指定索引范围的字符用新的`String`替换。

  ```java
    StringBuilder sb = new StringBuilder("Hello World");
    sb.replace(6, 11, "Java"); // "Hello Java"
  ```

7. **toString()**: 将`StringBuilder`对象转换为`String`。

  ```java
    StringBuilder sb = new StringBuilder("Hello");
    String str = sb.toString(); // "Hello"
  ```

8. **setCharAt(int index, char ch)**: 用指定的字符替换此序列中指定位置的`char`。

  ```java
    StringBuilder sb = new StringBuilder("Hello World");
    sb.setCharAt(6, 'J'); // "Hello Jorld"
  ```

9. **setLength(int newLength)**: 设置字符序列的长度。如果新长度小于当前长度，则长度将被改变，并丢弃索引newLength及之后的字符。

  ```java
    StringBuilder sb = new StringBuilder("Hello World");
    sb.setLength(5); // "Hello"
  ```

`StringBuilder`的这些方法都是可变操作，调用它们会改变对象本身的状态，而不是像`String`类那样创建一个新的对象。这就是`StringBuilder`在进行多次修改字符串内容的场景中比`String`更高效的原因。

 

 