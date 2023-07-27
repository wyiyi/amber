---
id: json-lines
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
description: Thymeleaf 是一种用于在服务器端和客户端之间渲染 HTML、XML、JavaScript、CSS 和文本的 Java 模板引擎。
它的目标是为 Web 和独立环境下的现代开发提供自然模板处理功能。
---

![](https://wyiyi.github.io/amber/contents/2023/thymeleaf.png)

Thymeleaf 是一种用于在服务器端和客户端之间渲染 HTML、XML、JavaScript、CSS 和文本的 Java 模板引擎。
它的目标是为 Web 和独立环境下的现代开发提供自然模板处理功能。

Thymeleaf 是一个功能强大且易于使用的模板引擎，它的目标是为现代化的 Java Web 开发提供灵活的模板处理功能。
相比于传统的 JSP 或类似技术，Thymeleaf 提供了更自然的模板语法，使得模板文件易于编写和理解。

## Thymeleaf 的特点

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

## Thymeleaf 的用法

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
  ```properties
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

## 如何使用标准方言

如果你看过使用[标准方言](https://www.thymeleaf.org/doc/articles/sayhelloextendingthymeleaf5minutes.html "标准方言")编写的代码片段，你应该注意到可处理的属性都是以 th: 开头。

这个 th 被称为方言前缀，它意味着所有由该方言处理的标签和属性都将以这个前缀开始。每个方言可以指定自己的前缀。

还要注意的是，一个模板引擎可以同时设置多个方言，从而允许处理包含所有指定方言特性的模板（把方言视为一种类似 JSP 标签库的增强功能）。
其中一些方言可以共享前缀，实际上充当一个聚合方言的作用。

### 1. 基本属性

Thymeleaf 提供了许多 th 属性，用于评估表达式并将这些属性的值设置为其结果。
它们的名称与它们所设置的属性的名称相似，用来对应不同的 XHTML 和 HTML5 属性。

标准方言中的几个最基本的属性：

```html
// 替换了标签的主体
<p th:text="#{msg.welcome}">Welcome everyone!</p>

// 语法等同于 foreach
<li th:each="book : ${books}" th:text="${book.title}">En las Orillas del Sar</li>

<form th:action="@{/createOrder}">

<input type="button" th:value="#{form.submit}" />

<a th:href="@{/admin/users}">
...
```

### 2. Thymeleaf 标准表达式

大多数 Thymeleaf 属性允许将其值设置为或包含表达式，由于它们使用的方言，我们将其称为标准表达式。

包含以下五种类型：

#### 2.1 \${...} : Variable expressions.

变量表达式是 OGNL 表达式（或者 Spring EL），
当 Thymeleaf 与 Spring 集成时，在上下文变量上执行
在 Spring 术语中也称为 模型属性。

```html
// 上下文变量
${session.user.name}

// （在 OGNL 和 SpringEL 中）等同于 ((Book)context.getVariable("book")).getAuthor().getName()
<span th:text="${book.author.name}">

// 复杂的处理，如条件、迭代....
<li th:each="book : ${books}">
...
```

#### 2.2 \*{...} : Selection expressions.

选择表达式与变量表达式一样，将在选定的对象上执行，而不是在整个上下文变量映射上执行。

```html
<div th:object="${user}">
   <p th:text="${user.name}"></p>
   <p th:text="*{age}"></p>
</div>
...
```

${user} 是一个对象绑定到上下文变量中的用户对象。th:object 指令会将该对象设置为当前选择对象。
然后，我们可以使用选择表达式 user.name 或 \*{age} 来获取对象的属性值。

选择表达式有两种不同的语法 \*{...} 和 \${...}，但它们的作用是相同的，都用于引用当前选择对象的属性或方法。

通过选择表达式，你可以在 Thymeleaf 模板中更方便地操作和展示特定对象的属性和方法。
这对于处理表单数据、回显对象属性等场景非常有用。

#### 2.3 #{...} : Message (i18n) expressions.

消息表达式（通常称为文本外部化、国际化或 i18n）允许我们从外部来源（.properties 文件）中检索与特定区域设置相关的消息，
通过键引用它们，并（可选地）应用一组参数。

Spring 框架可以在运行时根据当前的语言环境和上下文变量值来获取相应的消息，实现多语言支持和国际化功能。

```html
 // 从消息源中获取名为 main.title 的消息，并在模板中显示该消息的内容
 #{main.title}

 // 调用名为entrycreated的消息，并将entryId参数传递给该消息
 #{message.entrycreated(${entryId})}

 // 使用消息表达式指定表头（header）的内容为 header.address.city 对应的消息内容
 <table>
     <th th:text="#{header.address.city}">...</th>
     <th th:text="#{header.address.country}">...</th>
  </table>

 // 使用变量表达式和消息表达式的组合，根据 config.adminWelcomeKey 的值动态确定消息键，并使用 session.user.name 作为参数传递给该消息
 #{${config.adminWelcomeKey}(${session.user.name})}

...
```

#### 2.4 @{...} : Link (URL) expressions.

链接表达式用于构建 URL 并向其添加有用的上下文和会话信息（通常称为 URL 重写）。
对于部署在 Web 服务器的 /myapp 上下文中的 Web 应用程序，

在基于 Servlet 的 Web 应用程序中，对于每个输出的 URL（上下文相对的、相对的、绝对的...），Thymeleaf 在显示 URL 之前始终调用 HttpServletResponse.encodeUrl(...)机制。
如下面的表达式所示：

```html
 <a th:href="@{/order/list}">...</a>
 // 可以转换为以下形式：

 <a href="/myapp/order/list">...</a>
 <a href="/myapp/order/list;jsessionid=23fa31abd41ea093">...</a>

 // URL 带参数
 <a th:href="@{/order/details(id=${orderId},type=${orderType})}">...</a>
 // 注意：标签属性中的&符号应进行HTML转义
 <a href="/myapp/order/details?id=23&amp;type=online">...</a>

 // 相对的链接表达式
 <a th:href="@{../documents/report}">...</a>

 // 相对于服务器的链接表达式
 <a th:href="@{~/contents/main}">...</a>

 // 协议相关的链接表达式
 <a th:href="@{//static.mycompany.com/res/initial}">...</a>

 // 绝对的链接表达式
 <a th:href="@{http://www.mycompany.com/main}">...</a>
 ...
```

#### 2.5 ~{...} : Fragment expressions.

片段表达式是一种简单的方式，用于表示标记片段并在模板中移动它们。
通过这些表达式，片段可以复制、传递到其他模板作为参数等。

最常见的用法是使用 th:insert 或 th:replace 进行片段插入：

```html
 <div th:insert="~{commons :: main}">...</div>

 // 带参数
 <div th:with="frag=~{footer :: #main/text()}">
   <p th:insert="${frag}">
 </div>
 ...
```

#### 2.6 Literals and operations

也包含许多类型的文字和操作：

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

#### 2.7 Expression preprocessing

表达式预处理允许在表达式评估之前动态生成部分表达式。它由 "\_\_" 符号表示。

```html
 #{selection.__${sel.code}__}
```

表达式 #{selection.\${sel.code}} 包含两个部分：

1. 变量表达式：\${sel.code}
    - 此部分首先进行评估，并应返回一个值。在这种情况下，假设该值为 "ALL"。
2. 预处理表达式：selection.ALL
    - 将变量表达式的结果（\${sel.code}）插入 "\_\_" 符号之间，生成最终要评估的表达式。
    - 在本例中，生成的表达式为 selection.ALL。

最终的表达式 selection.ALL 通常在国际化等场景中使用，可以根据变量的值动态构建消息查找的键。
