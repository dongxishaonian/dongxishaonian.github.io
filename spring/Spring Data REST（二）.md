[返回上层](index)
# Spring Data REST不完全指南（二）

上一篇文章介绍了Spring Data REST的功能及特征，以及演示了如何在项目中引入Spring Data REST并简单地启动演示了Spring Data REST项目。在本文中，我们将深入了解Spring Data REST的特性，以此来满足我们日常api开发工作的要求。

如果仅仅是上一篇文章中对Spring Data REST的使用，那无法做到在日常开发中使用Spring Data REST,所以在上一篇文章中，我们列出了日常api开发中的一些必要功能：

> **需要满足的一些要求：**
> **1.针对字段级别，方法级别，类级别进行限制（禁止某些字段，方法，接口的对外映射）。**
> **2.对数据增删改查的限制（禁止某些请求方法的访问）。**
> **3.能个性化定义请求的路径。**
> **4.对所传参数进行值校验。**
> **5.响应统一处理。**
> **6.异常处理。**
> **7.数据处理的切面。**

➡️**本文，将演示这些要求中的前三个要求。**

---

## 针对接口级别，方法级别，字段级别进行访问限制

所谓的访问限制，这里我们的目的是指定某些资源不对外暴露，Spring Data REST使用注解来实现各级别的访问限制。

### 接口级别的访问限制：

```java
@RepositoryRestResource(exported = false)
public interface TenantRepository extends CrudRepository<Tenant, Long> {
    Page<Tenant> findAllByNameContaining(String name, Pageable page);

    Page<Tenant> findAllByIdCardContaining(String idCard, Pageable page);

    Tenant findFirstByMobile(String mobile);

    Tenant findFirstByIdCard(String idCard);
}
```

Spring Data REST中我们在接口级别增加`@RepositoryRestResource(exported = false)`来实现接口及接口中的所有方法不对外暴露，从而限制访问。

### 方法级别的访问限制：

```java
public interface TenantRepository extends CrudRepository<Tenant, Long> {
    Page<Tenant> findAllByNameContaining(String name, Pageable page);

    Page<Tenant> findAllByIdCardContaining(String idCard, Pageable page);

    Tenant findFirstByMobile(String mobile);

    @RestResource(exported = false)
    Tenant findFirstByIdCard(String idCard);
}
```

上述代码中，我们使用`@RestResource(exported = false)`注解在指定的方法上，从而实现该方法不对外暴露。

### 字段级别的访问限制

```java
@Entity
@Data
@Accessors(chain = true)
public class Tenant {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    //隐私信息不需要暴露
    @JsonIgnore
    private String idCard;

    private String mobile;

    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime rentDateTime;

    @OneToOne(cascade = CascadeType.ALL, orphanRemoval = true)
    private House house;

    public Tenant() {
    }

    public Tenant(String name, String idCard, String mobile, LocalDateTime rentDateTime, House hous) {
        this.name = name;
        this.idCard = idCard;
        this.mobile = mobile;
        this.rentDateTime = rentDateTime;
        this.house = hous;
    }
}
```

在上述代码中，我们在idCard字段上增加了`@JsonIgnore`注解，从而实现该字段不对外暴露。

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152117.jpg)

如上图，我们在HAL Browser中看到，输出的租客数据中不再包含idCard字段。

### 使用Projections和Excerpts来实现访问限制

```java
@Projection(name = "mobileAndName", types = {Tenant.class})
public interface TenantProjection {
    String getName();

    String getMobile();
}
```

如上，首先我们声明一个投影（Projection），在上面的投影中，我们使用指定字段的get方法来对外暴露需要暴露的字段name,mobile。

此时，我们访问HAL Browser

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152008.jpg)

可以看到Spring Data REST此时提供了一个投影的链接。

