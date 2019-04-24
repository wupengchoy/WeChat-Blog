- springcloud启动类增加注解**@EnableAutoConfiguration**
- 在pom文件中增加相关插件
```java  
  <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
          <artifactId>mybatis-spring-boot-starter</artifactId>
          <version>1.3.2</version>
      </dependency>
      <dependency>
          <groupId>mysql</groupId>
          <artifactId>mysql-connector-java</artifactId>
          <scope>runtime</scope>
  </dependency>
```

- 配置文件中配置数据库连接相关信息
```Java  
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/helloblog?setUnicode=true&characterEncoding=utf8&autoReconnect=true
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
```

- 创建接口类和xml映射文件
![mapper映射](/Users/Jeremy/Documents/MyBlog/HelloBlog/imgs/mapper.png)
1. 接口中添加两个注解**@Component**和**@Mapper**.
2. mapper映射文件中的namespace属性对应接口
3. 文件名字需要相同
