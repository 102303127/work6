# 一.任务一

## 1.项目结构(微服务架构)

```Plain text
##单个微服务架构
└─src
    └─main
        ├─java
        │  └─com
        │      └─zhang
        │          ├─config
        │          ├─controller
        │          ├─entity
        │          ├─mapper
        │          └─service
        │              └─impl
        └─resources
```



## 2.技术栈

### openfeign

>项目集成openfeign实现微服务之间的调用

- 实现远程调用，编写专门的common-api模块，利用starter实现，对接口统一管理
- 实现openfeign的同时，整合了loadbalancer



### nacos

> 项目中整合nacos，部署在服务器上，前面部署nacos在上面没有部署Java程序只是在本地连接都挺正常的，后面把java程序部署上去之后死活连接不上nacos，就在那查网络查配置都没问题，头都大了，后面看到有人说可能是服务器性能不够用了(这个nacos确实挺...),就按量搞了性能好一点的快照，最后也是成功部署了微服务。* _ *  。

- 利用nacos实现服务注册发现
- 使用nacos的config配置中心集中管理配置：mysql.yml，redis.yml

### Hystrix

> 项目中集成熔断降级功能

- 采用信号隔离的策略对接口方法进行服务降级，熔断

### Docker-compose

> 利用docker-compose部署微服务项目

- 编写docker-compose.yml文件，部署java项目

## 3.微服务详解

1. ### gateway:

    - 网关服务主要结合nacos实现调用服务
    - 结合springsecurity的拦截器链实现对请求token的检验，并将用户信息存储在userContext中提供给服务调用

2. ### user-service:

    - 实现用户注册，登录，查询用户信息，上传用户头像的操作

3. ### video-service:

    - 实现视频相关操作

4. ### comment-service:

    - 实现评论操作

5. ### like-service:

    - 实现点赞操作

6. ### follow-service:

    - 实现社交操作

7. ### common:

    - 利用starter实现
    - 公共模块实现统一工具类包括redis，Jwt，File，userContext等
    - 公共模块实现全局异常处理，自定义异常
    - 实现拦截器提供给各个模块获得用户信息
    - 实现封装返回vo

8. ### common-api

    - 利用starter实现
    - 公共接口调用服务，实现FeignClient,提供给其他接口调用



## 4.docker+docker-compose部署项目

### a.dockerfile(部分):

```
# 使用 OpenJDK 8 作为基础镜像
FROM openjdk:8

# 复制 Java JAR 文件到容器中
COPY user-service.jar /user-service.jar

# 暴露应用程序使用的端口（如果需要）
EXPOSE 8081

# 定义容器启动时执行的命令
CMD ["java", "-jar", "user-service.jar"]
```

### b.docker-compose(部分):

```
version: "3"
services:
  user-service:
    image: user-service
    container_name: user-service
    ports:
      - "8081:8081"
    networks:
      - work6-net
networks:
  work6-net:
```

### c.服务命令

创建镜像

```
docker build -t user-service .
```

检查镜像创建

```
docker images
```

镜像创建完毕后用docker-compose构建容器

```
docker compose up -d
```

停止并删除

```
docker compose down
```



# 二.任务二

## 1. 项目结构:

```
src
        └─main
            ├─java
            │  └─com
            │      └─zhang
            │          ├─api  ##这个文件下主要用于实现限流策略
            │          │  │  ControlManager.java   ##接口
            │          │  │  
            │          │  └─impl
            │          │          SlideWindowStrategy.java   ##接口实现滑动窗口限流
            │          │          
            │          ├─aspect
            │          │      ControlAspect.java    ##注解切入处理
            │          │      FrequencyControl.java     ##自定义限流注解
            │          │      
            │          ├─config
            │          │      FrequencyControlAutoConfiguration.java    ##自动装配类
            │          │      RedisConfig.java
            │          │      
            │          ├─core
            │          │      ControlProperties.java    ##核心配置类，可以在配置中自定义限流规则
            │          │      ControlRule.java    ##限流规则整理
            │          │      
            │          ├─exception
            │          │      ControlException.java    ##限流异常处理
            │          │      
            │          └─utils
            │                  RedisUtil.java    
            │                  WebUtil.java    
            │                  
            └─resources
                │  application-aliyun.yml
                │  application-local.yml
                │  application.yml
                │  
                ├─META-INF
                │      spring.factories
                │      
                └─script
                        SlideWindowStrategy.lua   ##使用lua脚本保证redis的频率计数的原子性
```

## 2.核心配置使用

![img](https://zquk8ow8fg6.feishu.cn/space/api/box/stream/download/asynccode/?code=OWYxNGU4NzBjYWNhZTM4OTQ5ZjMxZjNjOWY2ZmI0MTdfdENLM2Z6aW9rTHRlS3hMQ011VTQ4ZkRkdHpnTjFBM0lfVG9rZW46SzJibGJDQm82bzhJTlR4OXFJeGM4NW9wbnNlXzE3MTc1ODM3NDI6MTcxNzU4NzM0Ml9WNA)

## 3.项目实现

1. 利用自动装配实现
2. 限流策略实现滑动窗口：
    1. 也可以通过抽象类实现多种限流规则，可以由核心配置设置
3. 使用lua脚本保证redis的频率计数的原子性：
    1. lua的语法就是简单了解了一下call函数，实现了查询增加删除
4. 实现限流规则可以由核心配置设置，注解的规定大于配置


