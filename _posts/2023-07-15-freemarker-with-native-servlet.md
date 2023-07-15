---
layout: post
title: 在原生 Servlet 中使用 Freemarker
categories: 后端
tags: JavaWeb 模板引擎
author: ZekeTian
---



* content
{:toc}
我们在使用 Freemarker 时，通常会结合 Spring、SpringBoot 等框架使用，很少会在原生的 Servlet 中直接使用。因此，目前很多文章的内容都是讲解如何借助这些框架去使用 Freemarker，很少有文章讲解如何在原生的 Servlet 中使用 Freemarker。为了加深对 Freemarker 使用方式的理解，本文将介绍原生 Servlet 中的 Freemarker 使用，以及可能会遇到的一些坑。



## 1. 配置 Freemarker

在原生 Servlet 中使用 Freemarker，需要在 web.xml 中添加如下相关的配置：

```xml
<servlet>
    <servlet-name>freemarker</servlet-name>
    <servlet-class>freemarker.ext.servlet.FreemarkerServlet</servlet-class>
    <init-param>
        <!--
            设置 ftl 文件的根路径，FreemarkerServlet 会在此根路径下去寻找对应的 ftl 文件。
            如果要访问根路径下的子文件夹的 ftl 文件，那么在 url 中需要加上子文件夹名，例如访问 /ftl/test/test.ftl 文件，则 url 为：
            http://localhost:8080/contextPath/test/test.ftl （即在访问的 url 中添加了 "/test/" 子文件夹名）  -->
        <param-name>TemplatePath</param-name>
        <param-value>/ftl/</param-value>
    </init-param>
    <init-param>
        <param-name>default_encoding</param-name>
        <!--  template 文件的编码格式 -->
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>ResponseCharacterEncoding</param-name>
        <!-- Use the output_encoding setting of FreeMarker: -->
        <param-value>fromTemplate</param-value>
    </init-param>
    <init-param>
        <param-name>output_encoding</param-name>
        <!-- The encoding of the template output; Note that you must set
                 "ResponseCharacterEncodring" to "fromTemplate" for this to work! -->
        <param-value>UTF-8</param-value>
    </init-param>
</servlet>

<servlet-mapping>
    <servlet-name>freemarker</servlet-name>
    <url-pattern>*.ftl</url-pattern>
</servlet-mapping>
```

上述配置信息是对 FreemarkerServlet 进行配置，而 FreemarkerServlet  是 Freemarker 实现的 Servlet，其可以帮助我们处理 ftl 文件，渲染数据。

在 FreemarkerServlet 的初始化参数中，重点是对 ftl 文件的路径以及编码格式进行设置，需要注意如下几点：

- 设置完 ftl 文件的根路径后，注意访问的 url 与 ftl 文件夹路径的关系（具体解释可见上述配置的注释）
- 编码格式需要设置成 UTF-8，否则 ftl 文件的中文会乱码

