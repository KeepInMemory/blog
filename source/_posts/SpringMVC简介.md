---
title: SpringMVC
date: 2020-06-15 14:08:43
tags:
  - SpringMVC
---

SpringMVC是一种基于Java实现的MVC设计模型的请求驱动类型的轻量级web框架，是SpringFrameWork的后续产品

1.导入SpringMVC的坐标 2.配置Servlet前端控制器DispatherServlet 3.编写Controller类和视图页面 4.将Controller使用注解配置到Spring容器(@Controller)，配置请求映射的是哪个方法 5.配置spring-mvc.xml文件(配置组件扫描) 6.执行访问测试

<!--more-->

1.prm.xml导坐标

```java
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.0.5.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>5.0.5.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.0.1</version>
    </dependency>
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>javax.servlet.jsp-api</artifactId>
      <version>2.2.1</version>
    </dependency>
```

2.配置前端控制器

```java
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>

<!-- 配置SpringMVC的前端控制器-->
  <servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
<!--配置映射地址-->
  <servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

3.4.编写POJOController配置注解

```java
package com.it.controller;

import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.RequestMapping;

@Component
public class UserController {

    @RequestMapping("/quick")
    public String save() {
        System.out.println("Controller save running");
        return "success.jsp";
    }
}
```

UrlBasedViewResolver类里区分REDIRECT_URL_PREFIX="redirect:" FORWARD_URL_PREFIX="forward:"，通过这个区分重定向和转发，例如在上面save方法里写return "success.jsp"，这里默认省略了return "forward:success.jsp"，如果想改成重定向方式则改成return "redirect:success.jsp"

若要return "jsp/success.jsp"重新配置视图解析器可以省略前后缀

```java
<bean id="viewSolver" class="org.springframeowork.web.servlet.view.InternalResourceViewResolver">
	<property name="perfix" value="/jsp/"></property>
	<property name="suffix" value=".jsp"></property>
</bean>
```

重新配置视图解析器后就可写成return "success"

4.配置spring-mvc.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd">

<!--Controller组件扫描-->
        <context:component-scan base-package="com.it.controller"></context:component-scan>
</beans>
```

当有请求访问localhost:8080/quick时找Tomcat，Tomcat会找DispatcherServlet，因为作为前端控制器它的url-pattern是缺省的，所有请求都要过DispatcherServlet，根据DispatcherServlet指定的spring-mvc.xml去扫包去找对应的Web部分的Controller，找到@RequestMapping(/quick)时对应上，执行save方法

#### SpringMVC相关组件

前端控制器DispatcherServlet：调用其它功能组件

处理器映射器HandlerMapping：进行地址解析，返回执行链

处理器适配器HandlerAdapter：被前端控制器调用，去调用处理器

处理器Handler：相当于我们自己封装的Controller

视图解析器ViewResolver：将ModelAndView的View解析出来

视图View

#### SpringMVC的数据响应

##### 页面跳转

1.返回字符串形式：此种方式会将返回的字符串与视图解析器的前后缀拼接后跳转

2.返回ModelAndView

```java
package com.it.controller;

import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Component
public class UserController {

    @RequestMapping("/quick2")
    public ModelAndView save2() {
//        Model：模型 作用封装数据
//        View：视图 作用展示数据
        ModelAndView modelAndView = new ModelAndView();
        //设置模型数据,和之前的setAttribute一样，将数据放在域中可以在前端获取
        modelAndView.addObject("username","itcast");
        //设置视图名称
        modelAndView.setViewName("success");
        return modelAndView;
    }
        @RequestMapping("/quick3")
    public ModelAndView save3(ModelAndView modelAndView) {
        //设置模型数据,和之前的addAttribute一样，将数据放在域中可以在前端获取
        modelAndView.addObject("username","itcast");
        //设置视图名称
        modelAndView.setViewName("success");
        return modelAndView;
    }
    
        @RequestMapping("/quick4")
    public String save3(Model model) {
        model.addAttribute("username","bob");
        return "success.jsp";
    }
}
```

SpringMVC在解析方法的时候发现参数是ModelAndView、Model等需要提供，它就会从容器中将已有的对象注入

##### 回写数据

1.直接返回字符串

将需要回写的字符串直接返回，需要@ResponseBody注解告诉SpringMVC，方法返回的字符串不是跳转

```java
package com.it.controller;

import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import java.io.IOException;

@Component
public class UserController {

    @RequestMapping("/quick")
    @ResponseBody
    public String save()throws IOException {
        return "hello";
    }
}
```

2.返回对象和集合

使用在xml里用<mvc:annotation-driven>就能在方法上继续使用@ResponseBody(告知不是跳转)返回Json

#### SpringMVC获得请求数据

##### 基本类型参数

Controller中的方法参数名称要与请求参数名一致，参数值会自动映射匹配

http://localhost:8080/Spring-MVC/quick?username=zhangsan&age=12

```java
@RequestMapping("/quick")
@ResponseBody
public void quickMethod(String username,int age)throws IOException {
	System.out.println(username+age);
}
```

##### POJO类型参数

Controller中的方法的POJO参数的属性名与请求参数名一致，参数值会自动映射匹配

http://localhost:8080/Spring-MVC/quick?username=zhangsan&age=12

```java
public class User {
	private String username;
	private int age;
	getter/setter...
}
@RequestMapping("/quick")
@ResponseBody
public void quickMethod(User user)throws IOException {
	System.out.println(user);
}
```

##### 数组类型参数

Controller中的方法的数组名与请求参数名一致，参数值会自动映射匹配

http://localhost:8080/Spring-MVC/quick?strs=111&strs=222&strs=333

```java
@RequestMapping("/quick")
@ResponseBody
public void quickMethod(String[] strs)throws IOException {
	System.out.println(Arrays.asList(strs));
}
```

