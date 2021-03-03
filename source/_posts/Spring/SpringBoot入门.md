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

SpringBoot 出现原因：整合各种 Spring 框架，简化配置，快速上手开发。【约定大于配置】

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



### 1.3 了解自动配置原理（重点）

- SpringBoot 底层整合了 Sping, SpringMVC等



## 二、核心功能

### 2.1 配置文件

### 2.2 Web 开发

#### 2.2.1 请求参数处理

- REST 风格
  - GET-查询    DELETE-删除     PUT-修改      POST-保存



- **请求方法注解**

```java
// 原始写法
@RequestMapping(value = "/hello", method = RequestMethod.POST)
// 简略写法
@PostMapping("/hello")
```

#### 2.2.2 响应参数处理

