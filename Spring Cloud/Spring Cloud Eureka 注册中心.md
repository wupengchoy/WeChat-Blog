# Spring Cloud Eureka 注册中心

&nbsp;&nbsp;Spring Cloud注册中心可以统一管理各个服务组件，具有以下功能：

- 用来记录各个服务的基本信息，比如名称，IP，端口等等。
- 提供微服务的注册和注销功能，在微服务启动的时候，将自己的信息注册到服务中心，方便统一管理。
- 具有检查功能，通过心跳检测，定时检查注册的微服务是否可以访问，如果长时间不能访问，就会从服务注册表中移除实例。

## Eureka 服务端

- 创建一个项目，在项目pom文件中添加如下依赖：

```xml
<!--增加eureka-server的依赖-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

- 在启动类上添加@EnableEurekaServer注解，说明这是一个eureka服务。
- 编写配置文件application.yml，内容如下：

```yaml
spring:
  application:
    name: eureka-server

server:
  port: 8380

localhost: localhost

eureka:
  client:
    # 是否注册自身到其他eureka上
    register-with-eureka: false
    # 是否获取其他eureka注册信息，因为此处是单点的eureka，无需同步其他eureka的注册信息
    fetch-registry: false
    # 与eureka的交互地址，多个使用逗号分隔
    service-url:
      defaultZone: http://${localhost}:${server.port}/eureka/
```

&nbsp;&nbsp;以上一个eureka注册中心就编写完了，访问http://localhost:8380见到如下页面说明服务启动成功。此时instances currently registered with Eureka下为空是因为还没有服务注册到当前eureka中。

![eureka未注册](/Users/Jeremy/Documents/公众号/image/eureka未注册.png)

## Eureka 客户端

- 在服务的pom文件中添加如下依赖

```xml
<dependency> 
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```

- 在application.yml中配置注册中心地址，如果需要在多个注册中心注册，使用逗号相隔。eureka.instance.prefer-ip-address = true表示将自己的IP注册到eureka上，如果不设置该属性或者将其设置为false，则表示将微服务所在的操作系统的hostname注册到eureka上。

```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8380/eureka/
  instance:
    prefer-ip-address: false
```

- 在服务启动类上添加注解**@EnableEurekaClient**或者**@EnableDiscoveryClient**，启动服务，访问http://localhost:8380，发现在instances currently registered with Eureka下出现了当前服务信息，说明当前服务已经注册到eureka上。

![eureka已注册](/Users/Jeremy/Documents/公众号/image/eureka已注册.png)

>在status这一栏下有个UP，这个是Eureka的健康检查机制，除此之外还有DOWN,OUT_OF_SERVICE_UNKOWN等，只有标记为UP的才能正常被请求。
>
>启动Eureka的健康检查可以添加如下配置：
>
>```yaml
>eureka:
>  client:
>    healthcheck:
>      enable:true
>```
>
>

## Eureka Server 高可用

​      分布式开发环境中如果只配置一个注册中心，如果这个注册中心因为某些原因挂了，那么其他服务的注册信息就会丢失。下面我们部署一个高可用的Eureka环境，在其中某个服务挂掉之后还能获得其他的服务注册信息。

- 复制上面的eureka-server 服务，将名称修改为eureka-server-ha，修改端口号为8480。
- 在hosts文件中配置profiles，hosts文件在Windows下的路径是:C:\Windows\System32\driver\etc\hosts，在Linux，Mac OS下的路径是：/etc/hosts。

```xml
127.0.0.1 localhost localhost1
```

- 在两个Eureka Server的application.yml文件中添加如下配置

```yaml
spring:
  application:
    name: eureka-server

server:
  port: 8380

localhost: localhost

eureka:
  client:
    # 是否注册自身到其他eureka上
    register-with-eureka: false
    # 是否获取其他eureka注册信息
    fetch-registry: false
    # 与eureka的交互地址，多个使用逗号分隔
    service-url:
      defaultZone: http://${localhost}:${server.port}/eureka/,http://localhost2:8480/eureka/

---
spring:
  profiles: localhost
server:
  port: 8380
eureka:
  instance:
    host: localhost
---
spring:
  profiles: localhost2
server:
  port: 8480
eureka:
  instance:
    host: localhost2
```

​      defaultZone配置了每个eureka服务的地址信息，如果将register-with-eureka和fetch-registry设置为true，在启动两个服务之后，访问eureka界面，这两个服务将会相互注册并且将自身也注册到当前服务中。

​      在测试过程中如果将以上两个配置设置为true，在启动的时候会报出Connection Refused信息，网上查询可能跟eureka的默认配置相关，eureka默认的配置端口是8761，暂时还未找到具体的报错原因，以后有机会找到在更新。

-  修改Eureka Client服务的的application.yml配置文件

```yaml
spring:
  application:
    name: springcloud-study
server:
  port: 8280

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8380/eureka/,http://localhost2:8480/eureka/

```

- 分别启动两个Eureka Service再启动Eureka Client，访问http://localhost:8380和http://localhost:8480，在界面上都能看到客户端信息都注册到了这两个服务中心上去，说明高可用配置成功。

## 用户认证

​      在前面的实例中，Eureka Server是允许匿名访问的，下面创建一个需要登录的Eureka Server.

### 改造Eureka Server

- 在原有项目eureka-server中添加如下依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- 修改eureka-server中application.yml中的配置信息

```yml
security:
  basic:
    enabled: true
  user:
    name: yanzu
    password: yanzu123
```

- 将eureka-server和eureka-client中的defaultZone的注册信息修改为http://username:password@Eureka_host:Eureka_port/eureka这种格式

```yaml
eureka:
 client:
   service-url:
     defaultZone: http://yanzu:yanzu123@localhost:8380/eureka/,http://localhost2:8480/eureka/
```

​      只修改eureka-server而不修改eureka-client的注册地址信息，并不能将客户端注册上去，需要同时修改。

- 访问eureka-server界面，出现登录界面，输入设置好的用户名和密码后方能登入查看注册信息。