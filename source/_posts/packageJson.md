---
title: npm 规则：版本号的符号含义
date: 2022.06.01
tags: 
   - NodeJs
   - npm
categories: Technology   
mathjax: true 
comments: true
toc: true
description: 最近常遇到因依赖版本不同导致开发环境和测试环境出现种种问题。
---


## 起因
最近常遇到因依赖版本不同导致开发环境和测试环境出现种种问题。

在开发环境 package.json 文件中引入 `"pinyin-match": "^1.1.2"` 的 function(n,e)，正常是可以获取到的两个参数值。

在测试环境报错提示: "e.match is not a function"。

## 排查
猜测测试环境和本地环境不一样，排查打包的文件，vue 打包后会将代码压缩成 css、img、js 等文件， 其中 js 里都是 `chunk-xxx.xxx.js` 和 `app.xxx.js` 文件。

![](https://wyiyi.github.io/amber/contents/node/chunk-js.png)

根据图中提示的 `chunk-cafbd270.1652944515763.js` 文件，匹配到 match 方法提示引用文件的标识：`r("d22a")` ，
在 `app.xxx.js` 文件里全文检索 `d22a` 的定义，然后在 `d22a` 里发现 match 的 `function(n)` 方法只有一个参数，不是两个参数。
才会导致图中报错且测试环境不生效的问题。

## 解决
① 在 package.json 文件中将引用的 `"pinyin-match": "^1.1.2"` 改为 `"pinyin-match": "1.1.2"` 重新打包即可

② 重新 install 指定版本 1.1.2 重新打包即可

为什么去掉 ^ 会好使呢？

## 各种符号的含义
在 **[Node.js](http://nodejs.cn/learn/semantic-versioning-using-npm)** 中所有的版本号都有3个数字：x.y.z。
- 第一个数字是主版本。
- 第二个数字是次版本。
- 第三个数字是补丁版本。

在 package.json 文件中选择要将软件包更新到的版本，npm 设置了一些规则如下：

>`^`：只会执行不更改最左边非零数字的更新。 如果写入的是 ^0.13.0，则当运行 npm update 时，可以更新到 0.13.1、0.13.2 等，但不能更新到 0.14.0 或更高版本。 如果写入的是 ^1.13.0，则当运行 npm update 时，可以更新到 1.13.1、1.14.0 等，但不能更新到 2.0.0 或更高版本。
>
>`~`：如果写入的是 〜0.13.0，则当运行 npm update 时，会更新到补丁版本：即 0.13.1 可以，但 0.14.0 不可以。
>
>`>`：接受高于指定版本的任何版本。
>
>`>=`：接受等于或高于指定版本的任何版本。
>
>`<=`：接受等于或低于指定版本的任何版本。
>
>`<`：接受低于指定版本的任何版本。
>
>`=`：接受确切的版本。
>
>`-`：接受一定范围的版本。例如：2.1.0 - 2.6.2。
>
>`||`：组合集合。例如 `<` 2.1 || `>` 2.6。

## 在 package.json 文件应规范好所引用的版本，避免出现版本不一致导致的问题！
