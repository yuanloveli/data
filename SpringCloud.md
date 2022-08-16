# SpringCloud

## 服务拆分和远程调用

* 服务拆分总结：
  1. 微服务需要根据业务模块拆分，做到单一职责，不要重复开发相同业务
  2. 微服务可以将部分业务暴露为接口，供其它微服务使用
  3. 不同微服务都应该有自己独立的数据库

* 远程调用

  1. 注册在配置类中注入Bean，RestTemplate

     ```java
     @Bean public RestTemplate restTemplate(){
            return new RestTemplate();
     }
     ```

2.  调用restTemlate.getForObject(url,*.class)

   `User user=restTemlate.getForObject(url,User.class)`

## 服务调用关系

* 服务提供者：暴露接口供其它微服务使用
* 服务消费者：调用其它微服务提供的接口
* 提供者与消费这角色其实是相对的
* 一个微服务可以是消费者也是提供者

## Eureka注册中心

* 搭建EurakaServer服务（其实也是服务注册）  步骤如下：

  1. 创建**一个独立Springboot项目**,导入依赖 它也相当于是个服务

     ```java
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
         <version>3.1.3</version>
     </dependency>
     ```

  2. 编写启动类，添加@EnableEurakaServer注解

  3. 在application.yml文件中添加如下配置：

     ```java
     server:
       port: 10086
     spring:
       application:
         name: eurakaserver
     eureka:
       client:
         service-url:
           defaultZone: http://127.0.0.1:10086/eureka
     ```

     

* 注册服务（到EurekaServer中）  步骤如下：

  1. 在user-service项目中引入依赖

     ```
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
     //    <version>3.1.3</version>
     </dependency>
     ```

  2. 在application.yml中加入如下配置：

     ```java
     spring:
       application:
         name: userservice
     eureka:
       client:
         service-url:
           defaultZone: http://127.0.0.1:10086/eureka
     ```



* 服务发现
  1. 给RestTemplate这个Bean添加@LoadBalanced注解
  2. 给服务提供者的名称服务远程调用

## 负载均衡

* 修改负载均衡两种方式

  1. 代码方式：在order-service中Orderapplication类中，定义一个新的IRule:      

     ```java
     @Bean                               //作用于该类的全部服务
     public IRule randomRule(){
         return  new RandomRule();
     }
     ```

  2. 配置文件方式：在order-service的applicaiton.yml文件中，添加新的配置：

     ```java
     userservice: //服务名称          //只作用于这单个服务
       ribbon:
         NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule  #负载均衡规则
     ```

  3. 开启饥饿加载

     ```java
     ribbon:
       eager-load:
         enabled: true       #开启饥饿加载 作用：减少第一次访问时间
         clients:
          - userservice     #指定服务
     ```

## Nacos

###搭建nacos注册中心

1. 下载nacos  官网：nacos.io  下载1.4.1版本

2. 直接解压就能使用，在bin目录下，进入cmd，输入指令：startup.cmd -m standalone 启动服务 （服务端）

