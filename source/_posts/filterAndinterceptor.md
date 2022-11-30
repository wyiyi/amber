---
title: 我们不一样——拦截器与过滤器区别 
date: 2022.11.01 
tags: Servlet，Filter，Interceptor
categories: Technology  
mathjax: true 
comments: true
toc: true
description: 拦截器与过滤器区别。
---


# 我们不一样—拦截器与过滤器区别

在 [Servlet之Filter](https://wyiyi.github.io/amber/2021/10/01/filter/) 文章中介绍了 Filter 的用法，
[Spring之Interceptor](https://wyiyi.github.io/amber/2022/07/01/interceptor/) 文章中介绍 Interceptor 的用法。
二者有何区别？

## 简单介绍
过滤器（Filter）依赖于 Servlet 容器，在实现上基于函数回调。 
配置比较简单，直接实现 Filter 接口即可，也可以通过 @WebFilter 注解实现对特定 URL 拦截。
具体使用查看 [Servlet之Filter](https://wyiyi.github.io/amber/2021/10/01/filter/) 文章。

拦截器（Interceptor） 依赖于 Web 框架，在 SpringMVC 中就是依赖于 SpringMVC 框架。在实现上基于 Java 的反射机制。
它是链式调用，一个应用中可以同时存在多个拦截器 Interceptor， 一个请求也可以触发多个拦截器 ，而每个拦截器的调用会依据它的声明顺序依次执行。

## 相同点
拦截器与过滤器都是体现了 AOP 的思想，对方法实现增强，都可以拦截请求方法。

拦截器和过滤器都可以通过 Order 注解设定执行顺序。

## 不同点
### 1、实现原理
过滤器和拦截器底层实现方式不同。

Filter（过滤器）是基于函数回调的。

  自定义的过滤器要实现 doFilter() 方法，在示例一中看到该方法有 FilterChain 参数，其实它是一个回调接口。 
ApplicationFilterChain 是它的实现类，在实现类内部有 doFilter() 方法就是回调方法，可以获取到自定义的 xxxFilter 类，
在其内部回调方法 doFilter() 里调用各个自定义 xxxFilter 过滤器， 并执行 doFilter() 方法。 
而每个xxxFilter 会先执行自身的 doFilter() 过滤逻辑，最后在执行结束前会执行filterChain.doFilter(servletRequest, servletResponse)，也就是回调ApplicationFilterChain的doFilter() 方法，以此循环执行实现函数回调。
 

  如 [Servlet之Filter](https://wyiyi.github.io/amber/2021/10/01/filter/) 文章 FilterChain 的执行顺序：
 
![](https://wyiyi.github.io/amber/contents/filterAndInter/applicationFilterChain.png)
  
示例一：
````java
public interface Filter {
  void doFilter(ServletRequest var1, ServletResponse var2, FilterChain var3) throws IOException, ServletException;
}
````
示例二：
````java
public interface FilterChain {
    void doFilter(ServletRequest var1, ServletResponse var2) throws IOException, ServletException;
}
````
示例三：
````java
public final class ApplicationFilterChain implements FilterChain {
  public void doFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {
      // 省略 ...
      this.internalDoFilter(request, response);
      // 省略 ...
  }
  private void internalDoFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {
    if (this.pos < this.n) {
      ApplicationFilterConfig filterConfig = this.filters[this.pos++];
      Filter filter = filterConfig.getFilter();
      // 省略 ...
      filter.doFilter(request, response, this);
    }
}
````

Interceptor（拦截器）则是基于 Java 的反射机制（动态代理）实现的。


????

### 2、使用范围
Filter（过滤器）在 Servlet 规范中定义，实现 javax.servlet.Filter 接口，依赖于 Tomcat 等容器，在 web 程序中使用。

Interceptor（拦截器）是 SpringMVC 中实现的，实现 org.springframework.web.servlet.HandlerInterceptor 接口，
拦截器是一个 Spring 组件，由 Spring 容器进行管理。

### 3、触发时机不同
过滤器Filter是在请求进入容器后，但在进入 servlet 之前进行预处理，请求结束是在 servlet 处理完以后。

拦截器 Interceptor 是在请求进入 servlet 后，在进入Controller之前进行预处理的，Controller 中渲染了对应的视图之后请求结束。

![](https://wyiyi.github.io/amber/contents/filterAndInter/relationship.png)

### 4、拦截请求范围不同
条件：同时配置过滤器和拦截器，示例四：
````java
@RestController
public class LoginController {
    private static Logger log = LoggerFactory.getLogger(UserLoginInterceptor.class);

    @RequestMapping("/index")
    public Map<String,String> hello(Model model){
        Map<String,String> response= new HashMap<>();
        response.put("msg","hello controller !");
        log.info("hello controller !");
        return response;
    }
}
````

现象：在启动过程中的日志（如下）：看到过滤器 Filter 执行了两次，拦截器 Interceptor 只执行了一次。

执行顺序：AFilter 处理中 -> Interceptor 开始执行 -> hello controller ! -> Interceptor 执行过程中 -> Interceptor 执行结束

原因：因为过滤器 Filter 可以对所有进入容器的请求起作用，
而拦截器 Interceptor 只会对 Controller 中请求或访问 static 目录下的资源请求起作用。

Interceptor 可以访问 Action 的上下文，值栈里的对象，而 Filter 不能。

在 Action 的生命周期里，Interceptor 可以被多次调用，而 Filter 只能在容器初始化时调用一次。

````
2022-11-29 19:20:18.317  INFO 24388 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2022-11-29 19:20:18.317  INFO 24388 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.41]
2022-11-29 19:20:18.440  INFO 24388 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2022-11-29 19:20:18.440  INFO 24388 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 873 ms
2022-11-29 19:20:18.467  INFO 24388 --- [           main] com.amber.common.filter.AFilter          : A Filter初始化,只初始化一次...
2022-11-29 19:20:18.555  INFO 24388 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2022-11-29 19:20:18.708  INFO 24388 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2022-11-29 19:20:18.734  INFO 24388 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
2022-11-29 19:20:18.759  INFO 24388 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 5.4.25.Final
2022-11-29 19:20:18.882  INFO 24388 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.2.Final}
2022-11-29 19:20:18.942  INFO 24388 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
2022-11-29 19:20:19.044  INFO 24388 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
2022-11-29 19:20:19.049  INFO 24388 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2022-11-29 19:20:19.088  WARN 24388 --- [           main] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
2022-11-29 19:20:19.165  INFO 24388 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
 _ _   |_  _ _|_. ___ _ |    _ 
| | |\/|_)(_| | |_\  |_)||_|_\ 
     /               |         
                        3.3.1 
2022-11-29 19:20:19.483  INFO 24388 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2022-11-29 19:20:19.489  INFO 24388 --- [           main] com.amber.common.BronzeApplication       : Started BronzeApplication in 2.193 seconds (JVM running for 2.705)
2022-11-29 19:20:23.101  INFO 24388 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2022-11-29 19:20:23.101  INFO 24388 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2022-11-29 19:20:23.105  INFO 24388 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 4 ms
2022-11-29 19:20:23.110  INFO 24388 --- [nio-8080-exec-1] com.amber.common.filter.AFilter          : AFilter 处理中...
2022-11-29 19:20:23.113  INFO 24388 --- [nio-8080-exec-1] c.a.c.interceptor.UserLoginInterceptor   : Interceptor 开始执行时间：1669720823113
2022-11-29 19:20:23.149  INFO 24388 --- [nio-8080-exec-1] c.a.c.interceptor.UserLoginInterceptor   : hello controller !
2022-11-29 19:20:23.171  INFO 24388 --- [nio-8080-exec-1] c.a.c.interceptor.UserLoginInterceptor   : Interceptor 执行过程中
2022-11-29 19:20:23.172  INFO 24388 --- [nio-8080-exec-1] c.a.c.interceptor.UserLoginInterceptor   : Interceptor 执行结束时间：1669720823172耗时：59
````



### 5、注入Bean情况不同/ 使用资源不同
拦截器是 Spring 组件，受 Spring 容器管理，配置在 Spring 文件中，拦截器可以获取 IOC 容器中的各个 bean ，
切入点多如：Service 对象、数据源、事务管理等，控制更细致。 在拦截器里注入一个service，就可以调用业务逻辑。 

过滤器则不可以。

### 6、控制执行顺序不同
当出现多个过滤器或拦截器同时存在的情况，希望某个过滤器或拦截器能优先执行，就涉及到它们的执行顺序。

- 过滤器用 @Order 注解控制执行顺序，通过 @Order 控制过滤器的级别，值越小级别越高越先执行。
【注：[Servlet之Filter](https://wyiyi.github.io/amber/2021/10/01/filter/) 已提到过】

- 拦截器默认的执行顺序，就是它的注册顺序，也可以通过 Order 手动设置控制，值越小越先执行。 示例五：
````java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer { 
    // ...
  @Override
  public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(userLoginInterceptor2).addPathPatterns("/index").order(2);
    registry.addInterceptor(userLoginInterceptor1).addPathPatterns("/index").order(1);
    registry.addInterceptor(userLoginInterceptor).addPathPatterns("/index").order(3);
  }
}
````

## 拦截器和过滤器是比较常用的，但需要注意两者的区别！




https://github.com/yowko/docker-redis-5-cluster
https://docs.spring.io/spring-framework/docs/5.1.2.RELEASE/spring-framework-reference/web.html#filters
http://propersoft-cn.github.io/pep-refs/projects/servlet/3.0/servlet-3_0-mrel-spec.pdf


~~找 servlet 和 spring 就行，filter 是 servlet 里的，拦截器主要指 spring 里的~~
- - ~~拦截器 spring，过滤器 servlet~~
- - 拦截器可以 AbstractSpringTest，过滤器需要 AbstractIntegrationTest
- - ~~拦截器切入点更多，控制更细致~~
~~- - Filter chain.doFilter 之后 response is committed，不能再设置请求头 —— https://stackoverflow.com/questions/2030152/cannot-set-header-in-jsp-response-already-committed~~
- - 拦截器在未 @EnableWebMVC 的环境配置更复杂？
