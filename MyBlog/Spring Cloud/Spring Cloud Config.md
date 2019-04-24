> Spring Cloud Config是Spring Cloud的配置中心，将项目中的配置文件统一管理，其他服务通过配置中心获取相关的配置信息。

# 服务端Server
&ensp;&ensp;**pom.xml依赖**
```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.smartcity.envapp</groupId>
    <artifactId>gateway</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>

    <name>gateway</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Edgware.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <!-- 配置中心服务端组件 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        -->
        <!--热部署-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

&ensp;&ensp;**application.properties配置**
```java
# 端口号必须是8888--约定大于配置
server.port=8888
spring.application.name=config-server
# 配置中心信息
# 配置文件地址
spring.cloud.config.server.git.uri=https://gitee.com/wueryouting/CloudConfigServer
# 搜索文件夹，可配置多个用逗号隔开
spring.cloud.config.server.git.search-paths=testCloudProperties
# 代码管理登录的用户名和密码，如果是公共的可以不用填写
spring.cloud.config.server.git.username=wueryouting
spring.cloud.config.server.git.password=mtt20091201wp

management.security.enabled=false
```

&ensp;&ensp;**启动项**
```java
需要在服务端加上：
@EnableConfigServer
@SpringBootApplication
```

# 客户端Client
&ensp;&ensp;**pom.xml配置**
```Java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.smartcity.envapp</groupId>
    <artifactId>gateway</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>gateway</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.SR1</spring-cloud.version>
    </properties>
    <!-- 配置组建 -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--热部署-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

&ensp;&ensp;**application.properties配置**
```Java
# 客户端的端口
server.port=8881
# 名称需要与配置文件名称一致，比如配置文件需要为config-client-dev.properties,config-client与项目名称一致，dev表示开发环境
spring.application.name=config-client
# 代码分支版本
spring.cloud.config.label=master
# 访问环境
spring.cloud.config.profile=dev
# 访问配置中心的地址
spring.cloud.config.uri=https://localhost:8888/

# 配置rabbitmq
#开启通过服务访问config-server功能
spring.cloud.config.discovery.enabled=true
# 服务名称--配置中心服务的名称
spring.cloud.config.discovery.service-id=config-server
# rabiitmq的信息
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest

management.security.enabled=false
## 开启消息跟踪
spring.cloud.bus.trace.enabled=true
```
 &ensp;&ensp;**配置类的注解**
 ```java
 添加的注解：
 @SpringBootApplication
 @RestController
 @RefreshScope   //实现自动刷新
 @EnableAutoConfiguration
```

&ensp;&ensp;**使用和刷新**
- 先启动server,再启动client
- 在需要获取的属性上添加注解@Value("${key}"),文件中key值对应的配置value就会被注入到属性中
- 上传更新配置问价后需要运行**curl -X POST http://localhost:8888/bus/refresh**使用post方法触发刷新，在spring-cloud-starter-bus的机制下会将所有的客户端都刷新一遍，实现了自动刷新的功能。
