# 聊聊Spring Boot Actuator

## 概述

在本文中，我们将介绍Spring Boot Actuator。我们将首先介绍基础知识，然后详细讨论Spring Boot 1.x和2.x中的可用内容。

我们将在Spring Boot 1.x中学习如何使用，配置和扩展此监视工具。然后，我们将讨论如何利用反应式编程模型使用Boot 2.x和WebFlux进行相同的操作。

自2014年4月起，Spring Boot Actuator随Spring Boot一起发布。

随着SpringBoot2的发布，执行器进行了重新设计，并添加了新的激动人心的端点。本指南分为三个主要部分：

- 什么是执行器（Actuator）
- Spring Boot 1.x Actuator
- Spring Boot 2.x Actuator

👋：*执行器==Actuator*

---

## 什么是执行器（Actuator）

本质上，执行器为我们的应用带来了生产就绪功能。

**通过对他的依赖，监视我们的应用程序、收集度量、了解流量或数据库的状态变得轻松简单：**

这个库的主要好处是，我们可以获得生产级工具，而不必亲自实现这些功能。Actuator主要用于公开有关正在运行的应用程序的操作信息-运行状况，指标，信息，转储，环境等。它使用HTTP端点或JMX Bean使我们能够与其交互。一旦在类路径上使用执行器，便可以立即使用几个端点。与大多数Spring模块一样，我们可以通过多种方式轻松地对其进行配置或扩展。

### Getting Started

要启用SpringBootActuator，我们只需要将SpringBootActuator依赖项添加到包管理器中。在Maven：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

请注意，无论引导版本如何，这都保持有效，因为版本是在Spring Boot Bill of Materials（BOM）中指定的

---

## Spring Boot 1.x Actuator

在1.x中，执行器遵循R/W(read/write)模型，这意味着我们可以对其进行读取或写入。例如。我们可以检索指标或应用程序的运行状况。另外，我们可以优雅地终止我们的应用程序或更改日志记录配置。

为了让它工作，执行器需要Spring MVC通过HTTP公开其端点。不支持其他技术。

### Endpoints

**在1.x中，Actuator带来了自己的安全模型。它利用了Spring Security构造，但是需要与应用程序的其余部分独立配置。**

而且，大多数端点都是敏感的-表示它们不是完全公开的，换句话说，大多数信息将被省略-而少数端点不是敏感的比如/info。

以下是Boot提供的一些最常见的端点：

- /health –显示应用程序运行状况信息（通过未经身份验证的连接访问时为简单的“状态”，或通过身份验证时显示为完整的消息详细信息）；默认情况下不敏感

- /info –显示任意应用程序信息；默认不敏感

- /metrics –显示当前应用程序的“指标”信息；默认情况下是敏感的

- /trace –显示跟踪信息（默认情况下，最后几个HTTP请求）

  我们可以在[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints)中找到现有端点的完整列表。

### 配置现有端点

可以使用以下格式使用属性来自定义每个端点：*endpoints.[endpoint name].[property to customize]*

提供三个属性：

- id –将通过HTTP访问该端点
- enabled–如果为true，则可以访问，否则不能访问
- sensitive–如果为true，则需要授权通过HTTP显示关键信息

例如，添加以下属性将自定义/beans端点：

> ```properties
> endpoints.beans.id=springbeans
> endpoints.beans.sensitive=false
> endpoints.beans.enabled=true
> ```

⚠️：上述方法现已被弃用。



### /health端点

/health端点用于检查正在运行的应用程序的运行状况或状态。监视软件通常会执行此操作，以警告我们正在运行的实例出现故障或由于其他原因而变得不正常。例如。数据库的连接问题，磁盘空间不足…

默认情况下，在未授权状态下访问只会显示运行状况信息：

```json
{
  "status" : "UP"
}
```

此健康信息是从在我们的应用程序上下文中配置的，实现了HealthIndicator接口的所有bean收集的。

HealthIndicator返回的某些信息本质上是敏感的，但是我们可以配置endpoints.health.sensitive = false来公开更多详细信息，例如磁盘空间，消息传递代理连接，自定义检查等。

我们还可以实现自己的自定义运行状况指示器-可以收集特定于应用程序的任何类型的自定义运行状况数据，并通过/health端点自动将其公开：

