# springMVC

## 1.回顾mvc

### 1.1、什么是MVC

- MVC是模型（Model），视图（View），控制器（controller）的简写，是一种软件设计规范。
- 是将业务逻辑，数据，显示分离的方法来组织代码。
- MVC主要作用是**降低了视图与业务逻辑间的双向耦合**
- MVC不是设计模式，是一种架构模式

**Model（模型）**：数据模型，提供要展示的数据，因此包含数据和行为，可以认为是领域模型或JavaBean组件（包含数据和行为），不过现在一般都分离开来：Value Object（数据Dao） 和 服务层（行为Service）。也就是模型提供了模型数据查询和模型数据的状态更新等功能，包括数据和业务。

**View（视图）：**负责进行模型的展示，一般就是我们见到的用户界面，客户想看到的东西。

**Controller（控制器）：**接收用户请求，委托给模型进行处理（状态改变），处理完毕后把返回的模型数据返回给视图，由视图负责展示。 也就是说控制器做了个调度员的工作。

最典型的就是javabean+jsp+servlet

<img src="/Users/aixin/Desktop/u盘/java/studyNote/studyNote/springMVC/springMVC.assets/image-20210316160450903.png" alt="image-20210316160450903" style="zoom: 50%;" />

### 1.2、Model1时代

在web早期的时候，通常采用Model1模型。Model1中主要分为两层，视图层和模型层。

![image-20210316160823470](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/springMVC/springMVC.assets/image-20210316160823470.png)

Model1优点：架构简单，比较适合小型项目开发；

Model1缺点：JSP职责不单一，职责过重，不便于维护；

### 1.3、Model2时代

Model2把一个项目分成三部分，包括视图、控制、模型。

<img src="/Users/aixin/Desktop/u盘/java/studyNote/studyNote/springMVC/springMVC.assets/image-20210316160913952.png" alt="image-20210316160913952" style="zoom:50%;" />

步骤分为：

1. 用户发送请求到servlet
2. servlet接收请求并调用业务逻辑代码
3. 业务处理数据，并将结果返回给servlet
4. servlet转向到jsp，由jsp渲染页面
5. 将页面返回给客户端

职责分析：

controller（控制器）：

获取表单数据，调用业务逻辑，转向指定页面

model（模型）：

相关业务逻辑，处理数据

view（视图）：

显示页面

Model2这样不仅提高的代码的复用率与项目的扩展性，且大大降低了项目的维护成本。Model 1模式的实现比较简单，适用于快速开发小规模项目，Model1中JSP页面身兼View和Controller两种角色，将控制逻辑和表现逻辑混杂在一起，从而导致代码的重用性非常低，增加了应用的扩展性和维护的难度。Model2消除了Model1的缺点。

### 1.4、回顾servlet

搭建环境：

1.添加需要的依赖

```xml
<!-- https://mvnrepository.com/artifact/junit/junit -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.3.4</version>
</dependency>

<!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>

<!-- https://mvnrepository.com/artifact/javax.servlet.jsp/jsp-api -->
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
    <scope>provided</scope>
</dependency>

<!-- https://mvnrepository.com/artifact/javax.servlet/jstl -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
```

2.编写servlet代码

```java
public class HelloServlet extends HttpServlet {

    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("get方法");
        req.getSession().setAttribute("msg","hello xin!");
        //业务逻辑
        // 视图跳转
        req.getRequestDispatcher("/WEB-INF/jsp/hello.jsp").forward(req,resp);
    }

    @Override
    public void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("post方法");
    }
}
```

3.编写jsp

在WEB-INF中新建jsp文件夹，新建hello.jsp文件

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
${msg}
</body>
</html>

```

4.配置web.xml

如果项目中没有，idea可以鼠标右击项目-》Add FrameWork Support。然后选择web application。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    
    <servlet>
        <servlet-name>HelloServlet</servlet-name>
        <servlet-class>com.xin.servlet.HelloServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>HelloServlet</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>

</web-app>
```

使用idea配置tomcat，然后运行项目。

MVC框架做的事情：

将url路径映射到对应的java类的方法，封装用户提交的数据，处理请求调用业务逻辑处理，封装数据，将数据渲染到页面中，返回到客户端进行展示。

**说明：**

常见的服务器端MVC框架有：Struts，Spring MVC，ASP.NET MVC。

常见的前端MVC框架有：vue，angular js，react。

由MVC演化的另外一些模式：MVP，MVVM等