因为数组打印都是地址，List打印是值，所以这里转换成List

##### 集合类型参数

当使用ajax提交时，可以指定contentType为Json格式，Controller的方法参数位置加@RequestBody可以直接接收集合数据

```java
@RequestMapping("/quick")
@ResponseBody
public void quickMethod(@ResponseBody List<User> userList)throws IOException {
	System.out.println(userList);
}
```

##### 参数绑定注解@requestParam

http://localhost:8080/Spring-MVC/quick?name=zhangsan

value指请求的参数名称(和Controller方法参数可以不一致)

required请求的参数是否必须有它，没有就报错

defaultValue没有该参数时它的默认值是什么

```java
@RequestMapping("/quick")
@ResponseBody
public void quickMethod(@RequestParam(value="name",required=false,defaultValue="itcast") String username)throws IOException {
	System.out.println(username);
}
```

##### Restful风格参数

Restful风格：使用"url+请求方式"进行请求

/user/1 GET:得到id=1的user

/user/1 DELETE:删除id=1的user

/user/1 PUT:更新id=1的user

/user POST:新增user

http://localhost:8080/Spring-MVC/quick/zhangsan

```java
@RequestMapping("/quick/{name}")
@ResponseBody
public void quickMethod(@PathVariable(value="name",required=true) String name) {
	System.out.println(name);
}
```

在SpringMVC中可以使用占位符继续参数绑定，占位符{name}对应的就是zhangsan，Controller方法参数使用@PathVariable用value的值去匹配占位符，匹配到赋值给后面的变量，也就是说String name的参数名可以改，但是value与占位符必须是对应的

##### 获得请求头

1.@RequestHeader

可以获得指定请求头信息，相当于request.getHeader(name)

```java
@RequestMapping("/quick")
@ResponseBody
public void quickMethod(@RequestHeader(value="User-Agent",required=false) String name) {
	System.out.println(name);
}
```

value：请求头名称，required：是否必须携带此请求头

2.@CookieValue

可以获得指定Cookie的值，value和required同上

```java
@RequestMapping("/quick")
@ResponseBody
public void quickMethod(@CookieValue(value="JSESSIONID",required=false) String jsessionid) {
	System.out.println(jsessionid);
}
```

#### SpringMVC拦截器

用于对处理器进行预处理和后处理

将拦截器按一定顺序联结成一条链，叫拦截器链。在访问被拦截的方法或字段时，拦截链中的拦截器就会按其之前定义的顺序被调用。拦截器也是AOP思想的具体实现。

与过滤器Filter的区别

|   区别   |                      过滤器                      |                            拦截器                            |
| :------: | :----------------------------------------------: | :----------------------------------------------------------: |
| 使用范围 | 是servlet规范的一部分，任何JavaWeb项目都可以使用 |        属于SpringMVC框架，只有SpringMVC框架项目才能用        |
| 拦截范围 |          可以对所有要访问的资源进行拦截          | 只会拦截访问的Controller方法，如果访问的是jsp,html,css,js,image是不会进行拦截的 |

自定义拦截器步骤:

1.创建拦截器类实现HandlerInterceptor接口

```java
package com.it.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class MyInterceptor implements HandlerInterceptor {
    @Override
    //在目标方法执行之前执行
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle");
        return false;
    }

    @Override
    //在目标方法执行之后，视图对象返回执行之前
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle");
    }

    @Override
    //在所有流程执行完毕后执行
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion");
    }
}
```

preHandle方法返回false代笔拦截住，不继续执行，返回true代表放行。

 2.xml中配置拦截器

```java
<!--配置拦截器-->
        <mvc:interceptors>
                <mvc:interceptor>
                        <mvc:mapping path="/**"/>
                        <bean class="com.it.interceptor.MyInterceptor1"></bean>
                </mvc:interceptor>
                 <mvc:interceptor>
                        <mvc:mapping path="/**"/>
                        <bean class="com.it.interceptor.MyInterceptor2"></bean>
                </mvc:interceptor>
        </mvc:interceptors>
</beans>
```

拦截器链MyInterceptor1、MyInterceptor2，1的方法内套着2的方法

所以执行顺序是preHandle1->preHandle2->postHandle2->postHandle1->afterCompletion2->afterCompletion1

#### SpringMVC异常处理

![image-20200618105544097](SpringMVC简介\image-20200618105544097.png)

##### 异常处理两种方式

1.简单异常处理器SimpleMappingExceptionResolver 

SpringMVC已经定义好了，在使用时可以根据需求对异常和视图的映射进行配置

```java
<!--配置简单映射异常处理器-->
<bean class=""org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
	<property name="defaultErrorView" value="error"/>
	<property name="exceptionMappings">
		<map>
			<entry key="com.it.exception.MyException" value="error1"/>
			<entry key="java.lang.ClassCastException" value='error2'/>
		</map>
	</property>
</bean>
```

property的name指的是映射的类型，value是对应的错误页面。defaultErrorView是在底下的exceptionMappings都没匹配到就访问value的error界面，exceptionMappings是异常与对应的界面

2.使用Spring异常处理接口HandlerExceptionResolver自定义

创建异常处理器类实现HandlerExceptionResolver

```java
package com.it.resolver;

import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class MyExcpetionResovler implements HandlerExceptionResolver {

    @Override
    /*
    参数Exception：已经报的异常对象
    返回值ModelAndView：要跳转到的视图信息
     */
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
        ModelAndView modelAndView = new ModelAndView();
        if (e instanceof MyExcpetion)
            modelAndView.setViewName("error");
        return modelAndView;
    }
}
```

配置异常处理器

```java
<bean class="com.it.resolver.MyExceptionResolver"/>
```

