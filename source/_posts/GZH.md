---
title: 微信公众号自定义菜单消失？
date: 2021.07.31 
tags: 
    - Wechat
    - 公众号
categories: 
  - Technology
mathjax: true 
comments: true
toc: true
description:
---

[微信官方文档](https://developers.weixin.qq.com/doc/offiaccount/Custom_Menus/Creating_Custom-Defined_Menu.html) 中提供添加自定义菜单接口，可以通过**代码方式**实现自定义菜单。

在微信公众平台可以通过页面**增加菜单按钮方式**实现自定义菜单，如图：![](https://wyiyi.github.io/amber/contents/gzh/img.png)

一种通过代码方式，一种通过页面方式，各自使用是没问题。

那么问题来了，现在需要代码和页面同时使用，该怎么设置呢？
在官网中并不支持二者同时使用，但经过一顿操作是可以兼容的。

## 微信后台自定义的又菜单不见了？
当我们在微信公众平台，维护了自定义菜单且配置服务器并启用，服务生效后，发现手机端自定义的菜单却不见了？

根据之前的成功案例，自定义菜单和服务器支持同时使用的方法：

- 禁用服务器配置

- 修改菜单

- 启用服务器配置

- `+添加功能插件` 的自定义菜单，启用

由于微信公众平台更新导致 `+添加功能插件` 菜单消失了，此方法不在适用。

![](https://wyiyi.github.io/amber/contents/gzh/img_1.png)

不能同时用吗？

按照网上办法尝试，不起作用，联系了客服，没有回复，什么情况？

## 如何是好呢？

在 `+ 新的功能` - `自定义菜单` - `详情` ，点击 `开启` 即可。

![](https://wyiyi.github.io/amber/contents/gzh/img_2.png)
![](https://wyiyi.github.io/amber/contents/gzh/img_3.png)

WAIT A MOMENT！

如果还是没有效果，可以根据 [微信自定义菜单接口](https://developers.weixin.qq.com/doc/offiaccount/Custom_Menus/Creating_Custom-Defined_Menu.html)
重新打开或重新关注公众号，就能看到新自定义菜单已经生效。

成功！完美！