3. 在项目中引入依赖 （客户端）

   ```java
   <!-- nacos客户端依赖包 -->
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

4. 在application.yml文件中编写：

   ```java
   spring:
     application:
       name: orderservice
     cloud:
       nacos:
         server-addr: localhost:8848
         discovery:
           cluster-name: HZ #代指杭州
   ```

5. 实现分级模型，就是集群，在application.yml文件中修改spring.cloud.nacos.discovery.cluster-name属性，如上

###优先访问本地集群

1. 在order-service（消费者）application.yml中配置spring.cloud.nacos.discovery.cluster-name属性

2. 同时配置

   ```java
   userservice:
     ribbon:
       NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule //负载均衡策略
   ```

   注意：配置完后，优先找本地集群，本地无服务再找其它集群服务，找本地集群内服务时是随机

###nacos实例（服务）权重的控制：

1. Nacos控制台可以设置实例（服务）的权重值，0~1之间
2. 同集群内的多个实例，集群越高被访问的频率越高
3. 权重设置为0，则该实例（服务）完全不会被访问   作用：可以用来重启服务

### 命名空间 namespace

作用：环境隔离 使服务分隔开不能直接访问，不同namespace下实例互不可见

步骤：

1. 在nacos控制台（页面）创建命名空间，默认会在public命名空间下

2. 修改命名空间 修改spring.cloud.nacos.discovery.namespace属性 

   ```java
   spring:
     cloud:
       nacos:
         server-addr: localhost:8848
         discovery:
           cluster-name: HZ
           namespace: d4831eef-5222-49c5-9a33-6fcd15e71985  #命名id
   ```

### 临时实例

* 设置服务是否是临时实例：  

​    在 application.yml中配置spring.cloud.nacos.discovery.ephemeral属性     ephemeral: false #是否是 临时实例

### nacos与eureka总结

![image-20220725160747978](C:\Users\我家双九儿\AppData\Roaming\Typora\typora-user-images\image-20220725160747978.png)

### Nacos配置管理

* 将配置交给Nacos管理的步骤：  **要在同一个命名空间下进行**

  1. 在Nacos控制台中心添加配置文件

     注意data ID的格式：文件名称-环境.后缀  

     主要有两种格式：

     [spring.application.name]-[spring.profiles.active].yaml        例如 userservice-dev.yaml

     [spring.application.name].yaml        例如 userservice.yaml   作用范围大  多环境共享配置文件

  2. 在微服务中引入nacos的config依赖

     ```java
     <!--nacos配置管理客户端依赖-->
     <dependency>
         <groupId>com.alibaba.cloud</groupId>
         <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
     </dependency>
     ```

  3. 添加bootstrap.yml文件，配置nacos地址、当前环境、服务名称、文件后缀名。**告诉程序启动时去nacos中读取哪个文件**

* nacos实现热更新两种方式

  1. 在@Value注入的变量所在的类上添加注解@RefreshScope

  2.   2.1使用@ConfigurationProperties注解自动注入

     ```java
     @Component
     @Data
     @ConfigurationProperties(prefix = "pattern")
     public class PatternProperties {
         private String dateformat;
     }
     ```

     2.2 在需要使用该属性的类中，自动注入该类，getdateformat获得属性

  注意事项：

  1. 不是所有配置都放到配置中心，维护麻烦
  2. 建议将一些关键参数，需要运行时调整的参数放到nacos配置中心，一般都是自定义配置

* **微服务读取配置文件的优先级**

  [服务名]-[环境].yaml>[服务名].yml>本地配置

## Feign

* 基于feign远程调用  步骤如下

  1. 引入依赖(哪个服务使用就在哪个pom文件引入)

     ```java
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-openfeign</artifactId>
     </dependency>
     ```

  2. 在配置类中添加@EnableFeignClients注解

  3. 编写FeignClient接口 

     ```java
     @FeignClient("userservice")   //服务名称  
     public interface FeignClient {
         
         @GetMapping("/user/{id}")   //请求参数
         User findByid(@PathVariable("id") Long id);
         
     }
     ```

  4. 使用FeignClient中定义的方法代替RestTemplate

* 自定义Feign配置

  ![image-20220727102218488](C:\Users\我家双九儿\AppData\Roaming\Typora\typora-user-images\image-20220727102218488.png)

两种方式 1.  在application.yml中修改          

```java
feign:
  client:
    config:
      default(userservice):    //default就是全局配置，如果是服务名称，就是对单个服务的配置
        LoggerLevel: Full      //日志级别 默认为NONE
