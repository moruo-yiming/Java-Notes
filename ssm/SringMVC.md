# SpringMVC

### 1、介绍



### 2、原理

#### web.xml

```xml
<servlet>
    <!--绑定一个Servlet 处理所有请求-->
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--关联springmvc配置文件 配置处理器、映射器、视图解析器、Controller等-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc-servlet.xml</param-value>
    </init-param>
    <!--启动级别-->
    <load-on-startup>1</load-on-startup>
</servlet>
<!--/ 匹配所有请求，但不包含.jsp	/* 匹配所有请求，包含-->
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

#### springmvc-servlet.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--由DispatcherServlet调用，根据请求返回对应Controller信息
    处理映射器-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
    
    <!--由DispatcherServlet调用，
    	根据Controller信息执行具体的Controller并返回一个ModelandView。
   处理映射器-->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

    <!--由DispatcherServlet调用，
    	根据ModelAndView获取数据、拼接视图名字，并将结果返回，最终由Servlet呈现给用户。
    视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--业务Controller-->
    <bean id="/visit1" class="com.zone.controller.Controller1"/>
</beans>
```

#### Controller1.java

```java
public class Controller1 implements Controller {
    //返回一个模型和视图
    @Nullable
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        ModelAndView mv = new ModelAndView();
        //添加数据
        mv.addObject("msg","controller1");
        //设置视图的名字
        mv.setViewName("c1");
        return mv;
    }
}
```

#### c1.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h4>SpirngMVC原理</h4>
${msg}
</body>
</html>
```

小结：这种方式也可以实现mvc，其中处理映射器、适配器可以不配置（属于默认配置），但是由于一个控制器值对应一个方法，也就是多个页面得重复编写多个控制器类，所以用注解进行开发。

### 3、使用注解开发

#### web.xml 

同上

#### springmvc-servlet.xml	命名任意

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--包扫描 不用注入controllerbean-->
    <context:component-scan base-package="com.zone.controller"/>
    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    
    <!--具体原因：https://blog.csdn.net/j080624/article/details/66969987
    	下面两个注释是默认配置的，可以不写，如果只配置过滤静态资源，会导致404
	-->
    <!--mvc注解驱动 自动注入映射器和适配器，让@RequestMapping注解生效-->
    <mvc:annotation-driven/>
    <!--过滤静态资源-->
    <mvc:default-servlet-handler/>
</beans>
```

#### Controller1

```java
@Controller
public class Controller1 {

    @RequestMapping("/visit2")//实现映射关系，也可以加在类上，设置多级目录；或者直接写多级目录
    public String funcOne(Model model){
        model.addAttribute("msg","注解开发，成功访问");
        return "c2";
    }

    @RequestMapping("/visit21")
    public String funcTwo(Model model){
        model.addAttribute("msg","不同请求，可以指向同一视图，且视图结果不一样，视图可复用，控制器和视图属于弱耦合");
        return "c2";
    }
}
```

#### c2.jsp

同上

### 4、RESTful风格

#### 概念

​	传统：用不同参数(例如方法名)实现不同效果	RESTful风格：利用不同请求方式实现不同效果

#### 相同链接

```java
@Controller
public class RESTfulController {

    //相同链接访问，由于请求方式不同，从而实现不同效果
    @RequestMapping(path = "/admin", method = RequestMethod.POST)
    public String tPost(Model model) {
        model.addAttribute("msg", "POST请求");
        return "test";
    }

    //路径用path或value
    @RequestMapping(value = "/admin", method = RequestMethod.GET)
    public String tGET(Model model) {
        model.addAttribute("msg", "GET请求");
        return "test";
    }

    //简化：使用各种XxxMapping代替RequestMapping，每种请求都有对应的Mapping
}
```

#### 返回字符串

Controller中的方法除了可以返回到指定页面，也可以直接返回字符串或对象，这样就不用编写页面，同时能查看数据是否正确。

@ResponseBody注解：作用单个方法，不使用视图解析器，直接返回字符串

@RestController注解：作用类上，全局作用，同上。

#### 参数数据处理和注解

