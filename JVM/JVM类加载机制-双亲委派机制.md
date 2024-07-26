# JVM的作用是什么

我们运行Java程序时，要安装JDK，JDK包含JVM，不同环境的JDK都是不同的。



Java 代码在编译后会形成 class 的字节码文件，该字节码文件通过 JVM 解释器，生成特定系统的机器指令。从而实现**跨平台运行**

 

**也正是“一次编译，到处运行”**



下面我们讲一讲类的双亲委派机制



# **双亲委派机制**

当收到类加载的请求时，该类的加载器会将请求发送给其父类加载器直至根加载器，当父类加载器加载不了的话，就会自己尝试加载。



使用双亲委派机制，是防止重复加载两次字节码。确保只有一个字节码加载。



**类加载器**

**类加载器主要分为**

- **根加载器 Bootstrap ClassLoad，一般加载Java核心库**
- **扩展****类加载器 Extension ClassLoad，一般加载Java扩展包**
- **应用程序类加载器 Application ClassLoad，加载 classPath下的应用程序类**
- **自定义加载器 Customer ClassLoad**



一般 Java 的本地方法类由根加载器加载，扩展类Java的扩展目录（通常是jre/lib/ext）由扩展类加载器加载：，classPath 下的所有类都由应用程序类加载器。

- **java.util.ArrayList**：这个类也是Java核心库的一部分，由根加载器加载。
- **javax.swing.JFrame**：这个类属于Java的扩展包，由扩展加载器（ExtClassLoader）加载。
- **com.example.MyClass**：这是一个假设的应用程序类，它位于应用程序的类路径上，由系统加载器（AppClassLoader）加载。

## 加载流程

![img](https://cdn.nlark.com/yuque/0/2024/jpeg/33747484/1715074793663-7ba2e4d5-8652-4004-8c49-8a42b7d3fb80.jpeg)



**使用双亲委派机制的好处：**

1、可以避免类的重复加载，当父类加载器已经加载了该类时，就没有必要子类加载器 再加载一次。

2、考虑到安全因素，java核心api中定义类型不会被随意替换，假设通过网络传递一个名为java.lang.Object的类，通过双亲委托模式传递到启动类加载器，而启动类加载器在根加载器发现这个名字的类，发现该类已被加载，并不会重新加载网络传递的过来的java.lang.Object，而直接返回已加载过的Object.class，这样便可以防止核心API库被随意篡改。

