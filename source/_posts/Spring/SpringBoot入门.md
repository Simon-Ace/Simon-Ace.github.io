---
type: blog
title: SpringBoot 入门
date: 2021-02-05
categories: 教程
tags: SpringBoot
cover: false
meta:
  date: false
---

## 一、基础入门

### 1.1 Spring 与 SpringBoot

### 1.2 SpringBoot 第一个程序

#### 1.2.1 maven 设置

`pom.xml`中添加

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```

#### 1.2.2 主类

`MainApplication.java`

```java
@SpringBootApplication
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class, args);
    }
}
```

`HelloController.java`

```java
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello() {
        return "Hello springboot !";
    }
}
```

启动即可

完全参考官方文档即可，写的很清楚

[官方文档](https://docs.spring.io/spring-boot/docs/2.3.4.RELEASE/reference/html/getting-started.html#getting-started-first-application-dependencies)



### 1.3 了解自动配置原理

- SpringBoot 底层整合了 Sping, SpringMVC等

