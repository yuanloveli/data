# Spring

依赖注入是ioc控制反转的主要体现。

##一、使用spring框架的两种方式

###1. xml配置注入方式

```<bean id="UserDao" class="srping.Dao.Impl.UserDaoImpl"></bean>```

有引用对象时：

```java
<bean id="UserService" class="srping.Service.Impl.UserServiceImpl">
    <property name="userDao" ref="UserDao"></property>
</bean>
```

需要扫描外部properties文件时：

step1： 引入上下文

step2：添加：```<context:property-placeholder location="classpath:properties文件路径"/>```

需要引用其它applicationContext.xml时：``<import resource="classpath:"/>`

--------

###2.注解注入方式

注解的作用：使用注解尽可能替代掉xml配置文件

原始注解主要标签：

| @Conponent  | 在类上（web,service,dao）实例化Bean  |
| :---------: | :----------------------------------: |
| @Controller |          在web上实例化Bean           |
|  @Service   |       在Service层上实例化Bean        |
| @Repository |         在Dao层上实例化Bean          |
| @Autowired  |           根据类型依赖注入           |
| @Qualifier  | 结合@Autowired使用，按照名称进行注入 |
|  @Resource  |           上面两个结合使用           |
|   @Value    |             注入普通属性             |
|   @Scope    |          标注Bean的作用范围          |

原始注解能替代大部分xml文件内容，不能完全取代例如：

1. 非自定义的Bean配置：`<bean>`
2. 加载properties文件的配置：`<context:property-placehoder>`
3. 组件扫描的配置：`<context:component-scan>`
4. 引入其它文件：`<import>`

新注解由此诞生：

|                 |                                                              |
| :-------------: | :----------------------------------------------------------: |
| @Configuration  |               指定当前类**是一个Spring配置类**               |
| @ComponentScan  |              指定Spring**初始化容器时扫描该类**              |
|      @Bean      | 使用在==**方法**==上，标注将该方法的返回值存储到Spring容器中 |
| @PropertySource |                   加载.properties配置文件                    |
|     @Import     |                      导入其它**配置类**                      |

​        通过以上注解的结合使用，**xml配置文件可以完全被代替，无需使用applicationContext.xml**

注意：在使用时：`ApplicationContext app = new AnnotationConfigApplicationContext(SpringConfiguration.class);`

xml配置文件是：`ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");`

## 二、Spring集成

### Junit集成

* 集成Junit步骤：
  1. 导入spring集成Junit的坐标，**导入junit坐标，同时也要导入spring-junit坐标**
  2. 在测试类上使用**@Runwith**注解替换原来的运行期
  3. 使用@ContextConfiguration指定配置文件或配置类
  4. 创建方法进行测试

集成作用是：在进行测试时除去这两行重复代码

```java
ApplicationContext app = new AnnotationConfigApplicationContext(SpringConfiguration.class);
UserDao userDao = (UserDao) app.getBean("UserDao");
```

### Web环境集成

集成步骤：

step1：导入spring-web坐标

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.3.10</version>
</dependency>
```

step2：在webapp下配置web.xml文件

​              2.1 将servlet导入web环境中

```java
<servlet>
  <servlet-name>UserServlet</servlet-name>
  <servlet-class>srping.Web.UserServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>UserServlet</servlet-name>
  <url-pattern>/userServlet</url-pattern>
</servlet-mapping>
```

​            2.2配置监听器

```java
<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

​           2.3 配置全局初始化参数

```java
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>classpath:applicationContext.xml</param-value>
</context-param>
```

​          step3：编写UserServlet类

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    ServletContext servletContext = this.getServletContext();
    WebApplicationContext app = WebApplicationContextUtils.getWebApplicationContext(servletContext);
    UserService userService = app.getBean(UserService.class);
    userService.run();
    }
