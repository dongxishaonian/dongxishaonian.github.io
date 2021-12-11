[返回上层](index)
# A Guide to JUnit 5
### 准备

添加maven依赖：

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.1.0</version>
    <scope>test</scope>
</dependency>
```



### 基础注解

#### 1. **@BeforeAll and @BeforeEach**

```java
@BeforeAll
static void setup() {
    log.info("@BeforeAll - executes once before all test methods in this class");
}
 
@BeforeEach
void init() {
    log.info("@BeforeEach - executes before each test method in this class");
}
```

⚠️：*@BeforeAll* 注解的方法必须是static的，否则代码无法编译。



#### 2. **@DisplayName and @Disabled**

```java
@DisplayName("Single test successful")
@Test
void testSingleSuccessTest() {
    log.info("Success");
}
 
@Test
@Disabled("Not implemented yet")
void testShowSomething() {
}
```

@DisplayName：指定该测试显示的名称

@Disabled：禁用某个测试，并指定该测试的显示名称



#### 3. **@AfterEach and @AfterAll**

```java
@AfterEach
void tearDown() {
    log.info("@AfterEach - executed after each test method.");
}
 
@AfterAll
static void done() {
    log.info("@AfterAll - executed after all test methods.");
}
```

⚠️：*@AfterAll* 注解的方法必须是static的，否则代码无法编译。

---



### 断言和假设

#### 断言

断言已移至org.junit.jupiter.api.Assertions，并且已得到明显改善。如前所述，您现在可以在断言中使用lambda：

```java
@Test
void lambdaExpressions() {
    assertTrue(Stream.of(1, 2, 3)
      .stream()
      .mapToInt(i -> i)
      .sum() > 5, () -> "Sum should be greater than 5");
}
```

尽管上面的示例很简单，但是将lambda表达式用于断言消息的一个优点是可以对它进行延迟计算，如果消息的构建成本很高，则可以节省时间和资源。

现在还可以使用assertAll（）来对断言进行分组，这将使用MultipleFailuresError报告组内任何失败的断言：

```java
@Test
void groupAssertions() {
    int[] numbers = {0, 1, 2, 3, 4};
    assertAll("numbers",
        () -> assertEquals(numbers[0], 1),
        () -> assertEquals(numbers[3], 3),
        () -> assertEquals(numbers[4], 1)
    );
}
```

这意味着现在进行更复杂的断言更加安全，因为您将能够查明任何故障的确切位置。

#### 假设

假设仅在满足某些条件时才用于运行测试。这通常用于测试正常运行所需的外部条件，但这些条件与所测试的内容没有直接关系。

您可以使用assumeTrue（），assumeFalse（）和assumptionThat（）来声明一个假设。

```java
@Test
void trueAssumption() {
    assumeTrue(5 > 1);
    assertEquals(5 + 2, 7);
}
 
@Test
void falseAssumption() {
    assumeFalse(5 < 1);
    assertEquals(5 + 2, 7);
}
 
@Test
void assumptionThat() {
    String someString = "Just a string";
    assumingThat(
        someString.equals("Just a string"),
        () -> assertEquals(2 + 2, 4)
    );
}
```

如果假设失败，则抛出TestAbortedException，并且仅跳过测试。 假设也理解lambda表达式。



### 异常测试

JUnit 5中有两种异常测试方法。这两种方法都可以通过使用assertThrows（）方法来实现：

```java
@Test
void shouldThrowException() {
    Throwable exception = assertThrows(UnsupportedOperationException.class, () -> {
      throw new UnsupportedOperationException("Not supported");
    });
    assertEquals(exception.getMessage(), "Not supported");
}
 
@Test
void assertThrowsException() {
    String str = null;
    assertThrows(IllegalArgumentException.class, () -> {
      Integer.valueOf(str);
    });
}
```

第一个示例用于验证引发的异常的更多详细信息，第二个示例仅用于验证异常的类型。



### 测试套件

添加maven依赖：

```xml
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-runner</artifactId>
    <version>1.6.1</version>
    <scope>test</scope>
</dependency>
```



为了继续使用JUnit 5的新特性，我们将尝试了解在一个测试套件中聚合多个测试类的概念，以便我们可以一起运行这些类。JUnit 5提供了两个注释：@SelectPackages和@SelectClasses来创建测试套件。

```java
@RunWith(JUnitPlatform.class)
@SelectPackages("com.baeldung")
public class AllUnitTest {}
```

@SelectPackage用于指定运行测试套件时要选择的包的名称。在我们的示例中，它将运行所有测试。第二个注释@SelectClasses用于指定运行测试套件时要选择的类：

```java
@RunWith(JUnitPlatform.class)
@SelectClasses({AssertionTest.class, AssumptionTest.class, ExceptionTest.class})
public class AllUnitTest {}
```

例如，上面的类将创建一个包含三个测试类的套件。请注意，这些类不必在一个包中。



### 动态测试

我们要介绍的最后一个主题是JUnit 5动态测试功能，该功能允许声明和运行在运行时生成的测试用例。与静态测试在编译时定义了固定数量的测试用例相反，动态测试允许我们在运行时动态定义测试用例。

动态测试可以通过带有@TestFactory注释的工厂方法来生成。让我们看一下代码示例：

```java
@TestFactory
public Stream<DynamicTest> translateDynamicTestsFromStream() {
  List<String> in = Lists.newArrayList("1", "2", "3");
  List<String> out = Lists.newArrayList("one", "two", "three");
  return in.stream()
    .map(word ->
         DynamicTest.dynamicTest("Test translate " + word, () -> {
           int id = in.indexOf(word);
           assertThat(out.get(id).length()).isGreaterThan(2);
         })
        );
}
```

这个例子非常简单易懂。我们想使用分别命名为in和out的两个ArrayList翻译单词。工厂方法必须返回Stream，Collection，Iterable或Iterator。在本例中，我们选择Java 8 Stream。

请注意，@ TestFactory方法不得为私有或静态。测试的数量是动态的，并且取决于ArrayList的大小。



### 总结

我们可以看到JUnit 5的体系结构发生了很大变化，这与平台启动器，与构建工具，IDE，其他单元测试框架的集成等有关。此外，JUnit 5与Java 8的集成程度更高，尤其是与Lambdas和Stream概念。

---

注：本文翻译自原文：https://www.baeldung.com/junit-5 ，并且在此基础上有改动。



![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112520.jpg)