## 2.什么是spring MVC

### 2.1、概述

Spring MVC是Spring FrameWork的一部分，是基于Java实现MVC的轻量级MVC框架。

官方文档：https://docs.spring.io/spring-framework/docs/5.2.0.RELEASE/spring-framework-reference/web.html#spring-web

Spring MVC的特点：

1.轻量级，简单易学

2.高效，基于请求相应的MVC框架

3.与Spring兼容性好，无缝结合

4.约定优于配置

5.功能强大：RESTFUL、数据验证、格式化、本地化、主题等

6.简洁灵活

### 2.1、中心控制器DispatcherServlet

Spring 的web框架是围绕**DispatcherServlet**设计。

**DispatcherServlet**的作用是将请求分发给不同的处理器。从Spring2.5开始，使用java 5及以上版本的用户可以采用注解方式来进行开发。

Spring MVC像许多其他的MVC框架一样，以请求为驱动，围绕着一个中心Servlet分派请求及提供其他功能，**DispatcherServlet**是一个实际的Servlet

![image-20210316173520458](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/springMVC/springMVC.assets/image-20210316173520458.png)

SpringMVC原理：

当发起请求的时候首先被前置控制器拦截请求，根据参数生成代理请求找到对应的实际控制器，控制器处理请求，创建数据模型，连接数据库处理数据，将数据模型返还给中心控制器，中心控制器将数据模型传给视图解析器，视图解析器解析后将具体的view传给中心控制器，中心控制器再对view进行渲染视图（将模型数据填充到视图中）

<img src="/Users/aixin/Desktop/u盘/java/studyNote/studyNote/springMVC/springMVC.assets/image-20210317165803627.png" alt="image-20210317165803627" style="zoom:50%;" />

### 2.3、Spring执行原理

![image-20210317170329047](/Users/aixin/Desktop/u盘/java/studyNote/studyNote/springMVC/springMVC.assets/image-20210317170329047.png)

实现表示Spring已经做好的事情不需要开发者进行实现，需要表示开发者需要做的事情。

简单分析执行流程：

1.用户发送请求至前端控制器DispatcherServlet
2.DispatcherServlet收到请求调用处理器映射器HandlerMapping。
3.处理器映射器根据请求url找到具体的处理器，生成处理器执行链HandlerExecutionChain(包括处理器对象和处理器拦截器)一并返回给DispatcherServlet。
4.DispatcherServlet根据处理器Handler获取处理器适配器HandlerAdapter执行HandlerAdapter处理一系列的操作，如：参数封装，数据格式转换，数据验证等操作
5.执行处理器Handler(Controller，也叫页面控制器)。
6.Handler执行完成返回ModelAndView
7.HandlerAdapter将Handler执行结果ModelAndView返回到DispatcherServlet
8.DispatcherServlet将ModelAndView传给ViewReslover视图解析器
9.ViewReslover解析后返回具体View
10.DispatcherServlet对View进行渲染视图（即将模型数据model填充至视图中）。
11.DispatcherServlet响应用户。

## 3.Hello Spring MVC

### 3.1、配置版

1.环境搭建

新建一个module，添加web支持：idea鼠标右击项目-》Add FrameWork Support。然后选择web application。

2.配置web.xml

```xml
<!--1.注册DispatcherServlet-->
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--关联一个springmvc的配置文件:【servlet-name】-servlet.xml-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:hello2.xml</param-value>
    </init-param>
    <!--启动级别-1-->
    <load-on-startup>1</load-on-startup>
</servlet>

<!--/ 匹配所有的请求；（不包括.jsp）-->
<!--/* 匹配所有的请求；（包括.jsp）-->
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

3.在resources文件夹下新建hello2.xml(spring配置文件)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--处理映射器-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
    <!--处理器适配器-->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

    <!--视图解析器:DispatcherServlet给他的ModelAndView-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="InternalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!--后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>

    <bean id="/hello" class="com.xin.controller.HelloController"/>

</beans>
```

4.新建controller,将controller配置到spring配置文件中

```java
public class HelloController implements Controller {


    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        //ModelAndView 模型和视图
        ModelAndView mv = new ModelAndView();


        //封装对象，放在ModelAndView中。Model
        mv.addObject("msg","HelloSpringMVC!");
        //封装要跳转的视图，放在ModelAndView中
        mv.setViewName("hello"); //: /WEB-INF/jsp/hello.jsp
        return mv;
    }

}
```

