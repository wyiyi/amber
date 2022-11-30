---
title: Filter 中方法会执行两次？ 
date: 2022.12.01 
tags: Servlet，Filter
categories: Technology  
mathjax: true 
comments: true
toc: true
description: Filter 中方法会执行两次？
---

## 一、1 次 init()，2 次 doFilter()
### 1、现象：
添加一个 Filter ，当发起一个请求，在日志中发现：

init() 发放被初始化一次，

doFilter() 方法执行 2 次，

如下：代码示例一、日志一

示例一：
````java
//@WebFilter(filterName = "aFilter", value = "/*")
@Component
public class AFilter implements Filter {
    private static final Logger LOGGER = LoggerFactory.getLogger(AFilter.class);
    @Autowired
    private UserService userService;
    @Override
    public void init(FilterConfig filterConfig) {
        LOGGER.info("A Filter初始化,只初始化一次...");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        LOGGER.info("AFilter 处理中...");
        User user = userService.getUserInfo();
        LOGGER.info("USER 信息：" +  user);
    }

    @Override
    public void destroy() {
        LOGGER.info("A Filter销毁...");
    }
}
````
日志一：
````java
2022-11-30 23:55:43.928  INFO 22280 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 795 ms
2022-11-30 23:55:43.952  INFO 22280 --- [           main] com.amber.common.filter.AFilter          : A Filter初始化,只初始化一次...
2022-11-30 23:55:44.022  INFO 22280 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2022-11-30 23:55:44.170  INFO 22280 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2022-11-30 23:56:03.418  INFO 22280 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2022-11-30 23:56:03.418  INFO 22280 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2022-11-30 23:56:03.423  INFO 22280 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 5 ms
2022-11-30 23:56:03.431  INFO 22280 --- [nio-8080-exec-1] com.amber.common.filter.AFilter          : AFilter 处理中...
2022-11-30 23:56:03.431  INFO 22280 --- [nio-8080-exec-1] com.amber.common.filter.AFilter          : USER 信息：User(screen_name=amber, text=今天星期三, created_at=Wed Nov 30 23:56:03 CST 2022)
2022-11-30 23:56:03.455  INFO 22280 --- [nio-8080-exec-6] com.amber.common.filter.AFilter          : AFilter 处理中...
2022-11-30 23:56:03.456  INFO 22280 --- [nio-8080-exec-6] com.amber.common.filter.AFilter          : USER 信息：User(screen_name=amber, text=今天星期三, created_at=Wed Nov 30 23:56:03 CST 2022)
````