```java
	//Rest风格简化访问参数/param1/param2，即只需传值；也可给方法参数起别名    
	//使用@PathVariable注解，
	//获取所有参数Map<String,String>
    @RequestMapping(value = "/people/{id}/{pName}", method = RequestMethod.GET)
    public Object zone1(@PathVariable Integer id,
                        @PathVariable("pName") String name,
                        @PathVariable Map<String,String> pv) {
        Map<String,Object> map = new HashMap<>();
        map.put("id",id);
        map.put("name",name);
        map.put("pv",pv);
        return map;
    }

    //普通参数访问 http://localhost:8080/people?age=17&hobby=篮球&hobby=动画
    //@RequestParam("hobby")，可给方法参数起别名，对应访问链接参数
	//获取所有参数Map<String,String> or MultiValueMap<String,String>
    @GetMapping(value = "/people")
    public Object zone2(@RequestParam("age") int age,
                        @RequestParam("hobby")List<String>loves,
                        @RequestParam MultiValueMap<String,String> mvp){//MultiValueMap可存放相同的值，Map不行
        HashMap<String, Object> map = new HashMap<>();
        map.put("age",age);
        map.put("hobby",loves);
        map.put("mvp",mvp);
        return map;
    }

    //对象访问 http://localhost:8080/register?id=7&username=dige&password=123
    //要求访问参数字段名与对象字段名一致，否则不一致的字段为null
    @GetMapping("/register")
    public String register(User user){
        return user;
    }
	
	//*获取请求头的信息、根据Cookie的key获取Cookie信息
    @GetMapping(value = "/info")
    public Object zone3(@RequestHeader("Host") String host,
                        @RequestHeader Map<String,String> rh,
                        @CookieValue("需修改cookie key")Cookie cookie){//也可以是String接收
        HashMap<String, Object> map = new HashMap<>();
        map.put("host",host);
        map.put("Header",rh);
        map.put("cookie",cookie);
        return map;
    }

	//@RequestBody 接收json格式的数据，例如表单post提交，获取请求体信息
    @PostMapping(value = "/form")
    public Object zone4(@RequestBody String content){
        return content;
    }	

	
```

#### 请求转发和重定向

```java
//默认的就是请求转发(forward)、且重定向(redirect)不使用视图解析器
//方法的参数其实包含了很多ServletAPI，例如请求转发的类，可以实现我们的需求，但一般用mvc的简化方式。
@GetMapping("/redirect")
public String reDirect(){
    return "redirect:index.jsp";
}
```

#### 解决乱码

编写一个包含表单的jsp文件，设acton="/encdoe" method="post"

```java
@PostMapping("/encode")
public String enCode(@RequestParam("username") String name, @RequestParam("password") String pwd, Model model){
    //java层面的乱码，从控制台输出就可以看出，需要配置过滤器
    System.out.println("控制台输出：" + name +" and "+ pwd);
    model.addAttribute("name",name);
    model.addAttribute("pwd",pwd);
    return "login";
}
```

web.xml

```xml
<!--配置过滤器，防止乱码	CharacterEncodingFilter-->
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

### 5、JSON使用

##### controller测试类、实体类

```java
//lombok
@Data
@AllArgsConstructor
public class Phone {
    private Integer id;
    private String name;
}
```

分别测试了jackson, fastjson, gson 各种操作均自定义封装在了不同工具类

```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.gson.Gson;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class JsonController {

    private Phone p1 = new Phone("华为", 1899);
    private Phone p2 = new Phone("小米", 1899);
    private Phone p3 = new Phone("一加", 1899);
    
    /**
     * 直接返回对象的toString()--不使用json
     * 作用单个方法，ResponseBody注解：不使用视图解析器，直接返回字符串
     * 全局作用，RestController注解：不使用视图解析器，直接返回字符串
     * @return
     */
    @RequestMapping(value = "/str", produces = "application/json;charset=utf-8")
    @ResponseBody()
    public String Str() {
        return p1.toString();
    }
    
    @RequestMapping("/jks1")
    @ResponseBody()
    public String jackSon1() throws JsonProcessingException {
        //return JackSonUtil.getJsonString(new Date(),"yyyy-MM-dd");
        return JackSonUtil.getJsonString(p3);
    }

    @RequestMapping("/fjs1")
    @ResponseBody()
    public String fastJson1() {
        //return FastJsonUtil.getJsonString(new Date());
        return FastJsonUtil.getJsonString(p1);
    }

    @RequestMapping("/gson1")
    @ResponseBody()
    public String gson1() {
        //return GsonUtil.getJsonString(new Date());
        return GsonUtil.getJsonString(p2);
    }


}
```

##### 乱码

produces可解决乱码, 但局限于每个请求。可通过xml配置全局作用，springmvc-servlet.xml。处理效果比produces好

```xml
<mvc:annotation-driven>
    <mvc:message-converters register-defaults="true">
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <constructor-arg value="UTF-8"/>
        </bean>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper">
                <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                    <property name="failOnEmptyBeans" value="false"/>
                </bean>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

##### jackson工具类