```java
@Component
public class HealthCheck implements HealthIndicator {
  
    @Override
    public Health health() {
        int errorCode = check();//perform some specific health check
        if (errorCode != 0) {
            return Health.down()
              .withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }
     
    public int check() {
       //Our logic to check health
        return 0;
    }
}
```

输出结果如下所示：

```java
{
    "status" : "DOWN",
    "myHealthCheck" : {
        "status" : "DOWN",
        "Error Code" : 1
     },
     "diskSpace" : {
         "status" : "UP",
         "free" : 209047318528,
         "threshold" : 10485760
     }
}
```

### /info端点

我们还可以自定义/info端点显示的数据-例如：

```properties
info.app.name=Spring Sample Application
info.app.description=This is my first spring boot application
info.app.version=1.0.0
```

以及示例输出：

```json
{
    "app" : {
        "version" : "1.0.0",
        "description" : "This is my first spring boot application",
        "name" : "Spring Sample Application"
    }
}
```

### /metrics端点

metrics端点发布有关OS、JVM以及应用程序级度量的信息。启用后，我们将获得诸如内存、堆、处理器、线程、加载的类、卸载的类、线程池以及一些HTTP度量等信息。

以下是该端点的输出结果：

```json
{
    "mem" : 193024,
    "mem.free" : 87693,
    "processors" : 4,
    "instance.uptime" : 305027,
    "uptime" : 307077,
    "systemload.average" : 0.11,
    "heap.committed" : 193024,
    "heap.init" : 124928,
    "heap.used" : 105330,
    "heap" : 1764352,
    "threads.peak" : 22,
    "threads.daemon" : 19,
    "threads" : 22,
    "classes" : 5819,
    "classes.loaded" : 5819,
    "classes.unloaded" : 0,
    "gc.ps_scavenge.count" : 7,
    "gc.ps_scavenge.time" : 54,
    "gc.ps_marksweep.count" : 1,
    "gc.ps_marksweep.time" : 44,
    "httpsessions.max" : -1,
    "httpsessions.active" : 0,
    "counter.status.200.root" : 1,
    "gauge.response.root" : 37.0
}
```

**为了收集自定义指标，我们支持“仪表”（即数据的单值快照）和“计数器”（即增/减指标）。**

让我们在/metrics端点中实现我们自己的自定义指标。例如，我们将自定义登录流程以记录成功和失败的登录尝试：

```java
@Service
public class LoginServiceImpl {
 
    private final CounterService counterService;
     
    public LoginServiceImpl(CounterService counterService) {
        this.counterService = counterService;
    }
     
    public boolean login(String userName, char[] password) {
        boolean success;
        if (userName.equals("admin") && "secret".toCharArray().equals(password)) {
            counterService.increment("counter.login.success");
            success = true;
        }
        else {
            counterService.increment("counter.login.failure");
            success = false;
        }
        return success;
    }
}
```

输出结果如下所示：

```json
{
    ...
    "counter.login.success" : 105,
    "counter.login.failure" : 12,
    ...
}
```

**请注意，登录尝试和其他安全相关事件作为审计事件在执行器中是现成可用的。**

### 创建新端点

除了使用Spring Boot提供的现有端点之外，我们还可以创建一个全新的端点。

首先，我们需要让新的端点实现endpoint<T>接口：

```json
@Component
public class CustomEndpoint implements Endpoint<List<String>> {
     
    @Override
    public String getId() {
        return "customEndpoint";
    }
 
    @Override
    public boolean isEnabled() {
        return true;
    }
 
    @Override
    public boolean isSensitive() {
        return true;
    }
 
    @Override
    public List<String> invoke() {
       //Custom logic to build the output
        List<String> messages = new ArrayList<String>();
        messages.add("This is message 1");
        messages.add("This is message 2");
        return messages;
    }
}
```

为了访问此新端点，将使用其ID对其进行映射，即我们可以按/customEndpoint对其进行访问。 输出：

```json
[ "This is message 1", "This is message 2" ]
```

### 进一步定制

为了安全起见，我们可能选择通过非标准端口公开执行器端点-可以轻松地使用management.port属性进行配置。

同样，正如我们已经提到的，在1.x中。 Actuator基于Spring Security配置其自己的安全模型，但与应用程序的其余部分无关。

因此，我们可以更改management.address属性以限制可以从网络访问端点的位置：