5.在WEB-INF下新建jsp文件夹并新建hello.jsp文件

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
${msg}
</body>
</html>
```

6.记得在项目结构中在WEB-INF中新建lib文件夹，并将所有依赖的jar包让进去，否则会404，然后配置tomcat运行。如果存在500可能是tomcat配置问题，新建一个module测试。

### 3.2、注解版

1.环境搭建

新建一个module，添加web支持：idea鼠标右击项目-》Add FrameWork Support。然后选择web application。

依赖同配置版依赖。

此处可能存在maven资源过滤问题，需要在pom.xml中加入

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```

2.配置web.xml

```xml
<servlet>
    <servlet-name>springMVCAnnotation</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--通过初始化参数指定SpringMVC配置文件的位置，进行关联-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:helloAnnotation.xml</param-value>
    </init-param>
    <!-- 启动顺序，数字越小，优先级越高 -->
    <load-on-startup>1</load-on-startup>
</servlet>

<!--所有请求都会被springmvc拦截 -->
<servlet-mapping>
    <servlet-name>springMVCAnnotation</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

**/ 和 /* 的区别：**
< url-pattern > / </ url-pattern > 不会匹配到.jsp， 只针对我们编写的请求；即：.jsp 不会进入spring的 DispatcherServlet类 。< url-pattern > /* </ url-pattern > 会匹配 *.jsp，会出现返回 jsp视图 时再次进入spring的DispatcherServlet 类，导致找不到对应的controller所以报404错。

3.配置spring配置文件

注意要添加对应的头文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 自动扫描包，让指定包下的注解生效,由IOC容器统一管理 -->
    <context:component-scan base-package="com.xin.controller"/>
    <!-- 让Spring MVC不处理静态资源 -->
    <mvc:default-servlet-handler />
    <!-- 支持mvc注解驱动
    在spring中一般采用@RequestMapping注解来完成映射关系
    要想使@RequestMapping注解生效
    必须向上下文中注册DefaultAnnotationHandlerMapping 和一个AnnotationMethodHandlerAdapter实例
    这两个实例分别在类级别和方法级别处理。
    而annotation-driven配置帮助我们自动完成上述两个实例的注入。 -->
    <mvc:annotation-driven/>

    <!-- 视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```

4.编写controller

```java
@Controller
@RequestMapping("/hello")
public class HelloController {
    @RequestMapping("/test")
    public String test(Model model){
        model.addAttribute("msg","helloAnnotation");
        return "test";
    }
}
```

- @Controller是为了让Spring IOC容器初始化时自动扫描到；
- @RequestMapping是为了映射请求路径，这里因为类与方法上都有映射所以访问时应该是/hello/test
- 方法中声明Model类型的参数是为了把Action中的数据带到视图中
- 方法返回的结果是视图的名称test，加上配置文件中的前后缀变成/WEB-INF/jsp/test.jsp

5.tomcat配置，然后运行



## 4.controller及restful

### 4.1、controller控制器

- controller负责提供访问程序的行为，可以通过实现接口与注解方式来实现
- controller负责解析用户请求并返回一个模型
- 在spring MVC中，一个控制器类可以包含多个方法

两种实现方法在hello SpringMVC中已经都进行实现了。

使用实现接口方法来实现，每一个controller只能有一个方法，不灵活。

使用接口来实现，简单配置少，一个controller可以有多个方法。

### 4.2、@RequestMapping

@RequestMapping注解用于映射url到具体的类或方法上。如果用于类上则表示类中的所有方法都以此为父路径。

### 4.3、Restful风格

Restful就是一个资源定位及资源操作风格，不是标准也不是协议。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

```java
@Controller
@RequestMapping("/restful")
public class RestfulController {

    @RequestMapping(value = "/test/{a}/{b}",method = RequestMethod.POST)
    public String test(@PathVariable int a, @PathVariable int b, Model model){
        int c = a + b;
        model.addAttribute("msg",a+"+"+b+"="+c);
        return "test";
    }

    @RequestMapping(value = "/test/{a}/{b}",method = RequestMethod.GET)
    public String test1(@PathVariable int a, @PathVariable String b, Model model){
        String c = a + b;
        model.addAttribute("msg",a+"+"+b+"="+c);
        return "test";
    }
}
```

所有的地址栏请求默认都会是 HTTP GET 类型的。

```
@GetMapping 相当于@RequestMapping(method =RequestMethod.GET)
@PostMapping
@PutMapping
@DeleteMapping 
@PatchMapping
```

