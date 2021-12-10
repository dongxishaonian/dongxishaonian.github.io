[返回上层](index)
# Spring中的设计模式
## 1.介绍

设计模式是软件开发的重要组成部分。这些解决方案不仅解决了反复出现的问题，而且还通过识别通用模式来帮助开发人员了解框架的设计。

在本教程中，我们将研究Spring框架中使用的四种最常见的设计模式：

- **单例模式**
- **工厂方法模式**
- **代理模式**
- **模板模式**

我们还将研究Spring如何使用这些模式来减轻开发人员的负担并帮助用户快速执行繁琐的任务。

---

## 2.单例模式

单例模式是一种确保每个应用程序仅存在一个对象实例的机制。在管理共享资源或提供跨领域服务（例如日志记录）时，此模式很有用。

### 2.1 单例beans

通常，单例对于应用程序是全局唯一的，但是在Spring中，此约束更宽泛。Spring定义的单例是在spring IOC容器中唯一。实际上，这意味着Spring只会为每个应用程序上下文的每种类型创建一个bean。

Spring的方法不同于严格的单例定义，因为一个应用程序可以具有多个Spring容器。因此，如果我们有多个容器，则同一类的多个对象可以在单个应用程序中存在。

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-151337.png)

**默认情况下，Spring将所有bean创建为单例。**

### 2.2 自动装配单例对象

例如，我们可以在一个应用程序上下文中创建两个控制器，并将相同类型的bean注入每个控制器中。

首先，我们创建一个BookRepository管理我们的Book域对象。

接下来，我们创建LibraryController，它使用BookRepository返回库中的书数：

```java
@RestController
public class LibraryController {
     
    @Autowired
    private BookRepository repository;
 
    @GetMapping("/count")
    public Long findCount() {
        System.out.println(repository);
        return repository.count();
    }
}
```

最后，我们创建一个BookController，专注于特定于图书的操作，例如通过其ID查找一本书：

```java
@RestController
public class BookController {
      
    @Autowired
    private BookRepository repository;
  
    @GetMapping("/book/{id}")
    public Book findById(@PathVariable long id) {
        System.out.println(repository);
        return repository.findById(id).get();
    }
}
```

然后，我们启动此应用程序并在/ count和/ book / 1上执行GET：

```shell
curl -X GET http://localhost:8080/count
curl -X GET http://localhost:8080/book/1
```

在应用程序输出中，我们看到两个BookRepository对象具有相同的对象ID：

```
com.baeldung.spring.patterns.singleton.BookRepository@3ea9524f
com.baeldung.spring.patterns.singleton.BookRepository@3ea9524f
```

LibraryController和BookController中的BookRepository对象ID相同，这证明Spring将相同的bean注入到两个控制器中。

我们可以使用@scope（ConfigurableBeanFactory.scope_prototype）注释将bean范围从singleton更改为prototype，从而创建BookRepository bean的单独实例。

这样做指示Spring为它创建的每个BookRepository Bean创建单独的对象。因此，如果我们再次检查每个控制器中BookRepository的对象ID，我们将发现它们不再相同。

---

## 3.工厂方法模式

工厂方法模式要求工厂类具有用于创建所需对象的抽象方法。 通常，我们想基于特定的上下文创建不同的对象。

例如，我们的应用程序可能需要车辆对象。在航海环境中，我们想要制造船只，但是在航空航天环境中，我们想要制造飞机：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-151359.jpg)

为此，我们可以为每个所需的对象创建一个工厂实现，并从具体的工厂方法中返回所需的对象。

### 3.1 Application Context

Spring在其依赖注入（DI）框架的基础上使用了此技术。 从根本上讲，Spring将一个bean容器视为生成bean的工厂。 因此，Spring将BeanFactory接口定义为Bean容器的抽象：

```java
public interface BeanFactory {
 
    getBean(Class<T> requiredType);
    getBean(Class<T> requiredType, Object... args);
    getBean(String name);
 
    // ...
}
```

每个getBean方法均被视为工厂方法，该方法将返回一个与提供给该方法的条件相匹配的bean，例如bean的类型和名称。

然后，Spring使用ApplicationContext接口扩展BeanFactory，该接口引入了其他应用程序配置。 Spring使用此配置基于一些外部配置（例如XML文件或Java批注）来启动Bean容器。