```properties
#port used to expose actuator
management.port=8081 
 
#CIDR allowed to hit actuator
management.address=127.0.0.1 
 
#Whether security should be enabled or disabled altogether
management.security.enabled=false
```

此外，默认情况下，除/info外的所有内置终结点都是敏感的。如果应用程序使用的是Spring Security，我们可以通过在application.properties文件中定义默认的安全属性（用户名，密码和角色）来保护这些端点：

```properties
security.user.name=admin
security.user.password=secret
management.security.role=SUPERUSER
```

---

## Spring Boot 2.x Actuator

在2.x中，Actuator保持其基本意图，但简化了其模型，扩展了其功能并结合了更好的默认设置。

首先，该版本与技术无关。此外，它通过将其与应用程序合并来简化其安全模型。

最后，在各种更改中，请务必记住其中一些正在停用。这包括HTTP请求/响应以及Java API。

此外，与旧的RW（读/写）模型相反，最新版本现在支持CRUD模型。

### 技术支持

在第二个主要版本中，Actuator现在与技术无关，而在1.x中，它与MVC关联，因此与Servlet API关联。

在2.x中，Actuator定义了其模型，可插入且可扩展，而无需依赖MVC。

**因此，通过这种新模型，我们能够利用MVC和WebFlux作为基础Web技术。**

此外，可以通过实现正确的适配器来添加即将出现的技术。

最后，仍然支持JMX公开端点，而无需任何其他代码。

### 重要变化

与以前的版本不同，Actuator禁用了大多数端点。

因此，默认情况下仅有的两个可用的是/health和/info。

如果要启用所有这些功能，可以设置management.endpoints.web.exposure.include = *或者，我们可以列出应启用的端点。

现在，Actuator与我们的应用安全规则共享安全配置。因此，极大地简化了安全模型。

