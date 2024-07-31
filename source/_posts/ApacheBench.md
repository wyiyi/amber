---
title: 如何构建 ApacheBench 请求体上传文件
date: 2024.08.01
tags:
  - Postman
  - Apache Bench
categories: Technology
mathjax: true
comments: true
toc: true
description: 如何构建 ApacheBench 请求体上传文件
---

## 摘要

在 `Web` 开发过程中，文件上传功能是常见的需求。对于开发者而言，确保上传功能的稳定性和性能至关重要。
本文将带你了解如何使用`ApacheBench`文件上传功能进行性能测试，以评估和提高服务器在高并发情况下的处理能力。
特别是如何按照规范构造上传文件的请求体。

## 引言
![](https://wyiyi.github.io/amber/contents/2024/apache-bench.png)

通过 `Postman` 工具，我们可以直观地看到上传文件的请求体部分。
然而，当使用 `Apache Bench` 上传文件时，直接将文件内容以 `Base64` 编码后写入请求体并不会生效。
那么，我们该如何构造一个与 `Postman` 中相同的请求体呢？

## ApacheBench 简介

`ApacheBench`（简称`ab`）是 `Apache` 服务器自带的一个性能测试工具，它能够模拟多用户并发请求，从而评估服务器在高负载下的性能表现。
若系统中未安装 `Apache` 服务器，可前往 [Apache 官网](https://httpd.apache.org/)下载和安装。
更多关于 ApacheBench 的信息，请参考：[ApacheBench 简介](https://mp.weixin.qq.com/s/5lqaOphTwsWhGHT-VSH0Tg)。

在操作系统中，可按以下方式打开命令行工具：

- 在 `Windows` 系统中，可以使用 `CMD` 或 `PowerShell`
- 在 `Linux` 系统中，可以使用终端

## 构造请求体规范

在构造文件上传请求体时，我们需要遵循 `multipart/form-data` 协议的要求。
这种协议通常用于上传文件，它将请求体分为多个部分，每个部分都包含一个特定的头部信息，这些头部信息定义了每个部分的内容。
以下是构造请求体的规范步骤：

* 定义边界：首先，我们需要定义一个边界字符串，这个字符串将用于分隔请求体的各个部分。边界字符串应该是一个唯一的字符串，通常由 `WebKitFormBoundary` 开头，后面跟上一系列随机字符。
* 设置 `Content-Type`：在请求头中，我们需要设置 `Content-Type` 为 `multipart/form-data`，并使用 `boundary` 参数引用我们定义的边界字符串。
* 构造请求体：请求体由多个部分组成，每个部分以边界字符串开始，以 `--` 字符串结束。每个部分都包含一个头部，定义了该部分的内容类型、名称、文件名等信息，然后是文件内容的 `Base64` 编码。
* 添加文件内容：将文件内容转换为 `Base64` 编码，并将其添加到请求体的相应部分中。
* 添加结束标记：在最后一个部分之后，我们需要添加一个结束标记，表示请求体的结束。
* 检查完整性：确保所有部分都正确无误，并且边界字符串在请求体中正确使用。

## 构造请求体步骤

### 1. 准备测试文件

首先，需要准备一个用于上传的文件。如：创建一个名为`test.jpg`的图片文件作为测试对象。

### 2. 编写测试脚本文件
创建一个`test.txt`文件，该文件包含`HTTP`请求的 `body` 部分，在 `Postman` 图中右侧我们可以看到请求体部分包含了
`Content-Disposition`为`form-data`，`name`为`file`，`filename`为`test.jpg`和`Content-Type`为`image/jpeg`等信息。

以下是如何定义请求体信息前半部分的示例：

```
 ------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="test.jpg"
Content-Type: image/jpeg
```

### 3. 添加文件内容到测试脚本

接下来，我们需要按照规范将文件内容添加到请求体中，并正确添加请求结束标记。

#### 使用 Windows 系统
在 `git bash` 或其他命令行工具中，执行以下命令：

```
# 将 test.jpg 文件内容追加到 test.txt
$ cat test.jpg >> test.txt

# 将文件结尾添加到 test.txt
$ echo -e "\r\n------WebKitFormBoundary7MA4YWxkTrZu0gW--\r\n" >> test.txt
```

#### 使用 Linux 系统
在 `Linux` 系统中，可以使用以下命令：

```
# 将 test.jpg 文件内容追加到 test.txt
$ cat test.jpg >> test.txt

# 将文件结尾添加到 test.txt
$ echo -e "\n------WebKitFormBoundary7MA4YWxkTrZu0gW--\n" >> test.txt
```

确保在 `test.txt` 文件中的 `Content-Type` 和 `filename` 与实际文件类型和名称相匹配。

## 执行文件上传测试命令

使用以下命令执行文件上传测试：

```
$ ab -n 1 -c 1 -p test.txt -T "multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW" http://localhost:8080/upload
```

其中：[ApacheBench 简介](https://mp.weixin.qq.com/s/5lqaOphTwsWhGHT-VSH0Tg) 具体介绍了执行命令中参数及返回参数的含义：
* -n 表示请求次数
* -c 表示并发数
* -T 表示请求类型
* -p 表示上传文件的路径
* 最后一个参数为上传接口的 URL
* ...

## 测试结果分析

执行命令后，`ApacheBench` 输出测试结果，包括每秒请求数、请求平均响应时间等关键指标。
这些数据将帮助你分析文件上传的性能。

```
This is ApacheBench, Version 2.3 <$Revision: 1901567 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient)...INFO: POST header ==
---
POST /upload HTTP/1.0
Content-length: 24223
Content-type: multipart/form-data; boundary=---WebKitFormBoundary7MA4YWxkTrZu0gW
secret-id: 3e6bec50-3bbd-4443-b141-82aaee645cc7
secret-key: f1596671ce4d30a7847a91b8c674f43c
Host: localhost:8080
User-Agent: ApacheBench/2.3
Accept: */*


---
LOG: header received:
HTTP/1.1 200
Content-Type: text/plain;charset=UTF-8
Content-Length: 113
Date: Wed, 31 Jul 2024 13:37:18 GMT
Connection: close

File uploaded successfully: 123.txt with size: 24072 bytes and content written to: C:\Users\admin\Desktop\
..done


Server Software:
Server Hostname:        localhost
Server Port:            8080

Document Path:          /upload
Document Length:        113 bytes

Concurrency Level:      1
Time taken for tests:   0.056 seconds
Complete requests:      1
Failed requests:        0
Total transferred:      247 bytes
Total body sent:        24512
HTML transferred:       113 bytes
Requests per second:    17.73 [#/sec] (mean)
Time per request:       56.388 [ms] (mean)
Time per request:       56.388 [ms] (mean, across all concurrent requests)
Transfer rate:          4.28 [Kbytes/sec] received
                        424.51 kb/s sent
                        428.79 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.0      0       0
Processing:    56   56   0.0     56      56
Waiting:       53   53   0.0     53      53
Total:         56   56   0.0     56      56

```