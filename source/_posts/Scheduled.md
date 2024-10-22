---
title:  解析 Spring 计划任务执行多次之谜及解决方案
date: 2022.04.01 
tags: 
    - Java
    - Spring
categories:  
    - Technology
toc: true
mathjax: true 
comments: true
description: 在`Spring`项目中，计划任务（`Scheduled Tasks`）是一种常用的功能，用于定期执行特定操作。然而，在实际使用过程中，可能会遇到计划任务执行多次的现象。本文将深入探讨这一现象背后的原因，并提供有效的解决方案。
---

在`Spring`项目中，计划任务（`Scheduled Tasks`）是一种常用的功能，用于定期执行特定操作。然而，在实际使用过程中，可能会遇到计划任务执行多次的现象。本文将深入探讨这一现象背后的原因，并提供有效的解决方案。


## 一、现象描述

在`Spring`项目中，我们定义了一个计划任务类`ScheduledTaskParent`，以及两个继承该类的子类`FirstChild`和`SecondChild`。

```Java
@Component
public class ScheduledTaskParent {

    @Scheduled(fixedRate = 5000)
    public void performTask() {
        System.out.println("ScheduledTaskParent 执行计划任务");
    }
}

@Component
public class FirstChild extends ScheduledTaskParent {

    @Scheduled(fixedRate = 5000)
    public void firstTask() {
        System.out.println("FirstChild 执行特定的操作");
    }
}

@Component
public class SecondChild extends ScheduledTaskParent {

    @Scheduled(fixedRate = 5000)
    public void secondTak() {
        System.out.println("SecondChild 执行计划任务");
    }
}
```

当应用启动后，发现`ScheduledTaskParent`中的计划任务被执行了多次，具体表现为每个子类实例都执行了父类的计划任务，导致执行次数为子类数目加1。

```Text
FirstChild 执行特定的操作
ScheduledTaskParent 执行计划任务
ScheduledTaskParent 执行计划任务
SecondChild 执行计划任务
ScheduledTaskParent 执行计划任务
```

## 二、原因分析

### 1. 继承关系导致的问题

在`Java`中，子类会继承父类的所有属性和方法。在本例中，`FirstChild`和`SecondChild`继承了`ScheduledTaskParent`类，同时也继承其计划任务方法。由于，`ScheduledTaskParent`和其子类都被标记为`@Component`，`Spring`容器会将它们都注册为`Bean`，并调度其中的计划任务。

根据`Spring`官方文档，组件扫描会自动检测`Bean`，无需手动注册。这意味着，如果父类中有`@Scheduled`注解的方法，它会被视为一个独立的`Bean`并执行，而子类同样会执行这些继承来的方法。

> Spring’s component scanning feature enables you to automatically detect your beans, as opposed to manually registering them. This feature is particularly useful when you are developing larger applications because it lets you define beans with annotations and without having to write any XML configuration.


### 2. Spring 容器管理问题
`Spring`官方文档指出，要使用计划任务功能，需要在配置类中声明`@EnableScheduling`。这会使得`Spring`容器检测所有`Spring`管理的`Bean`中的`@Scheduled`注解，并将它们作为计划任务调度。因此，如果父类和子类都被注册为`Bean`，它们的方法都会被调度，导致重复执行。

> To use this capability, you must declare @EnableScheduling in one of your @Configuration classes. This will enable detection of @Scheduled annotations on any Spring-managed bean in the container.


## 三、解决方案

为了防止计划任务在子类中被重复执行，我们可以在父类中定义一个抽象方法，并在子类中实现具体的计划任务。如下所示：

```Java
public abstract class ScheduledTaskParent {

    public abstract void performTask();
}

@Component
public class FirstChild extends ScheduledTaskParent {

    @Scheduled(fixedRate = 5000)
    @Override
    public void performTask() {
        System.out.println("FirstChild 执行特定的操作");
    }
}

@Component
public class SecondChild extends ScheduledTaskParent {

    @Scheduled(fixedRate = 5000)
    @Override
    public void performTask() {
        System.out.println("SecondChild 执行计划任务");
    }
}
```

通过这种方式，每个子类只负责自己的计划任务，而不会继承父类的计划任务。