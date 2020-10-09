---
type: blog
title: SpringBoot 集成 Prometheus
date: 2020-09-23
categories: 教程
tags: SpringBoot, Prometheus
cover: false
meta:
  date: false
---

## 一、添加依赖

- Maven `pom.xml`

```xml
<!--  第一条必须加，否则会导致 Could not autowire. No beans of 'xxxx' type found 的错误  -->
<dependency>
	<groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-core</artifactId>
</dependency>
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

- Gradle `build.gradle`

```
implementation 'org.springframework.boot:spring-boot-starter-actuator'
compile 'io.micrometer:micrometer-registry-prometheus'
compile 'io.micrometer:micrometer-core'
```



- 打开 Prometheus 监控接口 `application.properties`

```
server.port=8088
spring.application.name=springboot2-prometheus
management.endpoints.web.exposure.include=*
management.metrics.tags.application=${spring.application.name}
```

可以直接运行程序，访问`http://localhost:8088/actuator/prometheus`可以看到下面的内容：

```
# HELP jvm_buffer_total_capacity_bytes An estimate of the total capacity of the buffers in this pool
# TYPE jvm_buffer_total_capacity_bytes gauge
jvm_buffer_total_capacity_bytes{id="direct",} 90112.0
jvm_buffer_total_capacity_bytes{id="mapped",} 0.0
# HELP tomcat_sessions_expired_sessions_total  
# TYPE tomcat_sessions_expired_sessions_total counter
tomcat_sessions_expired_sessions_total 0.0
# HELP jvm_classes_unloaded_classes_total The total number of classes unloaded since the Java virtual machine has started execution
# TYPE jvm_classes_unloaded_classes_total counter
jvm_classes_unloaded_classes_total 1.0
# HELP jvm_buffer_count_buffers An estimate of the number of buffers in the pool
# TYPE jvm_buffer_count_buffers gauge
jvm_buffer_count_buffers{id="direct",} 11.0
jvm_buffer_count_buffers{id="mapped",} 0.0
# HELP system_cpu_usage The "recent cpu usage" for the whole system
# TYPE system_cpu_usage gauge
system_cpu_usage 0.0939447637893599
# HELP jvm_gc_max_data_size_bytes Max size of old generation memory pool
# TYPE jvm_gc_max_data_size_bytes gauge
jvm_gc_max_data_size_bytes 2.841116672E9

# 此处省略超多字...
```

## 二、Prometheus 安装与配置

使用 docker 运行 Prometheus（仅初始测试）

```bash
docker run --name prometheus -d -p 9090:9090 prom/prometheus:latest
```

写配置文件`prometheus.yml`

```bash
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']
  # demo job
  -  job_name: 'springboot-actuator-prometheus-test' # job name
     metrics_path: '/actuator/prometheus' # 指标获取路径
     scrape_interval: 5s # 间隔
     basic_auth: # Spring Security basic auth 
       username: 'actuator'
       password: 'actuator'
     static_configs:
     - targets: ['docker.for.mac.localhost:18080'] # 实例的地址，默认的协议是http （这里开始有问题，直接写 localhost 是访问容器内的地址，而不是宿主机的。可通过在网页上方 status -> targets 查看对应的服务情况
```

运行 docker

```bash
docker run -d -p 9090:9090 -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus --config.file=/etc/prometheus/prometheus.yml
```

访问 `http://localhost:9090`，可看到如下界面

![img](https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/16fcafb94a6bf392.jpeg)

- 点击 `Insert metric at cursor` ，即可选择监控指标；点击 `Graph` ，即可让指标以图表方式展示；点击`Execute` 按钮，即可看到指标图

## 三、Grafana 安装和配置

1、启动

```bash
$ docker run -d --name=grafana -p 3000:3000 grafana/grafana 
```

2、登录

访问 `http://localhost:3000/login` ，初始账号/密码为：`admin/admin` 

3、配置数据源

- 点击左侧齿轮`Configuration`中`Add Data Source`，会看到如下界面：

<img src="https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/image-20200924110615218.png" alt="image-20200924110615218" style="zoom: 25%;" />

- 这里我们选择Prometheus 当做数据源，这里我们就配置一下Prometheus 的访问地址，点击 `Save & Test`

<img src="https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/image-20200924110807215.png" alt="image-20200924110807215" style="zoom:25%;" />

4、创建监控 Dashboard

- 点击导航栏上的 `+` 按钮，并点击Dashboard，将会看到类似如下的界面

<img src="https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/image-20200924110949259.png" alt="image-20200924110949259" style="zoom:25%;" />

- 点击`+ Add new panel`

<img src="https://raw.githubusercontent.com/shuopic/ImgBed/master/NoteImgs/image-20200924111151204.png" alt="image-20200924111151204" style="zoom:25%;" />

## 四、自定义监控指标

