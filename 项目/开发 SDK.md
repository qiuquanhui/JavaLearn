## 开发 SDK

为什么要开发SDK。

1. 减少代码的冗余
2. 提高代码的复用

如果实际项目中需要使用到该SDK，在pom.xml中注入就可以了。

类似于maven一样，把需要的SDK注入在中央仓库中。需要使用的使用几行代码实现注入。



以下来演示如何开发SDK。

下面演示一个开发场景：开发一个可以远程调用的SDK



### 新建项目

选择

**spring-boot-configurat-processor 的作用是配置文件中自动生成配置的代码提示**

**lombok**

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1679749674412-22eda84c-8f73-4782-b76f-39e8c8eba452.png)

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1679749681643-7042ea26-0bc0-4bed-a49d-31ee036c86b0.png)

### 修改pom文件

将版本号修改为 0.0.1

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1679749698335-de86cf8d-c1bc-4ec1-a5d2-71b824401199.png)

删除pom.xml里面的插件

**因为这个插件是用来构建jar包的**

而我们不是要开发jar包。

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1679749704698-1af9f6c3-5f86-4274-8d2e-1f5cea4dba50.png)

### 删除启动类

![img](https://cdn.nlark.com/yuque/0/2024/png/33747484/1713429573649-40a2a29e-0a7e-4d7a-be27-567ea58a4cd7.png)

### 创建配置类

![img](https://cdn.nlark.com/yuque/0/2024/png/33747484/1713429594852-278f2200-e350-49d7-814a-e389405b66eb.png)



```java
/**
 *
 *
 * @author: Hui
 **/
@Data
@Configuration
@ConfigurationProperties("api.client")
@ComponentScan
public class ApiClientConfig {

    private String accessKey;

    private String secretKey;

    @Bean
    public ApiClient apiClient() {
        return new ApiClient(accessKey, secretKey);
    }

}
```

### 复制之前的客户端

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1679749768843-731e89b8-91d1-4f6e-996b-d5f2eb843ff0.png)



![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1679749780506-78bc6eaa-47b6-487f-9c34-1cc9a232fe7b.png)



### 新建spring.factories

在resources下创建META-INF

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1679749837862-a32bf5a3-54a7-4a04-bff5-ab06c6e541b9.png)



![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1679749850173-27b0fa42-8874-4a46-b1d6-541cb5f3ebba.png)



```bash
#spring boot starter
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.jinze.jzapiclintsdk.JzApiClintConfig
```

![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1679749880023-e726c7e2-fe0d-4ba5-a29b-53b9d1332e5e.png)



### 打包



![img](https://cdn.nlark.com/yuque/0/2023/png/33747484/1679749905373-3061b505-5468-4ce1-a8b7-4c5f94a4999f.png)

这样你就可以在本地你的项目中的pom.xml文件中使用到这个sdk了。

引入方式代码：

```xml
<!--        客户端-->
        <dependency>
            <groupId>com.hui</groupId>
            <artifactId>api-client</artifactId>
            <version>0.0.1</version>
        </dependency>
```

application.yml文件中

```yml
api:
  client:
    access-key: hui
    secret-key: abcdefg
```



至此SDK完成。

> 文中的图片使用的是编程导航球友的图片，如建议，联系必删！！



编程导航的码，大家有兴趣可以加入

![image-20240424103919468](C:/Users/%E9%82%B1%E6%9D%83%E8%BE%89/AppData/Roaming/Typora/typora-user-images/image-20240424103919468.png)