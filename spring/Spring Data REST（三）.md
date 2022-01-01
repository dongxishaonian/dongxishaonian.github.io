[返回上层](index)
# Spring Data REST不完全指南（三）

上一篇我们介绍了使用Spring Data REST时的一些高级特性，以及使用代码演示了如何使用这些高级的特性。本文将继续讲解前面我们列出来的七个高级特性中的后四个。至此，这些特性能满足我们大部分的接口开发场景。

> **需要满足的一些要求：**
> **1.针对字段级别，方法级别，类级别进行限制（禁止某些字段，方法，接口的对外映射）。**
> **2.对数据增删改查的限制（禁止某些请求方法的访问）。**
> **3.能个性化定义请求的路径。**
> **4.对所传参数进行值校验。**
> **5.响应统一处理。**
> **6.异常处理。**
> **7.数据处理的切面。**

➡️**本文，将演示7个要求中的其余四个要求。**

---

## 对所传参数进行值校验

对于值校验，Spring 提供了Validator接口，Spring Data REST提供了使用Validator来进行值校验的功能。

**首先**我们通过实现Validator接口来创建一个校验器，**然后**在实现RepositoryRestConfigurer或Spring Data REST的RepositoryRestConfigurerAdapter的子类的配置中，重写configureValidatingRepositoryEventListener方法，并在ValidatingRepositoryEventListener上调用addValidator，传递要触发此校验器的事件和校验器的实例。以下示例显示了如何执行此操作：

```java
public class SaveTenantValidator implements Validator {
    @Override
    public boolean supports(Class<?> clazz) {
        return Tenant.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        Tenant tenant = (Tenant) target;
        if (StringUtils.isEmpty(tenant.getMobile())) {
            errors.rejectValue("mobile", "1001", "手机号不能为空");
        }
    }
}

```

如上，我们声明了一个Validator类，作为对手机号校验的Validator。接着我们通过以下代码注册我们的校验器。

```java
@Component
public class SpringDataRestCustomization implements RepositoryRestConfigurer {
    @Override
    public void configureValidatingRepositoryEventListener(ValidatingRepositoryEventListener validatingListener) {
        validatingListener.addValidator("beforeCreate", new SaveTenantValidator());
    }
}
```

`validatingListener.addValidator("beforeCreate", new SaveTenantValidator());`我们使用`validatingListener.addValidator()`来注册我们的校验器。该方法传入两个参数，第一个代表着要校验的事件，"beforeCreate"即代表着在插入新纪录之前，对插入数据进行校验。spring Data REST还提供了其他的事件：

- `BeforeCreateEvent`
- `AfterCreateEvent`
- `BeforeSaveEvent`
- `AfterSaveEvent`
- `BeforeLinkSaveEvent`
- `AfterLinkSaveEvent`
- `BeforeDeleteEvent`
- `AfterDeleteEvent`

我们都可以从字面意思进行理解。

方法中的第二个参数，就是指定我们要注册的校验器，如上代码中，我们对我们刚刚创建的校验器进行注册。

如下为验证效果：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152142.jpg)

---

## 响应统一处理

有时候我们需要对响应结果进行统一处理，比如，我们希望我们的响应结果中包含当前时间的时间戳又或者我们希望我们的HAL格式的响应数据中增加其他的链接。这时候，我们可以通过响应统一处理来完成这种看似重复性的工作。但是Spring Data REST并没有提供现成的功能，不过我们可以通过覆盖Spring Data REST响应处理程序，来实现这一目标。

```java
@RepositoryRestController
public class TenantController {
    private final TenantRepository tenantRepository;
		@Resource
    private RepositoryEntityLinks entityLinks;
    @Autowired
    public TenantController(TenantRepository tenantRepository) {
        this.tenantRepository = tenantRepository;
    }

    @GetMapping(value = "/tenantPath/search/mobile")
    public ResponseEntity<?> getByMobile(@RequestParam String mobile) {
        Tenant tenant = tenantRepository.findFirstByMobile(mobile);
        EntityModel<Tenant> resource = new EntityModel<>(tenant);
        resource.add(linkTo(methodOn(TenantController.class).getByMobile(mobile)).withSelfRel());
      resource.add(entityLinks.linkToSearchResource(Tenant.class, LinkRelation.of("findAllByIdCardContaining")));
        return ResponseEntity.ok(resource);
    }
}

```

如上代码，我们使用了`@RepositoryRestController`注解来创建了一个控制器，并定义了一个路径的请求，以此我们覆盖了之前Spring Data REST自动为我们提供的相同路径的接口。我们给接口的响应增加了两个链接。

**注意：上述代码中用到了Spring HATEOAS的库，所以我们需要增加Spring HATEOAS的依赖。**

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
<dependency>
  <groupId>org.atteo</groupId>
  <artifactId>evo-inflector</artifactId>
