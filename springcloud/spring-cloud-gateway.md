请求路由规则
```yml
spring:
  cloud:
    gateway:
      routes:
        id: 
        uri:
        predicates: #规定请求中的时间范围、cookie名、请求头匹配格式、域名格式、请求类型(Post Get)
      filters:
       
```

过滤规则
```yml
spring:
  cloud:
    gateway:
      routes:
        id: 
        uri:
        filters: 
       
```
过滤器校验token
1、从token解析身份(username, role, perm)
2、没有token则redirect至/login
3、有token,则查询uri对应的权限(例如, roleId=1, /api/get/**,对比request.uri是否与查询结果匹配)
4、通过则放行