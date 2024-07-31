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

## 摘要

在 `Web` 开发过程中，文件上传功能是常见的需求。对于开发者而言，确保上传功能的稳定性和性能至关重要。
本文将带你了解如何使用`ApacheBench`，对文件上传功能进行性能测试，以评估和提高服务器在高并发情况下的处理能力。

## ApacheBench 简介

`ApacheBench`（简称`ab`）是 `Apache` 服务器自带的一个性能测试工具，它能够模拟多用户并发请求，从而评估服务器在高负载下的性能表现。
若系统中未安装 `Apache` 服务器，可前往 [Apache 官网](https://httpd.apache.org/)下载和安装，请参考：[ApacheBench 简介](https://mp.weixin.qq.com/s/5lqaOphTwsWhGHT-VSH0Tg)。

在操作系统中，可按以下方式打开命令行工具：

- 在 `Windows` 系统中，可以使用 `CMD` 或 `PowerShell`
- 在 `Linux` 系统中，可以使用终端

## ApacheBench 文件上传测试脚本构建步骤

### 1. 准备测试文件

首先，需要准备一个用于上传的文件。如：创建一个名为`test.jpg`的图片文件作为测试对象。

### 2. 编写测试脚本文件

创建一个`test.txt`文件，该文件是一个包含`HTTP`请求的文本文件，需要按照规范在文件中定义好请求体信息前半部分，如：

```
 -----WebKitFormBoundary7MA4YWxkTrZu0gW Content-Disposition: form-data; name="file"; filename="test.txt"
```

### 3. 添加文件内容到测试脚本

使用以下命令将`test.jpg`文件的内容追加到`test.txt`文件中，并正确添加请求结束标记。

```
# 将 test.jpg 文件内容追加到 test.txt 
$ cat test.jpg >> test.txt

# 将文件结尾添加到 test.txt
$ echo -e "\r\n-----WebKitFormBoundary7MA4YWxkTrZu0gW--\r\n" >> 
```

完成后的`test.txt`文件结构如下：

```
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="test.txt"

[文件内容]
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

## 执行文件上传测试命令

使用以下命令执行文件上传测试：

```
$ ab -n 1 -c 1 -p upload.txt -T "multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW" http://localhost:8080/upload
```

其中：[ApacheBench 简介](https://mp.weixin.qq.com/s/5lqaOphTwsWhGHT-VSH0Tg) 具体介绍了执行命令中参数及返回参数的含义：
* -n 表示请求次数
* -c 表示并发数
* -T 表示请求类型
* -p 表示上传文件的路径
* 最后一个参数为上传接口的 URL
* ...

## 文件上传的 Base64 编码问题
在文件上传过程中，使用`Base64`编码传输文件可能会导致接收端接收到的是`Base64`编码的文本内容，而不是原始文件。为了避免这个问题，建议使用[流的形式进行文件上传](https://gist.github.com/chiller/dec373004894e9c9bb38ac647c7ccfa8)。

### 使用流形式上传文件
流形式上传文件可以避免`Base64`编码带来的额外处理开销，并且能够直接传输原始文件数据。

## 测试结果分析

```
Server Software:
Server Hostname:        localhost
Server Port:            8080

Document Path:          /upload
Document Length:        435 bytes

Concurrency Level:      10
Time taken for tests:   0.021 seconds
Complete requests:      100
Failed requests:        0
Non-2xx responses:      100
Total transferred:      59000 bytes
Total body sent:        94600
HTML transferred:       43500 bytes
Requests per second:    4656.58 [#/sec] (mean)
Time per request:       2.147 [ms] (mean)
Time per request:       0.215 [ms] (mean, across all concurrent requests)
Transfer rate:          2682.99 [Kbytes/sec] received
                        4301.88 kb/s sent
                        6984.87 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.3      0       1
Processing:     0    2   0.9      2       5
Waiting:        0    2   0.8      1       4
Total:          0    2   0.9      2       5
```

可看出：

- 每秒请求数（Requests per second）: 4656.58 [#/sec] (mean)
- 请求平均响应时间（第一个 Time per request）： 2.147 [ms] (mean)

## 总结

`Apache Bench` 和 `Postman` 各有所长，开发者可以根据实际需求选择合适的工具进行文件上传测试。

在开发过程中，`Postman` 可以帮助开发者快速调试接口； 而在评估服务器性能时，`Apache Bench`则能提供更准确的测试数据。

掌握这两种工具的使用，将有助于提高`Web`应用的质量和稳定性。