此时我们查询指定租客类的投影[http://localhost:8080/tenants/1?projection=mobileAndName](http://localhost:8080/tenants/1?projection=mobileAndName)

```json
{
  "name": "王一",
  "mobile": "186****3331",
  "_links": {
    "self": {
      "href": "http://localhost:8080/tenants/1"
    },
    "tenant": {
      "href": "http://localhost:8080/tenants/1{?projection}",
      "templated": true
    }
  }
}
```

可以看到只显示了我们投影的字段。

⚠️：我们声明的投影接口需要和数据类在同一个包中。

⚠️：否则，我们需要增加配置类，来告诉Spring Data REST投影接口的位置，如下图

```java
@Component
public class SpringDataRestCustomization implements RepositoryRestConfigurer {
    @Override
    public void configureRepositoryRestConfiguration(RepositoryRestConfiguration config) {
        config.getProjectionConfiguration().addProjection(TenantProjection.class);
    }
```

#### 数据内联

有时候，我们可能需要单独对House类进行数据操作，所以我们会声明一个HouseRepository

```java
public interface HouseRepository extends CrudRepository<House, Long> {
}

```

此时，我们访问某一租客数据（[http://localhost:8080/tenants/1](http://localhost:8080/tenants/1)）时，

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152013.jpg)

此时，house的数据就不会内联在Tenant里面。但是我们并不想要这种效果，我们希望house还是内联在Tenant中的。我们可以使用Projection来解决此问题。

```java
@Projection(name = "NameAndHouse", types = {Tenant.class})
public interface OnlyNameProjection {
    String getName();

    House getHouse();
}
```

如上，我们声明了一个NameAndHouse投影。然后我们使用`@RepositoryRestResource(excerptProjection = OnlyNameProjection.class)`注解放在TenantRepository接口上。

```java
@RepositoryRestResource(excerptProjection = OnlyNameProjection.class)
public interface TenantRepository extends CrudRepository<Tenant, Long> {
	...
}
```

此时，我们访问NameAndHouse投影([http://localhost:8080/tenants/1?projection=NameAndHouse](http://localhost:8080/tenants/1?projection=NameAndHouse))

```json
{
  "name": "王一",
  "house": {
    "houseNumber": "1101",
    "owner": "张三",
    "idCard": "330521******1"
  },
  "_links": {
    "self": {
      "href": "http://localhost:8080/tenants/1"
    },
    "tenant": {
      "href": "http://localhost:8080/tenants/1{?projection}",
      "templated": true
    },
    "house": {
      "href": "http://localhost:8080/tenants/1/house"
    }
  }
}
```

可以看到，House已经被内联进Tenant中。

---

## 对数据增删改查的限制

Spring Data REST提供了对资源请求的限制，比如对特定请求方法的限制，对特定资源访问的限制。

### Repository对外暴露限制

有时候我们希望，我们在Repository中定义的某些数据操作方法不对外暴露。Spring Data REST提供了了四个级别的资源限制级别：

> - **ALL**：公开所有Spring Data存储库，无论其Java可见性或注释配置如何。
> - **DEFAULT**：公开公共Spring数据存储库或使用`@RepositoryRestResource`显式注释的存储库，并且其导出属性未设置为false。
> - **VISIBILITY**：无论注释配置如何，仅公开公共Spring Data存储库。
> - **ANNOTATED**：仅公开使用`@RepositoryRestResource`显式注释的Spring Data存储库，并且其导出属性未设置为false。

如下代码实现了**ANNOTATED**级别的限制：

```java
@Component
public class SpringDataRestCustomization implements RepositoryRestConfigurer {
    @Override
    public void configureRepositoryRestConfiguration(RepositoryRestConfiguration config) { config.setRepositoryDetectionStrategy(RepositoryDetectionStrategy.RepositoryDetectionStrategies.ANNOTATED);
    }
}
```

### 增删改查限制

**在RESTful中定义：**

**GET（GET方法返回单个实体）**

**PUT（PUT方法用提供的请求主体替换目标资源的状态（存在则修改，不存在则新建）。）**

**PATCH（PATCH方法类似于PUT方法，但是部分更新资源状态。）**

**DELETE（删除信息）**

所以所谓的对增删改查的限制实际上就是对请求方法的限制。

如下，我们对Tenant类进行了两个操作

1. PUT操作禁止新增，但可以修改。
2. DELETE限制，也就是限制了删除操作。

```java
@Component
public class SpringDataRestCustomization implements RepositoryRestConfigurer {
    @Override
    public void configureRepositoryRestConfiguration(RepositoryRestConfiguration config) {
        ExposureConfiguration exposureConfiguration = config.getExposureConfiguration();
        exposureConfiguration.forDomainType(Tenant.class)
                .disablePutForCreation()
                .withItemExposure((metadata, httpMethods) -> httpMethods.disable(HttpMethod.DELETE));
    }
}
```

如下为请求测试：

1.禁止DELETE请求

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152102.jpg)

2.禁止PUT的新增操作

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152056.jpg)

3.允许PUT的修改操作

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152050.jpg)