### 2、原因：
![](https://wyiyi.github.io/amber/contents/flow/network.png)

打开 Network 发现实际的网络请求有两个：
- `/index`—— 实际请求
- `/favicon.ico`——浏览器展示tab上的网站图标。

在 doFilter() 方法中获取 HttpServletRequest 的 getRequestURI() 方法（示例二），在日志二中发现 `/favicon.ico` 请求。

示例二：
````java
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    LOGGER.info("AFilter 处理中...");
    User user = userService.getUserInfo();
    LOGGER.info("USER 信息：" +  user);
    HttpServletRequest servletRequest = (HttpServletRequest) request;
    LOGGER.info(servletRequest.getRequestURI());
}
````
日志二：
````java
2022-12-01 00:11:11.428  INFO 13616 --- [nio-8080-exec-2] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2022-12-01 00:11:11.428  INFO 13616 --- [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2022-12-01 00:11:11.431  INFO 13616 --- [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet        : Completed initialization in 3 ms
2022-12-01 00:11:11.436  INFO 13616 --- [nio-8080-exec-2] com.amber.common.filter.AFilter          : AFilter 处理中...
2022-12-01 00:11:11.436  INFO 13616 --- [nio-8080-exec-2] com.amber.common.filter.AFilter          : USER 信息：User(screen_name=amber, text=今天星期三, created_at=Thu Dec 01 00:11:11 CST 2022)
2022-12-01 00:11:11.436  INFO 13616 --- [nio-8080-exec-2] com.amber.common.filter.AFilter          : /index
2022-12-01 00:11:11.449  INFO 13616 --- [nio-8080-exec-6] com.amber.common.filter.AFilter          : AFilter 处理中...
2022-12-01 00:11:11.449  INFO 13616 --- [nio-8080-exec-6] com.amber.common.filter.AFilter          : USER 信息：User(screen_name=amber, text=今天星期三, created_at=Thu Dec 01 00:11:11 CST 2022)
2022-12-01 00:11:11.450  INFO 13616 --- [nio-8080-exec-6] com.amber.common.filter.AFilter          : /favicon.ico
````
### 3、方案：
这样我们就知道走了两次 doFilter() 方法的原因，就可以在拦截器中判断请求路径做处理即可。

## 二：2 次 init()，2 次 doFilter()
### 1、现象：
在示例一中：同时使用 `@WebFilter` 和 `@Component` 在启动日志，发起一个请求时发现：

init() 发放被初始化 2 次，

doFilter() 方法执行 2 次，

如下：代码示例三、日志三

示例三：
````java
@WebFilter(filterName = "aFilter", value = "/api/amber/*")
@Component
public class AFilter implements Filter {
  // ...
}
````
日志三：
````java
2022-12-01 00:59:06.606  INFO 2620 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2022-12-01 00:59:06.606  INFO 2620 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 822 ms
2022-12-01 00:59:06.637  INFO 2620 --- [           main] com.amber.common.filter.AFilter          : A Filter初始化,只初始化一次...
2022-12-01 00:59:06.637  INFO 2620 --- [           main] com.amber.common.filter.AFilter          : A Filter初始化,只初始化一次...
2022-12-01 00:59:06.707  INFO 2620 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2022-12-01 00:59:06.851  INFO 2620 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2022-12-01 01:00:29.819  INFO 21852 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 7 ms
2022-12-01 01:00:29.827  INFO 21852 --- [nio-8080-exec-1] com.amber.common.filter.AFilter          : AFilter 处理中...
2022-12-01 01:00:29.828  INFO 21852 --- [nio-8080-exec-1] com.amber.common.filter.AFilter          : USER 信息：User(screen_name=amber, text=今天星期三, created_at=Thu Dec 01 01:00:29 CST 2022)
2022-12-01 01:00:29.849  INFO 21852 --- [nio-8080-exec-5] com.amber.common.filter.AFilter          : AFilter 处理中...
2022-12-01 01:00:29.850  INFO 21852 --- [nio-8080-exec-5] com.amber.common.filter.AFilter          : USER 信息：User(screen_name=amber, text=今天星期三, created_at=Thu Dec 01 01:00:29 CST 2022)
````

### 2、原因：
在[文章中](https://blog.csdn.net/weixin_38152047/article/details/121244269)@WebFilter 
用于将一个类声明为过滤器，该注解将会在部署时候被容器处理，容器将根据具体的属性配置将相应的类部署为过滤器。
`value = "/api/amber/*"` 属性等价于 `urlPatterns` 属性。

① @WebFilter （示例四）

通过 WebFilter 进行 Filter 声明，这样容器在进行部署的时候就会处理该 Filter，
创建实例并创建配置对象 FilterConfig ，然后会将该 Filter 应用到 urlPatterns 所指定的 url ；

② @Component （示例四）
Filter 类增加 Component 注解，让该 Filter 被容器管理。

③ @ServletComponentScan （示例五）

在 init() 方法中获取到初始化参数，自定义的 excludedUrls，作为成员在后续执行过滤逻辑的时候使用；

在 doFilter 中进行 url 的鉴定，如果需要执行认证鉴权处理，则执行相应逻辑。不满足条件的情况下重定向到登录页；

在 SpringBoot 的 Application 中增加注解 @ServletComponentScan ，这样容器会扫描到 @Component 注解的 Filter 。


示例四：
````java
@WebFilter(filterName = "aFilter", value = "/api/amber/*")
@Component
public class AFilter implements Filter {
    
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }


    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws IOException, ServletException {
        chain.doFilter(req, resp);
    }

    @Override
    public void destroy() {
    }
}
````
示例五：
````java
@SpringBootApplication
@ServletComponentScan
@MapperScan({"com.amber.common.dao"})
public class BronzeApplication {
	public static void main(String[] args) {
		SpringApplication.run(BronzeApplication.class, args);
	}
}
````

访问的 url 为 /index/* 或者 /product/* 的时候，该过滤器也执行了！ 也就是说，WebFilter 注解配置的 urlPatterns 没有起作用。所以 doFilter() 执行两次，其中包含 /favicon.ico 路径。

// TODO 在查看容器启动日志的时候，发现WebAuthFilter被两次注册，两次映射，分别为WebAuthFilter和webAuthFilter
....文中内容。

### 3、方案：
#### a. @WebFilter
只保留 @WebFilter 注解，再次启动服务。查看日志，发现该 Filter 仅被映射一次，通过浏览器访问相应的 url 也表现正确。

#### b. @Component
只保留 @Component ，通过新增配置文件，对 FilterRegistrationBean 进行实例注册，
至于 配置的 UrlPatterns ，通过 `filterRegistrationBean.addUrlPatterns("/*");` addUrlPatterns() 属性解决。
在[Servlet之Filter](https://wyiyi.github.io/amber/2021/10/01/filter/)已提到，就不过多重复。

#### a 和 b 方案均可解决该现象出现。

