# Spring

## 1、介绍

- 各个版本下载地址：

  [http://repo.spring.io/release/org/springframework/spring](https://repo.spring.io/release/org/springframework/spring) 

- 学习文档(5.2.0)

  [Core Technologies (spring.io)](https://docs.spring.io/spring-framework/docs/5.2.0.RELEASE/spring-framework-reference/core.html#spring-core)

- 通过maven导入依赖

  ```xml
  <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.0.RELEASE</version>
  </dependency>
  ```

  

## 2、优点

- 开源的、免费的
- 轻量级的、非入侵式的
- 控制反转ioc，面向切面aop
- 支持事务的处理、支持框架整合

## 3、组成

### IOC

#### 	#介绍

​		**控制反转**：

​			控制：框架负责对象的创建，而不由程序本身创建；
​			反转：程序被动接收对象；	

​		简单点说，就是用于管理项目中的类

#### 	#IOC使用

> D:\IdeaProjects\Spring-Learn\spring1\src\test\java\TestS1.java

#### 	**#对象的创建**

> D:\IdeaProjects\Spring-Learn\spring1\src\test\java\TestS2.java

​		**无参**：默认访问

​		**有参**： constructor-arg 配置

```xml
<!--方式2 通过参数名 推荐使用-->
<bean id="user2" class="com.zone.pojo.User">
   <constructor-arg name="name" value="詹迪明"/>
   <constructor-arg name="age" value="21"/>
   <constructor-arg name="girlfriend" ref="user1"/>
</bean>
```

​		**特点**：对象的初始化在加载配置文件时，而不是获取对象时。
​					相同id多次获取的对象是同一个对象(默认是单例模式)

#### 	#**配置文件**

​		**bean配置**：
​			id--bean的位移标识符，等价对象名
​			class--bean对象的全限类名
​			name--id别名，可以取多个，用逗号、空格或分号分隔。
​			scope --作用域，singleton单例，prototype原型

​		**import配置**：
​			团队开发，将多个配置文件导入和合并为一个
​			`<import resource="文件名.xml"/>`

​		***alias配置：**

​			`<alias name="idName" alias="aliasName"/>`

#### #依赖注入

> D:\IdeaProjects\Spring-Learn\spring2\src\test\java\TestS1.java

- 介绍：利用set方法进行注入，是实现ioc的一种方法。

- 分类：

  - 构造器参数注入：参见对象的创建 constructor-arg		

  - Set方式属性注入： 		

    ```xml
     <!--空值-->
    <property name="name">
    	<null></null>
    </property>
    <property name="phones">
        <array>
            <value>按键手机</value>
        </array>
    </property>
    <!--集合-->
    <property name="hobbies">
        <list>
            <value>剪视频</value>
        </list>
    </property>
    <property name="friends">
        <set>
            <value>zzx</value>
        </set>
    </property>
    <property name="cards">
        <map>
            <entry key="schoolCard" value="10086"/>
        </map>
    </property>
    <!--特殊类-->
    <property name="info">
        <props>
            <prop key="nation">China</prop>
        </props>
    </property>
    ```

  - *其它注入：p命名空间和c命名空间

    ```xml
    <!--p命名空间注入使用  简化property，可直接注入属性的值
        使用时需在beans中加上xmlns:p="http://www.springframework.org/schema/p"
    -->
    <bean id="person1" class="com.zone.pojo.Person" p:age="21" p:name="詹迪明"></bean>
    
    <!--c命名空间注入使用  简化constructor-arg ,可直接注入到构造器参数
        使用时需在beans中加上xmlns:p="http://www.springframework.org/schema/p"
    -->
    <bean id="person2" class="com.zone.pojo.Person" c:age="18" c:name="zone"></bean>	
    ```


#### #自动装配

##### ##autowire配置 

> D:\IdeaProjects\Spring-Learn\spring2\src\test\java\TestS2.java

- byName   byType

  ```xml
  <bean id="dog" class="com.zone.pojo.Dog">
      <property name="name" value="dabao"/>
  </bean>
  
  <!--byName 查找字段名对应的beanid进行自动装配，找不到beanid则为null-->
  <bean id="person3" class="com.zone.pojo.Person" autowire="byName">
      <property name="name" value="詹山"/>
      <property name="age" value="18"/>
  </bean>
  
  <!--byType 根据字段对应类型进行自动装配  找不到beanclass则为null，
  要求容器中beanclass唯一，否则报异常，因为重复导致不可区分
  其中可省略beanid-->
  <bean id="person3" class="com.zone.pojo.Person" autowire="byType">
      <property name="name" value="詹山"/>
      <property name="age" value="18"/>
  </bean>
  ```

##### ##注解配置

> D:\IdeaProjects\Spring-Learn\spring2\src\test\java\TestS2.java

1. 导入约束和配置支持注解的标签

   ```xml
   <!--约束-->
   xmlns:context="http://www.springframework.org/schema/context"
   http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
   
   <!--支持-->
   <context:annotation-config/>
   ```

2. 使用

   - @Autowired

     介绍：自动装配

     ```java
     public @interface Autowired {
         //决定装配字段是否可为空，也就是ioc容器中是否可以不存在改类
         //false可为空
         boolean required() default true;
     }
     ```

     位置：属性、属性对应的set方法等

     好处：底层是反射，可省略set方法

     原理：优先使用byType方式装配，若beanclass相同，则使用byName，若找不到beanid,则报错

     ​			看似是byType和byName的结合，其实还是有些差别。

   - @Qualifier

     介绍：指定字段对应的beanid

     ```java
     public @interface Qualifier {
     	//通过value定义    
         String value() default "";
     }
     ```

     位置：字段等

     作用：辅助@Autowired

   - @Nullable

     介绍：定义字段是否为空--类似required

   - @Resource

     介绍：java注解自带的

     位置：字段等

     原理：优先使用byName，若beanid找不到，则使用ByType，若beanclass相同，则报错

#### #注解开发

**前提：**导入约束和配置支持注解的标签

```xml
<!--约束-->
xmlns:context="http://www.springframework.org/schema/context"
http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd

<!--全局扫描-->
<context:annotation-config/>
<!--包扫描 注解支持-->
<context:component-scan base-package="com.zone.pojo"/>
```

1. bean注册、属性注入、作用域（Component、Scope、Value）

   ```java
   //注册类
   @Component
   //作用域
   @Scope("singleton") //@Scope("prototype")
   public class User {
       //属性注入
       @Value("ZHANDIMING")
       private String name;
   }
   ```

2. 衍生注解

   @Component：代表将某个类注册到Spring中

   - dao：@Repository()
   - service：@Service()
   - controller：@Controller()

   四个注解功能一样，只是因为分层，从而定义不同名字

3. 自动装配

   详见：##注解配置

4. 小结

   xml和注解：

   - xml比较万能，适用任何场景，维护简单
   - 注解只能在自己的类中使用，维护复杂。

   实践：xml管理bean，注解负责属性注入。

   注意：使用注解时需要开启注解的支持，让注解生效

#### #纯Java配置Spring

核心：@Configuration

1. 配置文件.java：

   ```java
   //代表一个配置类，类似xml文件，本身也注册到了ioc容器中
   @Configuration
   //扫描包
   @ComponentScan("com.zone.pojo")
   public class ZoneConfig {
   	//注册一个bean，方法名相当于bean标签中的id属性，即beanid
       @Bean
       public User user(){
           return new User();
       }
   }
   ```

2. 测试类

   使用配置类的方式，需要通过AnnotationConfigApplicationContext来获取容器

   ```java
   public class TestS2 {
       @Test
       public void zone1(){
           ApplicationContext context = new  AnnotationConfigApplicationContext(ZoneConfig.class);
        	//beanid是方法名   
           System.out.println(context.getBean("user", User.class));
       }
   }
   ```

3. 实体类

   参看：注解开发  @Component......

   

### AOP

#### #介绍

**oop:**
	面向对象，一个功能模块对应一个类，最后通过类之间的调用实现业务需求。
	虽然逻辑清晰，但公共代码冗余，不可复用，不好维护（因为类之间相互调用紧密）

**aop:**
	把项目中具有公共功能的部分抽取出来，例如日志、事务、权限等。
	通过面向切面的方式来编程。底层：代理设计模式



#### #AOP使用

> D:\IdeaProjects\Spring-Learn\spring4\src\main\java\aop

前提：导入aop约束

```xml
xmlns:aop="http://www.springframework.org/schema/aop"
```

1. 法一：原生Spring API接口

   ```xml
   <!--法一 原生spirng API接口-->
   <!--作为切面的类需要实现通知类   advice-ref 引用的通知类需要实现Advice之类的接口-->
   <aop:config>
       <!--切入点
        切入点配置 expression
        切入点表达式：execution(* sp.PayDaoImpl.*(..))
        第一个*代表public class
        sp.PayDaoImpl.* 表示该类中的所有方法（不管是公有的，还是私有的）都是切入点
   	 (..)表示带任意参数的方法和不带参数的方法
       -->
       <aop:pointcut id="pointcut" expression="execution(* sp.PayDaoImpl.*(..))"/>
       <!--切面类-->
       <aop:advisor advice-ref="sc" pointcut-ref="pointcut"/>
       <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
       <aop:advisor advice-ref="se" pointcut-ref="pointcut"/>
   </aop:config>
   ```

   ```java
   public class Logging implements AfterReturningAdvice {......
   ```

2. 法二：自定义切面类

   ```xml
   <!--方法二 自定义切面类-->
   <aop:config>
       <aop:pointcut id="pointcut" expression="execution(* sp.PayDaoImpl.*(..))"/>
       <!--
       切面配置  order表示执行的顺序 ref表示引用切面类
       -->
       <aop:aspect ref="sc" order="4">
           <!--配置通知
           pointcut-ref 切入点id
           method 通知方法名
           -->
           <aop:before method="safeCheck" pointcut-ref="pointcut"/>
       </aop:aspect>
       <aop:aspect ref="log" order="7">
           <aop:after method="log" pointcut-ref="pointcut"/>
       </aop:aspect>
       <aop:aspect ref="se" order="6">
           <aop:after method="safeExit" pointcut-ref="pointcut"/>
       </aop:aspect>
   </aop:config>
   ```

3. 法三：注解开发

```xml
<!--xml开启注解支持  
属性proxy-target-class false默认使用jdk实现，true使用cglib实现 -->
<aop:aspectj-autoproxy />
```

```java
@Aspect //定义为切面类
public class AroundTest {

    @Before("execution(* sp.PayDaoImpl.*(..))")//定义前置通知
    public void before(){
        System.out.println("方法执行前");
    }
    
    @Around("execution(* sp.PayDaoImpl.*(..))")//定义环绕通知
    public void around(ProceedingJoinPoint jpp) throws Throwable {
        System.out.println("环绕前，比before先执行");
        Object proceed = jpp.proceed();//执行方法被代理
        System.out.println("环绕后，比after先执行");
    }

    @After("execution(* sp.PayDaoImpl.*(..))")//定义后置通知
    public void after(){
        System.out.println("方法执行后");
    }
}
```



## 4、整合mybatis

[官方文档](http://mybatis.org/spring/zh/index.html)

### 介绍

将mybatis的SqSession和mapper的创建注入到ioc容器中，由spring来创建。mybatis能参与到spring的事务管理中。

### 配置

- pom.xml

  必要的依赖：

  ```tex
  spring-webmvc  aspectjweaver(aop)
  mysql-connector-java(驱动)  spring-jdbc(spirng操作数据库) 
  mybatis   mybatis-spring(整合)  
  ```

  mybatis-spring:

  ```xml
  <dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.6</version>
  </dependency>
  ```

- spring-dao.xml

  ```tex
  配置spring 和 mybatis那些无需修改的内容，例如：
  ```

  mybatis 的DataSource、SqlSessionFactory、sqlSession

  ```xml
  <!--注入数据源 代替mybaits的DataSource-->
  <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
      <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
      <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
      <property name="username" value="root"/>
      <property name="password" value="root"/>
  </bean>
  
  <!--注入SqlSessionFactory-->
  <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
      <property name="dataSource" ref="dataSource"/>
      <!--引入mybatis的配置文件，也可以不保留-->
      <property name="configLocation" value="classpath:mybatis-config.xml"/>
      <!--mapper注册-->
      <property name="mapperLocations" value="classpath:com/zone/mapper/*.xml"/>
  </bean>
  
  <!--注入SqlSessionTemplate 等价于SqlSession-->
  <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
      <!--构造器方式注入，没有set函数-->
      <constructor-arg index="0" ref="sqlSessionFactory"/>
  </bean>
  ```

  spring AOP 例如事务通知

- mybatis-config.xml

  ```tex
   mybatis配置文件，可保留别名等一些设置的代码
  ```

- beans.xml

  ```
  需要配置注册的bean，编写在该文件，这里三个xml，实际通过import合并为一个
  ```

### 使用

> D:\IdeaProjects\Spring-Learn\spring5\src\test\java\TestSM1.java

XxxMapper接口和XxxMapper.xml 没有变化，但需要添加实现类XxxMapperIml

1. 方式一：声明实例变量

   ```java
   public class UserMapperImpl implements UserMapper {
   	//在ioc容器中，通过set方法，将属性sqlSessionTemplate注入到UserMapperImpl中
       private SqlSessionTemplate sqlSessionTemplate;//==SqlSession
       
       public void setSqlSessionTemplate(SqlSessionTemplate sqlSessionTemplate) {
           this.sqlSessionTemplate = sqlSessionTemplate;
       }
       //使用
       public List<User> queryUsers() {
           return sqlSessionTemplate.getMapper(UserMapper.class).queryUsers();
       }
   .....
   ```

   beans.xml

   ```xml
   <bean id="userMapperImpl" class="com.zone.mapper.UserMapperImpl">
    	<!--注入-->   
       <property name="sqlSessionTemplate" ref="sqlSession"/>
   </bean>
   ```

   

2. 方式二：通过继承SqlSessionDaoSupport

   ```java
   public class UserMapperImpl1 extends SqlSessionDaoSupport implements UserMapper {
   
       /*在ioc容器中，通过set方法，将属性sqlSessionFactory/SqlSession注入到UserMapperImpl1中
       然后由内部提供的getSqlSession()/getSqlSessionTemplate()方法，获得SqlSessionTemplate
       */
       public List<User> queryUsers() {
           return getSqlSession().getMapper(UserMapper.class).queryUsers();
       }
   ......
   ```

   beans.xml

   ```
   <bean id="userMapperImpl1" class="com.zone.mapper.UserMapperImpl1">
   	两个都可
       <!--<property name="sqlSessionTemplate" ref="sqlSession"/>-->
       <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
   </bean>
   ```


### 事务

> D:\IdeaProjects\Spring-Learn\spring5\src\test\java\TestSM2.java

配置事务通知，spring Aop

```xml
<!--配置声明式事务-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<!--配置AOP实现事务的织入-->
<!--配置事务通知-->
<tx:advice id="interceptor" transaction-manager="transactionManager">
    <!--可以省略  因为切入点已经指明-->
    <tx:attributes>
        <!--指定方法配置事务 propagation事务的传播特性
        name="*" 指定所有方法
        name="方法名" 指定方法
        name="串*" 指定包含该串的方法
        -->
        <tx:method name="abc" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>

<!--配置切面-->
<aop:config>
    <!--切入点 和 通知织入切入点-->
    <aop:pointcut id="pointCut" expression="execution(* com.zone.mapper..*.*(..))"/>
    <aop:advisor advice-ref="interceptor" pointcut-ref="pointCut"/>
</aop:config>
<!--aop时使用cglib实现-->
<aop:aspectj-autoproxy proxy-target-class="true"/>
```



## 5、问题：

context:annotation-config？

java.lang.IllegalStateException: org.springframework.context.annotation.AnnotationConfigApplicationContext@6a38e57f has not been refreshed yet



？底层源码任务：ClassPathXmlApplicationContext





aop cjlib and jdk  cglib



```
* com.zone.mapper.*.*(..)--而这里不能* com.zone.mapper.*(..)
* com.zone.mapper..*(..) --这个为什么不用加.* 也就是* com.zone.mapper..*.*(..)
```
