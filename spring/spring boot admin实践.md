[返回上层](index)
# spring boot admin实践
## Over View
[上一篇文章](https://juejin.im/post/5eba0439e51d454d9d3eda82)主要介绍了Spring Boot Admin的概况以及我们如何在系统中引入和使用Spring Boot Admin，以此来帮助我们更加了解自己的系统，做到能快速发现、排查问题。本篇文章将用代码演示Spring Boot Admin的消息通知功能，并利用这个开箱即用的特性来个性化我们的需求，优化我们在服务治理方面的工作效率。

Spring Boot Admin内置了多种开箱即用的系统通知渠道，包括邮件、Slack、Telegram、Hipchat等多种社交媒体的通知渠道。但是考虑到它所支持的大都是一些国外的主流社交媒体，在国内的本地化可能并不是那么的友好。不过没关系Spring Boot Admin也提供了通用的接口，使得用户可以基于他所有提供的接口来自定义通知方式。下面使用Spring Boot Admin的通知功能来实现基于邮件和国内办公软件“飞书”的服务健康预警。

---
## 邮件预警
### 依赖引入
在Spring Boot Admin的服务端项目中引入邮件相关依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```
### 添加配置
添加Spring Mail相关配置，我们配置好我们邮箱的Smtp服务器相关信息
```properties
spring.mail.host=your email smtp server
spring.mail.password=your password
spring.mail.port=your email smtp server port
spring.mail.test-connection=true
spring.mail.username=837718548@qq.com 
```
添加Spring Boot Admin（SBA）中相关的邮件配置，以下是SBA官方提供的邮件相关参数

| Property name                                       | Description                                                  | Default value                                                |
| :-------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| spring.boot.admin.notify.mail.enabled               | Enable mail notifications                                    | true                                                         |
| spring.boot.admin.notify.mail.ignore-changes        | Comma-delimited list of status changes to be ignored. Format: "<from-status>:<to-status>". Wildcards allowed. | "UNKNOWN:UP"                                                 |
| spring.boot.admin.notify.mail.template              | Resource path to the Thymeleaf template used for rendering.  | "classpath:/META-INF/spring-boot-admin-server/mail/status-changed.html" |
| spring.boot.admin.notify.mail.to                    | Comma-delimited list of mail recipients                      | "root@localhost"                                             |
| spring.boot.admin.notify.mail.cc                    | Comma-delimited list of carbon-copy recipients               |                                                              |
| spring.boot.admin.notify.mail.from                  | Mail sender                                                  | "Spring Boot Admin <noreply@localhost>"                      |
| spring.boot.admin.notify.mail.additional-properties | Additional properties which can be accessed from the template |                                                              |

我们这里使用如下配置
```properties
spring.boot.admin.notify.mail.from=837718548@qq.com
spring.boot.admin.notify.mail.ignore-changes=""
spring.boot.admin.notify.mail.to=目标邮箱
```
配置中的ignore-changes参数表示服务从一个状态变成其他状态时发出预警，例如："UNKNOWN:UP" 表示服务从未知状态变成UP时，发出通知。当其值是""时，表示任何状态变更都会发出预警。若想指定其他参数，参考上面的参数表。
完成上述操作后，重启Spring Boot Admin服务端，当客户端服务注册进来并且状态变为UP时，我们可以收到一封邮件：
![image-20200514220500095](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152859.jpg)

### 添加邮件模版
Spring Boot admin发送的邮件可以自定义模板样式，我们使用thymeleaf语法编写邮件模板，示例模板代码可参考本文在Github的[代码示例仓库](https://github.com/cg837718548/sba-server-demo/tree/master/src/main/resources/templates)，编写完模板文件之后，将文件放入项目src/main/resources/templates中，并且在配置文件中增加指定模板文件的地址：
```properties
spring.boot.admin.notify.mail.template=classpath:/templates/status-changed.html
```
重启Spring Boot Admin服务端，当客户端服务注册进来并且状态变为UP时，我们可以收到一封邮件，如下是我们对邮件进行本地化之后的样式：
![image-20200514220449191](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152852.jpg)

---
## 飞书预警
由于Spring Boot Admin内置的通知渠道都是国外的社交媒体，不过它也提供了自定义通知渠道的接口，所以我们很容易就可以自定义通知渠道，下面演示集成办公软件飞书的通知。
### 获取通知地址
飞书中提供了聊天机器人，我们只需调用机器人的WebHook就可以实现详细的推送（企业微信，钉钉也具有类似功能）。
![image-20200514232216500](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152843.jpg)

### 自定义通知渠道
Spring Boot Admin中提供了一个AbstractStatusChangeNotifier抽象类，我们可以通过继承它来自定义通知渠道
```java
public class FlyBookNotifier extends AbstractStatusChangeNotifier {

    private static final String DEFAULT_MESSAGE = "#{instance.registration.name} (#{instance.id}) 状态发生转变 #{lastStatus} ➡️ #{instance.statusInfo.status} " +
            "\n" +
            "\n 实例详情：#{instanceEndpoint}";

    private final SpelExpressionParser parser = new SpelExpressionParser();

    private RestTemplate restTemplate;

    private URI webhookUrl;

    private Expression message;

    public FlyBookNotifier(InstanceRepository repository, RestTemplate restTemplate) {
        super(repository);
        this.restTemplate = restTemplate;
        this.message = parser.parseExpression(DEFAULT_MESSAGE, ParserContext.TEMPLATE_EXPRESSION);
    }

    @Override
    protected Mono<Void> doNotify( InstanceEvent event,  Instance instance) {
        if (webhookUrl == null) {
            return Mono.error(new IllegalStateException("'webhookUrl' must not be null."));
        }
        return Mono
                .fromRunnable(() -> restTemplate.postForEntity(webhookUrl, createMessage(event, instance), Void.class));
    }

    public void setRestTemplate(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    protected Object createMessage(InstanceEvent event, Instance instance) {
        Map<String, Object> messageJson = new HashMap<>();
        messageJson.put("title", "👹警告&👼提醒");
        messageJson.put("text", getText(event, instance));

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        return new HttpEntity<>(messageJson, headers);
    }


    protected String getText(InstanceEvent event, Instance instance) {
        Map<String, Object> root = new HashMap<>();
        root.put("event", event);
        root.put("instance", instance);
        root.put("instanceEndpoint", instance.getEndpoints().toString());
        root.put("lastStatus", getLastStatus(event.getInstance()));
        StandardEvaluationContext context = new StandardEvaluationContext(root);
        context.addPropertyAccessor(new MapAccessor());
        return message.getValue(context, String.class);
    }


    public URI getWebhookUrl() {
        return webhookUrl;
    }

    public void setWebhookUrl(URI webhookUrl) {
        this.webhookUrl = webhookUrl;
    }

    public String getMessage() {
        return message.getExpressionString();
    }

    public void setMessage(String message) {
        this.message = parser.parseExpression(message, ParserContext.TEMPLATE_EXPRESSION);
    }
}

```
上面代码是一个示例，用户可以根据自己的需求来自定义消息体的格式和内容。
随后我们在Spring中创建该通知类的bean
```java
@Configuration
public static class NotifierConfiguration {
    @Bean
    @ConditionalOnMissingBean
    @ConfigurationProperties("spring.boot.admin.notify.flybook")
    public FlyBookNotifier flyBookNotifier(InstanceRepository repository) {
        return new FlyBookNotifier(repository, new RestTemplate());
    }
}
```
最后我们在项目的配置文件中添加我们飞书渠道的配置信息
```properties
spring.boot.admin.notify.flybook.ignore-changes=""
spring.boot.admin.notify.flybook.webhook-url=https://open.feishu.cn/open-apis/bot/hook...
```
完成上述操作后，重启Spring Boot Admin服务端，当客户端服务注册进来并且状态变为UP时，我们可以在飞书端收到Spring Boot Admin自动推过来的预警信息：
![image-20200514224218724](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152834.jpg)
至此，我们的自定义消息渠道就已经完成。通过继承AbstractStatusChangeNotifier抽象类，我们可以很轻易的自定义自己想要实现的推送渠道（<u>设计模式：模板方法模式</u>)。

---
## 总结
本文主要介绍了Spring Boot Admin中所提供的多种消息预警推送渠道，并且我们可以通过自定义消息预警渠道来满足我们自身的需求，整个过程并不需要耗费太多的人力和时间成本。我们用了两个示例来演示如何实现Spring Boot Admin的消息预警功能，分别是邮件预警和自定义的飞书预警。

**本文的示例代码**
SBA-client：https://github.com/cg837718548/sba-client-demo.git
SBA-server：https://github.com/cg837718548/sba-server-demo.git

---

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152827.jpg)
