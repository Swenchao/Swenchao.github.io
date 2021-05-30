---
title: SpringBoot常用注解
top: true
cover: true
toc: true
mathjax: true
date: 2020-05-11 22:01:14
password:
summary: SpringBoot之常用注解
tags:
- JAVA
categories:
- JAVA
---

# SpringBoot 常用注解

最近由于种种原因，开始学习使用 SpringBoot 了，但是上来发现了好多的注解，一脸懵 * ，于是像整理下，其中大多搜索与网上。

## 项目配置相关

### 1. @SpringBootApplication 注解

@SpringBootApplication是一个复合注解，包含了@SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan 这三个注解。

它们作用分别为：

#### @SpringBootConfiguration 继承于 @Configuration 注解，主要用于加载配置文件

标注当前类是配置类，并会将当前类内声明的一个或多个以 @Bean 注解标记的方法的实例纳入到srping容器中，并且实例名就是方法名。

#### @EnableAutoConfiguration 开启自动配置功能

可以帮助SpringBoot应用将所有符合条件的 @Configuration 配置都加载到当前 SpringBoot 创建并使用的 IoC 容器。借助于Spring框架原有的一个工具类 SpringFactoriesLoader 的支持。

#### @ComponentScan 主要用于组件扫描和自动装配

自动扫描并加载符合条件的组件或bean定义（被@Component，@Controller，@Service，@Repository注解标记的类），最终将这些bean定义加载到容器中。我们可以通过basePackages等属性指定@ComponentScan自动扫描的范围，如果不指定，则默认Spring框架实现从声明@ComponentScan所在类的package进行扫描，默认情况下是不指定的，所以SpringBoot的启动类最好放在root package下。等价于<context:component-scan>的xml配置文件中的配置项。

**因为一般这三个注解会一起使用，因此被包装成了现在的 @SpringBootApplication 注解**

### 2. @ServletComponentScan

Servlet、Filter、Listener 可以直接通过 @WebServlet、@WebFilter、@WebListener 注解自动注册。这样通过注解servlet ，拦截器，监听器的功能而无需其他配置。

### 3. @MapperScan:spring-boot 添加mybatis相应组建依赖之后就可以使用该注解

支持mybatis组件的一个注解，通过此注解指定mybatis接口类的路径，即可完成对mybatis接口的扫描。

它和@mapper注解是一样的作用，不同的地方是扫描入口不一样：@mapper需要加在每一个mapper接口类上面。所以大多数情况下，都是在规划好工程目录之后，通过@MapperScan注解配置路径完成mapper接口的注入。

## Controller 相关注解

### 1. @Controller 控制器，处理http请求

### 2. @RestController 复合注解


@RestController源码

```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Controller
    @ResponseBody
    public @interface RestController {
    	/**
    	* The value may indicate a suggestion for a logical component name,
     	* to be turned into a Spring bean in case of an autodetected component.
     	* @return the suggested component name, if any (or empty String otherwise)
     	* @since 4.0.1
     	*/
    	@AliasFor(annotation = Controller.class)
    	String value() default "";
	}
```

@RestController注解相当于@ResponseBody + @Controller合在一起的作用。RestController使用的效果是将方法返回的对象直接在浏览器上展示成json格式。

### 3. @RequestBody

通过HttpMessageConverter读取Request Body并反序列化为Object（泛指）对象

### 4. @RequestMapping

将 HTTP 请求映射到 MVC 和 REST 控制器的处理方法上

@GetMapping用于将HTTP get请求映射到特定处理程序的方法注解

注解简写：@RequestMapping(value = "/say",method = RequestMethod.GET)等价于：@GetMapping(value = "/say")

GetMapping源码等价于@RequestMapping(method = RequestMethod.GET) 

```java
    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @RequestMapping(method = RequestMethod.GET)
    public @interface GetMapping {
    //...
    }
```

PostMapping源码等价于@RequestMapping(method = RequestMethod.POST) 

```java
    是@RequestMapping(method = RequestMethod.GET)的缩写
    @PostMapping用于将HTTP post请求映射到特定处理程序的方法注解
    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @RequestMapping(method = RequestMethod.POST)
    public @interface PostMapping {
        //...
    }
```

## 取请求参数值

### 1. @PathVariable : 获取url中的数值

```java
    @Controller
    @RequestMapping("/Example")
    public class HelloWorldController {
        @RequestMapping("/getExample/{id}")
        public String getUser(@PathVariable("id")Integer id, Model model) {
            System.out.println("id:"+id);
            return "user";
        }
    }
```

