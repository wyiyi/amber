---
title: Mybatis-Plus更新对象时字段更新为空值
date: 2022.08.01 
tags: Mybatis-plus
categories: Technology  
mathjax: true
comments: true
description: 解决Mybatis-Plus更新对象时字段更新为空值的问题
---

## 场景：
mybatis-plus 执行更新操作，将某些字段值置为空，持久层执行后，需要更新为空值的字段仍然保持原本的值。 
显然和我们预期的结果不一致。 


## 解决方案：
在 [mybatis-plus 官网]() 的 @tablefield 注解中提供了该问题的解决方案。

### 1、updateStrategy
在 @tablefield 注解中有属性：`updateStrategy` ，字段验证策略，默认值为：`DEFAULT`

### 2、[FieldFill](https://www.mybatis-plus.com/guide/annotation.html#fieldfill)
在 @tablefield 注解中有属性：`fill` ，字段自动填充策略，默认值为：`FieldFill.DEFAULT`

|       值       |     描述     |
|:-------------:|:----------:|
|    DEFAULT    |   默认不处理    |
|    INSERT     |  插入时填充字段   |
|    UPDATE     |  更新时填充字段   |
| INSERT_UPDATE | 插入和更新时填充字段 |

### 3、[FieldStrategy](https://www.mybatis-plus.com/guide/faq.html#%E6%8F%92%E5%85%A5%E6%88%96%E6%9B%B4%E6%96%B0%E7%9A%84%E5%AD%97%E6%AE%B5%E6%9C%89-%E7%A9%BA%E5%AD%97%E7%AC%A6%E4%B8%B2-%E6%88%96%E8%80%85-null)

> FieldStrategy 有三种策略：
> - IGNORED：忽略
> - NOT_NULL：非 NULL，默认策略
> - NOT_EMPTY：非空
>
> 当用户有更新字段为 空字符串 或者 null 的需求时，需要对 FieldStrategy 策略进行调整：
>
> - 方式一：调整全局的验证策略
>
>   注入配置 GlobalConfiguration 属性 fieldStrategy
>
>
>- 方式二：调整字段验证注解
>
>  根据具体情况，在需要更新的字段中调整验证注解，如验证非空：
>
>
>     @TableField(strategy=FieldStrategy.NOT_EMPTY)
>
> - 方式三：使用 UpdateWrapper (3.x)
>
>   使用以下方法来进行更新或插入操作：
>
>
>     // updateAllColumnById(entity) // 全部字段更新: 3.0已经移除
>
>     mapper.update(
>       new User().setName("mp").setAge(3),
>       Wrappers.<User>lambdaUpdate()
>               .set(User::getEmail, null) //把email设置成null
>               .eq(User::getId, 2)
>     );
>     //也可以参考下面这种写法
>     mapper.update(
>      null,
>      Wrappers.<User>lambdaUpdate()
>              .set(User::getAge, 3)
>              .set(User::getName, "mp")
>              .set(User::getEmail, null) //把email设置成null
>              .eq(User::getId, 2)
>     );

## 应用到我的问题上：
1、使用 updateStrategy 属性
```
@ApiModelProperty(value = "结束时间")
@TableField(value = "ENDDATE", updateStrategy=FieldStrategy.IGNORED)
private Date enddate;
```

2、 使用 FieldFill 属性
```
@ApiModelProperty(value = "结束时间")
@TableField(value = "ENDDATE", fill = FieldFill.UPDATE)
private Date enddate;
```

3.1、针对 FieldStrategy 的方式一： 在 yml 文件中全局配置为：
```
mybatis-plus:
  global-config:
    db-config:
      update-strategy: IGNORED
```

3.2、针对 FieldStrategy 的方式二：
>
>     /**
>      * 字段验证策略
>      * <p>默认追随全局配置</p>
>      *
>      * @deprecated 3.1.2 , to use {@link #insertStrategy} and {@link #updateStrategy} and {@link #whereStrategy}
>      */
>     @Deprecated
>     FieldStrategy strategy() default FieldStrategy.DEFAULT;

在3.1.2版本后 strategy 方法被启用，更新为 insertStrategy、updateStrategy和whereStrategy 三个，
所以采用 `updateStrategy=FieldStrategy.IGNORED` 属性代替 `strategy=FieldStrategy.NOT_EMPTY` 方式。

3.3、针对 FieldStrategy 的方式三：
按照方式三的示例，UpdateWrapper中设置查询条件，再通过实体类去设置要修改的值。

## 当使用Mybatis-Plus更新对象操作时，可以参考以上方式！