---
layout: post
title: 何为devops
date: 2018-1-08
categories: spring
description: 俯视springMvc
author: 周瑞进
---


### 认识SpringMvc
   作为web的开发人员，你是否知道自己写的web代码本质是什么，为什么我们开发的java应用能在tomcat上应用(或其它java服务器)。因为我们的代码是根据servlet规范来开发的，它制定了java中处理请求的标准。继承了servlet我们需要实现它的几个方法，常见的就是init、service、destroy等，而一个tomcat服务它是有很多层次结构的，但它的每一层基本都实现了init等方法，当我们的应用部署到tomcat里面时，tomcat的顶层容器就逐一的调用下一层的init方法来触发子容器，到了最后一层就能触发到了我们的servlet程序，所以我们的应用就能够被访问了。
SpringMvc是一个很成熟并被广泛应用的开发框架，其实它的本质也是servlet，只是它帮我们封装好了，让我们可以根据自己的业务更灵活的开发，而不用生硬的遵守servlet的规范。

#### 1、servlet
servlet它是一套规范，所以它主要的工作就是定义一些接口，每一个接口解决不同的逻辑功能，但是具体的实现要取决于实现他的人。
servlet3.1.0中servlet的源码如下：
```
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();

    void destroy();
}
```

init方法会在容器(tomcat)启动时被调用，但是我们一个应用有非常多的servlet，所以当然不是每个servlet的init都会在启动时被调用，我们在web.xml配置servlet时会看到一个(load-on-start)的设置。

1. load-on-startup元素标记容器是否在启动的时候就加载这个servlet(实例化并调用其init()方法)。
2. 它的值必须是一个整数，表示servlet应该被载入的顺序
3. 当值为0或者大于0时，表示容器在应用启动时就加载并初始化这个    servlet；
4. 当值小于0或者没有指定时，则表示容器在该servlet被选择时才会去加载。
5. 正数的值越小，该servlet的优先级越高，应用启动时就越先加载。
6. 当值相同时，容器就会自己选择顺序来加载。

    一般我们写一个servlet会继承HttpServlet,从下面的源码中我们又可以看到，HttpServlet 继承了GenericServlet ，而GenericServlet 实现了Servlet和ServletConfig接口。（我是觉得看一个框架的结构树，还不如应用打开，一层层的跟进去看看它的代码实现可能映像更深刻）
```
public abstract class HttpServlet extends GenericServlet {
```

```
public abstract class GenericServlet implements Servlet, ServletConfig, Serializable {
```

 - ServletConfig：顾名思义存放的是servlet配置的。
 - Serializable ：将该java对象序列化（也就是将它在内存中的各个状态记录下来，保存成文件或者通过网络传到别的系统，其它应用读出这个序列话对象时就能还原成这个对象）
 - GenericServlet ：主要做了3件事，简单讲就是让我们更加方便的使用servlet。
  - 实现了ServletConfig接口
  - 提供了无参的init()方法
  - 提供了log方法

#### 2、tomcat
   Tomcat中最顶层的容器是server，它代表整个服务器，server中至少包含一个service，用于提供具体的服务。service主要包含两部分：connector和container。connect中主要处理链接相关的事情，并提供socket与request、response的转换。container用于封装和管理servlet，以及具体处理request的请求。
