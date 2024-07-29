---
title: 使用 Postman 和 ApacheBench 上传文件至服务器
date: 2024.08.01
tags:
  - Postman
  - Apache Bench
categories: Technology
mathjax: true
comments: true
toc: true
description: 使用Postman和ApacheBench上传文件至服务器
---

## 引言

在 `Web` 开发过程中，文件上传功能是常见的需求。
为了确保文件上传接口的稳定性和性能，常常需要对其进行测试。
`Postman` 和 `ApacheBench` 是两款常用的接口测试工具，本文将分别介绍如何使用它们上传文件。

## 使用 Postman 上传文件

打开 [Postman](https://www.postman.com/)：
- 在地址栏输入上传接口的 `URL`，如：`http://localhost:8080/upload`，
- 选择请求类型： `POST` ，
- 在 `Headers` 部分添加一个键值对，键名为 `Content-Type`，值为 `multipart/form-data`，
- 在 `Body` 部分选择 `form-data` 类型，然后添加一个键值对，键名为 `file`，选择要上传的文件。

发送请求，`Postman` 将自动上传文件至服务器，在 `Response`部分，可以查看服务器返回的结果。

## 使用 ApacheBench 上传文件

`ApacheBench` 是 `Apache` 服务器自带的一个压力测试工具，若未安装 `Apache` 服务器，可前往 [Apache 官网](https://httpd.apache.org/)下载安装，
具体安装请参考：[ApacheBench 简介](https://mp.weixin.qq.com/s/5lqaOphTwsWhGHT-VSH0Tg)。

打开命令行工具：
- 在 `Windows` 系统中，可以使用 `CMD` 或 `PowerShell`
- 在 `Linux` 系统中，可以使用终端

1、创建一个名为`upload.txt`的文件，内容如下：
```
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="example.jpg"
Content-Type: image/jpeg

<文件内容>
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```
【注意】：文件内容需替换为实际要上传的文件内容。

2、执行命令：
```
$ ab -n 1 -c 1 -T 'multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW' -p upload.txt http://localhost:8080/upload
```

其中：[ApacheBench 简介](https://mp.weixin.qq.com/s/5lqaOphTwsWhGHT-VSH0Tg) 具体介绍了执行命令中参数及返回参数的含义：
* -n 表示请求次数
* -c 表示并发数
* -T 表示请求类型
* -p 表示上传文件的路径
* 最后一个参数为上传接口的 URL
* .....

3、发送请求：

在命令行工具中执行命令，`ApacheBench` 将自动上传文件至服务器，在命令行输出中，可以查看服务器返回的结果：
- 每秒请求数（Requests per second）: 73.96 [#/sec] (mean)
- 请求平均响应时间（第一个 Time per request）：13.521 [ms] (mean)

```
Server Software:
Server Hostname:        localhost
Server Port:            8080


Document Path:          /doc.html#/default/file-upload-controller/uploadFile
Document Length:        435 bytes


Concurrency Level:      1
Time taken for tests:   0.014 seconds
Complete requests:      1
Failed requests:        0
Non-2xx responses:      1
Total transferred:      590 bytes
Total body sent:        25478
HTML transferred:       435 bytes
Requests per second:    73.96 [#/sec] (mean)
Time per request:       13.521 [ms] (mean)
Time per request:       13.521 [ms] (mean, across all concurrent requests)
Transfer rate:          42.61 [Kbytes/sec] received
                        1840.16 kb/s sent
                        1882.78 kb/s total


Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.0      0       0
Processing:    14   14   0.0     14      14
Waiting:       14   14   0.0     14      14
Total:         14   14   0.0     14      14
```

## 总结

本文介绍了如何使用 Postman 和 ApacheBench 上传文件至服务器。

通过实际操作，可以发现，Postman 操作简单，界面友好，适合开发和调试阶段使用；而 `ApacheBench` 则更适合进行性能测试。

掌握这两种工具的使用，有助于我们更好地进行文件上传接口的测试。