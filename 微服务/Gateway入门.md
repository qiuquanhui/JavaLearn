# Spring Cloud GateWay 用法



网关，类似于所有接口前面的一堵墙，统一来处理用户发起的请求，比如火车站售票前台。

关键是统一，起到路由，安全性等等。



官网：https://spring.io/projects/spring-cloud-gateway



官方文档：https://docs.spring.io/spring-cloud-gateway/docs/current/reference//html/



实例代码：https://spring.io/projects/spring-cloud-gateway#samples





## 核心概念

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680013471171-4fef045e-4aae-4fc1-8ac0-e0ec453f96c4.png)

1. 路由 (根据条件，进行url转发)
2. 断言（一组规则，条件，用来确定如何转发路由）
3. 过滤器：对请求进行一系列的处理，比如添加请求头，添加请求参数



## 请求流程

client：客户端发起请求

handler Mapping ：根据断言，将请求转发到对应的路由

web handler：处理请求，一层层的经过过滤器：例如可以用于鉴权，限流等等

最后调用实际的服务

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680013582238-17c452b7-3369-493f-bc66-8a9ea248985a.png)



## 两种配置方式

1. 配置式（推荐，方便，规范）

- 简化版

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680013678991-53e670c6-2b22-4ffb-a313-21ee62ed5ecc.png)

- 全称

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680013697198-d5de004f-f91e-4b93-917c-d07581ea202c.png)

1. 编程式（灵活、自由）



## 设置日志（建议设置）

设置了日志就可以知道请求什么开始断言转发到哪里了。

```yaml
logging:
   level:
      org:
        springframework:
          cloud:
             gateway: trace
```





## 路由的各种断言

## 断言

1. After在x时间之后
2. Before在x时间之前
3. Between在x时间之间
4. 请求类别
5. 请求头（包含Cookie）
6. 查询参数
7. 客户端地址
8. 权重(用于实现发布控制)



### The After Route Predicate Factory

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680013841385-9e428a26-a7f8-4d5a-b3e8-b459a87438cb.png?x-oss-process=image%2Fformat%2Cwebp)



### The Before Route Predicate Factory



当前时间在这个时间之前，就会访问当前这个路由

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680013878661-328ea0d4-b817-46a3-80de-c05b7b7cc301.png)





### The Between Route Predicate Factory



当前时间在这个时间之间，就会访问当前这个路由



![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680013901300-71f0a572-1b9d-4f8a-a7fc-e2dc1056592b.png)



### The Cookie Route Predicate Factory

## ![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680013976086-c4765c1b-e0d0-4dd6-a4b0-7ca71e617be1.png)



### The Method Route Predicate Factory



如果你的请求类别是这个post，get，就会访问当前这个路由



![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680013992291-896bc78f-b170-452e-80a7-63281ce791ca.png)



### （客户端访问地址）The Path Route Predicate Factory



如果你的访问的地址是以这些/red/{segment},/blue/{segment}路径作为前缀，就会访问当前这个路由

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: path_route
        uri: https://example.org
        predicates:
        - Path=/red/{segment},/blue/{segment}
```



### The Query Route Predicate Factory



根据查询条件，比如？green，就会访问当前这个路由

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680014044801-0fecacbe-8127-4629-bed5-a4620df1a16a.png)



### The RemoteAddr Route Predicate Factory



根据远程地址，比如你的用户的ip地址是192.168.1.1/24，就会访问当前这个路由



![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680014064363-a048b6b0-7973-4dee-9b6d-cb57912c6445.png)



### 发布控制 The Weight Route Predicate Factory



根据你设置的权重，给你把同一个访问的地址，重定到不同的服务，轻松实现**发布控制**



![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680014088024-493cbd48-f59a-492c-8bc0-6f95a02dbd03.png)





### The XForwarded Remote Addr Route Predicate Factory



从请求头中如果拿到XForwarded这个请求头的地址192.168.1.1/24，，就会访问当前这个路由    请求染色



![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680014113517-747ebc2e-898a-44ca-925e-9fa1e009bfd1.png)



### The Header Route Predicate Factory



如果你的请求头包含X-Request-Id这样一个请求头，并且，它的值符合正则表达式的规则，就会访问当前这个路由



![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680013953930-592b9c5c-11f7-4bfa-9af3-89cbbf546f37.png)





### The Host Route Predicate Factory



如果你的访问的是这个.somehost.org,.anotherhost.org，域名，就会访问当前这个路由



## 过滤器

基本功能：对请求头、请求参数、响应头的增删改查

1.添加清求头

2.添加请求参数

3.添加响应头	

4.降级

5.限流

6.重试



降级：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#spring-cloud-circuitbreaker-filter-factory



### The AddRequestHeader GatewayFilter Factory



增加请求头

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1680016806434-69c478d1-4b76-4be2-b0ab-d73cf1efd396.png)





## 小demo



创建SpringBoot项目

### 1.引入依赖

gateway

lombok



### 2.写配置文件

```yaml
server:
  port: 8090

spring:
  cloud:
    gateway:
      routes:
        - id: path_route
          uri: https://www.codefather.cn/
          predicates:
            - Path=/**


logging:
  level:
    org:
      springframework:
        cloud:
          gateway: trace
```

### 运行访问

地址访问 localhost:8090，就会跳转到 codefather.cn