## 5.结果跳转方式

### 5.1、ModelAndView

设置ModelAndView对象 , 根据view的名称 , 和视图解析器跳到指定的页面 .

页面 : {视图解析器前缀} + viewName +{视图解析器后缀}

```xml
<!-- 视图解析器 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

```java
public class TestController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        ModelAndView view = new ModelAndView();
        view.addObject("msg","ModelAndView");
        view.setViewName("test");
        return view;
    }
}
```

注意：实现controller要在spring配置文件中注入bean，id为访问地址，不能使用@RequestMapping注解来声明url。

### 5.2、ServletAPI

通过设置ServletAPI , 不需要视图解析器 。

- 通过HttpServletResponse进行输出
- 通过HttpServletResponse实现重定向
- 通过HttpServletResponse实现转发

```java
@Controller
@RequestMapping("/servlet")
public class ServletController {

    @RequestMapping("/t1")
    public void test1(HttpServletRequest req, HttpServletResponse rsp) throws IOException {
        rsp.getWriter().write("ServletController-t1");
    }

    @RequestMapping("/t2")
    public void t2(HttpServletRequest req, HttpServletResponse rsp) throws IOException {
        rsp.sendRedirect("/a.jsp");
    }

    @RequestMapping("/t3")
    public void t3(HttpServletRequest req, HttpServletResponse rsp) throws IOException, ServletException {
        req.setAttribute("msg","ServletController-t3");
        req.getRequestDispatcher("/WEB-INF/jsp/test.jsp").forward(req,rsp);
    }

}
```

注意：tomcat如果设置了项目的名称会：localhost:8080/projectName，这样转发和重定向就会有问题，再不就在转发与重定向的路径上加上设置的项目名称或者在tomcat配置上把项目名称去掉。

### 5.3、springMVC

使用springMVC来实现转发与重定向

```java
@Controller
@RequestMapping("/resultSpringMVC")
public class ResultSpringMVC {

    @RequestMapping("/t1")
    public String test1(){
        //转发
        return "test";
    }

    @RequestMapping("/t2")
    public String test2(){
        //转发
        return "forward:/WEB-INF/jsp/test.jsp";
    }

    @RequestMapping("/t3")
    public String test3(){
        //重定向
        return "redirect:/a.jsp";
    }

}
```

==在重定向那里我使用"redirect:/WEB-INF/jsp/test.jsp"就会报404，原因还未查明。==

## 6.数据处理

### 6.1、处理提交数据

1、提交的域名城与方法的参数名一致

提交数据：http://localhost:8080/hello/test?name=aixin

```java
@Controller
@RequestMapping("/hello")
public class HelloController {
    @RequestMapping("/test")
    public String test(String name,Model model){
        model.addAttribute("msg",name);
        return "test";
    }
}
```

2、提交的域名与方法的参数名不一致

提交数据：http://localhost:8080/hello/test?userName=aixin

```java
@Controller
@RequestMapping("/hello")
public class HelloController {
    @RequestMapping("/test")
    public String test(@RequestParam("userName") String name, Model model){
        model.addAttribute("msg",name);
        return "test";
    }
}
```

如果提交的数据再为name则会报错

3、提交的是一个对象

提交数据：http://localhost:8080/hello/test2?id=123aa&name=aixin&age=25

```java
public class User {
    private String id;
    private String name;
    private int age;
		//无参、有参构造器，getter、setter
}
```

```java
@RequestMapping("/test2")
public String test2(User user, Model model){
    model.addAttribute("msg",user.toString());
    return "test";
}
```

前端传的数据名称与接收的对象的属性名称需相同，否则会以该类型的默认值赋上。

### 6.2、数据显示到前端

ModelAndView与Model前面都已经写了。

ModelMap：

```java
@RequestMapping("/test2")
public String test2(User user, ModelMap model){
    model.addAttribute("msg",user.toString());
    return "test";
}
```

```properties
Model 只有寥寥几个方法只适合用于储存数据，简化了新手对于Model对象的操作和理解；

ModelMap 继承了 LinkedMap ，除了实现了自身的一些方法，同样的继承 LinkedMap 的方法和特性；

