[//]: # (---)

[//]: # (title: 解析 MyBatis-Plus 中 IService 和 BaseMapper 的关联关系—基础篇)

[//]: # (date: 2023.12.01)

[//]: # (tags: Mybatis-plus)

[//]: # (categories: Technology  )

[//]: # (mathjax: true)

[//]: # (comments: true)

[//]: # (description: Mapper 和 IService 有何关联？单独使用？一起使用是否重复了？)

[//]: # (---)

## 问题现象描述
- 常规操作：Service 实现类引入 Mapper，Mapper 继承 BaseMapper
- 现在的需求：将集合进行批量更新
- 出现的现象：BaseMapper 不支持批量更新集合的操作
- 继承 IService 即可实现批量更新的操作。

![](https://wyiyi.github.io/amber/contents/2023/BaseMapper.png)

![](https://wyiyi.github.io/amber/contents/2023/IService.png)

BaseMapper 和 IService 有何关联？单独使用？一起使用是否重复了？

## 引言
[MyBatis-Plus](https://mybatis.plus/guide/) 作为一个优秀的 ORM 框架，致力于简化和提高 Java 应用程序对数据库访问的效率。
在 MyBatis-Plus 中，IService 和 BaseMapper 是两个核心接口，它们分别负责定义 Service 层和 DAO 层的操作方法，相互之间有着紧密的关联。

## IService 接口
[IService](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-extension/src/main/java/com/baomidou/mybatisplus/extension/service/IService.java) 接口是 MyBatis-Plus 提供的高级 Service 接口，它继承自 Spring 的 CrudService 接口，并扩展了一些常用的数据库操作方法。
IService 接口的主要作用是定义 Service 层的业务逻辑方法，例如数据的增删改查等。

开发者可以通过继承 IService 接口，并指定对应的实体类，即可直接使用这些通用方法，无需手动编写业务逻辑代码，使得代码更加简洁和易于维护。

```
public interface IService<T> extends CrudService<T, Long> {
    // 定义常用的业务逻辑方法
    // ...
}
```

## BaseMapper 接口
[BaseMapper](https://github.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-core/src/main/java/com/baomidou/mybatisplus/core/mapper/BaseMapper.java) 接口是 MyBatis-Plus 提供的通用 Mapper 接口，它继承自 MyBatis 的 Mapper 接口，并扩展了一些常用的数据库操作方法。
BaseMapper 接口的主要作用是定义 DAO 层的数据库操作方法，例如数据的增删改查等。

开发者可以通过继承 BaseMapper 接口，并指定对应的实体类，即可直接使用这些通用方法，无需手动编写SQL语句，从而减少了代码量和重复劳动。

```
public interface BaseMapper<T> extends Mapper<T> {
    // 定义常用的数据库操作方法
    // ...
}
```

## IService 与 BaseMapper 的关联关系
在 MyBatis-Plus中，IService 与 BaseMapper 之间通过继承关系建立了联系。
IService 接口继承了 CrudService 接口，而 CrudService 又继承了 BaseService 接口，
最终 BaseService 接口继承了 BaseMapper 接口。

```
public interface UserService extends IService<User> {
    // 可以直接使用CrudService和BaseService提供的方法
    // ...
}
```
通过以上定义，UserService 接口就可以直接使用 IService、CrudService 和 BaseService 提供的常用数据库操作方法，无需再重复定义这些方法，大大简化了代码编写和维护的工作量。

## 同时使用
IService 接口中定义的方法主要是针对业务逻辑的操作，
而 BaseMapper 接口中定义的方法主要是针对数据库的操作。
通过将两者结合使用，可以实现数据访问层和业务逻辑层的解耦，使得代码更加清晰和可扩展。

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

@Mapper
public interface UserMapper extends BaseMapper<User> {
    ...
}
```

## 提示
当你的项目中只有简单的数据库操作，可以只使用 BaseMapper；
反之，如果只有简单的业务逻辑，也可以只使用 IService。
我们应该根据具体业务需求，合理使用 IService 和 BaseMapper 接口，并结合其他 MyBatis-Plus 提供的功能，充分发挥其优势。
