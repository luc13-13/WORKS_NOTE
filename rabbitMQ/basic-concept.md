
<div align=left>
<img width = 900 src = "../images/rabbitMQ/exchange-queue.png" alt="交换机的输出队列"/>
<center>消息传递过程</center>
</div>
<br>

## Spring Cloud Stream3.2 新特性

### binders声明
binder相当于消息来源服务器的规定, rabbitmq-node1、rocketmq、Kafka都可以被声明为一个binder, 注册到springCloud Stream的binders中.

通过binder, 可以将不同消息源对接、 手动选择消息源的配置等

```yaml
spring: 
  cloud:
    stream:
      binders: # binders 为 Map<String, BinderProperties>, 其中key 为声明的名称“rabbit”, BinderProperties包括 type、 enviroment属性, 用于配制消息服务器, 其中type可以为rabbit、kafka、rocket, 取决于引入了哪些消息队列的依赖
        rabbit: 
          type: rabbit
          environment:
            spring:
              rabbitmq:
                dynamic: false
                host: localhost
                port: 5672
                username: luc
                password: luc123456
                virtual-host: v1
  
```
### 消息生产者: 函数式声明
### 消息生产者: 手动发送消息

### 消费者与队列监听
在spring cloud stream2.x版本, 需要使用@Stream Listener创建对指定消息队列的监听. 

生产者于exchange的绑定声明


### 消费失败
类型转换失败, queue中消息会丢失

StreamBirdge(consumer-in-0, data)

将路由键放在headers, BaseMsg继承Message(org.springframework.messaging.Message;)
重写 getPayload和getHeaders方法

使用function方式声明生产者和消费者时, 需要发送Message子类才能成功路由到指定队列


auth:eyJ0eXBlIjoiSldUIiwiYWxnIjoiSFMzODQifQ.eyJzdWIiOiJsdWNoZW5nIiwiaWF0IjoxNjY3MzA2NTk0LCJleHAiOjE2Njc5MTEzOTR9.z07oiGOl2G0r1ELIOy3VH5VNUChotMJncSOhCwttFKIXi3Q9PCmvDknt1BLVynlj

rem:gtEd+M8ujgS6HJRPAoKlyNYG8n/GvPuE3sZ8r7YFrigTrhy3woQ7FeCGBYW38RTwNuRFjCBLpsaNE7+JMW3GPm1qx5BlfPNw9yMMUEWMDUo8uiEunTRBpxZg53xFt1ANwd6ciDKqsru5Y/LLDxGbXUPxsm8O2wvLoM6cCM49/nRen1vE51tWYlTMW2TYNmD1cZXbv1RjXgntA+hg6miMrbuKTBd+SDJzhH+tqyNfGXXAAHy7gjgkQg5NxGoLJjM/a47gt2EI0ycdXd1ir9tZRaxw8kVj15cuNE7UZqSxis2vP+ohc/GD0P1wMY+zVUhg+CLRBi2ANJv492YNDNz5mkKOxs+Es2HPLuKGAMrtrTIS9XQQAeNOGlovy6DIpArfPZb6Q6XoiHoCCxY9ZDN5UXusuIGb6MljVLDIbnoYe5/e1hALx/zbWvxVjdybip9zVI/TrYkuJpWNqOLMG5EXQNH+V6jzshmS25qgJdWRHRbKATcxneAkJKRhLjHUWyMLYXjFkKV+WfIXAQfLp4MYeIynN/TO4uvEyZAtFp+QMKxwqz+xb+DWqZ4HDInNp5552QA/rS/MMg60L9BLfdr/sTzoCc1zmeBNmd2GA2tar9uX/dD+PXY3Pda5GggNTs+RQDAoD9k/v7k5QTinOsh1Rwa+MFCbl9O8OBm3L/HMlZPri+Rzb1ARPPcyk960FF2DIsrgCqe/o3yIdSD2kcVuJZwn9CW5Im1CBnzok9GC4oz4RKd3zHYf7Jvjvuruu9dzx38SD4V8RQ9X40gWWA/qZ/iv5SxvX3eu2cxDcy1P6T07l3O2dM9iP1lVC0urFq4z/A2AS3p7ClAMb05iyuowpifcubQpsp1MkpHsZMNG2Nyg5ZKtnJGDSL//wO1kR6xqry/9BmikT1dWrVsvVUF7pTja+mfx2xxTUTwc1o1C74qwi05ZR+UDvWEJqzTA6e1LoWVyBCi7FpFl4YETVoligp7XLtm7j1Ho/ONAglYIndvcQKq2tlMsJuy8Fcbn3ByRor/3eUyl/SoKqruYhK3Ee65/Xw2e/fcasgRyX+L5uL4Fub2nCEbdGtbMAYu7UxQckbtb3GpDg3SB6lZdDf5sUQ==

sid: 33aee64e-1213-4576-91a7-23e011e660e5
