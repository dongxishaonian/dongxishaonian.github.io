[返回上层](index)
# Spring Boot2+Resilience4j实现容错之Bulkhead

> Resilience4j是一个轻量级、易于使用的容错库，其灵感来自Netflix Hystrix，但专为Java 8和函数式编程设计。轻量级，因为库只使用Vavr，它没有任何其他外部库依赖项。相比之下，Netflix Hystrix对Archaius有一个编译依赖关系，Archaius有更多的外部库依赖关系，如Guava和Apache Commons。
>
> Resilience4j提供高阶函数（decorators）来增强任何功能接口、lambda表达式或方法引用，包括断路器、速率限制器、重试或舱壁。可以在任何函数接口、lambda表达式或方法引用上使用多个装饰器。优点是您可以选择所需的装饰器，而无需其他任何东西。
>
> 有了Resilience4j，你不必全力以赴，你可以选择你需要的。
>
> https://resilience4j.readme.io/docs/getting-started

## 概览

Resilience4j提供了两种舱壁模式**（Bulkhead）**，可用于限制并发执行的次数：

- SemaphoreBulkhead（信号量舱壁，默认），基于Java并发库中的Semaphore实现。
- FixedThreadPoolBulkhead（固定线程池舱壁），它使用一个有界队列和一个固定线程池。

本文将演示在Spring Boot2中集成Resilience4j库，以及在多并发情况下实现如上两种舱壁模式。

## 引入依赖

在Spring Boot2项目中引入Resilience4j相关依赖

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot2</artifactId>
    <version>1.4.0</version>
</dependency>
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-bulkhead</artifactId>
    <version>1.4.0</version>
</dependency>
```

由于Resilience4j的Bulkhead依赖于Spring AOP，所以我们需要引入Spring Boot AOP相关依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

我们可能还希望了解Resilience4j在程序中的运行时状态，所以需要通过Spring Boot Actuator将其暴露出来

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

## 实现SemaphoreBulkhead（信号量舱壁）

resilience4j-spring-boot2实现了对resilience4j的自动配置，因此我们仅需在项目中的yml/properties文件中编写配置即可。

SemaphoreBulkhead的配置项如下：

| **属性配置**       | **默认值** | **含义**                                   |
| ------------------ | ---------- | ------------------------------------------ |
| maxConcurrentCalls | 25         | 舱壁允许的最大并行执行量                   |
| maxWaitDuration    | 0          | 尝试进入饱和舱壁时，应阻塞线程的最长时间。 |

### 添加配置

示例（使用yml）：

```yaml
resilience4j.bulkhead:
  configs:
    default:
      maxConcurrentCalls: 5
      maxWaitDuration: 20ms
  instances:
    backendA:
      baseConfig: default
    backendB:
      maxWaitDuration: 10ms
      maxConcurrentCalls: 20
```

如上，我们配置了SemaphoreBulkhead的默认配置为`maxConcurrentCalls: 5，maxWaitDuration: 20ms`。并在backendA实例上应用了默认配置，而在backendB实例上使用自定义的配置。**这里的实例可以理解为一个方法/lambda表达式等等的可执行单元。**

### 编写Bulkhead逻辑

**定义一个受SemaphoreBulkhead管理的Service类：**

```java
@Service
public class BulkheadService {
    private final Logger logger = LoggerFactory.getLogger(this.getClass());
    @Autowired
    private BulkheadRegistry bulkheadRegistry;

    @Bulkhead(name = "backendA")
    public JsonNode getJsonObject() throws InterruptedException {
        io.github.resilience4j.bulkhead.Bulkhead.Metrics metrics = bulkheadRegistry.bulkhead("backendA").getMetrics();
        logger.info("now i enter the method!!!,{}<<<<<<{}", metrics.getAvailableConcurrentCalls(), metrics.getMaxAllowedConcurrentCalls());
        Thread.sleep(1000L);
        logger.info("now i exist the method!!!");
        return new ObjectMapper().createObjectNode().put("file", System.currentTimeMillis());
    }
}
```

如上，我们将`@Bulkhead`注解放到需要管理的方法上面。并且通过`name`属性指定该方法对应的Bulkhead实例名字（这里我们指定的实例名字为backendA，所以该方法将会利用默认的配置）。

**定义接口类：**

```java
@RestController
public class BulkheadResource {
    @Autowired
    private BulkheadService bulkheadService;

    @GetMapping("/json-object")
    public ResponseEntity<JsonNode> getJsonObject() throws InterruptedException {
        return ResponseEntity.ok(bulkheadService.getJsonObject());
    }
}
```

**编写测试：**

首先添加测试相关依赖

```xml
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>3.0.5</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <version>4.0.2</version>
    <scope>test</scope>
