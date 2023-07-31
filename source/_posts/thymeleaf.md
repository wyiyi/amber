---
title: 简化 Java Web 开发的强大模板引擎—Thymeleaf
date: 2023.08.01
tags: 
   - Java Web
   - Spring
   - Thymeleaf
categories: Technology   
mathjax: true 
comments: true
toc: true
description: Thymeleaf 是一种用于在服务器端和客户端之间渲染 HTML、XML、JavaScript、CSS 和文本的 Java 模板引擎。它的目标是为 Web 和独立环境下的现代开发提供自然模板处理功能。
---

Thymeleaf 是一种用于在服务器端和客户端之间渲染 HTML、XML、JavaScript、CSS 和文本的 Java 模板引擎。
它的目标是为 Web 和独立环境下的现代开发提供自然模板处理功能。

Thymeleaf 是一个功能强大且易于使用的模板引擎，它的目标是为现代化的 Java Web 开发提供灵活的模板处理功能。
相比于传统的 JSP 或类似技术，Thymeleaf 提供了更自然的模板语法，使得模板文件易于编写和理解。

# Thymeleaf 的特点

1. 自然模板语法

   Thymeleaf 使用类似于 HTML 的模板语法，开发人员可以直接在模板中嵌入动态内容，并以自然的方式引用和操作模型数据。这种模板语法让开发者能够更快速地构建出富有动态性的页面。

2. 完整的 HTML5 支持

   Thymeleaf 与 HTML5 兼容，并提供了对 HTML5 标准元素和属性的良好支持。开发者可以充分利用最新的 HTML5 功能，如表单验证、视频和音频播放等，提升用户体验。

3. 广泛的模板功能

   Thymeleaf 提供了丰富的模板功能，包括条件判断、循环迭代、局部片段引入、模板布局、国际化支持等。这些功能使得模板更加灵活和可复用，能够满足各种复杂的业务需求。

4. 强大的表达式语言

   Thymeleaf 内置了强大的表达式语言，允许开发者在模板中动态地访问和操作模型数据。表达式语言支持各种操作符、方法调用、条件表达式等，使得模板能够进行复杂的逻辑处理和数据展示。