依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.3</version>
</dependency>
```

代码

```java
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

import java.text.SimpleDateFormat;

/**
 * Description:使用JackSon将java对象转换为json字符串的工具类
 * Param:
 * return:
 * Author: 詹迪明
 * Date:2021/12/2
 */
public class JackSonUtil {

    /**
     * 获得默认格式的json字符串
     *
     * @param obj
     * @return
     * @throws JsonProcessingException
     */
    public static String getJsonString(Object obj) throws JsonProcessingException {
        //默认格式
        return getJsonString(obj, "yyyy-MM-dd HH:mm:ss");
    }

    /**
     * 获得指定日期格式的json字符串
     *
     * @param obj
     * @param dateformat
     * @return
     * @throws JsonProcessingException
     */
    public static String getJsonString(Object obj, String dateformat) throws JsonProcessingException {
        ObjectMapper mapper = new ObjectMapper();
        //不使用时间戳的方式--加不加结果一样
        mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
        //定义并指定日期格式
        mapper.setDateFormat(new SimpleDateFormat(dateformat));
        //返回json字符串
        return mapper.writeValueAsString(obj);
    }
}
```

##### fastjson工具类

依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.75</version>
</dependency>
```

代码

```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.serializer.SerializerFeature;

/**
 * Description:使用FastJson将java对象转换为json字符串的工具类
 * Param:
 * return:
 * Author: 詹迪明
 * Date:2021/12/2
 */
public class FastJsonUtil {
    /**
     * 获得默认格式的json字符串
     * @param obj
     * @return
     */
    public static String getJsonString(Object obj) {
        //默认格式---yyyy-MM-dd HH:mm:ss,可已传也可以不传，FastJson里面有默认的
        return getJsonString(obj,null);
    }

    /**
     * 获得指定日期格式的json字符串
     * @param obj
     * @param dateformat
     * @return
     */
    public static String getJsonString(Object obj, String dateformat) {
        return JSON.toJSONStringWithDateFormat(obj,dateformat,SerializerFeature.WriteDateUseDateFormat);
    }
}
```

##### gson工具类

依赖

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.6</version>
</dependency>
```

代码

```java
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;

/**
 * Description:使用Gson将java对象转换为json字符串的工具类
 * Param:
 * return:
 * Author: 詹迪明
 * Date:2021/12/3
 */
public class GsonUtil {

    /**
     * 获得默认格式的json字符串
     * @param obj
     * @return
     */
    public static String getJsonString(Object obj){
        return getJsonString(obj,"yyyy-MM-dd HH:mm:ss");
    }

    /**
     * 获得指定日期格式的json字符串
     * @param obj
     * @param dateformat
     * @return
     */
    public static String getJsonString(Object obj,String dateformat){
        GsonBuilder gsonBuilder = new GsonBuilder();
        Gson gson = gsonBuilder.setDateFormat(dateformat).create();
        return gson.toJson(obj);
    }
}
```









```
老式spring官方文档
https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/spring-framework-reference/html/

springmvc
https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/spring-framework-reference/html/mvc.html

spring各种版本下载
https://repo.spring.io/ui/native/release/org/springframework/spring

```





```xml
<!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>

<!-- https://mvnrepository.com/artifact/javax.servlet.jsp/javax.servlet.jsp-api -->
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>javax.servlet.jsp-api</artifactId>
    <version>2.3.1</version>
    <scope>provided</scope>
</dependency>

<!-- https://mvnrepository.com/artifact/javax.servlet/jstl -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>


```

https://mp.weixin.qq.com/mp/homepage?__biz=Mzg2NTAzMTExNg==&hid=3&sn=456dc4d66f0726730757e319ffdaa23e&scene=18#wechat_redirect



自定义过滤器

```java
package com.kuang.filter;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.util.Map;

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