因此，要调整执行器安全性规则，我们可以为/actuator/**添加一个条目：

```java
@Bean
public SecurityWebFilterChain securityWebFilterChain(
  ServerHttpSecurity http) {
    return http.authorizeExchange()
      .pathMatchers("/actuator/**").permitAll()
      .anyExchange().authenticated()
      .and().build();
}
```

我们可以在全新的Actuator[官方文档](https://docs.spring.io/spring-boot/docs/2.0.x/actuator-api/html/)中找到更多详细信息。

同样，默认情况下，所有执行器端点现在都放在/actuator路径下。

与上一版本相同，我们可以使用新属性 management.endpoints.web.base-path调整此路径。

### 预定义端点

让我们看一下一些可用的端点，其中大多数已经在1.x中可用。

尽管如此，仍添加了一些端点，删除了一些端点，并重新构建了一些端点：

- /auditevents –列出与安全审核相关的事件，例如用户登录/注销。另外，我们可以在其他字段中按主体或类型进行过滤
- /beans –返回BeanFactory中所有可用的bean。与/auditevents不同，它不支持过滤
- /conditions –以前称为/autoconfig，围绕自动配置生成条件报告
- /configprops–允许我们获取所有@ConfigurationProperties bean
- /env –返回当前环境属性。此外，我们可以检索单个属性
- /flyway –提供有关我们的Flyway数据库迁移的详细信息
- /health –总结我们应用程序的健康状态
- /heapdump –从我们的应用程序使用的JVM构建并返回堆转储
- /info –返回常规信息。它可能是自定义数据，构建信息或有关最新提交的详细信息
- /liquibase –行为类似于/flyway，但对于Liquibase
- /logfile –返回普通的应用程序日志
- /loggers –使我们能够查询和修改应用程序的日志记录级别
- /metrics –详细说明我们应用程序的指标。这可能包括通用指标和自定义指标
- /prometheus –返回与上一个类似的指标，但其格式可以与Prometheus服务器一起使用
- /scheduledtasks –提供有关我们应用程序中每个计划任务的详细信息
- /sessions –列出我们正在使用Spring Session的HTTP会话
- /shutdown –正常关闭应用程序
- /threaddump –转储底层JVM的线程信息

### 健康指标

就像以前的版本一样，我们可以轻松添加自定义指标。与其他API相反，用于创建自定义健康状况端点的抽象保持不变。但是，添加了新接口ReactiveHealthIndicator来实现反应式健康检查。

让我们看一个简单的自定义反应式健康检查：

```java
@Component
public class DownstreamServiceHealthIndicator implements ReactiveHealthIndicator {
 
    @Override
    public Mono<Health> health() {
        return checkDownstreamServiceHealth().onErrorResume(
          ex -> Mono.just(new Health.Builder().down(ex).build())
        );
    }
 
    private Mono<Health> checkDownstreamServiceHealth() {
       //we could use WebClient to check health reactively
        return Mono.just(new Health.Builder().up().build());
    }
}
```

健康指标的一个方便功能是我们可以将它们汇总为层次结构的一部分。因此，按照前面的示例，我们可以将所有下游服务归为“下游服务”类别。只要可以访问每个嵌套服务，此类别都是健康的。

复合健康检查通过CompositeHealthIndicator在1.x中进行。另外，在2.x中，我们可以将CompositeReactiveHealthIndicator用作其响应式副本。

与Spring Boot 1.x中不同的是，*endpoints.*<id>.sensitive标志已被删除。要隐藏完整的健康报告，我们可以利用新的*management.endpoint.health.show-details*。默认情况下，此标志为false。

### Spring Boot 2中的指标

在Spring Boot 2.0中，内部指标已被Micrometer支持所取代。因此，我们可以期待重大的变化。如果我们的应用程序使用的是度量服务，例如GaugeService或CounterService，则它们将不再可用。

相反，我们应该直接与Micrometer进行交互。在Spring Boot 2.0中，我们将自动配置一个类型为MeterRegistry的bean。

此外，Micrometer现在是执行器依赖项的一部分。因此，只要执行器依赖项在类路径中，我们就应该继续。

此外，我们将从/metrics端点获得全新的响应：

```json
{
  "names": [
    "jvm.gc.pause",
    "jvm.buffer.memory.used",
    "jvm.memory.used",
    "jvm.buffer.count",
   //...
  ]
}
```

正如我们在前面的示例中观察到的，没有像1.x那样的实际指标。

要获取特定指标的实际值，我们现在可以导航至所需指标，即/actuator/metrics/jvm.gc.pause并获取详细的响应：

```json
{
  "name": "jvm.gc.pause",
  "measurements": [
    {
      "statistic": "Count",
      "value": 3.0
    },
    {
      "statistic": "TotalTime",
      "value": 7.9E7
    },
    {
      "statistic": "Max",
      "value": 7.9E7
    }
  ],
  "availableTags": [
    {
      "tag": "cause",
      "values": [
        "Metadata GC Threshold",
        "Allocation Failure"
      ]
    },
    {
      "tag": "action",
      "values": [
        "end of minor GC",
        "end of major GC"
      ]
    }
  ]
}
```

如我们所见，指标现在更加全面。不仅包括不同的值，还包括一些相关的元数据。

### 自定义/info端点

/info端点保持不变。和以前一样，我们可以使用Maven或Gradle各自的依赖项添加git详细信息：

```xml
 <plugin>
   <groupId>pl.project13.maven</groupId>
   <artifactId>git-commit-id-plugin</artifactId>
   <version>2.2.5</version>
   <executions>
     <execution>
       <goals>
         <goal>revision</goal>
       </goals>
     </execution>
   </executions>
   <configuration>
     <dotGitDirectory>${project.basedir}/.git</dotGitDirectory>
   </configuration>
</plugin>
```

同样，我们还可以使用Maven或Gradle插件来包含构建信息，包括名称，组和版本：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>build-info</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### 创建自定义端点

如前所述，我们可以创建自定义端点。但是，Spring Boot 2重新设计了实现此目标的方法，以支持与技术无关的新范例。

让我们创建一个Actuator端点来查询，启用和禁用应用程序中的功能标志：

```java
@Component
@Endpoint(id = "features")
public class FeaturesEndpoint {
 
    private Map<String, Feature> features = new ConcurrentHashMap<>();
 
    @ReadOperation
    public Map<String, Feature> features() {
        return features;
    }
 
    @ReadOperation
    public Feature feature(@Selector String name) {
        return features.get(name);
    }
 
    @WriteOperation
    public void configureFeature(@Selector String name, Feature feature) {
        features.put(name, feature);
    }
 
    @DeleteOperation
    public void deleteFeature(@Selector String name) {
        features.remove(name);
    }
 
    public static class Feature {
        private Boolean enabled;
 
       //[...] getters and setters 
    }
 
}
```

为了得到端点，我们需要一个bean。在我们的例子中，我们使用@Component来实现这一点。另外，我们需要用@Endpoint装饰这个bean。

端点的路径由@Endpoint的id参数确定，在本例中，它将请求路由到/actuator/features

准备就绪后，我们可以使用以下方法开始定义操作：

- *@ReadOperation –* it'll map to HTTP *GET*
- *@WriteOperation* – it'll map to HTTP *POST*
- *@DeleteOperation* – it'll map to HTTP *DELETE*

当我们使用应用程序中的上一个端点运行该应用程序时，Spring Boot将对其进行注册。

一种快速的验证方法是检查日志：

> ```
> [...].WebFluxEndpointHandlerMapping: Mapped "{[/actuator/features/{name}],
>   methods=[GET],
>   produces=[application/vnd.spring-boot.actuator.v2+json || application/json]}"
> [...].WebFluxEndpointHandlerMapping : Mapped "{[/actuator/features],
>   methods=[GET],
>   produces=[application/vnd.spring-boot.actuator.v2+json || application/json]}"
> [...].WebFluxEndpointHandlerMapping : Mapped "{[/actuator/features/{name}],
>   methods=[POST],
>   consumes=[application/vnd.spring-boot.actuator.v2+json || application/json]}"
> [...].WebFluxEndpointHandlerMapping : Mapped "{[/actuator/features/{name}],
>   methods=[DELETE]}"[...]
> ```

在前面的日志中，我们可以看到WebFlux是如何公开我们的新端点的。如果我们切换到MVC，它只需委托该技术，而不必更改任何代码。

此外，对于这种新方法，我们需要牢记一些重要的注意事项：

- 与MVC没有依赖关系
- 以前作为方法存在的所有元数据（敏感、已启用…）不再存在。但是，我们可以使用@endpoint启用或禁用端点（id=“features”，enableByDefault=false）
- 与1.x不同，不再需要扩展给定的接口
- 与旧的读/写模型相比，现在我们可以使用@DeleteOperation定义DELETE操作

### 扩展现有的端点

假设我们要确保应用程序的生产实例不是SNAPSHOT版本。我们决定通过更改返回实例信息的Actuator端点的HTTP状态代码（即/info）来执行此操作。如果我们的应用碰巧是快照。我们将获得不同的HTTP状态代码。

我们可以使用@EndpointExtension注释或其更具体的专门化@EndpointWebExtension或@EndpointJmxExtension轻松扩展预定义端点的行为：

```java
@Component
@EndpointWebExtension(endpoint = InfoEndpoint.class)
public class InfoWebEndpointExtension {
 
    private InfoEndpoint delegate;
 
   //standard constructor
 
    @ReadOperation
    public WebEndpointResponse<Map> info() {
        Map<String, Object> info = this.delegate.info();
        Integer status = getStatus(info);
        return new WebEndpointResponse<>(info, status);
    }
 
    private Integer getStatus(Map<String, Object> info) {
       //return 5xx if this is a snapshot
        return 200;
    }
}
```

### 启用所有端点

为了使用HTTP访问执行器端点，我们需要同时启用和公开它们。默认情况下，除/shutdown之外的所有端点均处于启用状态。默认情况下仅公开/health和/info端点。

我们需要添加以下配置以公开所有端点：

```
management.endpoints.web.exposure.include=*
```

要显式启用特定端点（例如/shutdown），我们使用：

```
management.endpoint.shutdown.enabled=true
```

要公开除一个（例如/loggers）以外的所有启用的端点，我们使用：

```
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=loggers
```

---

## 总结

在本文中，我们讨论了Spring Boot Actuator。我们开始定义执行器的含义及其对我们的作用。 接下来，我们关注当前Spring Boot版本1.x的Actuator。讨论如何使用它，并对它进行扩展。 然后，我们在Spring Boot 2中讨论了Actuator。我们专注于新功能，并利用WebFlux公开了端点。

此外，我们还讨论了在新迭代中可以找到的重要安全更改。我们讨论了一些流行的端点以及它们如何发生变化。

最后，我们演示了如何自定义和扩展执行器。





---

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-153651.jpg)