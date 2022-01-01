[返回上层](index)
# [译]谈谈SpringBoot 事件机制

要“监听”事件，我们总是可以将“监听器”作为事件源中的另一个方法写入事件，但这将使事件源与监听器的逻辑紧密耦合。

对于实际事件，我们比直接方法调用更灵活。我们可以根据需要动态注册和注销某些事件的侦听器。我们还可以为同一事件设置多个侦听器。

本教程概述了如何发布和侦听自定义事件，并解释了 Spring Boot 的内置事件。

---



## 为什么我应该使用事件而不是直接方法调用？

事件和直接方法调用都适合于不同的情况。使用方法调用，就像断言一样-无论发送和接收模块的状态如何，他们都需要知道此事件的发生。

对于事件，另一方面，我们只知道发生了一个事件，哪些模块会被通知并不是我们关心的问题。当我们想要将某些业务处理传递给另一个线程时（例如：在某些任务完成时发送电子邮件），最好使用事件。此外，事件对于测试驱动的开发也很有用。



## 什么是应用程序事件（ Application Events）？

Spring 应用程序事件允许我们发送和接收特定应用程序事件，我们可以根据需要处理这些事件。事件用于在松散耦合的组件之间交换信息。由于发布者和订阅者之间没有直接耦合，因此可以在不影响发布者的情况下修改订阅者，反之亦然。

让我们看看如何在 Spring Boot 应用程序中创建、发布和侦听自定义事件。



##  创建ApplicationEvent

我们可以使用 Spring Framework 的事件发布机制发布应用程序事件。

让我们通过扩展来创建调用的自定义事件：

```java
class UserCreatedEvent extends ApplicationEvent {
  private String name;

  UserCreatedEvent(Object source, String name) {
    super(source);
    this.name = name;
  }
  ...
}
```

代码中super(source)中的source应该是最初发生事件的对象或与事件相关联的对象。

**从Spring 4.2开始，我们还可以将对象发布为事件，而无需扩展ApplicationEvent：**

```java
class UserRemovedEvent {
  private String name;

  UserRemovedEvent(String name) {
    this.name = name;
  }
  ...
}
```



## 发布一个ApplicationEvent

我们使用ApplicationEventPublisher接口发布事件：

```java
@Component
class Publisher {
  
  private final ApplicationEventPublisher publisher;
    
    Publisher(ApplicationEventPublisher publisher) {
      this.publisher = publisher;
    }

  void publishEvent(final String name) {
    // Publishing event created by extending ApplicationEvent
    publisher.publishEvent(new UserCreatedEvent(this, name));
    // Publishing an object as an event
    publisher.publishEvent(new UserRemovedEvent(name));
  }
}
```

当我们发布的对象不是ApplicationEvent时，Spring会自动为我们将其包装在PayloadApplicationEvent中。



## 接收应用程序事件

现在，我们知道如何创建和发布自定义事件，让我们看看如何侦听该事件。事件可以有多个侦听器并且根据应用程序要求执行不同的工作。

有两种方法可以定义侦听器。我们可以使用注解（@EventListener）或实现接口（ApplicationListener）。在这两种情况下，侦听器类都必须由 Spring 管理。

### 注解

从Spring 4.1开始，可以使用@EventListener注解的方法，以自动注册与该方法签名匹配的ApplicationListener：

```java
@Component
class UserRemovedListener {

  @EventListener
  ReturnedEvent handleUserRemovedEvent(UserRemovedEvent event) {
    // handle UserRemovedEvent ...
    return new ReturnedEvent();
  }

  @EventListener
  void handleReturnedEvent(ReturnedEvent event) {
        // handle ReturnedEvent ...
  }
  ...
}
```

启用注解驱动的配置时，不需要其他配置。我们的方法可以监听多个事件，或者如果我们想完全不使用任何参数来定义它，那么事件类型也可以在注解本身上指定。示例：@EventListener（{ContextStartedEvent.class，ContextRefreshedEvent.class}）。

对于使用@EventListener注解并定义为具有返回类型的方法，Spring会将结果作为新事件发布给我们。在上面的示例中，第一个方法返回的ReturnedEvent将被发布，然后由第二个方法处理。

如果指定SpEL条件，Spring仅在某些情况下才允许触发我们的侦听器：

```java
@Component
class UserRemovedListener {

  @EventListener(condition = "#event.name eq 'reflectoring'")
  void handleConditionalListener(UserRemovedEvent event) {
    // handle UserRemovedEvent
  }
}
```

仅当表达式的计算结果为true或以下字符串之一时才处理该事件：**“ true”，“ on”，“ yes”或“ 1”**。方法参数通过其名称公开。条件表达式还公开了一个“ root”变量，该变量引用原始ApplicationEvent（＃root.event）和实际方法参数（＃root.args）

在以上示例中，仅当＃event.name的值为'reflectoring'时，才会使用UserRemovedEvent触发监听器。

### 实现ApplicationListener接口

侦听事件的另一种方法是实现ApplicationListener接口：

```java
@Component
class UserCreatedListener implements ApplicationListener<UserCreatedEvent> {

  @Override
  public void onApplicationEvent(UserCreatedEvent event) {
    // handle UserCreatedEvent
  }
}
```

只要侦听器对象在Spring应用程序上下文中注册，它就会接收事件。当Spring路由一个事件时，它使用侦听器的签名来确定它是否与事件匹配。