5. [与 Spring 框架集成](https://spring.io/guides/gs/serving-web-content/ "与 Spring 框架集成")

   Thymeleaf 很好地与 Spring 框架集成，为开发者提供了便捷的使用方式。它可以与 Spring MVC、Spring Boot 等框架无缝配合，实现高效的视图渲染和数据绑定。

# Thymeleaf 的用法

1. **添加 Thymeleaf 依赖**

   在项目的构建文件中添加 Thymeleaf 的相关依赖：

- Maven 依赖
  ```xml
  <dependencies>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-thymeleaf</artifactId>
      </dependency>
  </dependencies>
  ```
- Gradle 依赖
  ```
  dependencies {
      implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
  }
  ```

2. **编写模板文件**

   创建一个以 .html 为后缀的模板文件，在其中使用 Thymeleaf 的模板语法。

   通过使用 Thymeleaf 的标签和表达式，我们可以插入动态数据、进行条件判断、循环迭代等操作。

   ```html
    <!DOCTYPE HTML>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <title>Getting Started: Serving Web Content</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    </head>
    <body>
        <p th:text="'Hello, ' + ${name} + '!'" />
    </body>
    </html>
   ```

3. **控制器中使用 Thymeleaf**

   在 Spring MVC 的控制器中，参数的值将添加到 Model 对象，传递给 Thymeleaf 模板。
   Thymeleaf 会自动根据模板中的表达式来渲染数据，并生成最终的 HTML 页面。

   ```java
    package com.example.servingwebcontent;

    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RequestParam;

    @Controller
    public class GreetingController {

        @GetMapping("/greeting")
        public String greeting(@RequestParam(name="name", required=false, defaultValue="World") String name, Model model) {
            model.addAttribute("name", name);
            return "greeting";
        }

    }
   ```

# 如何使用标准方言

如果你看过使用[标准方言](https://www.thymeleaf.org/doc/articles/sayhelloextendingthymeleaf5minutes.html "标准方言")编写的代码片段，你应该注意到可处理的属性都是以 th: 开头。

这个 th 被称为方言前缀，它意味着所有由该方言处理的标签和属性都将以这个前缀开始。每个方言可以指定自己的前缀。

还要注意的是，一个模板引擎可以同时设置多个方言，从而允许处理包含所有指定方言特性的模板（把方言视为一种类似 JSP 标签库的增强功能）。
其中一些方言可以共享前缀，实际上充当一个聚合方言的作用。

## 1. 基本属性

Thymeleaf 提供了许多 th 属性，用于评估表达式并将这些属性的值设置为其结果。
它们的名称与它们所设置的属性的名称相似，用来对应不同的 XHTML 和 HTML5 属性。

标准方言中的几个最基本的属性：

```html
// 替换了标签的主体
<p th:text="#{msg.welcome}">Welcome everyone!</p>

// 语法等同于 foreach
<li th:each="book : ${books}" th:text="${book.title}">En las Orillas del Sar</li>

// 提交表单时，浏览器将向 /createOrder 发送POST请求 
<form th:action="@{/createOrder}" method="post">
    
// 使用表达式 #{form.submit} 的求值结果作为按钮的值
<input type="button" th:value="#{form.submit}" />

// 跳转到 /admin/users 地址
<a th:href="@{/admin/users}">
...
```

## 2. Thymeleaf 标准表达式

大多数 Thymeleaf 属性允许将其值设置为或包含表达式，由于它们使用的方言，我们将其称为标准表达式。

包含以下五种类型：

### 2.1 ${...} : Variable expressions.

变量表达式用于在模板中访问和显示变量的值。
变量可以是通过控制器传递给模板的模型属性、请求参数、会话属性等。

1. 访问模型属性：
    ```html
    <p th:text="${user.name}"></p>
    ```
    `${user.name}` 表达式用于访问模型中名为 "user" 的属性，并将其值插入到 `<p>` 元素中。

2. 访问请求参数：
    ```html
        <p th:text="${param.email}"></p>
    ```
    `${param.email}` 表达式用于获取名为 "email" 的请求参数的值，并将其插入到 `<p>` 元素中。

3. 访问会话属性：
    ```html
    ${session.user}
    ```
    ${session.user} 表达式用于获取会话中名为 "user" 的属性的值，并将其插入到 `<p>` 元素中。

【注意】：${...} 变量表达式在模板中只能读取变量的值，不能修改变量的值。如果修改，使用 Thymeleaf 的内联表单处理或链接处理功能。

### 2.2 *{...} : Selection expressions.

选择表达式与变量表达式一样，将在选定的对象上执行，而不是在整个上下文变量映射上执行。

```html
<div th:object="${user}">
   <p th:text="${user.name}"></p>
   <p th:text="*{age}"></p>
</div>
```
${user} 是一个对象绑定到上下文变量中的用户对象。th:object 指令会将该对象设置为当前选择对象。
然后，我们可以使用选择表达式 user.name 或 \*{age} 来获取对象的属性值。

【注意】：选择表达式有两种不同的语法 \*{...} 和 \${...}，但它们的作用是相同的，都用于引用当前选择对象的属性或方法。

### 2.3 #{...} : Message (i18n) expressions.

#{...} 是一种用于国际化（i18n）的消息表达式。它的主要目的是在软件中支持多语言的消息处理。

```JS
const name = 'Alice';
const greeting = `Hello, #{name}!`;
console.log(greeting);
```
`#{name}` 表达式用来将变量 name 的值动态地插入到问候消息中，生成适当的本地化文本。

### 2.4 @{...} : Link (URL) expressions.

@{...} 表达式是链接表达式（Link Expressions），用于生成动态链接（URL）。

可以轻松地处理路由和参数传递，而无需手动构建URL。

1. 生成相对路径链接：
    ```html
     <a th:href="@{/home}">Home</a>
    ```
    `@{/home}` 表达式将生成一个相对于当前上下文路径的链接，指向 "home" 路径。当用户点击链接时，将导航到 "/home" 页面。

2. 生成带参数的链接：
    ```html
    <a th:href="@{/user/details(userId=${user.id})}">View Details</a>
    ```
   `@{/user/details(userId=${user.id})}` 表达式生成一个带有参数的链接。
   `${user.id}` 是一个变量表达式，表示用户的 ID。生成的链接将包含用户 ID 作为查询参数，例如： "/user/details?userId=123"。

3. 生成URI片段链接：
    ```html
   <a th:href="@{#section-1}">Go to Section 1</a>
    ```
   `@{#section-1}` 表达式生成一个链接，可以直接跳转到具有 ID 为 "section-1" 的页面片段。

【注意】：
- 链接表达式只能在 HTML 标签的属性中使用，用于生成正确的链接。
- 片段表达式在 `Thymeleaf 3.0` 及更高版本中引入。

### 2.5 ~{...} : Fragment expressions.

~{...} 表达式是片段表达式（Fragment Expressions），用于引入和使用模板片段，重用可独立使用的模块或组件。
通过引入和参数化片段，在不同的地方使用相同的代码片段，提高了模板的可维护性和重用性。

最常见的用法是使用 th:insert 或 th:replace 进行片段插入：
1. 引入片段：
    ```html
    <div th:insert="~{fragment :: myFragment}"></div>
    ```
   `~{fragment :: myFragment}` 表达式用于引入名为 myFragment 的模板片段，并将其插入到 `<div>` 元素中。
2. 参数化片段：
    ```html
    <div th:replace="~{fragment :: myFragment(param1='value1', param2=42)}"></div>
    ```
    `~{fragment :: myFragment(param1='value1', param2=42)}` 表达式引入 myFragment 模板片段，并为其传递参数。在模板片段中可以通过 `${param1}` 和 `${param2}` 来获取参数的值。
3. 引入片段：
    ```html
    <div th:replace="~{:: myFragment}"></div>
    ```
   `~{:: myFragment}` 表达式用于引入当前模板中名为 myFragment 的片段。

【注意】：片段表达式在 `Thymeleaf 3.0` 及更高版本中引入。

### 2.6 Literals and operations

在Thymeleaf模板引擎中，字面量（Literals）和表达式操作（Operations）用于在模板中进行数据处理和计算。
这些表达式可以用于文本替换、条件判断、循环迭代等操作，从而使模板更具动态性和灵活性。
- 文字：
    - 文本文字：'一段文字'，'另一个！'，...
    - 数字文字：0，34，3.0，12.3，...
    - 布尔文字：true，false
    - 空文字：null
    - 文字标记：one，sometext，main，...
- 文字操作：
    - 字符串连接：+
    - 文字替换：|The name is \${name}|
- 算术操作：
    - 二元运算符：+，-，\*，/，%
    - 减号（一元运算符）：-
- 布尔操作：
    - 二元运算符：and，or
    - 布尔否定（一元运算符）：！，not
- 比较和相等：
    - 比较运算符：>，<，>=，<=（gt，lt，ge，le）
    - 相等运算符：==，!=（eq，ne）
- 条件运算符：
    - If-then: (if) ? (then)
    - If-then-else: (if) ? (then) : (else)
    - Default: (value) ?: (defaultvalue)

示例：
1. 字符串字面量：
    ```html
    <p th:text="'Hello, Thymeleaf!'"></p>
    ```
    `'Hello, Thymeleaf!'` 是一个字符串字面量，可以在 `<p>` 元素中显示文本 "Hello, Thymeleaf!"。

2. 数字字面量：
     ```html
    <p th:text="42"></p>
    ```
    `42` 是一个数字字面量，可以在 `<p>` 元素中显示数字 42。

3. 变量引用：
     ```html
    <p th:text="${username}"></p>
    ```
    `${username}` 是一个变量引用，通过该表达式可以获取名为 "username" 的变量的值，并在 `<p>` 元素中显示。
     
4. 算术操作：
     ```html
    <p th:text="${number1 + number2}"></p>
    ```
    `${number1 + number2}`是一个算术操作，通过该表达式可以对 number1 和 number2 变量进行加法运算，并在 `<p>` 元素中显示结果。

5. 逻辑操作：
    ```html
    <p th:if="${age >= 18}">You are an adult.</p>
    ```
    `${age >= 18}` 是一个逻辑操作，通过该表达式可以判断 age 变量是否大于或等于 18，如果条件满足，则显示 "You are an adult."。

### 2.7 Expression preprocessing

表达式预处理（Expression Preprocessing）是一种用于对表达式进行预处理和修改的机制。可以使用预处理器在表达式求值之前对其进行操作和转换。
预处理器提供了一些特殊语法和功能，可以扩展表达式的功能并提供更好的灵活性和可读性。

预处理的具体用法：
1. 转义表达式：
    ```html
    <p th:text="|Hello, \${name}!|"></p>
    ```
    `|\${name}|` 是一个转义表达式，通过在表达式外添加竖线字符 "|"，可以防止表达式被求值，而直接显示为文本 "Hello, ${name}!"。

2. 默认值设置：
   ```html
    <p th:text="${username} ?: 'Guest'"></p>
    ```
   `${username} ?: 'Guest'` 是一个默认值设置，如果 username 变量为空或不存在，将使用默认值 "Guest"。

3. 集合选择：
   ```html
    <ul>
        <li th:each="item : ${items}" th:text="${item.name}"></li>
    </ul>
    ```
   `${item.name}` 是一个集合选择表达式，用于从 items 集合中选取每个元素的 name 属性并显示在列表项中。
4. 字符串拼接：
   ```html
    <p th:text="'Hello ' + ${name}"></p>
    ```
   `'Hello ' + ${name}` 是一个字符串拼接表达式，可将字符串 "Hello " 和 name 变量的值进行拼接。

【注意】：片段表达式在 `Thymeleaf 3.0` 及更高版本中引入。


# 遇到的问题
Java Web 开发过程中，遇到了些问题（建议先查看官网，避免走弯路）。

## 情景一：获取跳转的路径的部分值

1. 需求： 跳转路径为：`https://xxx.com/wechat/memberInfoIndex/wx9e24xxx443` ， 
`wx9e24xxx443` 为appid，需要获取访问的路径中的 appid 的值。
2. 解决方案：
     ```html
     <input id="requestURI" th:value="${#request.requestURI}"/>
    
     // 显示在 Console 中：
     <input id="requestURI" value="/wechat/memberInfoIndex/wx9e24xxx443"/>
     ```
3.分析：`${#request.requestURI}` 的写法可以获取到访问路径，经过处理（拆分、正则或其它方式）即可得到 appid。

## 情景二：页面动态获取 title
1. 需求：页面中的 title 不固定，需要根据接口返回的值，显示到 title 上。

2. 解决方案：
   ```html
    $('title').html($('#comname').val());
    ```
3. 分析：正常 Html 中直接写：`<title>Amazing！</title>` ，通过 Jquery 获取标签元素即可动态更改 title 值。
   
4. 请求后跳转页面也如此。
   ```html
    $('#container').html(res);
    ```
