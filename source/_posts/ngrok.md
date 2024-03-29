---
title: 内网穿透？试试 ngrok
date: 2022.01.01 
tags: Ngrok
categories: Technology  
mathjax: true
comments: true
description: 提到内网穿透，首先想到的可能是花生壳，但很多年前的一次使用经历，感觉不是一般的费劲。如果有类似的需求，可以试试 ngrok。
---

提到内网穿透，首先想到的可能是花生壳，但很多年前的一次使用经历，感觉不是一般的费劲。

如果有类似的需求，可以试试 [ngrok][ngrok] 。

1. 到 [下载页面][download] 下载客户端压缩包，并解压，如 `unzip /path/to/ngrok.zip`
2. 启动服务，转发到本地端口（如 8888） `./ngrok http 8888`

```bash
ngrok by @inconshreveable                                                     (Ctrl+C to quit)

Session Status                online
Session Expires               1 hour, 59 minutes
Version                       2.3.35
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://e9f8c28f1c08.ngrok.io -> http://localhost:8888
Forwarding                    https://e9f8c28f1c08.ngrok.io -> http://localhost:8888
```

之后便可根据 console 中提示的 `*.ngrok.io` 的地址，从外网通过 http 或 https 方式访问本地服务。

从上面的提示信息可以看到，这个地址的有效期是 2 小时。若要获得更长的有效期，可以选择注册账号，使用免费服务或者付费服务。

[创建账号][login] 后从 [引导界面][setup] 获得 auth token，之后通过 `./ngrok authtoken <your_auth_token>` 将 token 信息写入 `~/.ngrok2/ngrok.yml` 文件中。再启动 ngrok 服务后，可以看到 `Session Expires` 消失。

除了 Console UI 外，还可以通过服务启动后提示的 Web Interface 地址，访问自带的 Web 界面，如 http://127.0.0.1:4040 。在界面中，可以查看更多请求信息，以及进行请求的重放等。

更多用法，可参见 [官方文档][docs] 。

原文地址：https://alphahinex.github.io/2021/02/06/ngrok/

[ngrok]:https://ngrok.com/
[download]:https://ngrok.com/download
[login]:https://dashboard.ngrok.com/login
[setup]:https://dashboard.ngrok.com/get-started/setup
[docs]:https://ngrok.com/docs