```

### springmvc

实现前端控制器，处理大量重复代码

1. 导入SpringMVC包

   ```java
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-webmvc</artifactId>
       <version>5.3.10</version>
   </dependency>
   ```

   配置Servlet前端控制器(dispacherServlet)

   ```java
   <servlet>
     <servlet-name>DispatcherServlet</servlet-name>
     <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
     <load-on-startup>1</load-on-startup>//加载时自动初始化次数
   </servlet>
     <servlet-mapping>
       <servlet-name>DispatcherServlet</servlet-name>
       <url-pattern>/</url-pattern>
     </servlet-mapping>
   ```

2. 编写Controller

   ```java
   @Controller
   public class UserController  {
       @RequestMapping("/quick")
       public String  save(){
           System.out.println("Save Running.....");
           return "index.jsp";
       }
   }
   ```

3. jiangController使用注解配置到Spring容器中（@Controller,@Component）

   上方代码

4. 在applicationContext.xml文件中配置组件扫描（因为使用了注解），但是一般在spring-mvc.xml文件中配置。

   **重要的是告诉前端控制器这个配置文件在哪：**下方是以applicationContext.xml为例

   ```java
   <servlet>
     <servlet-name>DispatcherServlet</servlet-name>
     <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
     <init-param>
       <param-name>contextConfigLocation</param-name>
       <param-value>classpath:applicationContext.xml</param-value>
     </init-param>
     <load-on-startup>1</load-on-startup>
   </servlet>
   ```

5. 客户端发请求测试

## 三、Intercepter拦截器



|                            Filter                            |              Interceptor               |
| :----------------------------------------------------------: | :------------------------------------: |
| filter过滤器是servlet的一部分,web项目就能使用，拦截全部访问是：/* | springmvc的一部分，拦截全部访问是：/** |

###自定义拦截器

步骤：

1. 创建自定义类实现HandlerInterceptor接口

   ```java
   public class myInterceptor implements HandlerInterceptor {
   
       @Override
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
           System.out.println("拦截成功！");
           return false;
       }
   }
   ```

2. 在spring-mvc.xml中配置拦截器

   ```java
   <mvc:interceptors>
           <mvc:interceptor>
   <!--对哪些资源进行拦截-->
               <mvc:mapping path="/**"/>
   <!--对哪些资源不进行拦截-->
               <mvc:exclude-mapping path="/quick"/>
   <!--指定那个用拦截器-->
               <bean class="srping.interceptor.myInterceptor"></bean>
           </mvc:interceptor>
       </mvc:interceptors>
   ```

3. 测试拦截器拦截效果

# Springboot

## lombok

使用lombok简化实体类开发步骤：

1. 导入lombok坐标

   ```java
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
   </dependency>
   ```

2. 在实体类上使用@data或其它

## mabatis集成

步骤：

1. 创建项目时导入相关坐标，要哪个就选哪个。mabatis ,mysql的
2. 编写Dao层和实体类，在yml配置mysql的连接信息
3. 测试

## Mybatis-pulus(MP)快速开发

1. 首先导入MP的坐标

   ```java
   <dependency>
       <groupId>com.baomidou</groupId>
       <artifactId>mybatis-plus-boot-starter</artifactId>
       <version>3.5.0</version>
   </dependency>
   ```

2. Dao层

继承Basemapper<实体类>   **注意：该实体需要有数据库表中的 所有字段**

```java
public interface UserDao extends BaseMapper<User>{}
```

3. service层

   1. 编写service接口继承 Iservice<T>

      ```java
      public interface UserService extends IService<User> {
      }
      ```

   2. 编写serviceImpl实现类

      ```java
      @Service
      public class UserServiceImpl extends ServiceImpl<UserDao, User> implements UserService {
      }
      ```

4. 编写Controller层

   1. 导入web坐标，因为@RestController、@Requestmapping都是这里带过来的

      

   2. ```java
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
      ```

   3. 编写Controller类

      ```java
      @RestController
      @RequestMapping("/user")
      public class Controller{
      
          @Autowired
          private UserService userService;
      
          @GetMapping
          public R list(){
              return new R(true,userService.list());
          }
      
          @PutMapping
          public R put(@RequestBody User user){
                  return new R(userService.save(user));
          }
          @DeleteMapping("{id}")
          public R deleteById(@PathVariable int id){
              return new R(userService.removeById(id));
          }
      }
      ```

   4. 接收参数的两种方式

      1. 接收实体用@RequestBody
      2. 接受个别参数使用@PathVariable，多个就使用多次

## 表现层对象一致性处理（R对象）

作用：将后端数据统一格式返回给前端，方便前端处理数据

```java
public class R {
    public Boolean flag;
    public Object data;
    
    public R(){}
    
    public R(Boolean flag){
        this.flag=flag;
    }
    
    public R(Boolean flag,Object data){
        this.flag=flag;
        this.data=data;
    }
}
```

## SpringBoot项目打包快速启动

### windows版

step1：对SpringBoot项目打包（执行Maven指令）

​     `mvn package（名称）`

​       或者在idea中打包，记得跳过test测试，有个"skip Tests" 按钮

step2：在cmd中运行项目，先到该jar包目录下，执行启动指令

​     `java -jar springboot.jar（名称）`

注意事项：

1.windows需要安装有jdk环境

2.jar支持命令行启动需要依赖maven插件支持，请确认打包时是否具有SpringBoot对应的Maven插件

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

**如果在cmd启动时出现没有"没有主清单属性"的错误，就是因为打包时没有添加maven插件**

### Linux版

