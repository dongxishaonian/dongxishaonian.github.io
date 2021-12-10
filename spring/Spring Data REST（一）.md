# Spring Data REST不完全指南（一）

## 简介

Spring Data REST是Spring Data项目的一部分，可轻松在Spring Data存储库上构建超媒体驱动的REST Web服务。

Spring Data REST 构建在 Spring 数据存储库之上，分析应用程序的域模型，并公开模型中包含的聚合的超媒体驱动的 HTTP 资源。

### 特征：

- 使用 HAL 媒体类型来公开域模型的 REST API。
- 适用[集合、项目（item）和关联资源](https://docs.spring.io/spring-data/rest/docs/current/reference/html/#repository-resources)表示你的模型。
- 通过链接导航支持分页。
- 允许动态过滤收集资源。
- 通过资源api来暴露你repositories中定义的资源查询方法。
- 允许通过处理Spring ApplicationEvents来处理REST请求。
- 公开有关ALPS和JSON Schema模型的元数据。
- 允许通过投影定义客户特定的表示形式。
- 发布一个定制的HAL浏览器变体以利用公开的元数据。
- 目前支持JPA，MongoDB，Neo4j，Solr，Cassandra，Gemfire。
- 允许对公开的默认资源进行高级自定义。

💡：***目前对Spring Data REST适用分析：快速生成数据库资源对外的接口（适用于一些逻辑简单的数据对外接口）***

---

## 分析

**使用Spring Data REST并实现以下功能来满足日常api的开发过程：**

> ```
> 需要满足的一些要求：
> 1.针对字段级别，方法级别，类级别进行限制（禁止某些字段，方法，接口的对外映射）。
> 2.对数据增删改查的限制（禁止某些请求方法的访问）。
> 3.能个性化定义请求的路径。
> 4.对所传参数进行值校验。
> 5.响应统一处理。
> 6.异常处理。
> 7.数据处理的切面。
> ```

以上列出了我们日常接口开发中比较常见的一些功能需求，这里将演示使用Spring Data REST并结合实现上述功能来快速开发HAL REST API。

---

## 准备

### 条件：

> jdk11
>
> Springboot 2.2.6.RELEASE
>
> maven
>
> Spring Data JPA

### 添加依赖

本文中演示Spring Data JPA结合Spring Data REST

#### 1.添加Spring Data持久层依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
 <dependency>
   <groupId>com.h2database</groupId>
   <artifactId>h2</artifactId>
</dependency>
```

目前Spring Data REST支持JPA，MongoDB，Neo4j，Solr，Cassandra，Gemfire,所以使用时可根据自己的需求引入不同的Spring Data依赖，本文将使用JPA作为演示。

#### 2.添加Spring Data REST相关依赖

```xml
<!--Spring Data REST-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
<!--Spring Data REST 数据浏览工具-->
<dependency>
  <groupId>org.springframework.data</groupId>
  <artifactId>spring-data-rest-hal-browser</artifactId>
</dependency>
```

---

## 简单尝试

示例中将用一个简单的租客系统来做演示。

**创建一个房子类**

```java
@Entity
@Data
@Accessors(chain = true)
public class House {
    @Id
    @GeneratedValue
    private Long id;

    private String houseNumber;

    private String owner;

    private String idCard;

    public House() {
    }

    public House(String houseNumber, String owner, String idCard) {
        this.houseNumber = houseNumber;
        this.owner = owner;
        this.idCard = idCard;
    }
}
```

**创建一个租客类**

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

此时，我们新建一个租客的Reopsitory

```java
public interface TenantRepository extends CrudRepository<Tenant, Long> {
    Page<Tenant> findAllByNameContaining(String name, Pageable page);

    Page<Tenant> findAllByIdCardContaining(String idCard, Pageable page);

    Tenant findFirstByMobile(String mobile);

    Tenant findFirstByIdCard(String idCard);
}

```

运行前我们准备好初始化数据

```java
@SpringBootApplication
public class SpringDataRestDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringDataRestDemoApplication.class, args);
    }

    @Resource
    private TenantRepository tenantRepository;

    @PostConstruct
    public void initRepo() {
        //准备房子信息
        List<House> houses = new ArrayList<>();
        House zhangsan = new House("1101", "张三", "330521******1");
        House zhangsi = new House("1102", "张四", "330521******2");
        House zhangwu = new House("1103", "张五", "330521******3");
        House zhangliu = new House("1104", "张六", "330521******4");
        House zhangqi = new House("1105", "张七", "330521******5");
        House zhangba = new House("1106", "张八", "330521******6");
        //准备租客信息
        List<Tenant> tenants = new ArrayList<>();
        tenants.add(new Tenant("王一", "330522******1", "186****3331", LocalDateTime.now().minusDays(1), zhangsan));
        tenants.add(new Tenant("王二", "330522******2", "186****3332", LocalDateTime.now().minusDays(2), zhangsi));
        tenants.add(new Tenant("王三", "330522******3", "186****3333", LocalDateTime.now().minusDays(3), zhangwu));
        tenants.add(new Tenant("王四", "330522******4", "186****3334", LocalDateTime.now().minusDays(4), zhangliu));
        tenants.add(new Tenant("王五", "330522******5", "186****3335", LocalDateTime.now().minusDays(5), zhangqi));
        tenants.add(new Tenant("王六", "330522******6", "186****3336", LocalDateTime.now().minusDays(6), zhangba));
        tenantRepository.saveAll(tenants);
    }
}

```

启动项目，并且访问localhost:8080,如下图：

![image-20200420130723041](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-151951.jpg)

上图是Spring Data REST的HAL数据浏览器，通过它能高效的查询和调试Spring Data REST对外提供的接口。

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-151945.jpg)

我们可以看到响应内容的格式，正是符合HAL类型的格式。

访问http://localhost:8080/tenants/search

![image-20200420133931527](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-151939.jpg)

上图可以看到，Spring Data REST对外暴露了我们在Repository中定义的查询方法，并且可以看到response Body中数据格式符合HAL格式类型，通过HAL格式的响应数据，我们轻松就能知道这些查询方法对应的请求路径，起到了一个文档的作用，使api能够自发现。

---

总结

本文初步的介绍了Spring Data REST的功能及特征，并且演示了如何在项目中引入Spring Data REST，并结合Spring Data REST实现了简单的演示Demo。**下一篇文章将介绍并演示如何在Spring Data REST中实现一些必要的功能，以此来满足我们日常的接口开发工作。**

本文代码示例：https://gitee.com/jeker8chen/spring-data-rest-in-practice.git



---

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-151932.jpg)