![tomcat服务器结构（图片来自互联网）](http://img.blog.csdn.net/20160512222519535)
      
   server中提供了init和start方法，当容器启动后他们分别循环调用了每个init方法和start方法来启动说有service，service中又调用了子容器的init和start方法。以此递推进去就能调到servlet中的init方法了。

### 环境搭建
开发换件用IDEA，并使用maven架构。(maven如果不清楚可以自己额外去了解)
#### 新建工程
File --》 new Module --》 Maven  (注：Idea中的module就是项目，等同于eclipse的project)
![这里写图片描述](http://img.blog.csdn.net/20160512224425887)

项目的简单工程结构如下(用脚手架自动生成的工程要是没有相应的结构可以根据自己的需要新建)
![工程结构](http://img.blog.csdn.net/20160512224952482)


#### 配置环境
 1.首先要在pom.xml中加入springmvc 和 servlet的依赖
```
 <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>4.1.5.RELEASE</version>
    </dependency>
  </dependencies>
```
2.web.xml中配置servlet
```
    <!-- spring mvc配置开始  -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```
contextConfigLocation指定了springmvc配置文件的具体位置，如果没有指定就默认使用WEB-INFO/[ServletName]-servlet.xml文件。
3.创建SpringMvc的配置文件
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	   xmlns:context="http://www.springframework.org/schema/context"
	   xmlns:mvc="http://www.springframework.org/schema/mvc"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
   http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

	<mvc:annotation-driven/>
	<context:component-scan base-package="com.sunshine.springmvc.web" />

	<!-- 声明viewResolver -->
	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="view/" />
		<property name="suffix" value=".jsp" />
	</bean>
</beans>
```
`<mvc:annotation-driven/>`会帮我们做一些自动注册组件的事情。

```
<context:component-scan base-package="com.sunshine.springmvc.web"/>指定了制定了要扫描配置的类的地方。
```
在这一串配置中最令人难懂的应该就是头部的一大串声明了。这一串称之为Schema-based XML的声明，引入了这个声明就可以简单的在下面定义我们的组件。如：mvc:annotation、context:component-scan

其实这2个和下面通过`<bean>` 标签的作用是相同的：对SpringMVC中的组件进行声明，指定组件的行为方式。
- 基于Schema-based XML的配置定义模式 
- 基于Traditional XML的配置定义模式   
毫无疑问第一种的配置是比较简单的，第二种要一个个servlet去定义，而第二种是根据整个逻辑功能引入的，像mvc:annotation-driven，这个用来注册组件的功能里就注册了好几个servlet。
有关Schema-based XML的概念，可以参考Spring官方的reference： 
[Appendix C. XML Schema-based configuration ](http://docs.spring.io/spring/docs/3.1.0.RELEASE/reference/html/xsd-config.html+%20+%E2%80%9CAppendix%20C.%20XML%20Schema-based%20configuration%20%E2%80%9D)
### 组件介绍
![srpingmvc继承结构图](http://img.blog.csdn.net/20160512235329776)

从上图我们可以看到springmv组要的结构是HttpServletBean、FrameworkServlet、DispatcherServlet，其它的是上面讲的java servlet api中的结构。

 - HttpServletBean
   - 将servlet中配置的参数设置到DispacherServlet的相关属性
   - 调用模板方法initServletBean，子类就可以通过这个方法初始化
 - FrameworkServlet
    - 初始化了WebApplication
      - 获取了spring的根容器rootContext
      - 设置webApplicationContext并根据情况调用onRefresh方法。
      - 将webApplicationContext设置到ServletContext中
 - DispatcherServlet
    上面的onRefresh方法是DispatcherServlet的入口方法。onRefresh中简单调用了initStrategies,在initStrategies中调用了9个初始化方法：
```
    protected void onRefresh(ApplicationContext context) {
        this.initStrategies(context);
    }

    protected void initStrategies(ApplicationContext context) {
        this.initMultipartResolver(context);
        this.initLocaleResolver(context);
        this.initThemeResolver(context);
        this.initHandlerMappings(context);
        this.initHandlerAdapters(context);
        this.initHandlerExceptionResolvers(context);
        this.initRequestToViewNameTranslator(context);
        this.initViewResolvers(context);
        this.initFlashMapManager(context);
    }

```

#### 1. HanderMapping
根据request找到相应的处理器hander 和 Interceptors

#### 2. HandlerAdapter
handler是具体处理事情的，HandlerAdapter就是使用handler来完成这件事的。adapter是适配器的意思，可能到adapter的请求各种各样，但他们要完成的都是同一个handler这件事。
#### 3. HandlerExceptionResolver
只用于对请求做处理过程中产生的异常，而渲染环节产生的异常不归它管。
#### 4. ViewResolver
用来将String类型的视图名和locale（国际化用的有默认值）解析为view类型的视图。
#### 5. RequestToViewNameTranslator
ViewResolver是根据viewname查找view，当viewname没有的话，就要从request中获取了。
#### 6. LocaleResolver
用于从request中解析出locale，提供给ViewResolver作为参数用。
#### 7. ThemeResolver
解析主体的。
#### 8. MultipartResolver
用于处理上传请求。
#### 9. FlashMapManager
主要用于redirect中传递参数的

###总结（处理流程）
1. 请求发送到服务器，服务器分配一个socket线程跟它链接，接着创建出request和response,接着交给对应的servlert处理。
2. servlet中的请求首先会被httpservlet处理，将servletrequest和servletresponse转换为httpservletrequest和httpservletresponse，然后调用service方法。
3. 接着请求就来到了springmvc,首先到达的是FrameworkServlet,做了一些初始化动作后讲请求交给DispatcherServlet
4. DispatcherServlet是SpringMvc的核心内容，它需要将请求交给对应的hander处理，但是hander有很多个，它需要先通过handerMapping,找到对应的hander.
5. 找到了hander后，DispatcherServlet将它交给具体的handerAdapter处理。
6. 处理完业务逻辑后会调用modelAndView对结果进行处理。
7. 之后交给viewResovler,根据数据(model)和试图(view)渲染成页面返回。


----------


*以上是自己学习之后的总结，可能有些地方自己的理解有偏误，欢迎指正，仅供参考。*