## 异步事件侦听器

默认情况下，spring事件是同步的，这意味着发布者线程将阻塞，直到所有侦听器都完成对事件的处理为止。

要使事件侦听器以异步模式运行，我们要做的就是在该侦听器上使用@Async注解：

```java
@Component
class AsyncListener {

  @Async
  @EventListener
  void handleAsyncEvent(String event) {
    // handle event
  }
}
```

为了使@Async注解起作用，我们还必须使用@EnableAsync注解我们的@Configuration类之一或@SpringBootApplication类。

上面的代码示例还显示了我们可以将String用作事件。使用风险自负。最好使用特定于我们用例的数据类型，以免与其他事件冲突。

## Transaction-绑定事件

Spring允许我们将事件侦听器绑定到当前事务的某个阶段。如果当前事务的结果对侦听器很重要时，这使事件可以更灵活地使用。

当我们使用@TransactionalEventListener注释方法时，我们将获得一个扩展的事件侦听器，该侦听器可以了解事务：

```java
@Component
class UserRemovedListener {

  @TransactionalEventListener(phase=TransactionPhase.AFTER_COMPLETION)
  void handleAfterUserRemoved(UserRemovedEvent event) {
    // handle UserRemovedEvent
  }
}
```

仅当当前事务完成时才调用UserRemovedListener。

我们可以将侦听器绑定到事务的以下阶段：

AFTER_COMMIT：事务成功提交后，将处理该事件。如果事件侦听器仅在当前事务成功时才运行，则可以使用此方法。

AFTER_COMPLETION：事务提交或回滚时将处理该事件。例如，我们可以使用它在事务完成后执行清理。

AFTER_ROLLBACK：事务回滚后将处理该事件。

BEFORE_COMMIT：该事件将在事务提交之前进行处理。例如，我们可以使用它来将事务性ORM会话刷新到数据库。

##  Spring Boot的 Application Events

Spring Boot提供了几个与SpringApplication生命周期相关的预定义ApplicationEvent。

在创建ApplicationContext之前会触发一些事件，因此我们无法将这些事件注册为@Bean。我们可以通过手动添加侦听器来注册这些事件的侦听器：

```java
@SpringBootApplication
public class EventsDemoApplication {

  public static void main(String[] args) {
    SpringApplication springApplication = 
        new SpringApplication(EventsDemoApplication.class);
    springApplication.addListeners(new SpringBuiltInEventsListener());
    springApplication.run(args);
  }
}
```

通过将META-INF/spring.factories文件添加到我们的项目中，我们还可以注册侦听器，而不管如何创建应用程序，并使用org.springframework.context.ApplicationListener键引用侦听器：

org.springframework.context.ApplicationListener= com.reflectoring.eventdemo.SpringBuiltInEventsListener

```java
class SpringBuiltInEventsListener 
    implements ApplicationListener<SpringApplicationEvent>{

  @Override
  public void onApplicationEvent(SpringApplicationEvent event) {
    // handle event
  }
}
```

确定事件监听器已正确注册后，便可以监听所有Spring Boot的SpringApplicationEvents。让我们按照它们在应用程序启动过程中的执行顺序来进行观察。

### ApplicationStartingEvent

除了运行侦听器和初始化程序的注册之外，ApplicationStartingEvent在运行开始时但在任何处理之前都会触发。

### ApplicationEnvironmentPreparedEvent

当上下文中使用的环境可用时，将触发ApplicationEnvironmentPreparedEvent。

由于此时环境已准备就绪，因此我们可以在其他Bean使用它之前对其进行检查和修改。

### ApplicationContextInitializedEvent

当ApplicationContext准备就绪并且调用ApplicationContextInitializers但尚未加载bean定义时，将触发ApplicationContextInitializedEvent。

在bean初始化到Spring容器之前，我们可以使用它来执行任务。

### ApplicationPreparedEvent

准备好ApllicationContext但未刷新时会触发ApplicationPreparedEvent。

该环境已准备就绪，可以使用，并且将加载Bean定义。

### WebServerInitializedEvent

如果我们使用的是网络服务器，则在网络服务器准备就绪后会触发WebServerInitializedEvent。 ServletWebServerInitializedEvent和ReactiveWebServerInitializedEvent分别是servlet和反应式网络服务。

**WebServerInitializedEvent不扩展SpringApplicationEvent。**

### ApplicationStartedEvent

在刷新上下文之后但在调用任何应用程序和命令行运行程序之前，将触发ApplicationStartedEvent。

###  ApplicationReadyEvent

触发ApplicationReadyEvent来指示该应用程序已准备就绪，可以处理请求。

建议此时不要修改内部状态，因为所有初始化步骤都将完成。

### ApplicationFailedEvent

如果存在异常并且应用程序无法启动，则会触发ApplicationFailedEvent。在启动期间的任何时间都可能发生这种情况。

我们可以使用它来执行一些任务，例如执行脚本或在启动失败时发出通知。

## 结论

事件是为在同一应用程序上下文内的Spring Bean之间进行简单通信而设计的。从Spring 4.2开始，基础结构已得到显着改进，并提供了基于注释的模型以及发布任意事件的功能。



英文原文：https://reflectoring.io/spring-boot-application-events-explained/



---

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152551.jpg)


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