</dependency>
```

这里我们使用[rest-assured](https://github.com/rest-assured/rest-assured)和[awaitility](https://github.com/awaitility/awaitility)编写多并发情况下的API测试

```java
public class SemaphoreBulkheadTests extends Resilience4jDemoApplicationTests {
    @LocalServerPort
    private int port;
    @BeforeEach
    public void init() {
        RestAssured.baseURI = "http://localhost";
        RestAssured.port = port;
    }

    @Test
    public void 多并发访问情况下的SemaphoreBulkhead测试() {
        CopyOnWriteArrayList<Integer> statusList = new CopyOnWriteArrayList<>();
        IntStream.range(0, 8).forEach(i -> CompletableFuture.runAsync(() -> {
                statusList.add(given().get("/json-object").statusCode());
            }
        ));
        await().atMost(1, TimeUnit.MINUTES).until(() -> statusList.size() == 8);
        System.out.println(statusList);
        assertThat(statusList.stream().filter(i -> i == 200).count()).isEqualTo(5);
        assertThat(statusList.stream().filter(i -> i == 500).count()).isEqualTo(3);
    }
}
```

可以看到所有请求中只有前五个顺利通过了，其余三个都因为超时而导致接口报500异常。我们可能并不希望这种不友好的提示，因此Resilience4j提供了自定义的失败回退方法。当请求并发量过大时，无法正常执行的请求将进入回退方法。

首先我们定义一个回退方法

```java
private JsonNode fallback(BulkheadFullException exception) {
        return new ObjectMapper().createObjectNode().put("errorFile", System.currentTimeMillis());
    }
```

**注意：回退方法应该和调用方法放置在同一类中，并且必须具有相同的方法签名，并且仅带有一个额外的目标异常参数。**

然后在`@Bulkhead`注解中指定回退方法：`@Bulkhead(name = "backendA", fallbackMethod = "fallback")`

最后修改API测试代码：

```java
@Test
public void 多并发访问情况下的SemaphoreBulkhead测试使用回退方法() {
    CopyOnWriteArrayList<Integer> statusList = new CopyOnWriteArrayList<>();
    IntStream.range(0, 8).forEach(i -> CompletableFuture.runAsync(() -> {
            statusList.add(given().get("/json-object").statusCode());
        }
    ));
    await().atMost(1, TimeUnit.MINUTES).until(() -> statusList.size() == 8);
    System.out.println(statusList);
    assertThat(statusList.stream().filter(i -> i == 200).count()).isEqualTo(8);
}
```

运行单元测试，成功！可以看到，我们定义的回退方法，在请求过量时起作用了。

## 实现FixedThreadPoolBulkhead（固定线程池舱壁）

FixedThreadPoolBulkhead的配置项如下：

| 配置名称           | 默认值                                           | 含义                                                         |
| ------------------ | ------------------------------------------------ | ------------------------------------------------------------ |
| maxThreadPoolSize  | `Runtime.getRuntime().availableProcessors()`     | 配置最大线程池大小                                           |
| coreThreadPoolSize | `Runtime.getRuntime().availableProcessors() - 1` | 配置核心线程池大小                                           |
| queueCapacity      | 100                                              | 配置队列的容量                                               |
| keepAliveDuration  | 20ms                                             | 当线程数大于核心时，这是多余空闲线程在终止前等待新任务的最长时间 |

### 添加配置

示例（使用yml）：

```yaml
resilience4j.thread-pool-bulkhead:
  configs:
    default:
      maxThreadPoolSize: 4
      coreThreadPoolSize: 2
      queueCapacity: 2
  instances:
    backendA:
      baseConfig: default
    backendB:
      maxThreadPoolSize: 1
      coreThreadPoolSize: 1
      queueCapacity: 1
