# 一、简介

今天学习完了springboot，做一个简单的博客项目练练手

以代码为主，前端界面直接省略拷贝资源的

然后简单记录一些开大过程中遇到的坑和不是特别会的技术点

## 搭建环境

### 导入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com</groupId>
    <artifactId>myblog</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>myblog</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>



        <!--这三个jar包作用是将markdown格式转成html格式-->
        <dependency>
            <groupId>com.atlassian.commonmark</groupId>
            <artifactId>commonmark</artifactId>
            <version>0.10.0</version>
        </dependency>

        <dependency>
            <groupId>com.atlassian.commonmark</groupId>
            <artifactId>commonmark-ext-heading-anchor</artifactId>
            <version>0.10.0</version>
        </dependency>
        <dependency>
            <groupId>com.atlassian.commonmark</groupId>
            <artifactId>commonmark-ext-gfm-tables</artifactId>
            <version>0.10.0</version>
        </dependency>

        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.2.13</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

#### 定义核心配置文件**application.yml**

```yml
spring:
  #关闭缓存
  thymeleaf:
    cache: false
  #激活指定的环境
  profiles:
    active: dev
 
```

#### 开发环境**application-dev.yml**

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/blog?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456

pagehelper:                #分页插件
  helper-dialect: mysql
  reasonable: true
  support-methods-arguments: true
  params:

mybatis:
  type-aliases-package: com.blog.pojo   #设置别名
  mapper-locations: classpath:mapper/*.xml   #ָ指定myBatis的核心配置文件与Mapper映射文件

logging:  #日志级别
  level:
    root: info
    com.blog: debug
  file: log/blog-dev.log

  #开发环境
```

#### 生产环境**application-pro.yml**

```yml

server:
  port: 8080 #默认


spring:
  datasource:
    url: jdbc:mysql://localhost:3306/blog?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 12456

pagehelper:                #分页插件
  helper-dialect: mysql
  reasonable: true
  support-methods-arguments: true
  params:

mybatis:
  type-aliases-package: com.blog.pojo   #设置别名
  mapper-locations: classpath:mapper/*.xml   #ָ指定myBatis的核心配置文件与Mapper映射文件

logging:  #日志级别
  level:
    root: warn
    com.blog: info
  file: log/blog-pro.log

```

## 定义切面ExceptionController

就是整个项目中 只要跑了异常 我们的切面异常处理映射器就会拦截下来

```java
package com.myblog.handler;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.core.annotation.AnnotationUtils;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;

@ControllerAdvice//拦截多有带Controller的切面
public class ExceptionControllerHandler {

    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    @ExceptionHandler(Exception.class)  //表示该方法可以处理所有类型异常
    public ModelAndView exceptionHandler(HttpServletRequest request, Exception e) throws Exception {

        //日志打印异常信息  两个括号{}是占位符， 将后面的信息一次填入
        logger.error("Request url: {}, Exception: {}", request.getRequestURI(), e);

        //不处理带有ResponseStatus注解的异常  就是如果我们标注了状态码的异常就不会走error界面 直接按照状态码解析出来的异常走模糊匹配或者精确匹配的界面
        if (AnnotationUtils.findAnnotation(e.getClass(), ResponseStatus.class) != null) {
            throw e;
        }

        //返回异常信息到自定义error页面
        ModelAndView mv = new ModelAndView();
        mv.addObject("url", request.getRequestURI());
        mv.addObject("Exception", e);
        mv.setViewName("error/error");
        return mv;
    }

}
```

然后我们自定义404,500这两个异常界面，只要控制器出现的是我们的404，或者500异常就交给springboot帮我们管理，使用模板引擎到具体错误的界面

404界面

```java
package com.myblog.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

//HTTP状态码
@ResponseStatus(HttpStatus.NOT_FOUND)      //自定义NotFoundException异常,会跳转到404页面
public class NotFoundException extends RuntimeException {
    public NotFoundException() {
        super();
    }

    public NotFoundException(String message) {
        super(message);
    }

    public NotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

500界面

```java
package com.myblog.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)////自定义服务器错误处理异常,会跳转到500页面
public class InternalServerErroreException extends RuntimeException {
    public InternalServerErroreException() {
        super();
    }

    public InternalServerErroreException(String message) {
        super(message);
    }

    public InternalServerErroreException(String message, Throwable cause) {
        super(message, cause);
    }

}
```

定义日志切面

```java
package com.myblog.aspect;


import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;

@Aspect    //定义切面 使用@Aspect 注解的类， Spring 将会把它当作一个特殊的Bean（一个切面），也就是不对这个类本身进行动态代理
@Component //添加到组件
public class LogAspect {

    //获得日志对象
    private Logger log = LoggerFactory.getLogger(this.getClass());


    //这个就是说在这个包com.myblog.controller下面发生的任何事情都会被切面织入

    //定义切点 和切点表达式 表示任意返回值 这个包下面的任意类下面的任意方法和方法下面是任意参数
    @Pointcut("execution(* com.myblog.controller.*.*(..))")
    public void log(){}

    //下面引入切点
    @Before("log()")   //注意导入的包
    public void before(JoinPoint joinPoint){
        //在这个方法开始前应该做什么事
        //我们需要他帮我们记录是什么路径地址调用了什么方法得到了什么结果参数是什么
        //所以我们将这这些信息封装成一个对象,然后获取并且输出  这个类就是RequestLog
        //接下来就是获取这个对象，然后封装起来
        //1.获取HttpServletRequest对象
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        String url =request.getRequestURL().toString();//获得访问路径
        String ip = request.getRemoteAddr();//获得访问ip
        //获得类名.方法名
        String classMethod = joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName();
        //获得方法参数
        Object[] args = joinPoint.getArgs();

        RequestLog requestLog = new RequestLog(url,ip, classMethod,args);
        //打印请求信息
        log.info("Request: {}", requestLog);
    }

    //around和AfterReturning的配置区别如下，
    // 注意AfterReturning配置必须有argNames参数，且参数值和returning开麦你的值一样，这样在织入代码里面便可通过returning的值获取被织入函数的返回值。

    @AfterReturning(returning = "result", pointcut = "log()")
    public void doAfterReturn(Object result){

        //打印返回值
        log.info("Result: {}", result);
    }

}
```