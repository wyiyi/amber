---
title: 解析 MyBatis-Plus 中 IService 和 BaseMapper 的关联关系—基础篇
date: 2023.12.01
tags: Mybatis-plus
categories: Technology  
mathjax: true
comments: true
description: BaseMapper 和 IService 怎么用？有何关联？一起使用是否重复了？有一种剪不断理还乱的感觉。
---

## 问题现象描述
- 常规操作：Service 实现类引入 Mapper，Mapper 继承 BaseMapper
- 现在的需求：将集合进行批量更新
- 出现的现象：BaseMapper 不支持批量更新集合的操作的方法
- 继承 IService 即可实现批量更新的操作。

![](https://wyiyi.github.io/amber/contents/2023/BaseMapper.png)

![](https://wyiyi.github.io/amber/contents/2023/IService.png)

BaseMapper 和 IService 怎么用？有何关联？一起使用是否重复了？有一种剪不断理还乱的感觉。

## 引言
[MyBatis-Plus](https://mybatis.plus/guide/) 作为一个优秀的 ORM 框架，致力于简化和提高 Java 应用程序对数据库访问的效率。
在 MyBatis-Plus 中，IService 和 BaseMapper 是两个核心接口，它们分别负责定义 Service 层和 DAO 层的操作方法，相互之间有着紧密的关联。

## BaseMapper 接口
[BaseMapper](https://baomidou.com/pages/49cc81/#mapper-crud-%E6%8E%A5%E5%8F%A3) 接口是 MyBatis-Plus 提供的通用 Mapper 接口，它继承自 MyBatis 的 Mapper 接口，并扩展了一些常用的数据库操作方法。

> 说明:
>
> - 通用 CRUD 封装 `BaseMapper` 接口，为 `Mybatis-Plus` 启动时自动解析实体表关系映射转换为 `Mybatis` 内部对象注入容器
> - 泛型 `T` 为任意实体对象
> - 参数 `Serializable` 为任意类型主键 `Mybatis-Plus` 不推荐使用复合主键约定每一张表都有自己的唯一 `id` 主键
> - 对象 `Wrapper` 为 `条件构造器`


[BaseMapper](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-core/src/main/java/com/baomidou/mybatisplus/core/mapper/BaseMapper.java) 接口的主要作用是定义 DAO 层的数据库操作方法，例如数据的增删改查等。

开发者可以通过继承 BaseMapper 接口，并指定对应的实体类，即可直接使用这些通用方法，无需手动编写SQL语句，从而减少了代码量和重复劳动。

```
public interface BaseMapper<T> extends Mapper<T> {
    // 定义常用的数据库操作方法
    // ...
}
```

## IService 接口
[IService](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-extension/src/main/java/com/baomidou/mybatisplus/extension/service/IService.java) 接口是 MyBatis-Plus 提供的高级 Service 接口，它继承自 Spring 的 [CrudService](https://baomidou.com/pages/49cc81/#service-crud-%E6%8E%A5%E5%8F%A3) 接口，并扩展了一些常用的数据库操作方法。

> 说明:
>
> - 通用 Service CRUD 封装 IService 接口，进一步封装 CRUD 采用 `get 查询单行` `remove 删除` `list 查询集合` `page 分页` 前缀命名方式区分 `Mapper` 层避免混淆，
> - 泛型 `T` 为任意实体对象
> - 建议如果存在自定义通用 Service 方法的可能，请创建自己的 `IBaseService` 继承 `Mybatis-Plus` 提供的基类
> - 对象 `Wrapper` 为 `条件构造器`

IService 接口的主要作用是定义 Service 层的业务逻辑方法，例如数据的增删改查等。

开发者可以通过继承 IService 接口，并指定对应的实体类，即可直接使用这些通用方法，无需手动编写业务逻辑代码，使得代码更加简洁和易于维护。

```
public interface IService<T> extends CrudService<T, Long> {
    // 定义常用的业务逻辑方法
    // ...
}
```

## 引出 ServiceImpl
[ServiceImpl](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-extension/src/main/java/com/baomidou/mybatisplus/extension/service/impl/ServiceImpl.java#L60C22-L60C22)
是 IService 默认实现类，ServiceImpl 是针对业务逻辑层的实现，并调用 BaseMapper 来操作数据库。
传入的参数为 M 和 T：
- M：Mapper 接口类型
- T：对应实体类的类型

```
public class ServiceImpl<M extends BaseMapper<T>, T> implements IService<T> {
    ...
}
```

## 关联
- IService 是对 BaseMapper 的扩展
- IService 提供批量处理操作，BaseMapper 没有（文章开篇图片已说明）
- IService 依赖于 Spring 容器，而 BaseMapper 不依赖
- BaseMapper 可以继承并添加新的数据库操作，IService 要扩展的话还是得调用 Mapper，略显重复

举个例子说明 IService、ServiceImpl、BaseMapper 三者的类关系如下：

```
public interface UserService extends IService<User> {
    ...
}

@Service
public class UserServiceImpl implements UserService {
    @Resource
    UserMapper userMapper;
    ...
}

实现类或者可以写成：
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> {
    ...
}

@Mapper
public interface UserMapper extends BaseMapper<User> {
    ...
}
```

## 既用之，则听之
看大家对于 mybatis plus 的 `BaseMapper`以及 `IService` 以及 `ServiceImpl` 在 [issue](https://github.com/baomidou/mybatis-plus/issues) 中火热讨论，最后还是以官方为主，若有改动文章在做后续调整。

- [关于改进 IService 和 ServiceImpl](https://github.com/baomidou/mybatis-plus/issues/5764)
- [关于 mybatis plus 中的 BaseMapper<T> 以及 IService<T> 以及 ServiceImpl<M extends <Basemapper>，T> 这几个类](https://github.com/baomidou/mybatis-plus/issues/59)
- [mybatis-plus 的一种很别扭的用法](https://github.com/baomidou/mybatis-plus/issues/926)

当你的项目中只有简单的数据库操作，可以只使用 BaseMapper；
若只有简单的业务逻辑，也可以只使用 IService；
若需要批量处理的方法就需要 IService 和 BaseMapper 一起使用。

合理使用 IService 和 BaseMapper 接口，并结合其他 MyBatis-Plus 提供的功能，充分发挥其优势。
