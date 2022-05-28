## 服务注册与发现

Spring Cloud提供了多种服务治理方案，netfix eureka、consul、zookeeper

### 搭建eureka服务注册中心 eureka-server
eureka为一个单独的springboot微服务，首先需要注意pom.xml中依赖引入问题

```xml{.line-numbers}
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>

<!--需要指定springcloud依赖的版本-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2021.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```


#### （1）配置文件
注册中心为 ip:port/eureka, 其它服务启动时将实例交给eureka管理
```yml{.line-numbers}
server:
  port: 8889
spring:
  application:
    name: eureka-server
  cloud:
    inetutils:
      ignored-interfaces: 'VMware Virtual Ethernet Adapter for VMnet1, VMware Virtual Ethernet Adapter for VMnet8' #如果电脑中有虚拟机，需要忽略一些网卡，否则无法识别本地ip

eureka:
  instance:
    hostname: localhost #部署eureka注册中心的IP
  client:
    register-with-eureka: false #该服务为注册中心，用于发现服务，不需要将自身暴露于注册中心
    fetch-registry: false # 表示不在注册中心中检索服务，只维护服务实例
    service-url: # 其它服务的注册地址和发现地址
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
logging:
  config: classpath:logback-spring.xml
  level:
    root: debug

```

#### （2）主启动类
加上@EnableEurekaServer注解，将本服务标识为注册中心，访问本服务的ip:port进入eureka服务管理界面，可以查看到已注册的微服务实例
```java
@SpringBootApplication
@EnableEurekaServer
public class Bootstrap {
    public static void main(String[] args) {
        SpringApplication.run(Bootstrap.class, args);
    }
}
```

### 注册微服务实例 uid-center

#### （1）修改pom.xml文件

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2021.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
在之前的springboot项目中加入依赖，这里spring-cloud-starter-netfix-eureka-client提供注解@EnableDiscoveryClient
#### （2）修改配置文件
将eureka-server的地址作为默认服务注册地址
```yml
eureka: 
    client: 
        service-url:
            defaultZone: http://localhost:8889/eureka
```

#### （3）主启动类
```java
@SpringBootApplication
@MapperScan("mapper/*.xml")
@EnableDiscoveryClient
public class Bootstrap {
    public static void main(String[] args) {
        SpringApplication.run(Bootstrap.class, args);
    }
}
```
使用注解@EnableDiscoveryClient，将uid-center服务的ip、端口等信息注册到eureka-server

### 服务调用openFeign

#### （1）引入依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2021.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

#### （2）主启动类
加上注解@EnableFeignClients，扫描所有被@FeignClient标注的接口
```java
@SpringBootApplication
@MapperScan("mapper/*.xml")
@EnableFeignClients
public class Bootstrap {
    public static void main(String[] args) {
        SpringApplication.run(Bootstrap.class, args);
    }
}
```

#### （3）调用服务
```java
FeignClient(name = "uid-center")
public interface TestFeignUid {
    @GetMapping("/uid/nextId")
    long getUid();
}
从服务中心调用uid-center服务的/uid/nextId接口，注意返回值保持一致
```