然后使用诸如AnnotationConfigApplicationContext之类的ApplicationContext类实现，我们可以通过从BeanFactory接口继承的各种工厂方法来创建bean。

首先，我们创建一个简单的应用程序配置：

```java
@Configuration
@ComponentScan(basePackageClasses = ApplicationConfig.class)
public class ApplicationConfig {
}
```

接下来，我们创建一个简单的类Foo，它不接受构造函数参数：

```java
@Component
public class Foo {
}
```

然后创建另一个接受单个构造函数参数的类Bar：

```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class Bar {
  
    private String name;
      
    public Bar(String name) {
        this.name = name;
    }
      
    // Getter ...
}
```

最后，我们通过ApplicationContext的AnnotationConfigApplicationContext实现创建我们的bean：

```java
@Test
public void whenGetSimpleBean_thenReturnConstructedBean() {
     
    ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class);
     
    Foo foo = context.getBean(Foo.class);
     
    assertNotNull(foo);
}
 
@Test
public void whenGetPrototypeBean_thenReturnConstructedBean() {
     
    String expectedName = "Some name";
    ApplicationContext context = new AnnotationConfigApplicationContext(ApplicationConfig.class);
     
    Bar bar = context.getBean(Bar.class, expectedName);
     
    assertNotNull(bar);
    assertThat(bar.getName(), is(expectedName));
}
```

使用getBean工厂方法，我们可以仅使用类类型和构造函数参数（对于Bar而言）来创建已配置的bean。

### 3.2外部配置

这种模式是通用的，因为我们可以根据外部配置完全更改应用程序的行为。

如果我们希望更改应用程序中自动装配对象的实现，则可以调整我们使用的ApplicationContext实现。

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-151415.png)

例如，我们可以将AnnotationConfigApplicationContext更改为基于XML的配置类，例如ClassPathXmlApplicationContext：

```java
@Test
public void givenXmlConfiguration_whenGetPrototypeBean_thenReturnConstructedBean() { 
 
    String expectedName = "Some name";
    ApplicationContext context = new ClassPathXmlApplicationContext("context.xml");
  
    // Same test as before ...
}
```

---

## 4.代理模式

代理在我们的数字世界中是一种方便的工具，我们经常在软件（例如网络代理）之外使用它们。在代码中，代理模式是一种技术，它允许一个对象（代理）控制对另一对象（主题或服务）的访问。

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-151421.png)

### 4.1 事务

要创建代理，我们创建一个对象，该对象实现与主体相同的接口，并包含对该主体的引用。

然后，我们可以使用代理代替主体。

在Spring中，代理Bean以控制对基础Bean的访问。我们在使用事务时会看到这种方法：

```java
@Service
public class BookManager {
     
    @Autowired
    private BookRepository repository;
 
    @Transactional
    public Book create(String author) {
        System.out.println(repository.getClass().getName());
        return repository.create(author);
    }
}
```

在我们的BookManager类中，我们使用@Transactional注释对create方法进行注释。该注释指示Spring自动执行我们的create方法。没有代理，Spring将无法控制对我们的BookRepository bean的访问并确保其事务一致性。

### 4.2 CGLib代理

相反，Spring创建了一个代理，该代理包装了我们的BookRepository bean，并检测了我们的bean以自动执行我们的create方法。

当我们调用BookManager＃create方法时，我们可以看到输出：

```
com.baeldung.patterns.proxy.BookRepository$$EnhancerBySpringCGLIB$$3dc2b55c
```

通常，我们希望看到一个标准的BookRepository对象ID。相反，我们看到了EnhancerBySpringCGLIB对象ID。

在后台，Spring将我们的BookRepository对象包装为EnhancerBySpringCGLIB对象。 Spring因此控制对BookRepository对象的访问（确保事务一致性）。

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-151427.png)

通常，Spring使用两种类型的代理：

```
CGLib Proxies – Used when proxying classes
JDK Dynamic Proxies – Used when proxying interfaces
```

当我们使用事务公开底层代理时，Spring将在必须控制对bean的访问的任何情况下使用代理。

---

## 5.模版模式

在许多框架中，大部分代码是样板代码。

例如，在数据库上执行查询时，必须完成相同的一系列步骤：