---

## 个性化定义请求的路径

Spring Data REST提供了个性化请求路径的功能

### 自定义项目资源URI

默认情况下，项目资源的URI包含用于集合资源的路径段，并附加了数据库标识符。这样一来，您就可以使用存储库的findOne（…）方法来查找实体实例。从Spring Data REST 2.5开始，可以通过使用RepositoryRestConfiguration上的配置API（在Java 8上首选）或通过将EntityLookup的实现注册为应用程序中的Spring bean来自定义此属性。 Spring Data REST会选择它们并根据其实现来调整URI生成。

```java
@Component
public class SpringDataRestCustomization implements RepositoryRestConfigurer {
    @Override
    public void configureRepositoryRestConfiguration(RepositoryRestConfiguration config) {
        config.withEntityLookup()
                .forRepository(TenantRepository.class)
                .withIdMapping(Tenant::getMobile)
                .withLookup(TenantRepository::findFirstByMobile);
    }
}
```

如上，我们个性化地指定了Tenant的标识符为mobile字段，此时我们可以通过HAL Browser来看到路径中的标识符变成了mobile字段。

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152042.jpg)

### 配置REST URL路径

我们使用`@RepositoryRestResource`和`@RestResource`注解直接指定资源在路径中的名字。

如下：

```java
@RepositoryRestResource(path = "tenantPath")
public interface TenantRepository extends CrudRepository<Tenant, Long> {
    @RestResource(path = "mobile")
    Tenant findFirstByMobile(String mobile);
}
```

此时我们通过HAL Browser访问此资源：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152036.jpg)

可以看到，被我们注解的findFirstByMobile对应的资源路径已经被定制化。

另外我们看到"links"中的属性名还是findFirstByMobile，我们可以通过`@RestResource(path = "mobile",rel = "mobile")`来个性化指定其名字，例如此注解中，我们指定了"links"中的findFirstByMobile属性名为mobile。

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152030.jpg)

---

## 总结

本文列出了接口开发常用的7个功能，并且演示如何实现**针对接口级别，方法级别，字段级别进行访问限制**，我们使用`@RepositoryRestResource`，`@RestResource`，`@JsonIgnore`分别实现接口，方法，字段级别的访问限制，并且我们利用了Projections和Excerpts来实现自定义数据格式。我们还是实现了对**数据增删改查的限制**，我们通过RepositoryDetectionStrategy的四个级别来控制数据接口的对外暴露，使用ExposureConfiguration来限制某些资源对特定请求方式的限制。最后我们使用RepositoryRestConfiguration以及`@RepositoryRestResource`和`@RestResource`注解实现了**个性化定义请求的路径**。



本文代码示例：[https://gitee.com/jeker8chen/spring-data-rest-in-practice.git](https://gitee.com/jeker8chen/spring-data-rest-in-practice.git)



---

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-152021.jpg)


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

