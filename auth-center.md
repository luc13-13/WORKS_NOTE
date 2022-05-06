# SpringBoot + MybatisPlus + Shiro + Jwt的认证管理系统

----
##一、 MybatisPlus配置
###1.1 采用com.lc.uid.provider项目中的服务生成 uid
####(1) 指定DO类的主键字段
MybatisPlus的uid生成方式为存储在枚举类IdType中，主要有五种AUTO/NONE/INPUT/ASSIGN_ID/ASSIGN_UUID，
在实体类DO中为主键字段添加注解

```java
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.core.incrementer.IdentifierGenerator;

public class TableDO {
    /**
     *   value与table中主键一致
     *   如果type=AUTO,则需要表中主键为自增
     *   如果type=ASSIGN_ID,则从{@link IdentifierGenerator#nextId(Object)} 获得 {@link Number} 类型的uid.
     * 因此，为使用com.lc.uid.provider提供的uid，需要重写 {@link IdentifierGenerator#nextId(Object)}
     *   同理，如果type=ASSIGN_UUID,则从{@link IdentifierGenerator#nextUUID(Object)} 获得uuid，
     * 区别在于该方法返回的类型为String
     */
    @TableId(value = "id", type = IdType.ASSIGN_ID)
    private Long id;
}
```

----

####(2) 引入com.lc.uid.provider，并重写 IdentifierGenerator.nextId(Object)

```java
import com.baomidou.mybatisplus.core.incrementer.IdentifierGenerator;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

import java.util.Optional;
import java.util.UUID;

/**
 * 目前还没有集成nginx，uid-provider服务只在虚拟机的docker容器中运行，只能通过服务地址访问
 * 后期考虑将uid-provider升级为SpringCloud服务，以feign接口的形式暴露服务
 */
@Component
public class IUidGenerator implements IdentifierGenerator {

    @Autowired
    private RestTemplate restTemplate;


    /**
     * 采用{@link Optional#ofNullable(Object)}的方式判断目标服务是否返回结果
     * 如果请求超时，返回JVM生成的uid {@link UUID#getLeastSignificantBits()}
     * @param entity
     * @return
     */
    @Override
    public Number nextId(Object entity) {
        Long uid = Optional.ofNullable(
                restTemplate.getForObject("ip:port/uid/nextId", Long.class)
        ).orElse(UUID.randomUUID().getLeastSignificantBits());
        return uid;
    }
}
```

###1.2 为时间相关字段设置自动填充
createTime只在记录生成时填充, modifyTime则在记录被修改时填充，利用@TableField注解为DO类属性完成自动填充
####(1)首先修改MybatisPlus的字段填充策略

```java
import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import com.baomidou.mybatisplus.core.metadata.TableInfo;

import org.apache.ibatis.reflection.MetaObject;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.time.ZoneId;
import java.util.List;
import java.util.function.Supplier;

import static TZoneEnum.SHANGHAI;

/**
 *
 */
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    /**
     *   追踪最后的源码发现，插入和更新都最终执行 {@link MetaObjectHandler#strictFill(boolean, TableInfo, MetaObject, List)}
     * strictFill的执行策略是：
     * 1.对metaObject的字段进行插入时，需要字段被注解标记为INSERT或INSERT_UPDATE - @TableField(fill=FieldFill.INSERT)；
     * 2.对metaObject的字段进行更新时，需要字段被注解标记为UPDATE或INSERT_UPDATE - @TableField(fill=FieldFill.INSERT)；
     * 3.strictFill方法中，如果字段存在且类型匹配，则继续用{@link MetaObjectHandler#strictFillStrategy(MetaObject, String, Supplier)}
     * 改方法只有在字段值为null时对字段进行填充，因此，在更新记录时，要将dtModified设置为null
     * @param metaObject
     */
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "dtCreated", LocalDateTime.class,
                LocalDateTime.now(ZoneId.of(SHANGHAI.getZone())));
        this.strictInsertFill(metaObject, "dtModified", LocalDateTime.class,
                LocalDateTime.now(ZoneId.of(SHANGHAI.getZone())));
        // 如果表中没有invalid字段，则不执行这个操作。 0表示invalid 1表示valid
        this.strictInsertFill(metaObject, "invalid", Integer.class, 1);

    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "dtModified", LocalDateTime.class,
                LocalDateTime.now(ZoneId.of(SHANGHAI.getZone())));
    }
}

```


## 二、Shiro
###2.1 用户权限校验过程

```java
package com.lc.blog.shiro.web.controller;

import UserDO;
import UserService;
import UserVO;
import com.luc.framework.core.mvc.WebResult;

import org.apache.catalina.User;
import org.apache.shiro.authz.annotation.RequiresPermissions;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.Collection;
import java.util.Collections;
import java.util.List;
import java.util.stream.Collectors;

import lombok.AllArgsConstructor;

import org.apache.shiro.mgt.SecurityManager;

import ConditionalOnRedisCache;

/**
 * //以测试接口/user/getUserList为例子
 * @author: lucheng
 * @data: 2022/5/1 20:51
 * @version: 1.0
 */
@RestController
@RequestMapping("/user")
@AllArgsConstructor
public class UserController {
    private final UserService userService;

    /**
     * 1、接收到请求后，会校验servlet中是否有cookie和token
     * 2、从token中获取用户账号信息，在数据库中查找用户角色集合和权限集合，这里以Set<String>方式放入shiro.AccountRealm
     * 3、这里的RequiresPermissions("浏览全部内容")，
     * 则是对用户权限集合进行校验，如果没有“浏览全部内容”,则抛出认证失败的异常，捕获到这个异常可以将request重定向
     * 4、具体细节:
     * (1) AccountRealm需要被放入{@link SecurityManager}, 在ShiroConfig中注入时不可以使用new AccountRealm()的形式,
     * 因为AccountRealm中有其它用于校验用户的服务，JwtUtils、UserService等，如果new出来实例的话，Spring将跳过对其的装配，导致其中成员为null。
     * 因此ShiroConfig中需要以自动装配的形式引入AccountRealm
     * (2)token的生成和解析
     * Shiro中有UserNamePassword实现了生成token，我们可以实现其父类构建自定义JwtToken
     * Jwts的token主要包括header、claims、payload、algorithm和key
     * TODO:补充token的生成和解析袭击
     * (3) 目前使用的是EhCacheManager，但是引入spring-boot-stater-data-redis包后，
     * 会触发RedisAutoConfiguration,导致创建了两个cacheManager,但只有一个被消费。
     * 如果只采用一个cacheManager，则需要在创建bean时指定为@Primary。
     * TODO:采用{@link ConditionalOnRedisCache}的方式，将redisCacheManager也进行消费，
     * TODO:或者创建CacheManagerConfig，将可能用到的cacheManager都封装在内部，实现对bean的消费
     * @return
     */
    @GetMapping("/getUserList")
    @RequiresPermissions("浏览全部内容")
    public WebResult<List<UserDO>> getUserList() {
        List<UserDO> userDOList = userService.list();
        return WebResult.successData(userDOList);
    }
}

```