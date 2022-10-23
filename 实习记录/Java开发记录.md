# 面向大厂

# SpringBoot

社区版没有Spring Initializr，可以安装Alibaba Cloud Toolkit

[教程](https://www.one-tab.com/page/0xMhi2TTQNmcLNKudKFWWA)

## SpringBoot配置Swagger3.0

### 使用Swagger的好处

- 代码变，文档变。只需要少量的注解，Swagger 就可以根据代码自动生成 API 文档，很好的保证了文档的时效性。
- 跨语言性，支持 40 多种语言。
- Swagger UI 呈现出来的是一份可交互式的 API 文档，我们可以直接在文档页面尝试 API 的调用，省去了准备复杂的调用参数的过程。
- 还可以将文档规范导入相关的工具（例如 Postman、SoapUI）, 这些工具将会为我们自动地创建自动化测试。

### Swagger3.0

#### 兼容说明

- 需要Java 8
- 需要Spring5.x
- 需要SpringBoot2.2+
- 示例

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.1.RELEASE</version>
</parent>
<dependencyManagement>
    <dependencies>
    	<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR4</version>
            <type>pom</type>
            <scope>import</scope>
    	</dependency>
    </dependencies>
</dependencyManagement>
```

#### 依赖

- application.yaml配置

```yaml
spring:
  application:
    name: springfox-swagger
server:
  port: 8080
# ===== 自定义swagger配置 ===== #
swagger:
  enable: true
  application-name: ${spring.application.name}
  application-version: 1.0
  application-description: springfox swagger 3.0整合Demo
  try-host: http://localhost:${server.port}
```

- 需要注意与Spring Boot、SpringCloud等依赖的版本对应关系

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>3.0.0</version>
</dependency>
```

#### 编写Swagger的配置类

```java
@Configuration
@EnableOpenApi
public class SwaggerConfig {
}
```

项目启动后访问：http://ip:port/swagger-ui/index.html

出现Swagger接口文档页面，说明基本配置环境成功。

<img src="Java%E5%BC%80%E5%8F%91%E8%AE%B0%E5%BD%95/3e5a1ec0d80ab2655490e190e74d3775cb44542a.png" alt="image" style="zoom: 67%;" />

#### Swagger详细配置

1. 配置Swagger首先需要构建Bean实例Docket对象

```java
@Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.OAS_30).pathMapping("/")
				// DocumentationType与Swagger版本对应
                // 定义是否开启swagger，false为关闭，可以通过变量控制
                .enable(swaggerProperties.getEnable())
                // 将api的元信息设置为包含在json ResourceListing响应中。 
                .apiInfo(apiInfo())
                // 接口调试地址
                .host(swaggerProperties.getTryHost())
                // 选择哪些接口作为swagger的doc发布
                .select()
                // RequestHandlerSelectors,配置要扫描接口的方式
                // basePackage指定要扫描的包
                // any()扫描所有，项目中的所有接口都会被扫描到
                // none()不扫描
                // withClassAnnotation()扫描类上的注解
                // withMethodAnnotation()扫描方法上的注解
                .apis(RequestHandlerSelectors.any())
            	// 过滤某些路径
                .paths(PathSelectors.any())
                .build()
                // 支持的通讯协议集合
                .protocols(newHashSet("https", "http"))
                // 授权信息设置，必要的header token等认证信息
                .securitySchemes(securitySchemes())
                // 授权信息全局应用
                .securityContexts(securityContexts());
        		// 配置Api文档分组，一个Docket对应一个分组
        		// .groupName("moruoyiming")
    }
```

2. 构建Docket需要创建ApiInfo实例，其主要作用是构建我们接口文档的页面显示的基础信息

```java
private ApiInfo apiInfo() {
        return new ApiInfoBuilder().title(swaggerProperties.getApplicationName() + " Api Doc")
                .description(swaggerProperties.getApplicationDescription())
                .contact(new Contact("lighter", null, "123456@gmail.com"))
                .version("Application Version: " + swaggerProperties.getApplicationVersion() + ", Spring Boot Version: " + SpringBootVersion.getVersion())
                .build();
    }
```

3. 实体类配置

```java
@ApiModel("用户实体")
public class User {
   @ApiModelProperty("用户名")
   public String username;
   @ApiModelProperty("密码")
   public String password;
}
// 只要这个实体在请求接口的返回值中，即便是泛型，都能映射到实体项中。
```

- 常用注解

| Swagger注解                                                | 简单说明                                             |
| :--------------------------------------------------------- | :--------------------------------------------------- |
| [@Api](https://springboot.io/u/api)(tags = “xxx模块说明”)  | 作用在模块类上                                       |
| **@ApiOperation**(“xxx接口说明”)                           | 作用在接口方法上                                     |
| **@ApiModel**(“xxxPOJO说明”)                               | 作用在模型类上：如VO、BO                             |
| **@ApiModelProperty**(value = “xxx属性说明”,hidden = true) | 作用在类方法和属性上，hidden设置为true可以隐藏该属性 |
| **@ApiParam**(“xxx参数说明”)                               | 作用在参数、方法和字段上，类似@ApiModelProperty      |

### 总结（一个常见的Swagger配置方案）

- SwaggerConfiguration类

```java
@EnableOpenApi
@Configuration
public class SwaggerConfiguration implements WebMvcConfigurer {
    private final SwaggerProperties swaggerProperties;

    public SwaggerConfiguration(SwaggerProperties swaggerProperties) {
        this.swaggerProperties = swaggerProperties;
    }

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.OAS_30).pathMapping("/")

                // 定义是否开启swagger，false为关闭，可以通过变量控制
                .enable(swaggerProperties.getEnable())

                // 将api的元信息设置为包含在json ResourceListing响应中。 
                .apiInfo(apiInfo())

                // 接口调试地址
                .host(swaggerProperties.getTryHost())

                // 选择哪些接口作为swagger的doc发布
                .select()
                .apis(RequestHandlerSelectors.any())
                // 过滤某个路径
                .paths(PathSelectors.any())
                .build()

                // 支持的通讯协议集合
                .protocols(newHashSet("https", "http"))

                // 授权信息设置，必要的header token等认证信息
                .securitySchemes(securitySchemes())

                // 设置分组名
                .groupName("默认分组")

                // 授权信息全局应用
                .securityContexts(securityContexts());
    }

    /**
     * API 页面上半部分展示信息
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder().title(swaggerProperties.getApplicationName() + " Api Doc")
                .description(swaggerProperties.getApplicationDescription())
                .contact(new Contact("lighter", null, "123456@gmail.com"))
                .version("Application Version: " + swaggerProperties.getApplicationVersion() + ", Spring Boot Version: " + SpringBootVersion.getVersion())
                .build();
    }

    /**
     * 设置授权信息
     */
    private List<SecurityScheme> securitySchemes() {
        ApiKey apiKey = new ApiKey("BASE_TOKEN", "token", In.HEADER.toValue());
        return Collections.singletonList(apiKey);
    }

    /**
     * 授权信息全局应用
     */
    private List<SecurityContext> securityContexts() {
        return Collections.singletonList(
                SecurityContext.builder()
                        .securityReferences(Collections.singletonList(new SecurityReference("BASE_TOKEN", new AuthorizationScope[]{new AuthorizationScope("global", "")})))
                        .build()
        );
    }

    @SafeVarargs
    private final <T> Set<T> newHashSet(T... ts) {
        if (ts.length > 0) {
            return new LinkedHashSet<>(Arrays.asList(ts));
        }
        return null;
    }

    /**
     * 通用拦截器排除swagger设置，所有拦截器都会自动加swagger相关的资源排除信息
     */
    @SuppressWarnings("unchecked")
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        try {
            Field registrationsField = FieldUtils.getField(InterceptorRegistry.class, "registrations", true);
            List<InterceptorRegistration> registrations = (List<InterceptorRegistration>) ReflectionUtils.getField(registrationsField, registry);
            if (registrations != null) {
                for (InterceptorRegistration interceptorRegistration : registrations) {
                    interceptorRegistration
                            .excludePathPatterns("/swagger**/**")
                            .excludePathPatterns("/webjars/**")
                            .excludePathPatterns("/v3/**")
                            .excludePathPatterns("/doc.html");
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

- SwaggerProperties类

```java

@Component
@ConfigurationProperties("swagger")
public class SwaggerProperties {
    /**
     * 是否开启swagger，生产环境一般关闭，所以这里定义一个变量
     */
    private Boolean enable;
    
    /**
     * 项目应用名
     */
    private String applicationName;

    /**
     * 项目版本信息
     */
    private String applicationVersion;

    /**
     * 项目描述信息
     */
    private String applicationDescription;

    /**
     * 接口调试地址
     */
    private String tryHost;

    public Boolean getEnable() {
        return enable;
    }

    public void setEnable(Boolean enable) {
        this.enable = enable;
    }

    public String getApplicationName() {
        return applicationName;
    }

    public void setApplicationName(String applicationName) {
        this.applicationName = applicationName;
    }

    public String getApplicationVersion() {
        return applicationVersion;
    }

    public void setApplicationVersion(String applicationVersion) {
        this.applicationVersion = applicationVersion;
    }

    public String getApplicationDescription() {
        return applicationDescription;
    }

    public void setApplicationDescription(String applicationDescription) {
        this.applicationDescription = applicationDescription;
    }

    public String getTryHost() {
        return tryHost;
    }

    public void setTryHost(String tryHost) {
        this.tryHost = tryHost;
    }
}
```

## SpringBoot部署到服务器

### Maven项目的打包方式

- jar（默认方式）：是Java打的包，一般只包括class和一些部署文件，常用于内部，接口，服务部署。
- war（web application archive）：可以理解为Javaweb打的包，不仅包括jar中有的文件类型，还包括网站页面如html、jsp等。此类型的包需要发布到容器里（如Tomcat），在war应用中会有WEB-INF目录，该目录下包括web.xml（应用配置文件）和classes目录（存放编译好的Servlet类、Jsp、JavaBean）。
- pom：Maven模块化管理时，父级项目为此方式。

> 使用maven进行模块划分管理时，一般都会有一个父级项目，pom文件除了GAV(groupId, artifactId, version)是必须要配置的，另一个重要的属性就是packaging打包类型，所有的父级项目的packaging都为pom（可以到pom.xml文件里面进行手动配置），packaging默认是jar类型，如果不作配置，maven会将该项目打成jar包。
>
> 作为父级项目，还有一个重要的属性，那就是modules，通过modules标签将项目的所有子项目引用进来，在build父级项目时，会根据子模块的相互[依赖关系](https://so.csdn.net/so/search?q=依赖关系&spm=1001.2101.3001.7020)整理一个build顺序，然后依次build。
>
> 子类项目的packaging值只能是war或者jar。

### 常用部署方式（Linux）

1. Package一下SpringBoot项目，完成之后target文件下会生成jar包。
2. 脚本在Linux下进行部署

- 后台启动startTask.sh

```shell
#设置工程路径
project_path=/root/test
cd $project_path
#nohup后台启动，输出日志到test.log
nohup java -jar test.jar > test.log &
#打印日志
tail -f test.log
```

```shell
#执行权限
chmod +x startTask.sh
#执行
./startTask.sh
```

- 根据应用端口关闭服务stopTask.sh

```shell
#设置关闭的端口
port=8080
#获取此端口运行的进程
pid=`lsof -t -i:$port`
#判断如果进程号不为空则，关闭进程
if test -z "$pid";then
   echo "test 工程未启动！"
else
  kill -9 $pid
  echo "test 工程进程$pid 关闭成功！"
```

执行同上。

- 使用应用名获取其进程号

```shell
 pid=$(jps -l|grep app-provider-1.0-SNAPSHOT.jar |awk '{print $1}')
echo " app-provider-1.0-SNAPSHOT.jar pid "+${pid}+"will be kill"
```

## Maven仓库配置镜像

<img src="Java%E5%BC%80%E5%8F%91%E8%AE%B0%E5%BD%95/watermark,size_14,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=.png" alt="IntelliJ IDEA 中为Maven 配置阿里云镜像源_.net" style="zoom:50%;" />

[配置文件](https://gitee.com/yang-peijie/codes/15p4qjcrut0swz28gilxn52)

# Git

## 分支操作

### 基本操作

```shell

git checkout master # 切换分支
git pull 
// 创建并切换到新分支相当于：git branch xxx, git checkout xxx
git checkout -b newBranchName
git branch -a # 白色显示为本地分支、绿色显示为当前分支、红色显示为远程分支；
```

### 常见操作

#### 基于远程master分支创建新分支

```shell
git checkout remotes/origin/master	# 切换至远程分支
git checkout -b newBranchName 		# 基于创建，为本地新分支
```

#### 创建远程分支

```shell
// git push <远程主机名> <本地分支名>:<远程分支名>
git push origin newBranch:newBranch		# 将本地新分支推送创建远程新分支
```

<img src="Java%E5%BC%80%E5%8F%91%E8%AE%B0%E5%BD%95/image-20220729180437139.png" alt="image-20220729180437139" style="zoom:50%;" />

#### 本地的新分支关联远程同名新分支

```shell
git checkout dev # 切换到新分支下
# git branch --set-upstream-to=origin/<远程分支名> <本地分支名>如设置当前分支，第二个参数可省略,；
git branch --set-upstream-to=origin/dev
```

<img src="Java%E5%BC%80%E5%8F%91%E8%AE%B0%E5%BD%95/image-20220729180832110.png" alt="image-20220729180832110" style="zoom: 67%;" />

#### 合并分支

- 将新分支合并至develop分支

```shell
// 切换到develop分支
git checkout develop
// 合并newBranch代码
git merge newBranch
// 提交commit到远程（newBranch分支有多少个commit就会生成几个）
git push
```

- 将远程分支合并至本地develop分支

```shell
git pull origin newBranch
```

#### 删除分支

- 本地

```shell
// "-d" 如果该分支代码未合并到其他分支，将无法删除;
// "-D" 强制删除分支，不会出现任何提示；
git branch -D xxxx
```

- 远程

```shell
git push origin --delete newBranch
```

### 常见问题

#### 工作区有新代码切换分支(使用工作现场存放)

```shell
// 如 直接 "git stash"则将上次commit注释作为说明
git stash save "存储说明"
git checkout B
// 返回原分支
git checkout A
git stash pop
```

- 如果本来想在A分支上开发， 开发过程中才发现当前处在B分支，想强制将工作区间代码迁到A分支也可以借助“工作现场”完成：

```shell
git stash save "存储说明"
git checkout B
git stash pop
// 如有冲突且处理完所有冲突
git add -A
```

#### 切换分支出现异常

如，在A分支编写了新代码，突然切换到B分支，虽然没异常，但B分支却有了大量A分支的代码，可以使用远程分支进行回退

```shell
// 将所有改动提交到本地仓库	
git add -A
git commit -m "这个commit会被覆盖"
//B 是当前分支名
git reset --hard origin/B
```

<img src="Java%E5%BC%80%E5%8F%91%E8%AE%B0%E5%BD%95/4225993309-58f44e1362fab_articlex.png" alt="图片描述" style="zoom:67%;" />

#### 合并分支出现无法自动合并的冲突

手动解决冲突后进行告知

```shell
git add -A
git commit -m "[master]-合并newBranch代码"
git push
```

#### 简化commit

建议先`git rebase xxx`，如有冲突，`git rebase --abort`，再换用`git merge xxx`。

#### 相关文章

[图解分支](https://blog.csdn.net/weixin_34342207/article/details/89064319?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165908745516782248596424%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165908745516782248596424&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-89064319-null-null.142^v35^new_blog_fixed_pos&utm_term=git%E5%88%86%E6%94%AF%E6%93%8D%E4%BD%9C&spm=1018.2226.3001.4187)

[代码回退](https://segmentfault.com/a/1190000007070302)

[区域详解](https://segmentfault.com/a/1190000007067265)

## 同一个电脑配置多个git账号

先说结论，

- 同一台电脑，同一个github账号，只对应一个SSH key，不要对应多个SSH key

- 同一台电脑，n个不同的github账号（或n个gitlab或同时存在），需要对应n个不同github账号的SSH key

- 同一个github账号，n台电脑，需要n个该github账号的SSH key

### 解决方案

1. 对每一个git账号，生成对应的私钥

```shell
ssh-keygen -t rsa -C "git邮箱" # 重命名以避免覆盖,且必须存放在.ssh目录下
```

2. 默认SSH只会读取id_rsa，所以为了让SSH识别新的私钥id_rsa_xx，需要将其添加到SSH agent

```shell
ssh-agent bash # 打开ssh-agent
ssh-add ~/.ssh/id_rsa_xx # 添加新私钥
# 其他命令
ssh-add -D 删除ssh-agent中的所有密钥
ssh-add -d 删除ssh-agent中的所有密钥
ssh-add -L 删除ssh-agent中的所有密钥
ssh-add -l 删除ssh-agent中的所有密钥
```

3. 配置config文件(需要注意是文件不是文本文档)

```
# 配置
Host 自定义别名，随意，但会影响git相关命令（建议取账号user.name 或 账号邮箱user.email）
HostName 远程仓库真实域名(github.com或git.oschina.net) 或 ip地址
PreferredAuthentications publickey
IdentityFile 本地私钥id_rsa的路径
User 配置使用用户名，写账号user.name或写git就行
# Port 端口号（默认22）
```

4. 将私钥分别添加到对应的平台，不赘述

### 测试

```shell
ssh -T git@Host # 有提示信息表示配置成功
git clone git@Host自定义别名 # clone远程仓库需要将真实域名改成自定义别名
git remote set-url origin git@Host别名:gitrepo # 老项目需要修改远程仓库的URL，或者直接重新clone
```

### 示例

```shell
# 配置
Host aeroht
HostName git.aeroht.local
PreferredAuthentications publickey
IdentityFile C:\Users\HT\.ssh\id_rsa
User git
# Port 端口号（默认22）
# 配置
Host gitee
HostName gitee.com
PreferredAuthentications publickey
IdentityFile C:\Users\HT\.ssh\id_rsa_gitee
User git
# Port 端口号（默认22）
```

# Linux



```shell
sudo apt install net-tools
sudo apt install openssh-server
# 安装jdk
# 写入环境变量
```

