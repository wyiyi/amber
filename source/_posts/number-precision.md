---
title: 精度丢失的解决办法
date: 2021.03.25 
tags: Others
categories: 
  - Technology
mathjax: true 
---

# JS 中保留固定位数以及四舍五入
在保留有效数字的时候我们经常会使用到toFixed()函数，但发现这个方法会存在一些奇怪的问题。

## toFixed() 的值错误？
在JS中会有如下的现象，我们需要对最后的结果值进行保留固定位数且四舍五入处理，但发现结果不是所期望的。
```
1.5.toFixed(0) // 2 正确
1.35.toFixed(1) // 1.4 正确
1.335.toFixed(2) // 1.33  错误
1.3335.toFixed(3) // 1.333 错误
1.33335.toFixed(4) // 1.3334 正确
1.333335.toFixed(5)  // 1.33333 错误
1.3333335.toFixed(6) // 1.333333 错误
```
发现元凶：小数点位数为1，2，5时四舍五入是正确的，其它是错误的。
根据 [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/toFixed#description) 手册的 Warning 中也表示浮点数不能以二进制精确表示所有小数。这可能会导致意外结果。
~~~~
Warning: Floating point numbers cannot represent all decimals precisely in binary. 
This can lead to unexpected results, such as 0.1 + 0.2 === 0.3 returning false .
 ~~~~

## 为什么会导致 toFixed() 的值不准确呢？
~~~~
Number.prototype.toFixed(fractionDigits)
NOTE 1 : 
    toFixed returns a String containing this Number value represented in decimal fixed-point notation with fractionDigits digits after the decimal point. If fractionDigits is undefined, 0 is assumed.

The following steps are performed:

    1. Let x be thisNumberValue(this value).
    2. ReturnIfAbrupt(x).
    3. Let f be ToInteger(fractionDigits). (If fractionDigits is undefined, this step produces the value 0).
    4. ReturnIfAbrupt(f).
    5. If f < 0 or f > 20, throw a RangeError exception. However, an implementation is permitted to extend the behaviour of toFixed for values of f less than 0 or greater than 20. In this case toFixed would not necessarily throw RangeError for such values.
    6. If x is NaN, return the String "NaN".
    7. Let s be the empty String.
    8. If x < 0, then
        Let s be "-".
        Let x = –x.
    9. If x ≥ 10^21, then
        Let m = ToString(x).
    10.Else x < 1021,
         a. Let n be an integer for which the exact mathematical value of n ÷ 10f – x is as close to zero as possible. If there are two such n, pick the larger n.
         b. If n = 0, let m be the String "0". Otherwise, let m be the String consisting of the digits of the decimal representation of n (in order, with no leading zeroes).
         c. If f ≠ 0, then
            i.  Let k be the number of elements in m.
            ii. If k ≤ f, then
                1. Let z be the String consisting of f+1–k occurrences of the code unit 0x0030.
                2. Let m be the concatenation of Strings z and m.
                3. Let k = f + 1.
            iii. Let a be the first k–f elements of m, and let b be the remaining f elements of m.
            iv.  Let m be the concatenation of the three Strings a, ".", and b.
    11. Return the concatenation of the Strings s and m.
~~~~
根据[规范算法](https://262.ecma-international.org/6.0/#sec-number.prototype.tofixed) 我们将 `1.335.toFixed(2)` 值代入:
~~~~
1.令x为1.335，
2.突然完成则返回参数 （不满足）
3.令f为2，如果f是undefined 会产生0 （不满足）
4.突然完成则返回参数 （不满足）
5.如果f < 0 或f > 20， 会抛出异常，也可以扩展f <0 或者f>20 toFixed() 的值，由于f=2 所以不会异常 （不满足）
6.如果x是NaN，则就会返回字符串型的NaN （不满足）
7.s 为空的字符串，
8.如果x<0，（不满足）
   a.则s变成"-"
   b.x = -x
9. 如果x>=10^21 则令m=字符串 （不满足）
10.否则x<10^21
   由于x=1.335 满足 1.335<10^21
   a. n 为整数： 假设 n = 132 或者 133 或者 134 或者 135 时， n/10^2 - 1.335 尽可能接近于0，如果存在两个这样的n，选择较大的n 
       ' 132/100-1.335 ' --- - 0.014999999999999902
       ' 133/100-1.335 ' --- - 0.004999999999999893
       ' 134/100-1.335 ' --- 0.0050000000000001155
       ' 135/100-1.335 ' --- 0.015000000000000124
       当n的值为133 和 134 最接近0，选择n/10^2 - 1.335较大的值  所以 n = 134
   b. 如果n = 0，则让m为 String "0"，否则，让m是由n的十进制表示的数字组成的字符串（按顺序，没有前导零），所以 m = 134  
   c. 如果f ≠0， 则
       i. k为m的元素数，所以k = 3，
      ii. 如果k<f 则 （k=3，f=2 不满足）
          1.设z是由代码单元 0x0030的f +1– k次出现组成的字符串。
          2.让m是字符串z和m的串联。
          3.令k = f + 1。
      iii. a为m的k-f个元素 所以a = 1，b为m的余下个元素 所以b = 34，
       iv. m 为a.b 所以m = 1.34，
11.返回字符串s和m的串联 s+m = 1.34
所以1.335.toFixed(2)的值为1.34
~~~~

1.335.toFixed(2) = 1.34 是因为我们按照正常的十进制方式计算得出结果为 `1.34`
而 在js 中我们看到的是 `1.335.toFixed(2) // 1.33  错误` ，是因为在计算机中使用的二进制计算，[文章：十进制与二进制数转换](https://alphahinex.github.io/2021/03/28/float-binary-conversion/) 中已介绍过在计算的过程中会存在丢失精度的问题。 
这也就是toFixed()会存在错误值的原因。


## 怎么可以既保留固定位数又可以进行四舍五入呢？
一： 转换为整数再处理，避免浮点数精度的损失
```
function toFixed(num, m) {
    var des =  Math.round(Math.pow(10, m) * num) / Math.pow(10, m);
    return des;
}
console.log(toFixed(1.333335, 5))
```

二： 数值加上最小偏差 `Number.EPSILON`

ES6 在 Number 对象上新增了一个极小的常量：[Number.EPSILON](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/EPSILON) ，可以利用这个常量，使正确的计算结果更接近于0。

```
function toFixed(num) {
    var des =  (num + Number.EPSILON).toFixed()
    return des;
}
console.log(toFixed(1.333335)) 
```