```

如上，我们定义了一段简单的FixedThreadPoolBulkhead配置，我们指定的默认配置为：`maxThreadPoolSize: 4，coreThreadPoolSize: 2，queueCapacity: 2`，并且指定了两个实例，其中backendA使用了默认配置而backendB使用了自定义的配置。

### 编写Bulkhead逻辑

**定义一个受FixedThreadPoolBulkhead管理的方法：**

```java
@Bulkhead(name = "backendA", type = Bulkhead.Type.THREADPOOL)
public CompletableFuture<JsonNode> getJsonObjectByThreadPool() throws InterruptedException {
    io.github.resilience4j.bulkhead.ThreadPoolBulkhead.Metrics metrics = threadPoolBulkheadRegistry.bulkhead("backendA").getMetrics();
    logger.info("now i enter the method!!!,{}", metrics);
    Thread.sleep(1000L);
    logger.info("now i exist the method!!!");
    return CompletableFuture.supplyAsync(() -> new ObjectMapper().createObjectNode().put("file", System.currentTimeMillis()));
}
```

如上定义和**SemaphoreBulkhead**的方法大同小异，其中`@Bulkhead`显示指定了type的属性为`Bulkhead.Type.THREADPOOL`，表明其方法受**FixedThreadPoolBulkhead**管理。由于`@Bulkhead`默认的**Bulkhead**是**SemaphoreBulkhead**，所以在未指定type的情况下为**SemaphoreBulkhead**。另外，**FixedThreadPoolBulkhead**只对CompletableFuture方法有效，所以我们必创建返回CompletableFuture类型的方法。

**定义接口类方法**

```java
@GetMapping("/json-object-with-threadpool")
public ResponseEntity<JsonNode> getJsonObjectWithThreadPool() throws InterruptedException, ExecutionException {
    return ResponseEntity.ok(bulkheadService.getJsonObjectByThreadPool().get());
}
```

**编写测试代码**

```java
@Test
public void 多并发访问情况下的ThreadPoolBulkhead测试() {
    CopyOnWriteArrayList<Integer> statusList = new CopyOnWriteArrayList<>();
    IntStream.range(0, 8).forEach(i -> CompletableFuture.runAsync(() -> {
            statusList.add(given().get("/json-object-with-threadpool").statusCode());
        }
    ));
    await().atMost(1, TimeUnit.MINUTES).until(() -> statusList.size() == 8);
    System.out.println(statusList);
    assertThat(statusList.stream().filter(i -> i == 200).count()).isEqualTo(6);
    assertThat(statusList.stream().filter(i -> i == 500).count()).isEqualTo(2);
}
```

测试中我们并行请求了8次，其中6次请求成功，2次失败。根据FixedThreadPoolBulkhead的默认配置，最多能容纳maxThreadPoolSize+queueCapacity次请求（根据我们上面的配置为6次）。

同样，我们可能并不希望这种不友好的提示，那么我们可以指定回退方法，在请求无法正常执行时使用回退方法。

```java
private CompletableFuture<JsonNode> fallbackByThreadPool(BulkheadFullException exception) {
    return CompletableFuture.supplyAsync(() -> new ObjectMapper().createObjectNode().put("errorFile", System.currentTimeMillis()));
}
@Bulkhead(name = "backendA", type = Bulkhead.Type.THREADPOOL, fallbackMethod = "fallbackByThreadPool")
public CompletableFuture<JsonNode> getJsonObjectByThreadPoolWithFallback() throws InterruptedException {
    io.github.resilience4j.bulkhead.ThreadPoolBulkhead.Metrics metrics = threadPoolBulkheadRegistry.bulkhead("backendA").getMetrics();
    logger.info("now i enter the method!!!,{}", metrics);
    Thread.sleep(1000L);
    logger.info("now i exist the method!!!");
    return CompletableFuture.supplyAsync(() -> new ObjectMapper().createObjectNode().put("file", System.currentTimeMillis()));
}
```

**编写测试代码**

```java
@Test
public void 多并发访问情况下的ThreadPoolBulkhead测试使用回退方法() {
    CopyOnWriteArrayList<Integer> statusList = new CopyOnWriteArrayList<>();
    IntStream.range(0, 8).forEach(i -> CompletableFuture.runAsync(() -> {
            statusList.add(given().get("/json-object-by-threadpool-with-fallback").statusCode());
        }
    ));
    await().atMost(1, TimeUnit.MINUTES).until(() -> statusList.size() == 8);
    System.out.println(statusList);
    assertThat(statusList.stream().filter(i -> i == 200).count()).isEqualTo(8);
}
```

由于指定了回退方法，所有请求的响应状态都为正常了。

------

## 总结

本文首先简单介绍了Resilience4j的功能及使用场景，然后具体介绍了Resilience4j中的Bulkhead。演示了如何在Spring Boot2项目中引入Resilience4j库，使用代码示例演示了如何在Spring Boot2项目中实现Resilience4j中的两种Bulkhead（SemaphoreBulkhead和FixedThreadPoolBulkhead），并编写API测试验证我们的示例。

**本文示例代码地址：**https://github.com/cg837718548/resilience4j-demo

------

🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152705.jpg)


---
---
---


## 🤔  💭 👇👇👇

<script src="https://utteranc.es/client.js"
        repo="dongxishaonian/issue-posted"
        issue-term="pathname"
        label="🙂🙃😡🥶😬🤣😄"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>

