[返回上层](../index)

# Java – Try with Resources

## 1.介绍

Java 7中引入的对try-with-resources的支持使我们能够声明将在try块中使用的资源，并确保在执行该块后将关闭资源。

**⚠️：声明的资源必须实现AutoCloseable接口。**

---

## 2.使用try-with-resources

简单地说，要自动关闭，必须在try中声明和初始化资源，如下所示：

```java
try (PrintWriter writer = new PrintWriter(new File("test.txt"))) {
    writer.println("Hello World");
}
```

---

## 3.使用try-with-resources代替try–catch-finally 

使用简单明显的try-with-resources代替传统和冗长的*try-catch-finally* 块。

让我们比较以下代码示例–首先是一个典型的try-catch-finally块，然后是新方法，使用等效的try-with-resources块：

```java
Scanner scanner = null;
try {
    scanner = new Scanner(new File("test.txt"));
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException e) {
    e.printStackTrace();
} finally {
    if (scanner != null) {
        scanner.close();
    }
}
```

这是使用try-with-resources的超级简洁解决方案：

```java
try (Scanner scanner = new Scanner(new File("test.txt"))) {
    while (scanner.hasNext()) {
        System.out.println(scanner.nextLine());
    }
} catch (FileNotFoundException fnfe) {
    fnfe.printStackTrace();
}
```

你最钟意那种呢？

---

## 4.具有多个资源的try-with-resources

通过用分号分隔多个资源，可以在try-with-resources块中声明多个资源：

```java
try (Scanner scanner = new Scanner(new File("testRead.txt"));
    PrintWriter writer = new PrintWriter(new File("testWrite.txt"))) {
    while (scanner.hasNext()) {
    writer.print(scanner.nextLine());
    }
}
```

---

## 5.自定义实现AutoCloseable接口的资源

要构造一个将由try-with-resources块正确处理的自定义资源，该类应需要实现Closeable或AutoCloseable接口，并覆盖close方法：

```java
public class MyResource implements AutoCloseable {
    @Override
    public void close() throws Exception {
        System.out.println("Closed MyResource");
    }
}
```

---

## 6.资源关闭顺序

资源首先将会被定义（创建），然后使用，最后被关闭。让我们看一下示例：

**资源1:**

```java
public class AutoCloseableResourcesFirst implements AutoCloseable {
 
    public AutoCloseableResourcesFirst() {
        System.out.println("Constructor -> AutoCloseableResources_First");
    }
 
    public void doSomething() {
        System.out.println("Something -> AutoCloseableResources_First");
    }
 
    @Override
    public void close() throws Exception {
        System.out.println("Closed AutoCloseableResources_First");
    }
}
```

**资源2:**

```java
public class AutoCloseableResourcesSecond implements AutoCloseable {
 
    public AutoCloseableResourcesSecond() {
        System.out.println("Constructor -> AutoCloseableResources_Second");
    }
 
    public void doSomething() {
        System.out.println("Something -> AutoCloseableResources_Second");
    }
 
    @Override
    public void close() throws Exception {
        System.out.println("Closed AutoCloseableResources_Second");
    }
}
```

**使用资源：**

```java
private void orderOfClosingResources() throws Exception {
    try (AutoCloseableResourcesFirst af = new AutoCloseableResourcesFirst();
        AutoCloseableResourcesSecond as = new AutoCloseableResourcesSecond()) {
 
        af.doSomething();
        as.doSomething();
    }
}
```

**输出：**

> *Constructor -> AutoCloseableResources_First*
> *Constructor -> AutoCloseableResources_Second*
> *Something -> AutoCloseableResources_First*
> *Something -> AutoCloseableResources_Second*
> *Closed AutoCloseableResources_Second*
> *Closed AutoCloseableResources_First*

---

## 7.catch&finally

一个带有资源的try块仍然可以包含catch和finally块，其工作方式与传统try块相同。

## 8.总结

在本文中，我们讨论了如何使用try-with-resources，如何用try-with-resources替换try，catch和finally，如何使用AutoCloseable构建自定义资源以及了解关闭资源的顺序。
