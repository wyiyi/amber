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

在 `Web` 开发过程中，文件上传功能是常见的需求。对于开发者而言，确保上传功能的稳定性和性能至关重要。
本文将带你了解如何使用`Postman` 和 `ApacheBench`进行文件上传测试，并对比两者在实际应用中的优劣。

## ApacheBench：性能测试利器

`ApacheBench` 是 `Apache` 服务器自带的一个性能测试工具，它能够模拟多用户并发请求，从而评估服务器在高负载下的性能表现。
若未安装 `Apache` 服务器，可前往 [Apache 官网](https://httpd.apache.org/)下载安装，请参考：[ApacheBench 简介](https://mp.weixin.qq.com/s/5lqaOphTwsWhGHT-VSH0Tg)。

打开命令行工具：
- 在 `Windows` 系统中，可以使用 `CMD` 或 `PowerShell`
- 在 `Linux` 系统中，可以使用终端

1、编写测试脚本：创建一个文本文件`upload.txt`，包含`HTTP`请求的详细信息，内容如下：
```
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="upload.txt"
Content-Type: text/plain


------WebKitFormBoundary7MA4YWxkTrZu0gW--
```
【注意】：文件内容需替换为实际要上传的文件内容。

2、执行命令：
```
$ ab -n 100 -c 10 -p upload.txt -T "multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW" http://localhost:8080/upload
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
- 每秒请求数（Requests per second）: 4656.58 [#/sec] (mean)
- 请求平均响应时间（第一个 Time per request）： 2.147 [ms] (mean)

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


## Postman：接口调试与测试专家

打开 [Postman](https://www.postman.com/)：
- 在地址栏输入上传接口的 `URL`，如：`http://localhost:8080/upload`，
- 选择请求类型： `POST` ，
- 在 `Headers` 部分添加一个键值对，键名为 `Content-Type`，值为 `multipart/form-data`，
- 在 `Body` 部分选择 `form-data` 类型，然后添加一个键值对，键名为 `file`，选择要上传的文件。

发送请求，`Postman` 将自动上传文件至服务器，在 `Response`部分，可以查看服务器返回的结果。

## 二者对比

| 特性          | Postman                       | ApacheBench |
|:------------|:------------------------------|:-----------|
| 易用性    | 直观的图形界面                       | 命令行工具      |
| 功能丰富度 | 支持多种HTTP请求方法，添加、修改请求参数和Header | 主要用于性能测试，功能相对单一 |
| 测试场景 | 适合进行接口功能测试和调试                 | 适合进行大规模并发测试，评估服务器在高负载下的性能|
| 适用范围 | 适用于接口开发和测试                    |服务器性能评估|

## 总结

`Apache Bench` 和 `Postman` 各有所长，开发者可以根据实际需求选择合适的工具进行文件上传测试。

在开发过程中，`Postman` 可以帮助开发者快速调试接口； 而在评估服务器性能时，`Apache Bench`则能提供更准确的测试数据。

掌握这两种工具的使用，将有助于提高`Web`应用的质量和稳定性。