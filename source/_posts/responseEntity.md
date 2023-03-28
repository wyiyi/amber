

# ResponseEntity.BodyBuilder 

## 简单说一说 ResponseEntity 
[ResponseEntity](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/ResponseEntity.html) 
继承了 HttpEntity ，增加了 HttpStatusCode 状态码，在 RestTemplate 和 Controller 方法中使用。

> Extension of HttpEntity that adds an HttpStatusCode status code. Used in RestTemplate as well as in @Controller methods.

在下图中能看见 ResponseEntity.class 中所包含得接口以及方法：
![](https://wyiyi.github.io/amber/contents/2023/responseEntity.png)

## 重点来了：ResponseEntity.BodyBuilder 
[ResponseEntity.BodyBuilder](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/ResponseEntity.BodyBuilder.html) builder 是 Spring 框架中的一个类，用于构建响应实体。

在 Spring MVC 中，我们可以使用它来构建 HTTP 响应，包括状态码、头信息和响应体等。

它提供了一些方法来设置响应的各个部分，例如 status()、header() 和 body() 等。

## 用法
在本文中，将介绍如何使用 ResponseEntity.BodyBuilder 来构建响应体。

根据请求的不同情况来设置不同的状态码和响应体，以及添加不同的头信息，并使用 builder.build() 方法来构建最终的 ResponseEntity 对象。

### 创建一个 HTTP 状态码为 200 的响应体
```java
ResponseEntity.BodyBuilder builder = ResponseEntity.ok();
```

### 使用 builder 对象来设置响应体的内容
例如：我们可以使用`builder.body()`方法来设置响应体的主体内容。
示例代码是将一个字符串设置为响应体的主体内容。

```java
builder.body("Hello World");
```

### 使用 builder.header() 方法来设置响应头
自定义设置成响应头：
```java
builder.header("Content-Type", "text/plain");
```

### builder.build() 方法来构建最终的 ResponseEntity 对象

构建一个 HTTP 状态码为 200、响应体为 `"Hello World"` 、自定义响应头 `"Content-Type", "text/plain"` 的响应体：

```java
@GetMapping("/hello")
public ResponseEntity<String> hello() {
    // ResponseEntity.BodyBuilder builder = ResponseEntity.ok().header("Content-Type", "text/plain").body("Hello World");

    ResponseEntity.BodyBuilder builder = ResponseEntity.ok();
    builder.header("Content-Type", "text/plain");
    builder.body("Hello, World!");
    return builder.build();
  }
```

## ResponseEntity.BodyBuilder 好处
使用 ResponseEntity.BodyBuilder builder 可以帮助我们更加灵活的控制响应的各个部分。此外，使用 builder 还可以使代码更加简洁和易于维护。
