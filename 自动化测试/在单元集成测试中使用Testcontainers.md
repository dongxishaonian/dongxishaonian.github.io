[返回上层](index)
# java单元/集成测试中使用Testcontainers

## 1.Testcontainers介绍：

Testcontainers是一个Java库，它支持JUnit测试，提供公共数据库、SeleniumWeb浏览器或任何可以在Docker容器中运行的轻量级、一次性实例。

**测试容器使以下类型的测试更加容易：**

**数据访问层集成测试：**

使用MySQL，PostgreSQL或Oracle数据库的容器化实例测试您的数据访问层代码，但无需在开发人员的计算机上进行复杂的设置，并且测试将始终从已知的数据库状态开始，避免“垃圾”数据的干扰。也可以使用任何其他可以容器化的数据库类型。

**应用程序集成测试：**

用于在具有相关性（例如数据库，消息队列或Web服务器）的短期测试模式下运行应用程序。

**UI /验收测试：**

使用与Selenium兼容的容器化Web浏览器进行自动化UI测试。每个测试都可以获取浏览器的新实例，而无需担心浏览器状态，插件版本或浏览器自动升级。您将获得每个测试会话或测试失败的视频记录。

**更多：**

可以签出各种贡献的模块，或使用 [GenericContainer](https://www.testcontainers.org/features/creating_container/)作为基础创建自己的自定义容器类。

---

## 2.Testcontainers实践示例：

Testcontainers提供了多种现成的与测试关联的应用程序容器，如下图：

![image-20200407113553146](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112633.png)



在本文中，将演示集成postgresql容器和mockserver容器的测试。

> **Testcontainers必要条件：**
>
> 1.Docker
>
> 2.支持的JVM测试框架：JUnit4，JUnit5，spock...



### 2.1 集成postgresql测试

#### **依赖：**

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.12.5</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
  	<!--指定数据库名称，mysql，mariadb等等-->
    <artifactId>postgresql</artifactId>
    <version>1.12.5</version>
    <scope>test</scope>
</dependency>
```

#### **配置：**

在项目的src/test/resources/application.yml文件中配置postgresql相关信息

```properties
#将驱动程序设置为org.testcontainers.jdbc.ContainerDatabaseDriver，它是一个Testcontainers JDBC代理驱动程序。初始化数据源时，此驱动程序将负责启动所需的Docker容器。
spring.datasource.driver-class-name=org.testcontainers.jdbc.ContainerDatabaseDriver

#将JDBC URL设置为JDBC:tc:<database image>：<version>：///以便Testcontainers知道要使用哪个数据库。
#TC_INITSCRIPT=指定的数据库初始化的脚本文件位置
spring.datasource.url=jdbc:tc:postgresql:9.6:///?TC_INITSCRIPT=file:src/main/resources/init_db.sql

#将方言明确设置为数据库的方言实现，否则在启动应用程序时会收到异常。当您在应用程序中使用JPA时（通过Spring Data JPA），此步骤是必需的
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQL9Dialect
```

#### **测试示例：**

为了在@DataJpaTest中使用TC，您需要确保使用了应用程序定义的（自动配置的）数据源。您可以通过使用@AutoConfigureTestDatabase注释测试来轻松完成此操作，如下所示：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class OwnerRepositoryTests {

    @Autowired
    private OwnerRepository ownerRepository;

    @Test
    void findAllReturnsJohnDoe() { // as defined in tc-initscript.sql
        var owners = ownerRepository.findAll();
        assertThat(owners.size()).isOne();
        assertThat(owners.get(0).getFirstName()).isEqualTo("John");
        assertThat(owners.get(0).getLastName()).isEqualTo("Doe");
    }
}
```

**以上测试将使用Testcontainers提供的postgresql容器进行测试，从而排除了外部环境对测试的干扰。**

当需要用本地数据库进行集成测试时，我们只要使用@SpringBootTest替换如上两个注解即可：

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
public class OwnerResourceTests {

    @Autowired
    WebApplicationContext wac;

    @Test
    void findAllReturnsJohnDoe() throws Exception {
        given()
               .webAppContextSetup(wac)
        .when()
                .get("/owners")
        .then()
                .status(HttpStatus.OK)
                .body(
                        "_embedded.owners.firstName", containsInAnyOrder("John"),
                        "_embedded.owners.lastName", containsInAnyOrder("Doe")
                );
    }
}
```

**以上测试将使用真实运行环境的数据库进行测试。**

---

### 2.2 集成mockServer测试

Mock Server可用于通过将请求与用户定义的期望进行匹配来模拟HTTP服务。

#### **依赖：**

```xml
				<dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>mockserver</artifactId>
            <version>1.12.5</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.mock-server</groupId>
            <artifactId>mockserver-netty</artifactId>
            <version>5.5.4</version>
        </dependency>
        <dependency>
            <groupId>org.mock-server</groupId>
            <artifactId>mockserver-client-java</artifactId>
            <version>5.5.4</version>
        </dependency>
```

#### **测试示例：**

```java
//创建一个MockServer容器
@Rule
public MockServerContainer mockServer = new MockServerContainer();
```

以及使用Java MockServerClient设置简单的期望。

```java
new MockServerClient(mockServer.getContainerIpAddress(), mockServer.getServerPort())
                .when(request()
                        .withPath("/person")
                        .withQueryStringParameter("name", "peter"))
                .respond(response()
                        .withBody("Peter the person!"));
//...当一个get请求至'/person?name=peter' 时会返回 "Peter the person!"
```

测试(使用restassured进行测试)：

```java
RestAssured.baseURI = "http://" + mockServer.getContainerIpAddress();
RestAssured.port = mockServer.getServerPort();
given().queryParam("name", "peter")
                .get("/person")
                .then()
                .statusCode(HttpStatus.OK.value())
                .body(is("Peter the person!"));
```

完整代码如下：

```java
@RunWith(SpringJUnit4ClassRunner.class)
public class OneTests {
    @Rule
    public MockServerContainer mockServer = new MockServerContainer();

    @Test
    public void v() {
        RestAssured.baseURI = "http://" + mockServer.getContainerIpAddress();
        RestAssured.port = mockServer.getServerPort();

        new MockServerClient(mockServer.getContainerIpAddress(), mockServer.getServerPort())
                .when(request()
                        .withPath("/person")
                        .withQueryStringParameter("name", "peter"))
                .respond(response()
                        .withBody("Peter the person!"));

        given().queryParam("name", "peter")
                .get("/person")
                .then()
                .statusCode(HttpStatus.OK.value())
                .body(is("Peter the person!"));
    }
}
```

---

## 3.总结：

Testcontainers轻松的解决了集成测试时测试代码与本地组件耦合，从而出现各种意外失败的问题（比如本地数据库中存在脏数据影响到了集成测试，多个集成测试同时运行时相互干扰导致测试结果意外失败）。笔者之前专门为集成测试准备了一套数据库，使数据和其他环境隔离掉，但还是会遇到多个集成测试一起跑相互干扰的问题，Testcontainers轻松的解决了笔者的问题。



---
**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**
![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112625.jpg)