```java
org.apache.catalina.loader.WebappClassLoaderBase.clearReferencesJdbc The web application [ROOT] registered the JDBC driver [com.mysql.jdbc.Driver] but failed to unregister it when the web application was stopped. To prevent a memory leak, the JDBC Driver has been forcibly unregistered.
04-Dec-2021 11:30:48.225 警告 [RMI TCP Connection(7)-127.0.0.1] org.apache.catalina.loader.WebappClassLoaderBase.clearReferencesThreads The web application [ROOT] appears to have started a thread named [C3P0PooledConnectionPoolManager[identityToken->1hgeqzgal10gyc4x1s63irh|146b7f05]-AdminTaskTimer] but has failed to stop it. This is very likely to create a memory leak. Stack trace of thread:
 java.lang.Object.wait(Native Method)
 java.util.TimerThread.mainLoop(Timer.java:552)
 java.util.TimerThread.run(Timer.java:505)
04-Dec-2021 11:30:48.225 警告 [RMI TCP Connection(7)-127.0.0.1] org.apache.catalina.loader.WebappClassLoaderBase.clearReferencesThreads The web application [ROOT] appears to have started a thread named [C3P0PooledConnectionPoolManager[identityToken->1hgeqzgal10gyc4x1s63irh|146b7f05]-HelperThread-#0] but has failed to stop it. This is very likely to create a memory leak. Stack trace of thread:
 java.lang.Object.wait(Native Method)
 com.mchange.v2.async.ThreadPoolAsynchronousRunner$PoolThread.run(ThreadPoolAsynchronousRunner.java:683)
04-Dec-2021 11:30:48.225 警告 [RMI TCP Connection(7)-127.0.0.1] org.apache.catalina.loader.WebappClassLoaderBase.clearReferencesThreads The web application [ROOT] appears to have started a thread named [C3P0PooledConnectionPoolManager[identityToken->1hgeqzgal10gyc4x1s63irh|146b7f05]-HelperThread-#1] but has failed to stop it. This is very likely to create a memory leak. Stack trace of thread:
 java.lang.Object.wait(Native Method)
 com.mchange.v2.async.ThreadPoolAsynchronousRunner$PoolThread.run(ThreadPoolAsynchronousRunner.java:683)
04-Dec-2021 11:30:48.225 警告 [RMI TCP Connection(7)-127.0.0.1] org.apache.catalina.loader.WebappClassLoaderBase.clearReferencesThreads The web application [ROOT] appears to have started a thread named [C3P0PooledConnectionPoolManager[identityToken->1hgeqzgal10gyc4x1s63irh|146b7f05]-HelperThread-#2] but has failed to stop it. This is very likely to create a memory leak. Stack trace of thread:
 java.lang.Object.wait(Native Method)
 com.mchange.v2.async.ThreadPoolAsynchronousRunner$PoolThread.run(ThreadPoolAsynchronousRunner.java:683)
04-Dec-2021 11:30:48.225 警告 [RMI TCP Connection(7)-127.0.0.1] org.apache.catalina.loader.WebappClassLoaderBase.clearReferencesThreads The web application [ROOT] appears to have started a thread named [Abandoned connection cleanup thread] but has failed to stop it. This is very likely to create a memory leak. Stack trace of thread:
 java.lang.Object.wait(Native Method)
 java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
 com.mysql.jdbc.AbandonedConnectionCleanupThread.run(AbandonedConnectionCleanupThread.java:64)
 java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
 java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
 java.lang.Thread.run(Thread.java:748)
04-Dec-2021 11:30:48.756 信息 [Abandoned connection cleanup thread] org.apache.catalina.loader.WebappClassLoaderBase.checkStateForResourceLoading Illegal access: this web application instance has been stopped already. Could not load []. The following stack trace is thrown for debugging purposes as well as to attempt to terminate the thread which caused the illegal access.
 java.lang.IllegalStateException: Illegal access: this web application instance has been stopped already. Could not load []. The following stack trace is thrown for debugging purposes as well as to attempt to terminate the thread which caused the illegal access.
	at org.apache.catalina.loader.WebappClassLoaderBase.checkStateForResourceLoading(WebappClassLoaderBase.java:1355)
	at org.apache.catalina.loader.WebappClassLoaderBase.getResource(WebappClassLoaderBase.java:1025)
	at com.mysql.jdbc.AbandonedConnectionCleanupThread.checkContextClassLoaders(AbandonedConnectionCleanupThread.java:90)
	at com.mysql.jdbc.AbandonedConnectionCleanupThread.run(AbandonedConnectionCleanupThread.java:63)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
```



任务：利用aop实现一个拦截器



文件上传和下载---狂神博客 multipart/form-data









```
solve:启动类加上@MapperScan("com.example.dao")


Description:

Field userMapper in com.example.service.impl.UserServiceImpl required a bean of type 'com.example.dao.UserMapper' that could not be found.

The injection point has the following annotations:
	- @org.springframework.beans.factory.annotation.Autowired(required=true)


Action:

Consider defining a bean of type 'com.example.dao.UserMapper' in your configuration.
```





文件上传和下载

1. 文件应放在外界无法直接访问的目录，以保证服务器安全
2. 文件名字唯一，防止覆盖
3. 限制文件上传最大值，减少服务器压力
4. 限制文件上传类型
5. 
