

@[TOC](文章目录)

---

# 前言

昨天跟大家讲了MySQL的基础架构（链接：[MySQL的基础架构](https://qiuqiu.blog.csdn.net/article/details/136158147?ydreferer=aHR0cHM6Ly9tcC5jc2RuLm5ldC9tcF9ibG9nL21hbmFnZS9hcnRpY2xlP3NwbT0xMDExLjIxMjQuMzAwMS41Mjk4)）

今天讲一讲我们的高频面试题MySQL中SQL语句的执行流程。


---
# SQL语句的执行流程
SQL 语句分为查询语句与更新语句。更新语句就是增加、删除、修改。本文从这两种不同的语句讲述SQL语句的执行流程。

## 查询语句的执行流程
我们先来看看MySQL的基础架构（如下图），各司其职。
* 连接器：用于身份认证与权限鉴定。
* 分析器（解析器）：用于词法分析与语句分析，用于判断 SQL 语句的准确性。
* 优化器：用于优化 SQL 语句，MySQL 自带，为了让 SQL 语句执行时可以提高性能，但是注意优化器优化后的 SQL 语句并不是最佳的。
* 执行器：执行 SQL 语句返回存储引擎返回的读写数据
* 存储引擎：用于存储读写数据。
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/2c2708fed69f435f96a6cf2e2978048c.png#pic_center)


正如图所示，查询语句的执行流程如下

查询语句：```select * from students where name = '张三'```

1. 客户端通过连接器连接数据库。连接器对用户进行身份验证与权限校验。
2. 分析器进行词法和语法分析，判断SQL语句的词法、语法是否正确。
3. 先查询缓存，有则返回，无则继续第 4 步，得到结果后将数据存入缓存。（注意在MySQL8.0以后查询缓存功能已取消，因为实用性不大。）
4. 优化器将SQL语句进一下的优化，达到性能优化。（注意 MySQL 的优化不一定是最佳，底层涉及索引的使用等等，不展开说）
5. 执行器调用存储引擎的 API 接口执行SQL语句，最后返回名字等于‘张三’的数据。

## 更新语句的执行流程
更新语句的前程跟查询的一样，不同的是多了日志模块的使用。MySQL 的 binglog（归档日志），与存储引擎 Innodb 自带的 redo log（重做日志）。MySQL 5.5 之后默认是使用 Innodb 当做存储引擎。

```update students set  name = '李四' where name = '张三' ```

1. 先按照上述的流程在数据库中查询出 name = ‘张三’ 的数据。
2. 执行更新操作，记录在 redo log 并记为 prepare 状态。
3. 将操作记录在 binglog 中。
4. 将 redo log 的该记录改为 commit 状态。

问：为什么 redo log 有两种状态呢？

答：为了防止在修改中数据库崩溃的时候可以恢复数据

问：binglog 起到什么作用？

答：binglog 是归档日志。是MySQL自带的，而存储引擎 Innodb 自带的 redo log。redo log 是支持事务的，在数据库崩溃的时候可以使用 redo log 来恢复数据。而 binglog 是多用于恢复备份。

---
# 总结
查询语句与更新语句前面都是相同，只是更新语句需要加入日志模块，是为了能够恢复数据记录等等的作用，涉及到 Innodb 的 redo log。这个是面试的高频考点，大家要多加熟悉。


有启发点个赞 🌹





> 我是小辉，正在进行 Java 实习的 24 届应届毕业生。持续分享，包括但不限于技术文章。全网同名...