请求示例：http://localhost:8080/Example/getExample/123

### 2. @RequestParam : 获取请求参数的值

```java
    @Controller
    @RequestMapping("/Example")
    public class HelloWorldController {
    @RequestMapping("/getExample")
    public String getUser(@RequestParam("id")Integer id, Model model) {
        System.out.println("id:"+id);
        return "user";
    }
    }
```

请求示例：http://localhost:8080/Example/getExample?id=123

### 3. @RequestHeader 把Request请求header部分的值绑定到方法的参数上

### 4. @CookieValue 把Request header中关于cookie的值绑定到方法的参数上

## 注入 bean 相关

### 1. @Repository

DAO层注解，DAO层中接口继承 JpaRepository<T,ID extends Serializable>，需要在build.gradle中引入相关jpa的一个jar自动加载。

Repository注解源码

```java
    @Target({ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Component
    public @interface Repository {

        /**
         * The value may indicate a suggestion for a logical component name,
         * to be turned into a Spring bean in case of an autodetected component.
         * @return the suggested component name, if any (or empty String otherwise)
         */
        @AliasFor(annotation = Component.class)
        String value() default "";

    }
```

### 2. @Service

```java
    @Target({ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Component
    public @interface Service {

        /**
         * The value may indicate a suggestion for a logical component name,
         * to be turned into a Spring bean in case of an autodetected component.
         * @return the suggested component name, if any (or empty String otherwise)
         */
        @AliasFor(annotation = Component.class)
        String value() default "";
    }
```

- @Service是@Component注解的一个特例，作用在类上
- @Service注解作用域默认为单例
- 使用注解配置和类路径扫描时，被@Service注解标注的类会被Spring扫描并注册为Bean
- @Service用于标注服务层组件,表示定义一个bean
- @Service使用时没有传参数，Bean名称默认为当前类的类名，首字母小写
- @Service(“serviceBeanId”)或@Service(value=”serviceBeanId”)使用时传参数，使用value作为Bean名字

### 3. @Scope作用域注解

@Scope作用在类上和方法上，用来配置 spring bean 的作用域，它标识 bean 的作用域

@Scope源码

```java
    @Target({ElementType.TYPE, ElementType.METHOD})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface Scope {

        /**
         * Alias for {@link #scopeName}.
         * @see #scopeName
         */
        @AliasFor("scopeName")
        String value() default "";

        @AliasFor("value")
        String scopeName() default "";

        ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;
    }
```

属性介绍
value
    singleton   表示该bean是单例的。(默认)
    prototype   表示该bean是多例的，即每次使用该bean时都会新建一个对象。
    request     在一次http请求中，一个bean对应一个实例。
    session     在一个httpSession中，一个bean对应一个实例。

proxyMode
    DEFAULT         不使用代理。(默认)
    NO              不使用代理，等价于DEFAULT。
    INTERFACES      使用基于接口的代理(jdk dynamic proxy)。
    TARGET_CLASS    使用基于类的代理(cglib)。

### 4. @Entity实体类注解

- @Table(name ="数据库表名")，这个注解也注释在实体类上，对应数据库中相应的表。
- @Id、@Column注解用于标注实体类中的字段，pk字段标注为@Id，其余@Column。

### 5. @Bean产生一个bean的方法

@Bean明确地指示了一种方法，产生一个bean的方法，并且交给Spring容器管理。支持别名@Bean("xx-name")

### 6. @Autowired 自动导入

- @Autowired注解作用在构造函数、方法、方法参数、类字段以及注解上
- @Autowired注解可以实现Bean的自动注入

### 7. @Component

虽然有了@Autowired,但是我们还是要写一堆bean的配置文件,相当麻烦,而@Component就是告诉spring,我是pojo类,把我注册到容器中吧,spring会自动提取相关信息。那么我们就不用写麻烦的xml配置文件了

## 事务注解 @Transactional

在Spring中，事务有两种实现方式，分别是编程式事务管理和声明式事务管理两种方式。

编程式事务管理
    编程式事务管理使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。

声明式事务管理
    建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务，通过@Transactional就可以进行事务操作，更快捷而且简单。推荐使用

## 全局异常处理

@ControllerAdvice 统一处理异常

@ControllerAdvice 注解定义全局异常处理类
```java
    @ControllerAdvice
    public class GlobalExceptionHandler {
    }
```

@ExceptionHandler 注解声明异常处理方法

```java
    @ControllerAdvice
    public class GlobalExceptionHandler {

        @ExceptionHandler(Exception.class)
        @ResponseBody
        String handleException(){
            return "Exception Deal!";
        }
    }
```