```

 2 .Java代码修改

1. 

   ```java
   public class DefaultFeignConfiguration {
       @Bean
       public Logger.Level LogLevel(){
           return Logger.Level.BASIC;
       }
   }
   ```

2. 添加@EnableFeignClients(defaultConfiguration = DefaultFeignConfiguration.class)

   注意：如果是在@EnableFeignClients注解中声明则代表全局

   ​           如果是在@FeignClient注解中声明则代表某个服务

* Feign性能优化

  Feign底层的客户端实现：

  * URLConnection:默认实现，不支持连接池
  * apache HttpClient:支持连接池
  * OKHttp: 支持连接池

  1. 日志尽量使用Basic
  2. ![image-20220727105234602](C:\Users\我家双九儿\AppData\Roaming\Typora\typora-user-images\image-20220727105234602.png)

## Gateway 网关

###网关的作用：

* 对用户请求做身份认证，权限校验
* 将用户请求路由到微服务，并实现负载均衡
* 对用户请求做限流

SpirngCloud中网关的实现包括两种：

* gateway 
*  zuul

Zuul是基于Servlet的实现，属于阻塞式编程。而SpringCloudGateway则是基于Spring5中提供的WebFlux，属于响应式编程，具备更好的性能。 

###如何搭建网关

* 搭建网关步骤：

  1. 创建一个独立的moudle（也相当于是一个服务）

  2. 导入依赖

     ```java
     <dependencies>
     <!--        nacos服务注册发现依赖-->    //在nacos中心注册微服务
             <dependency>
                 <groupId>com.alibaba.cloud</groupId>
                 <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
             </dependency>
     <!--        网关gateway依赖-->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-gateway</artifactId>
             </dependency>
         </dependencies>
     ```

  3. 在application.yml文件中编写网关信息

     ```java
     server:
       port: 10010
     spring:
       application:
         name: gateway
       cloud:
         nacos:
           server-addr: localhost:8848  #nacos地址
         gateway:
           routes:
             - id: user-service       #路由标识，必须唯一
               uri: lb://userservice  #路由目标地址,http代表固定地址，lb代表根据服务名称负载均衡
               predicates:       #路由断言，判断请求是否符合规则
                 - Path=/user/** # 判断路径是否以/user开头，如果是则符合
             - id: order-service
               uri: lb://orderservice
               predicates:
                 - Path=/order/**
     ```

  4. 发起请求进行测试

### 路由断言工厂（规则）

![image-20220727153856364](C:\Users\我家双九儿\AppData\Roaming\Typora\typora-user-images\image-20220727153856364.png)

predicateFactory作用就是：读取用户配置的路由规则，解析成对应的判断条件，对将来用户发起的请求做判断，判断其符不符合规则。

### 网关全局过滤器

原因：因为仅仅由springcloud提供的断言规则不能满足某些业务的断言需求，所以有时候得要自定义一些全局过滤器来进行断言判断。

* 自定义一个全局过滤器步骤
  1. 在网关这个module下创建一个类，实现GlobalFilter接口
  2. 然后添加@component注解，成为一个Bean自动装配
  3. 添加@Order(-1)  作用：提高优先级 或者实现Orderd接口

例子如下：

```java
//@Order(-1)
@Component
public class AuthorizeFilter implements GlobalFilter , Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        //1.获取请求参数
        ServerHttpRequest request = exchange.getRequest();
        MultiValueMap<String, String> params = request.getQueryParams();
        //2.获取参数中的authorization 参数
        String authorization = params.getFirst("authorization");
        //3.判断参数值是否等于 admin
        if("admin".equals(authorization)){
            //4.是，放行
            return chain.filter(exchange);
        }
        //5.否，拦截
        //5.1.设置状态码
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        //5.2 拦截请求
        return exchange.getResponse().setComplete();
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```



## docker

镜像名称一般分两部分组成：[repository]:[tag]  不加标签默认最新版latest

**镜像**基本命令：

* 查看帮助： docker --help    docker pull --help
* 拉取镜像（获取镜像）：docker pull 镜像名称 
* 打包镜像：docker sava -o 打包文件名 镜像名称
* 删除镜像：docker rmi 镜像名称
* 解压镜像：docker load -i 压缩包名

容器基本命令：

* 运行一个容器：docker run --name containerName -p 80:80 -d nginx

  --name 指定容器名称 -p  主机映射端口：镜像端口 -d 后台运行此镜像  nginx 镜像名称

* 查看所有容器及状态：docker ps

* 查看容器运行日志：docker logs containerName

* 删除容器：docker rm containerName

* 进入容器执行命令：docker exec -it containerName bash

  -it 建立标准输入输出，允许我们于容器交互   bash 进入容器后执行的命令，bash是Linux终端交互命令

  1. docker pause 进入暂停
  2. docker unpause 取消暂停
  3. docker stop 停止容器
  4. docker start 启动容器

数据卷的基本操作：

* docker volume create 卷名称
* docker volume ls   查看所有数据卷
* docker volume rm 卷名称        删除数据卷
* docker volume prune        **删除所有没有正在使用的数据卷**
* docker volume inspect 卷名称   查看该数据卷的文件位置

数据卷挂载：

1. docker run 的命令中通过-v 参数挂载文件或目录到容器中：
   1. -v volume名称：容器内目录
   2. -v 宿主机文件：容器文件                   宿主机文件会覆盖容器文件
   3. -v 宿主机目录：容器内目录
2. 数据卷挂载与目录直接挂载的区别
   1. 数据卷挂载耦合度低，有docker来管理目录，但是目录较深，不好找
   2. 目录挂载耦合度高，需要我们自己管理，不过目录容易寻找查看