关于其它的配置信息，可以查看[官方文档](https://freemarker.apache.org/docs/pgui_misc_servlet.html)。

此外，需要说明的是，本文所使用的 Freemarker 版本为 `2.3.31`，对应的 maven 信息如下：
```xml
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.31</version>
</dependency>
```



## 2. 使用介绍

### 2.1 ftl 的作用域对象

在 JSP 的 EL 表达式中，其内置了 pageScope、requestScope、sessionScope、applicationScope 四种作用域对象，方便我们获取数据。而在 FreemarkerServet 中，其同样也为我们内置了如下的四种作用域对象：

- Request：对应 HttpServletRequest
- RequestParameters：对应 HttpServletRequest 中的参数
- Session：对应 HttpSession
- Application：对应 ServletContext



#### 2.1.1 基本使用方法

在 ftl 文件中，获取作用域对象的数据，有如下两种方式：

- 不省略作用域对象名，语法：`${作用域对象.属性名}`，示例如下：

  ```
  ${Application.attrName}
  ${Request.attrName}
  ```

  

- 省略作用域对象名，语法：`{属性名}`，示例如下：

  ```
  ${attrName}
  ```

  在省略作用域对象名时，Freemarker 会先在模板中查找这个 attrName 变量。如果模板中没有创建这个变量，就会在 Request 中查找该变量，然后在 Session 中查找，最后在 Application 中查找。



#### 2.1.2 底层实现

在了解完基本用法之后，我们再来看看 Freemare 底层是如何实现这两种数据获取方式。



**不省略作用域对象名**

在不省略作用域对象名时，使用方式是`${作用域对象.属性名}`，Freemarer 在解析这个表达式时，会进入方法 `freemarker.core.Dot#_eval` 去获取作用域对象以及对应的属性值。 `freemarker.core.Dot#_eval`的代码如下：

```java
TemplateModel _eval(Environment env) throws TemplateException {
    TemplateModel leftModel = target.eval(env);
    if (leftModel instanceof TemplateHashModel) {
        return ((TemplateHashModel) leftModel).get(key);
    }
    if (leftModel == null && env.isClassicCompatible()) {
        return null; // ${noSuchVar.foo} has just printed nothing in FM 1.
    }
    throw new NonHashException(target, leftModel, env);
}
```

假设表达式为 `${Request.attrName}`，那么在上述代码中的第 2 行 `TemplateModel leftModel = target.eval(env);` 将会获得 HttpServletRequest 对应的对象（HttpRequestHashModel ）。

然后，在第 4 行 ` return ((TemplateHashModel) leftModel).get(key)` 中，get 方法实际上就是调用`freemarker.ext.servlet.HttpRequestHashModel#get` 方法。HttpRequestHashModel 的 get 方法代码如下：

```java
public TemplateModel get(String key) throws TemplateModelException {
    return wrapper.wrap(request.getAttribute(key));
}
```

根据上述代码，我们可以知道该方法实际上就是去 HttpServletRequest 中获取 attrName 对应的 attribute 值。

因为 TemplateHashModel 还可能是 HttpRequestParametersHashModel、HttpSessionHashModel、ServletContextHashModel，所以此处附上这些类的 get 方法实现。

- freemarker.ext.servlet.HttpRequestParametersHashModel#get

  ```java
  public TemplateModel get(String key) {
      String value = request.getParameter(key);
      return value == null ? null : new SimpleScalar(value);
  }
  ```

- freemarker.ext.servlet.HttpSessionHashModel#get

  ```java
  public TemplateModel get(String key) throws TemplateModelException {
      checkSessionExistence();
      return wrapper.wrap(session != null ? session.getAttribute(key) : null);
  }
  ```

- freemarker.ext.servlet.ServletContextHashModel#get

  ```java
  public TemplateModel get(String key) throws TemplateModelException {
      return wrapper.wrap(servletctx.getAttribute(key));
  }
  ```

在了解了 Freemarker 的底层代码实现后，我们也可以确定 **FreemarkerServlet 只能帮我们获取 attribute 值，不能获取类的 property 值**，如：不能获取 ServletContext 中的 contextPath （contextPath 是 ServletContext  类的 property 值，而不是 attribute 值）。





**省略作用域对象名**

在省略作用域对象名时，使用方式是`${attrName}`，而 Freemarer 在解析这个表达式时，会进入方法 `freemarker.ext.servlet.AllHttpScopesHashModel#get` 。而 `AllHttpScopesHashModel#get` 方法则会去各个作用域对象中尝试获取对应的 attribute 值，具体实现代码如下：

```java
public TemplateModel get(String key) throws TemplateModelException {
    // Lookup in page scope
    TemplateModel model = super.get(key);
    if (model != null) {
        return model;
    }

    // Look in unlisted models （unlistedModels 用于存储 Freemarker 的内置对象，如作用域对象 Request、Session 等） 
    model = (TemplateModel) unlistedModels.get(key);
    if (model != null) {
        return model;
    }

    // Lookup in request scope
    Object obj = request.getAttribute(key);
    if (obj != null) {
        return wrap(obj);
    }

    // Lookup in session scope
    HttpSession session = request.getSession(false);
    if (session != null) {
        obj = session.getAttribute(key);
        if (obj != null) {
            return wrap(obj);
        }
    }

    // Lookup in application scope
    obj = context.getAttribute(key);
    if (obj != null) {
        return wrap(obj);
    }

    // return wrapper's null object (probably null).        
    return wrap(null);
}
```

根据上述代码，我们也清楚了在省略作用域对象名时，Freeamarker 获取 attribute 值的顺序。



#### 2.1.3 总结

在讲解完 ftl 的作用域对象的基本用法和实现代码之后，我们再根据[官方文档](https://freemarker.apache.org/docs/api/freemarker/ext/servlet/FreemarkerServlet.html) 中 FreemarkerServlet 的介绍进行回顾总结：

> - It makes all request, request parameters, session, and servlet context attributes available to templates through `Request`, `RequestParameters`, `Session`, and `Application` variables.
> - The scope variables are also available via automatic scope discovery. That is, writing `Application.attrName`, `Session.attrName`, `Request.attrName` is not mandatory; it's enough to write `attrName`, and if no such variable was created in the template, it will search the variable in `Request`, and then in `Session`, and finally in `Application`.

根据文档中的第一点，也可以看出 FreemarkerServlet 只能帮我们获取 attribute 值，不能获取类的 property 值。

而文档中的第二点也解释了省略作用域对象名时，获取 attribute 值的顺序。



### 2.2 获取 ContextPath

#### 2.2.1 尝试的方案及可能的错误

在开发 html 页面时，通常会使用到 css 等静态文件，而这些静态文件的路径需要获取 contextPath。所以，我们在 ftl 文件中也要想办法获取到 contextPath。

那么，如何获取呢？首先，我们看看在 JSP 中是如何获取 contextPath。

JSP 中的获取方式如下：

```jsp
<link rel="stylesheet" type="text/css" href="${pageContext.request.contextPath}/css/bootstrap/bootstrap.css"/>
```

如果，你借鉴上面的写法，尝试在 ftl 文件中通过 Request 对象获取 contextPath。也许，你可能会在 ftl 文件中用如下代码获取。

```html
context path : ${Request.contextPath}
```

但是，当实际运行时，你会看到如下的错误信息：

```
	FreeMarker template error:
	The following has evaluated to null or missing:
==> Request.contextPath  [in template "test.ftl" at line 11, column 18]
```

原因是，` ${Request.contextPath}` 这种方式**是去 HttpServletRequest 中获取名为 contextPath 的 attrbiute 值**，相当于是：`request.getAttribute("contextPath")` ，而不是 `request.getContextPath()`。具体的底层实现，可见上一章节中的 Freemarker 实现代码，对应 `freemarker.ext.servlet.HttpRequestHashModel#get`。



#### 2.2.2 解决方案

当遇到上面的错误之后，你可能会在网上搜索 “freemarker 如何获取 contextPath”。但是，网上提供的解决方案大多是基于 Spring 框架去实现的，而我们目前的场景是在原生 Servlet 环境中使用 Freemarker。

那么在没有 Spring 等其它框架的帮助下，我们如何在 ftl 文件中获取 contextPath 呢？

实际上，很简单！我们可以借助 ServletContextListener，在 context 初始化完后，将 contextPath 放进 ServletContext 的 attributes 中。具体代码如下：

```java
@WebListener
public class WebContextListener implements ServletContextListener {

    public static final String CONTEXT_PATH = "contextPath";

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        ServletContext servletContext = sce.getServletContext();
          // 将 contextPath 放进 ServletContext 的 attribute 中，从而使 ftl 文件可以获得 contextPath。
        servletContext.setAttribute(CONTEXT_PATH, servletContext.getContextPath());
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        ServletContext servletContext = sce.getServletContext();
        if (servletContext.getAttribute(CONTEXT_PATH) != null) {
            servletContext.removeAttribute(CONTEXT_PATH);
        }
    }
}
```

当将 contextPath 放进 ServletContext 的 attributes 后，可以在 ftl 文件中通过如下方式获取：

```html
context path : ${Application.contextPath}
```

至此，你可以在 Freemarker 中像 JSP 一样轻松地获取到 ContextPath。