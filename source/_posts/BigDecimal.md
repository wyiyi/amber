---
title:  BigDecimal 的 equals() 和 compareTo() 的用法与区别
date: 2022.04.01 
tags: 
    - Java
    - BigDecimal
categories:  
    - Technology
toc: true
mathjax: true 
comments: true
description: 在日常涉及金额和数量时离不开 BigDecimal 类型。在 java.math.BigDecimal API中可以看到 提供了很多方法，除了加减乘除计算外，下面说两个常用且需谨慎的两个方法：compareTo()、equals()。
---

## BigDecimal 常使用方法
在日常涉及金额和数量时离不开 BigDecimal 类型。
在 java.math.BigDecimal API中可以看到 提供了很多方法，除了加减乘除计算外，
下面说两个常用且需谨慎的两个方法：compareTo()、equals()。

## 比较大小 compareTo(BigDecimal val)
### 两个数比较大小：
```
a.compareTo(b) < 0  // a小于b
a.compareTo(b) == 0 // a等于b
a.compareTo(b) > 0  // a大于b
```
### 与 0 比较：    
BigDecimal 提供与个别数字比较，
例如：BigDecimal.ZERO、BigDecimal.ONE、BigDecimal.TEN...
```
a.compareTo(BigDecimal.ZERO) > 0 // a大于0
```
## 是否相等 equals(Object x)
```
* Compares this {@code BigDecimal} with the specified
* {@code Object} for equality.  Unlike {@link
* #compareTo(BigDecimal) compareTo}, this method considers two
* {@code BigDecimal} objects equal only if they are equal in
* value and scale (thus 2.0 is not equal to 2.00 when compared by
* this method).
```
在源码注释中能看到 equals() 方法不像 compareTo() 方法，equals() 方法考虑到value（值）和 scale（精度）两个因素。

```
BigDecimal a = new BigDecimal(0.1);
BigDecimal b = new BigDecimal(0.10);
assert a.equals(b);

BigDecimal c = new BigDecimal("0.1");
BigDecimal d = new BigDecimal("0.10");
assert !c.equals(d);
```

我们发现 BigDecimal 的 equals() 方法对0.1和0.10进行比较的时候：

- 当使用int、long、double 定义 BigDecimal时，结果为 true

- 当使用 String 定义 BigDecimal 时，结果为 false

### 为什么 a 和 b 相等？ c 和 d 不等呢？

BigDecimal(int)、BigDecimal(long)、BigDecimal(double)、BigDecimal(String) 创建出来的 BigDecimal 的精度是不同的。

①由于 int 和 long 是整数，在下面源码注释中声明它们的精度为 0

```
* Translates an {@code int} into a {@code BigDecimal}.  The
* scale of the {@code BigDecimal} is zero.
```
```
* Translates a {@code long} into a {@code BigDecimal}.  The
* scale of the {@code BigDecimal} is zero.
```

②double 表示的是一个近似值

new BigDecimal(0.1) 或者 new BigDecimal(0.10) 创建的 BigDecimal，创建出来的值并不是正好等于 0.1，
值都是0.1000000000000000055511151231257827021181583404541015625。

在控制台执行发现 `new BigDecimal(0.10).scale == 55` 和 `new BigDecimal(0.1).scale == 55` 精度都是 55，其中 scale（精度）为小数的位数，所以 a 与 b 是相等的。

```
* One might assume that writing {@code new BigDecimal(0.1)} in
* Java creates a {@code BigDecimal} which is exactly equal to
* 0.1 (an unscaled value of 1, with a scale of 1), but it is
* actually equal to
* 0.1000000000000000055511151231257827021181583404541015625.
```
③String: BigDecimal(String val) 
- new BigDecimal("0.1") 创建出来的值正好等于 0.1，精度为 1
- new BigDecimal("0.10") 创建出来的值正好等于 0.10，精度为 2

由于精度不同，所以 c 和 d 是不相等的。

```
* "0"            [0,0]
* "0.00"         [0,2]
* "123"          [123,0]
* "-123"         [-123,0]
* ......
```
### c 和 d 如何相等？

```
* Compares this {@code BigDecimal} with the specified
* {@code BigDecimal}.  Two {@code BigDecimal} objects that are
* equal in value but have a different scale (like 2.0 and 2.00)
* are considered equal by this method.
```
为了使 c 和 d 相等，在 compareTo() 源码注释中可看到，不会比较 scale，而是直接比较值的大小。

``` 
BigDecimal c = new BigDecimal("0.1");
BigDecimal d = new BigDecimal("0.10");
assert c.compareTo(d) == 0;
```

## 小心使用 BigDecimal！