1、创建 Prometheus 监控管理类`PrometheusCustomMonitor`

```java
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.DistributionSummary;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import java.util.concurrent.atomic.AtomicInteger;

@Component
public class PrometheusCustomMonitor {

    private Counter requestErrorCount;
    private Counter orderCount;
    private DistributionSummary amountSum;
    private AtomicInteger failCaseNum;

    private final MeterRegistry registry;

    @Autowired
    public PrometheusCustomMonitor(MeterRegistry registry) {
        this.registry = registry;
    }

    @PostConstruct
    private void init() {
        requestErrorCount = registry.counter("requests_error_total", "status", "error");
        orderCount = registry.counter("order_request_count", "order", "test-svc");
        amountSum = registry.summary("order_amount_sum", "orderAmount", "test-svc");
        failCaseNum = registry.gauge("fail_case_num", new AtomicInteger(0));
    }

    public Counter getRequestErrorCount() {
        return requestErrorCount;
    }

    public Counter getOrderCount() {
        return orderCount;
    }

    public DistributionSummary getAmountSum() {
        return amountSum;
    }

    public AtomicInteger getFailCaseNum() {
        return failCaseNum;
    }
}
```

2、新增`/order`接口

当 `flag="1"`时，抛异常，模拟下单失败情况。在接口中统计`order_request_count`和`order_amount_sum`

```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import java.util.Random;

@RestController
public class TestController {
    @Resource
    private PrometheusCustomMonitor monitor;

    @RequestMapping("/order")
    public String order(@RequestParam(defaultValue = "0") String flag) throws Exception {
        // 统计下单次数
        monitor.getOrderCount().increment();
        if ("1".equals(flag)) {
            throw new Exception("出错啦");
        }
        Random random = new Random();
        int amount = random.nextInt(100);
        // 统计金额
        monitor.getAmountSum().record(amount);

        monitor.getFailCaseNum().set(amount);
        return "下单成功, 金额: " + amount;
    }
}
```

3、新增全局异常处理器`GlobalExceptionHandler`

统计下单失败次数`requests_error_total`

```java
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.annotation.Resource;

@ControllerAdvice
public class GlobalExceptionHandler {
    @Resource
    private PrometheusCustomMonitor monitor;

    @ResponseBody
    @ExceptionHandler(value = Exception.class)
    public String handle(Exception e) {
        monitor.getRequestErrorCount().increment();
        return "error, message: " + e.getMessage();
    }
}
```

4、测试

启动项目，访问`http://localhost:8080/order`和`http://localhost:8080/order?flag=1`模拟下单成功和失败的情况，然后我们访问`http://localhost:8080/actuator/prometheus`，可以看到我们自定义指标已经被 `/prometheus` 端点暴露出来

```
# HELP requests_error_total  
# TYPE requests_error_total counter
requests_error_total{application="springboot-actuator-prometheus-test",status="error",} 41.0
# HELP order_request_count_total  
# TYPE order_request_count_total counter
order_request_count_total{application="springboot-actuator-prometheus-test",order="test-svc",} 94.0
# HELP order_amount_sum  
# TYPE order_amount_sum summary
order_amount_sum_count{application="springboot-actuator-prometheus-test",orderAmount="test-svc",} 53.0
order_amount_sum_sum{application="springboot-actuator-prometheus-test",orderAmount="test-svc",} 2701.0
```

5、使用 Prometheus 监控

重新运行 docker

```bash
docker run -d -p 9090:9090 -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus --config.file=/etc/prometheus/prometheus.yml
```

选择对应指标后可以看到数据变化

6、使用 Grafana 展示

在 Dashboard 界面选择对应的监控指标即可

---

参考资料：

[Metric types | Prometheus](https://prometheus.io/docs/concepts/metric_types/)

[IntelliJ IDEA创建第一个Spring Boot项目_Study Notes-CSDN博客](https://blog.csdn.net/typa01_kk/article/details/76696618)

[Spring Boot 使用 Micrometer 集成 Prometheus 监控 Java 应用性能 【springboot 2.0】](https://cloud.tencent.com/developer/article/1508319)

[Micrometer Application Monitoring【官方文档】](https://micrometer.io/docs/concepts)

[Spring Boot 微服务应用集成Prometheus + Grafana 实现监控告警 ★](https://juejin.im/post/6844904052417904653)

[Monitoring Java Spring Boot applications with Prometheus: Part 1 | by Arush Salil | Kubernauts 【放弃这个教程？】client java 不支持 springboot 2.x，最高支持 1.5](https://blog.kubernauts.io/https-blog-kubernauts-io-monitoring-java-spring-boot-applications-with-prometheus-part-1-c0512f2acd7b)

[Spring Boot 参考指南（端点）_风继续吹 - SegmentFault 思否](https://segmentfault.com/a/1190000015309478)