ModelAndView 可以在储存数据的同时，可以进行设置返回的逻辑视图，进行控制展示层的跳转。
```

### 6.3、乱码问题

处理方式：

1.在web.xml中添加一个过滤器

```xml
<filter>
    <filter-name>encoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encoding</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```



2.修改tomcat配置文件，设置编码

```xml
<Connector URIEncoding="utf-8" port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
```

自己编写过滤器

```java
 /**
 * 解决get和post请求 全部乱码的过滤器
 */
public class GenericEncodingFilter implements Filter {
 
    @Override
    public void destroy() {
    }
 
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        //处理response的字符编码
        HttpServletResponse myResponse=(HttpServletResponse) response;
        myResponse.setContentType("text/html;charset=UTF-8");
 
        // 转型为与协议相关对象
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        // 对request包装增强
        HttpServletRequest myrequest = new MyRequest(httpServletRequest);
        chain.doFilter(myrequest, response);
    }
 
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }
 
}
 
//自定义request对象，HttpServletRequest的包装类
class MyRequest extends HttpServletRequestWrapper {
 
    private HttpServletRequest request;
    //是否编码的标记
    private boolean hasEncode;
    //定义一个可以传入HttpServletRequest对象的构造函数，以便对其进行装饰
    public MyRequest(HttpServletRequest request) {
        super(request);// super必须写
        this.request = request;
    }
 
    // 对需要增强方法 进行覆盖
    @Override
    public Map getParameterMap() {
        // 先获得请求方式
        String method = request.getMethod();
        if (method.equalsIgnoreCase("post")) {
            // post请求
            try {
                // 处理post乱码
                request.setCharacterEncoding("utf-8");
                return request.getParameterMap();
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
        } else if (method.equalsIgnoreCase("get")) {
            // get请求
            Map<String, String[]> parameterMap = request.getParameterMap();
            if (!hasEncode) { // 确保get手动编码逻辑只运行一次
                for (String parameterName : parameterMap.keySet()) {
                    String[] values = parameterMap.get(parameterName);
                    if (values != null) {
                        for (int i = 0; i < values.length; i++) {
                            try {
                                // 处理get乱码
                                values[i] = new String(values[i]
                                        .getBytes("ISO-8859-1"), "utf-8");
                            } catch (UnsupportedEncodingException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }
                hasEncode = true;
            }
            return parameterMap;
        }
        return super.getParameterMap();
    }
 
    //取一个值
    @Override
    public String getParameter(String name) {
        Map<String, String[]> parameterMap = getParameterMap();
        String[] values = parameterMap.get(name);
        if (values == null) {
            return null;
        }
        return values[0]; // 取回参数的第一个值
    }
 
    //取所有值
    @Override
    public String[] getParameterValues(String name) {
        Map<String, String[]> parameterMap = getParameterMap();
        String[] values = parameterMap.get(name);
        return values;
    }
}
```

然后在web.xml中配置这个过滤器即可。

**乱码问题，需要平时多注意，在尽可能能设置编码的地方，都设置为统一编码 UTF-8！** 

## 7.整合SSM

==整合遇到的问题：junit测试控制台显示class not found，mvn clean install就好了，每次修改代码只有mvn clean install才生效解决办法：将项目关闭，然后将.idea与.iml文件删除，再重新打开就ok==

### 7.1、数据库与依赖

1.数据库

```sql
CREATE DATABASE `ssmbuild`;

USE `ssmbuild`;

DROP TABLE IF EXISTS `books`;

CREATE TABLE `books` (
  `bookID` int(10) NOT NULL AUTO_INCREMENT COMMENT '书id',
  `bookName` varchar(100) NOT NULL COMMENT '书名',
  `bookCounts` int(11) NOT NULL COMMENT '数量',
  `detail` varchar(200) NOT NULL COMMENT '描述',
  KEY `bookID` (`bookID`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

```

2.依赖

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>

<!--数据库 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.16</version>
</dependency>
<!-- 数据库连接池 c3p0-->
<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.5.2</version>
</dependency>

<!--Servlet - JSP -->
<!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/javax.servlet.jsp/jsp-api -->
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/javax.servlet/jstl -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>

<!--Mybatis-->
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.6</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.6</version>
</dependency>

<!--  spring-->
<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.1.9.RELEASE</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.3.5</version>
</dependency>
<!-- lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.8</version>
</dependency>
```

3.静态资源问题

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```

4.项目基本结构与配置文件

- com.xin.beans

- com.xin.dao

- com.xin.service

- com.xin.controller

- mybatis-config.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
  
  </configuration>
  ```

- applicationContext.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd">
  
  </beans>
  ```

  