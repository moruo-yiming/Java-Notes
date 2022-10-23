[toc]

[官网文档](https://mybatis.net.cn/)

### 1、mybatis5步骤实现

mybatis-config.xml（核心配置文件，名字随意）；实体类；mapper（访问数据库的接口）；mapper.xml（sql代码）,  需要注册到核心配置文件中； 编写工具类获取SqlSession；

### 2、生命周期和作用域

- SqlSessionFactoryBuilder

  一旦创建SqlSessionFactory，就不需要了，故声明为局部变量

- SqlSessionFactory

  理解为连接池，运行期间要一直存在，没必要关闭。

- SqlSession

  每个线程都应该有自己的SqlSession实例，该实例不是线程安全的，作用域应为请求或方法作用域，用完需立即关闭，否则会被占用。

- 映射器：每个Mapper代表一个业务

工具类：

```java
package com.zone.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class MyBatisUtils {
    // sqlSessionFactory会在运行期间一直存在，除非你关闭--这里采用了单例模式
    private static SqlSessionFactory sqlSessionFactory = null;//提升作用域--下面方法才可调用
    static {
        try {
            //通过SqlSessionFactoryBuilder加载配置文件获取SqlSessionFactory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    //SqlSession 用于执行sql语句，线程不安全，每次请求完毕需关闭
    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();//返回sqlSession实例
    }

}
```

注：执行增删改操作时，需要使用sqlSession.commit()提交事务，因为mybatis默认是不自动提交事物的，这样数据才会写入数据库。或者使用带参的openSession(true)，这样就全局开启自动提交事务，无需调用commit()方法。

### 3、配置文件

> [代码演示]() mybatis2

核心配置文件--mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <!--&amp; 代表 &-->
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

解析：

- environments

  环境变量，可以配置多种环境，但只能选择一种(根据default属性)

  ```xml
  <environments default="development">
  	每个环境根据id来识别
      <environment id="development">
          事务管理器有两种类型
          <transactionManager type="JDBC"/>
          数据源（连接池） 有池的连接 POOLED
          <dataSource type="POOLED">
           	......
          </dataSource>
      </environment>
  </environments>
  ```

- properties

  属性，用于在配置文件中引入外部配置文件。如果两个文件中有相同字段，优先使用外部配置文件。

  例如，抽取数据库连接配置为db.properties

  ```properties
  driver=com.mysql.jdbc.Driver
  url=jdbc:mysql://localhost:3306/mydbs?useSSL=true&useUnicode=true&characterEncoding=UTF-8
  username=root
  password=root
  ```

  引入，通过${}获取其它配置文件的值，这样可以动态配置主配置文件

  ```xml
  <properties resource="db.properties">
  	<!-- 额外属性-->
  </properties>
  
  <property name="password" value="${password}"/>
  ```

- typeAliases

  类型别名，减少类完全限定名的冗余，给实体类写别名  这样在UserMapper.xml里面就可以使用别名

  - 单个类，用短的名字代替完全限定名

  ```xml
  <typeAliases>
      <typeAlias type="com.zone.pojo.User" alias="uname"/>
  </typeAliases>
  ```

  - 多个类，指定包名即可，框架会自动去搜索该位置下的类，并以小写类名作为别名

  ```xml
  <typeAliases>
      <package name="com.zone.pojo"/>
  </typeAliases>
  ```

  注：对于指定包名，类的别名若要自定义，可通过在实体类添加别名注解@Alias("")

- settings

  设置，会改变框架运行时的行为，例如日志，缓存，命名规则

  ```xml
  <settings>
      <!--开启驼峰命名法 将带下划线的数据库列名映射到java的驼峰命名-->
      <setting name="mapUnderscoreToCamelCase" value="true"/>
  </settings>
  ```
  
- mappers

  每一个mapper.xml都需要在核心配置文件中注册

  映射器，注册绑定Mapper文件

  - 使用资源路径

    ```xml
    <mappers>
        <mapper resource="com/zone/dao/UserMapper.xml"/>
    </mappers>
    ```

  - 使用class文件

    ```xml
    <mappers>
    	<mapper class="com.zone.dao.UserMapper"/>
    </mappers>
    ```

  - 扫描包

    ```xml
    <mappers>
    	<package name="com.zone.dao"/>
    </mappers>
    ```
    
    > 注意：后两种方式，Mapper和Mapper.xml要求同名且位于同一包下

注意：标签引入是有顺序的，否则编译报错

![](https://img-blog.csdnimg.cn/825df4fc2e87403cb3bf4957ce751517.png)


### 4、属性名和字段名不一致问题 

> [代码演示]() mybatis3

```
数据库：id name pwd
实体类：id name password
```

导致查询出来时password为null

解决：

1. 查询语句使用别名 as password

2. 使用ResultMap标签--将实体类和数据库的字段对应起来

### 5、Lombok插件--偷懒

1. 实体类偷懒工具，只需定义好实体类字段，添加注解自动实现set，get，toString，构造器等方法。

2. idea安装插件，导入jar包

3. 通过编写注解来实现

   @Data：无参、get、set、toString、hashcode、equals
   @AllArgsConstructor：有参
   @NoArgsConstructor：无参
   @Getter
   @Setter
   @ToString
   @EqualsAndHashCode

### 6、使用注解

> [代码演示]() mybatis4

1. 在接口的方法上写注解
2. 在核心配置文件绑定接口

crud

1. @Param("")注解

   ```java
   @Select("select * from mybatis.user where id= #{id} and name = #{n}")
   User getUserByID(@Param("id") int id, @Param("n") String name);
   ```

   接口方法参数若为基本类型、String类型，需使用@Param()设定属性名，sql才能获取得到，引用集合则不用。
   


### 7、查询

#### 简单查询

```xml
<resultMap id="resultmap" type="User">
    column字段名  property实体类属性名
    <result column="pwd" property="password"/>
</resultMap>
resultMap映射到上面的标签id 
<select id="getUserById" parameterType="int" resultMap="resultmap">
    select * from mybatis.user where id = #{id};
</select>
```

#### 复杂查询

> [代码演示 ]()mybatis5

​	复杂的属性需要使用对象association和集合collection处理

- association

  按照查询嵌套处理--类似子查询
  ```xml
  <select id="getStudent" resultMap="StudentTeacher">
      select id,name,tid from student;
  </select>
  
  <resultMap id="StudentTeacher" type="Student">
      <!-- 下面两句可以不写，因为字段对应的上 -->
      <result property="id" column="id"/>
      <result property="name" column="name"/>
      <!--通过select 调用子查询  这里的tid就需要与数据库对应上-->
      <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
  </resultMap>
  
  <select id="getTeacher" resultType="Teacher">
      <!-- 虽然这里的#{tid}的名称可以随便定义，但是规范最好 -->
      SELECT  * from teacher where id = #{tid}
  </select>
  ```
  
  按照结果嵌套处理--类似联表查询
  ```xml
  <select id="getStudent" resultMap="st">
      select s.id,s.name as sname,t.name as tname from student as s, teacher as t where s.tid = t.id;
  </select>
  
  <resultMap id="st" type="Student">
      <result property="id" column="id"/>
      <result property="name" column="sname"/>
      <association property="teacher" javaType="Teacher">
          <result property="name" column="tname"/>
      </association>
  </resultMap>
  ```

- collection

  按照查询嵌套处理
  ```xml
  <select id="showTeacher" resultMap="ts">
        select * from teacher where id = #{tid};
  </select>
  
  <resultMap id="ts" type="Teacher">
      <!--根据tid 查询学生集合List<Student>-->
      <!--如果不加，查询的结果id为空-->
      <result property="id" column="id"/>
      <collection property="students" column="id" ofType="Student" select="getStudent"/>
  </resultMap>
  
  <select id="getStudent" resultType="Student">
      select * from student where tid = #{id};
  </select>
  ```
  
  按照结果嵌套处理
  
  ```xml
  <select id="showTeacher" resultMap="ts">
      select t.id tid, t.name tname, s.id sid, s.name sname from teacher t, student s where
      t.id = #{tid} and t.id = s.tid;
  </select>
  
  <resultMap id="ts" type="Teacher">
      <result property="id" column="tid"/>
      <result property="name" column="tname"/>
      <!--集合 泛型类型需要使用ofType-->
      <collection property="students" ofType="Student">
          <result property="id" column="sid"/>
          <result property="name" column="sname"/>
      </collection>
  </resultMap>
  ```

#### 延迟加载--懒加载

原理：懒加载原理是调用的时候才触发，而不是初始化时就加载，例如a.getStudent()用于获取学生及对应导师的信息，但如果使用懒加载，调用a.getStudent的时候只返回学生的信息，当调用a.getSutdent().getTeacher()才返回学生和教师的信息。

要求：仅适用于有子查询的联合查询

全局配置：

```xml
<!--<setting name="aggressiveLazyLoading" value="false"/> 某个版本之后默认为false，所以无需配置-->
<setting name="lazyLoadingEnabled" value="true"/>
<setting name="lazyLoadTriggerMethods" value=""/>
<!--value 默认为equals,clone,hashCode,toString这些方法 
也就是调用这些方法就会触发懒加载操作，容易误以为认为懒加载无效-->
```

局部配置：只需指定某条查询使用懒加载，在association或collection中添加`fetchType="lazy"`属性

:zap:注意：除了配置的lazyLoadTriggerMethods为空，如果有输出操作，注意实体类是否使用了lombok，因为重写的toString会输出所有字段，包括关联对象(触发了关联对象的toString())，此时也会调用懒加载，导致失效。所以要么手动重写toString不输出关联对象，要么整个过程都不输出，看调用的sql语句也能知道懒加载是否生效。

> [推荐阅读](https://cloud.tencent.com/developer/article/1587223)
>
> [更多配置信息](https://mybatis.org/mybatis-3/zh/configuration.html)



### 8.日志

> [代码演示]()  mybatis6

以前：debug sout	现在：日志工厂

重点学习：

- STDOUT_LOGGING (最简单)，只需要如下配置

  ```xml
  #核心配置文件
  <settings>
         <setting name="logImpl" value="STDOUT_LOGGING"/>
  </settings>
  ```

- LOG4J

  Apache的开源；输出位置、日志输出格式可控；可定义级别；通过配置文件配置，无需修改代码。

  1. 需要导包
  
  2. log4j.properties
  
  3. 核心配置
  
     ```xml
     <settings>
            <setting name="logImpl" value="LOG4J"/>
     </settings>
     ```
  
     

### 9.动态SQL

> [代码演示]() mybatis6

概念：根据不同的条件生成不同的SQL语句

类似JSTL，在SQL层面添加逻辑代码

- if

  ```xml
  <select id="queryBlogIF" parameterType="map" resultType="Blog">
      select * from blog where 1=1
      <if test="title != null">
          and title = #{title}
      </if>
      <if test="author != null">
          and author = #{author}
      </if>
  </select>
  ```

- where

  where后面若直接遇到and，则去除and

  若where里面无满足条件，则不添加where

  ```xml
  <select id="queryBlogWhere" parameterType="map" resultType="Blog">
      select * from blog
      <where>
          <if test="title != null">
              title = #{title}
          </if>
          <if test="author != null">
              and author = #{author}
          </if>
      </where>
  </select>
  ```

- set

  ```xml
  <update id="updateBlog" parameterType="map">
      update blog
      <set>
          <!--如果只传入title 则逗号会被自动去除-->
          <if test="title != null">
              title = #{title},
          </if>
          <if test="author != null">
              author = #{author}
          </if>
      </set>
      where id = #{id}
  </update>
  ```

- choose-when-otherwise

  ```xml
  <select id="queryBlogChoose" parameterType="map" resultType="Blog">
      select * from blog
      <where>
          <choose>
              <!-- 只会有一个选项成立 -->
              <when test="title != null">
                  title = #{title}
              </when>
              <when test="author != null">
                  and author = #{author}
              </when>
              <otherwise>
                  and views = #{views}
              </otherwise>
          </choose>
      </where>
  </select>
  ```

- sql片段

  sql + include

  抽取公用部分，方便复用：抽取部分最好基于单表；不要存在where标签；

  ```xml
  <!--sql抽取-->
  <sql id="if-title-author">
      <if test="title != null">
          and title = #{title}
      </if>
      <if test="author != null">
          and author = #{author}
      </if>
  </sql>
  ```

  ```xml
  <select id="queryBlogWhere" parameterType="map" resultType="Blog">
      select * from blog
      <where>
          <!--在需要的地方引入-->
          <include refid="if-title-author"/>
      </where>
  </select>
  ```

- Foreach

  对集合的遍历---使用in的时候

  ```xml
  <!--查询指定id集合的博客-->
  <select id="queryBlogForeach" parameterType="map" resultType="Blog">
      select * from blog
      方式1
      <where>
         <!--需要传入一个包含id的集合 --> 
          <foreach collection="ids" item="id" separator="or">
              id = #{id}
          </foreach>
      </where>
      方式2
      <where>
          id IN
          <!--separator open close 可自定义 -->
          <foreach collection="ids" item="id" open="(" separator="," close=")">
              #{id}
          </foreach>
      </where>
  </select>
  ```

### 10.缓存

> 代码演示 mybatis7

- 介绍：存在内存的东西，便于下次查询获取。提高效率，解决高并发问题。
- 适合：经常查询且很少改变的数据
- mybatis缓存：默认定义了两级缓存，可以方便地定制和配置缓存
- 缓存原理：先二级后一级最后数据库

#### 一级缓存（本地缓存）

介绍：默认只开启一级缓存(SqlSession级别----从开启到关闭，会话期间有效)

缓存失效：查询不同东西；增删改操作（刷新缓存）；手动清除缓存---sqlSession.clearCache()；查询不同Mapper.xml

#### 二级缓存

1. 介绍：基于命名空间级别的缓存，不同会话可以通过二级缓存获取内容。

​			同一mapper有效，一级缓存先作用，一级失效，才使用二级

2. 使用：

   * 显示开启全局缓存（默认开启）
     ```.xml
     <setting name="cacheEnabled" value="true"/>
     ```

   * mapper.xml中加入`<cache/>`
     ```
     <cache/>
     语句的作用：select被缓存、增删改时刷新缓存
     参数可选：readOnly="true" eviction="FIFO" flushInterval="60000" size="520"
     	eviction清除策略：默认的清除策略是LRU
             LRU – 最近最少使用：移除最长时间不被使用的对象。
             FIFO – 先进先出：按对象进入缓存的顺序来移除它们。
             SOFT – 软引用：基于垃圾回收器状态和软引用规则移除对象。
             WEAK – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。
         flushInterval="60000"：开启定时刷新（缓存每隔60秒刷新一次）
         
     <cache/>作用详解
     映射语句文件中的所有 select 语句的结果将会被缓存。
     映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
     缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
     缓存不会定时进行刷新（也就是说，没有刷新间隔）。
     缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
     缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。
     ```

3. 注意：实体类序列化，只读问题？
   readOnly（只读）属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓存对象的相同实例。 因此这些对象不能被修改。这就提供了可观的性能提升。而可读写的缓存会（通过序列化）返回缓存对象的拷贝（深拷贝），速度上会慢一些，但是更安全，因此默认值是 false，其中false的话，实体类需要可序列化

4. 缓存失效：一旦中间有增删改，则失效，清除缓存没用(只清除一级)；

#### 自定义缓存ehcache

使用第三方ehcache，开源JAVA分布式缓存，主要面向通用缓存

1.导包

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis.caches/mybatis-ehcache -->
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-ehcache</artifactId>
    <version>1.1.0</version>
</dependency>
```

2.新建配置文件ehcache.xml

:zap: 项目中一般用Redis数据库做缓存

### 11、mybatis原理

#### Mybatis一次查询操作的执行流程

1.通过SqlSessionFactoryBuilder读取配置文件，加载配置信息，如数据源，并返回SqlSessionFactory对象
2.SqlSessionFactory通过openSession()获取SqlSession，SqlSession包含Executer
3.SqlSession通过getMapper()获取Mapper接口的代理对象，即MapperProxy(jdk动态代理)，MapperProxy包含SqlSession
4.MapperProxy最终会调用SqlSession的增删改查方法（方法内部具体通过使用Executer）
5.`Executor`，负责创建`StatementHandler`对象（内部也创建了`ParameterHandler`和`ResultSetHandler`）和调用StatementHandler处理操作数据库
6.StatementHandler处理sql参数，调用jdbc执行sql，并返回结果，其中使用ParameterHandler处理参数，使用ResultSetHandler处理结果集

四大对象：`ParameterHandler` 、 `ResultSetHandler` 、 `StatementHandler` 、 `Executor` 

理解：四大对象创建之后，都需要通过拦截器interceptorChain.pluginAll(parameterHandler); 这也是自定义插件实现的底层原理

```java
public Object pluginAll(Object target) {
    for (Interceptor interceptor : interceptors) {
        target = interceptor.plugin(target);
    }
    //当然，如果没有配置拦截器的话，也是直接返回
    return target;
}
```

#### 执行器Executer

* SimpleExecuter：每执行一次 update 或 select，就开启一个 Statement 对象，用完立刻关闭 Statement 对象。
* ReuseExecutor：执行 update 或 select，以 sql 作为 key 查找 Statement 对象，存在就使用，不存在就创建，用完后，不关闭 Statement 对象，而是放置于 Map<String, Statement>内，供下一次使用。简言之，就是重复使用 Statement 对象。
* BatchExecutor：执行 update（没有 select，JDBC 批处理不支持 select），将所有 sql 都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个 Statement 对象，每个 Statement 对象都是 addBatch()完毕后，等待逐一执行 executeBatch()批处理。与 JDBC 批处理相同。

### 12、分页

> [代码演示]() mybatis3

#### **分页方式**

- limit
- RowBounds
- [PageHelper](https://pagehelper.github.io/docs/howtouse/)

逻辑分页：使用RowBounds进行分页，它从数据库是一次性查询很多数据，然后再对数据进行分页。

物理分页：手写sql或者分页插件，直接从数据库查询指定条数的数据。

:zap:RowBounds是一次性查询很多数据，但不是全部，首先mybatis是对jdbc的封装，而jdbc中规定了每次从数据库查询的最大条数，也就是内部也是分多次查询的。逻辑分页是再内存层面上的操作，存在消耗内存的弊端，

#### 分页插件原理

分页插件的基本原理是使用 MyBatis 提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql，根据不同数据库，添加对应的物理分页语句和物理分页参数。

#### 如何实现自定义插件

实现 MyBatis 的 Interceptor 接口并重写 `intercept()` 方法，然后再给插件添加注解@Intercepts，指定要拦截哪一个接口的哪些方法即可，最后在配置文件中配置你编写的插件。

```java
package com.zone.chajian;

import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.plugin.*;
import org.apache.ibatis.session.ResultHandler;
import org.apache.ibatis.session.RowBounds;

import java.util.Properties;

/**
 1.实现Interceptor和重写intercept
 2.添加@Intercepts注解，指明拦截谁
 @Signature 插件签名，告诉mybatis拦截谁
 type指定拦截那个对象，method指定拦截方法，args指定方法参数
 3.在全局配置文件中进行配置。
 注：多个插件都拦截同一对象，会按配置的顺序执行多层代理，而拦截方法的执行则是反过来的。
 */
@Intercepts(
        {@Signature(type = org.apache.ibatis.executor.Executor.class, method = "query",
        args ={MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class})}
)
public class MyInterceptor implements Interceptor {

    //拦截时执行的方法
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        System.out.println("MyInterceptor拦截到的目标对象"+invocation.getTarget());
        System.out.println("执行之前===============MyInterceptor");        
        Object res = invocation.proceed();//执行目标对象方法
        System.out.println("执行之后===============MyInterceptor");
        return res;
    }
    
    //封装目标对象，可以返回本身，也可以返回代理
    //四大对象都会经过该方法，但只有被拦截对象才返回代理类，否则返回本身
    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target,this); //包装目标对象，创建代理类
    }

    //获取插件配置信息===全局配置文件中配置
    @Override
    public void setProperties(Properties properties) {
        System.out.println("MyInterceptor配置信息"+ properties);
    }
}
```

配置拦截器插件

```xml
<plugins>
    <plugin interceptor="com.zone.chajian.MyInterceptor">
        <property name="name" value="MyInterceptor"/>
    </plugin>
</plugins>
```

#### 自定义插件原理

MyBatis自定义插件用于拦截 `ParameterHandler` 、 `ResultSetHandler` 、 `StatementHandler` 、 `Executor` 这 4 种接口，MyBatis 使用 JDK 的动态代理为拦截对象生成代理类，这样就能在某个方法执行前后进行操作（AOP，面向切面）。

### 13、批处理

一次提交多个语句给数据库执行

实现一：使用带参的openSession获取SqlSession
```java
sqlSessionFactory.openSession(ExecutorType.BATCH);
```

实现二：整合spring，注入spring时，配置批处理属性

### 14、问题



尝试maven项目--不加杠-

```xml
<!--在build中配置resources, 来防止我们资源导出失败问题 也就是class文件中有BlogMapper.xml文件-->
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <!--<filtering>false</filtering>-->
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <!--<filtering>false</filtering>-->
        </resource>
    </resources>
</build>
```



**cannot download sources**

idea无法下载源码，通过idea控制台，进入pom.xml所在的目录，执行

**mvn dependency:resolve -Dclassifier=sources**

[文章](https://www.jianshu.com/p/a259e322794c)

下载之后，在choose Sources，选中下好的源码即可

 <img src="../../../JuniorFirst/typora/imgs/image-20220421163112674.png" alt="image-20220421163112674" style="width: 70%;" />



9.xml注释问题

```xml
<select id="showTeacher" resultMap="ts">
    无效注释
    --           where t.id = #{id} and t.id = s.tid;
    select * from teacher where id = #{id};
</select>
```

```java
### The error occurred while setting parameters
### SQL: --           where t.id = ? and t.id = s.tid;           select * from teacher where id = ?;
### Cause: org.apache.ibatis.type.TypeException: Could not set parameters for mapping: 
```

```xml
有效注解
<!--where t.id = #{id} and t.id = s.tid;-->
```