1. **建立连接**
2. **执行查询**
3. **执行清理**
4. **关闭连接**

这些步骤是模板方法模式的理想场景。

### 5.1 模板和回调

模板方法模式是一种定义某些操作所需的步骤，实现样板步骤并将可自定义步骤保留为抽象的技术。然后，子类可以实现此抽象类，并为缺少的步骤提供具体的实现。

对于数据库查询，我们可以创建一个模板：

```java
public abstract DatabaseQuery {
 
    public void execute() {
        Connection connection = createConnection();
        executeQuery(connection);
        closeConnection(connection);
    } 
 
    protected Connection createConnection() {
        // Connect to database...
    }
 
    protected void closeConnection(Connection connection) {
        // Close connection...
    }
 
    protected abstract void executeQuery(Connection connection);
}
```

另外，我们可以通过提供回调方法来提供缺少的步骤。

回调方法是一种允许主体向客户端发信号通知某些所需操作已完成的方法。

在某些情况下，主体可以使用此回调执行操作-例如映射结果。

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-151435.png)



例如，代替使用executeQuery方法，我们可以为execute方法提供查询字符串和回调方法来处理结果。

首先，我们创建一个回调方法，该方法接受一个Results对象并将其映射到T类型的对象：

```java
public interface ResultsMapper<T> {
    public T map(Results results);
}
```

然后我们更改我们的DatabaseQuery类以利用此回调：

```java
public abstract DatabaseQuery {
 
    public <T> T execute(String query, ResultsMapper<T> mapper) {
        Connection connection = createConnection();
        Results results = executeQuery(connection, query);
        closeConnection(connection);
        return mapper.map(results);
    }
 
    protected Results executeQuery(Connection connection, String query) {
        // Perform query...
    }
}
```

这种回调机制正是Spring与JdbcTemplate类一起使用的方法。

### 5.2 JdbcTemplate

JdbcTemplate类提供了查询方法，该方法接受查询String和ResultSetExtractor对象：

```java
public class JdbcTemplate {
 
    public <T> T query(final String sql, final ResultSetExtractor<T> rse) throws DataAccessException {
        // Execute query...
    }
 
    // Other methods...
}
```

ResultSetExtractor将代表查询结果的ResultSet对象转换为T类型的域对象：

```java
@FunctionalInterface
public interface ResultSetExtractor<T> {
    T extractData(ResultSet rs) throws SQLException, DataAccessException;
}
```

通过创建更多特定的回调接口，Spring进一步减少了样板代码。

例如，RowMapper接口用于将单行SQL数据转换为类型T的域对象。

```java
public class JdbcTemplate {
 
    public <T> List<T> query(String sql, RowMapper<T> rowMapper) throws DataAccessException {
        return result(query(sql, new RowMapperResultSetExtractor<>(rowMapper)));
    }
 
    // Other methods...
}
```

除了提供用于转换整个ResultSet对象（包括在行上进行迭代）的逻辑之外，我们可以提供用于如何转换单行的逻辑：

```java
public class BookRowMapper implements RowMapper<Book> {
 
    @Override
    public Book mapRow(ResultSet rs, int rowNum) throws SQLException {
 
        Book book = new Book();
         
        book.setId(rs.getLong("id"));
        book.setTitle(rs.getString("title"));
        book.setAuthor(rs.getString("author"));
         
        return book;
    }
}
```

使用此转换器，我们可以使用JdbcTemplate查询数据库并映射每个结果行：

```java
JdbcTemplate template = // create template...
template.query("SELECT * FROM books", new BookRowMapper());
```

除了JDBC数据库管理，Spring还使用以下模板：

- **[Java Message Service (JMS)](https://www.baeldung.com/spring-jms)**
- **[Java Persistence API (JPA)](https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa)**
- **[Hibernate](https://www.baeldung.com/persistence-layer-with-spring-and-hibernate#no_template) (now deprecated)**
- **[Transactions](https://www.baeldung.com/spring-programmatic-transaction-management#transaction-template)**

---

## 6.总结

在本教程中，我们研究了Spring框架中应用的四种最常见的设计模式。

我们还探讨了Spring如何利用这些模式来提供丰富的功能，同时减轻开发人员的负担。