</dependency>
```

现在我们访问`http://localhost:8080/tenantPath/search/mobile?mobile=186****3331`,看到响应结果：

```json
{
  "name": "王一",
  "mobile": "186****3331",
  "rentDateTime": "2020-04-22 17:48:40",
  "_links": {
    "self": {
      "href": "http://localhost:8080/tenantPath/search/mobile?mobile=186****3331"
    },
    "findAllByIdCardContaining": {
      "href": "http://localhost:8080/tenantPath/search/findAllByIdCardContaining{?idCard,page,size,sort,projection}",
      "templated": true
    }
  }
}
```

可以看到，links属性中链接已经变成我们指定的链接了。

---

## 异常统一处理

Spring Data REST中并没有提供异常处理的功能，但是我们可以使用Springboot中自带的异常处理功能来实现我们的要求。

```java
@Slf4j
@ControllerAdvice
public class ExceptionTranslator {
    @ExceptionHandler
    public ResponseEntity<Object> handleEmailAlreadyUsedException(NullPointerException ex, NativeWebRequest request) {
        log.info("遇到空指针");
        return ResponseEntity.ok(List.of("拦截到空指针异常"));
    }
}
```

如上，我们声明了一个异常处理器。接下来我人为制造一个错误。

```java
@RepositoryRestController
public class TenantController {
    private final TenantRepository tenantRepository;

    @Autowired
    public TenantController(TenantRepository tenantRepository) {
        this.tenantRepository = tenantRepository;
    }

    @GetMapping(value = "/tenantPath/search/mobile")
    public ResponseEntity<?> getByMobile(@RequestParam String mobile) {
      	if (1 == 1) {
           throw new NullPointerException();
        }
        Tenant tenant = tenantRepository.findFirstByMobile(mobile);
        EntityModel<Tenant> resource = new EntityModel<>(tenant);
        resource.add(linkTo(methodOn(TenantController.class).getByMobile(mobile)).withSelfRel());
        return ResponseEntity.ok(resource);
    }
}
```

此时，我们请求此接口：

```json
[
  "拦截到空指针异常"
]
```

可以看到，我们的异常被我们的异常处理器拦截掉了。

---

## 数据切面处理

Spring Data REST提供了类似的Aop切面操作，虽然不能和Spring的原生aop相比，但是其简洁性也能满足需求。Spring Data REST提供的是基于事件的切面。如下我们声明了一个切面。

```java
@Component
@Slf4j
@RepositoryEventHandler
public class TenantEventHandler {
    @HandleBeforeDelete
    protected void onBeforeDelete(Tenant entity) {
        log.info("现在要开始删除操作了，删除对象:{}", entity);
    }
    @HandleAfterDelete
    protected void onAfterDelete(Tenant entity) {
        log.info("删除对象完成，删除对象:{}", entity);
    }

}
```

如上，我们声明了一个切面，我们可以在删除操作之前和之后进行额外的逻辑处理，示例中很简单，我们使用日志记录事件的发生。

此时，我们访问项目的删除接口`curl --location --request DELETE 'http://localhost:8080/tenantPath/1'` 

我们可以看到控制输出了相应的日志：

> 2020-04-23 17:26:29.950  INFO 38077 --- [nio-8080-exec-1] c.e.d.configuration.TenantEventHandler   : 现在要开始删除操作了，删除对象:Tenant(id=1, name=王一, idCard=330522******1, mobile=1863331, rentDateTime=2020-04-22T17:24:46.105897, house=House(id=2, houseNumber=1101, owner=张三, idCard=330521******1))
>
> 2020-04-23 17:26:30.035  INFO 38077 --- [nio-8080-exec-1] c.e.d.configuration.TenantEventHandler   : 删除对象完成，删除对象:Tenant(id=1, name=王一, idCard=330522******1, mobile=1863331, rentDateTime=2020-04-22T17:24:46.105897, house=House(id=2, houseNumber=1101, owner=张三, idCard=330521******1))

此时，我们的数据切面处理生效了，除此之外，Spring Data REST还提供了如下几个基于事件的切面：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152153.jpg)

---

## 总结

至此，我们先前列出的所有功能特性三篇文章中都有涉及到，通过引入这些功能特性，我们能更加轻松的使用Spring Data REST，并且也能满足我们大部分接口开发的场景。当然三篇文章不能涉及Spring Data REST的全部，有兴趣的小伙伴可以访问Spring Data REST的[官方文档](https://docs.spring.io/spring-data/rest/docs/3.2.6.RELEASE/reference/html/#preface)查看更多关于Spring Data REST的特性及信息。



---

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152157.jpg)


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

