[返回上层](index)
# 云原生时代高性能Java框架—Quarkus（一）

*[**——— Quarkus&GraalVM介绍、创建并启动第一个项目**](http://blog.dongxishaonian.tech/?p=824)*

------

***Quarkus系列博文***

- *[Quarkus&GraalVM介绍、创建并启动第一个项目](http://blog.dongxishaonian.tech/archives/789（在新窗口中打开）)*
- *构建Quarkus本地镜像、容器化部署Quarkus项目*
- *...*

------

## Quarkus介绍

Quarkus 是一个为 **Java 虚拟机（JVM）**和**原生编译（native compilation）**而设计的全栈Kubernetes 原生 [Java 框架](https://www.redhat.com/zh/topics/cloud-native-apps/what-is-a-Java-framework)，用于优化Java特别是Java项目的容器化，并使其成为[serverless](https://www.redhat.com/en/topics/cloud-native-apps/what-is-serverless)、[云](https://www.redhat.com/en/topics/cloud)和 [Kubernetes](https://www.redhat.com/en/topics/containers/what-is-kubernetes) 环境的高效平台。

Quarkus 可与常用 Java 标准、框架和库协同工作，例如 Eclipse MicroProfile、Apache Kafka、RESTEasy（JAX-RS）、Hibernate ORM（JPA）、Spring、Infinispan、Camel 等。

Quarkus 的依赖注入解决方案基于 CDI（上下文和依赖注入），且包含一个扩展框架来扩展功能并将其配置、引导并集成到您的应用中。添加[扩展](https://quarkus.io/extensions/)就像添加依赖项一样容易；或者，您可以使用 Quarkus 工具。

此外也是引人注目的一个特点，它还向 GraalVM（一种通用虚拟机，用于运行以多种语言（包括 Java 和 JavaScript）编写的应用）提供正确信息，以便对应用进行原生编译。

Rad Hat列出了一下清单来表明使用Quarkus的好处：[检查清单](https://www.redhat.com/cms/managed-files/cl-4-reasons-try-quarkus-checklist-f19180cs-201909-a4-zh.pdf)

### Quarkus与传统Java框架对比

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-110824.jpg)Quarkus与传统技术栈对比

来自官方的一张图，展示了使用Quarkus框架开发项目和使用传统框架开发的一些运行时数据明细对比，可以看到Quarkus项目在JVM中运行时所消耗的内存和接口响应能力要明显好于传统的Java技术栈。而将Quarkus编译成本地可执行文件（本地镜像）之后，其优势可以说非常明显了。

## GraalVM简介

GraalVM是一种高性能的虚拟机，它可以显著的提高程序的性能和运行效率，非常适合微服务。其设计初衷是实现可以运行不同语言（Java、JavaScript、基于LLVM的语言（例如C和C ++）以及其他动态语言）编写的应用程序。它消除了不同编程语言之间的隔阂，并实现了多语言共享运行时的互操作性。它可以独立运行，也可以在OpenJDK，Node.js或Oracle数据库的上下文中运行。

![GraalVM system diagram](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-110833.jpg)

对于Java应用程序，GraalVM可以带来很多有价值的好处：更快地运行它们，通过脚本语言（JavaScript, R, Python...）提供可扩展性或创建提前编译的本机映像（native-image）。

更多关于GraalVM的信息可参考：[此篇文章](http://blog.dongxishaonian.tech/archives/799)。

## GraalVM安装

本文我们使用[SDKMAN](https://sdkman.io/)来安装GraalVM。SDKMAN是一款用于在大多数基于Unix的系统上管理多个软件开发套件的并行版本的工具。它提供了一个方便的命令行界面（CLI）和API，用于安装，切换，删除和列出候选人。它以前被称为Groovy enVironment Manager （GVM），受到了非常有用的RVM和rbenv工具的启发，该工具在Ruby社区中广泛使用。

### 安装SDKMAN

运行如下命令进行安装：

```bash
$ curl -s "https://get.sdkman.io" | bash
$ source "$HOME/.sdkman/bin/sdkman-init.sh"	
```

运行如下命令，验证是否已安装ADKMAN：

```bash
$ sdk version
```

### 安装GraalVM

运行如下命令：

```bash
$ sdk list java
```

可以看到SDKMAN列出了所支持的所有Java发行版

![image-20200717142755813](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-110839.jpg)

我们找到GraalVM的发行版

![image-20200717142840033](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-110845.jpg)

截至编写本文时，GraalVM的最新版本为20.1.0.r11-grl，所以我们会安装此版本。运行如下命令安装GraalVM：

```bash
$ sdk install java 20.1.0.r11-grl
```

至此，GraalVM安装完毕！我们可以运行如下命令来判断GraalVM是否已安装：

```bash
$ java -version
```

![image-20200717143216643](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-110850.jpg)

## 创建项目

我们有多种方式创建Quarkus项目

### 使用Intellij IDEA创建Quarkus项目

点击菜单栏File>New>Project... 创建新项目

![image-20200717150609041](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-110854.jpg)

点击Next，并填写适当的信息，Next>Next...,创建完毕。

### 使用Maven命令行创建Quarkus项目

运行如下命令，创建Quarkus项目：

```bash
mvn io.quarkus:quarkus-maven-plugin:1.6.0.Final:create \
    -DprojectGroupId=org.acme \
    -DprojectArtifactId=getting-started \
    -DclassName="org.acme.getting.started.GreetingResource" \
    -Dpath="/hello"
cd getting-started
```

至此，创建项目完毕！

## 启动项目

我们使用IDEA打开项目

![image-20200717152929281](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-110902.jpg)

Quarkus并没有类似Spring Boot、Helidon之类框架一样的启动类，我们需要通过运行Maven命令来启动项目。

在IDEA控制台运行如下命令来启动项目:

```bash
./mvnw compile quarkus:dev
```

启动成功！

![image-20200717153314096](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-110907.jpg)

当然每次运行命令行会显得不便，我们可以通过如下配置来配置项目快捷启动：

![image-20200717153444892](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-110915.jpg)

点击左上角"+"图标添加一个Maven配置如左边栏，在右边栏中的Command line中填入"compile quarkus:dev"，点击OK。

![image-20200717153542033](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-110920.jpg)

此时可以点下下图所示图标来便捷启动项目

![image-20200717153914940](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-110927.jpg)

## 运行测试

打开项目中的测试类，看到如下代码：

```java
@QuarkusTest  //1
public class ExampleResourceTest {
    @Test
    public void testHelloEndpoint() {
        given()
            .when().get("/hello")
            .then()
            .statusCode(200) //2
            .body(is("hello"));
    }
}
```

1. 通过使用@QuarkusTest注解运行程序，可以指示JUnit在测试之前启动应用程序。
2. 检查HTTP响应状态代码和内容。

默认情况下，测试将在端口8081上运行，以免与正在运行的应用程序冲突。Quarkus自动将RestAssured配置为使用此端口。如果要测试其他路径，则可以使用@TestHTTPResource注解将被测试的URL直接注入到测试类的字段中。该字段的类型可以是字符串，URL或URI。我们需要为该注解指定测试路径的值。例如，如果我要测试映射到/myservlet的Servlet，只需在测试中添加以下内容：

```java
@QuarkusTest  
public class ExampleResourceTest {
    @TestHTTPResource("/myservlet")
    URL testUrl;

    @Test
    public void testHelloEndpoint() {
        given()
            .when().get(testUrl)
            .then()
            .statusCode(200) 
            .body(is("hello"));
    }
}
```

可以通过在项目配置文件中配置quarkus.http.test-port属性控制测试端口。 Quarkus还创建了一个名为test.url的系统属性，该属性值将被设置成基础测试URL（BasePath）。

## 总结

我们进入了云原生、微服务的时代，我们告别了大型单体应用的庞大和复杂，并且收获了微服务带来的极大的好处 。但是一些问题也开始接踵而至。随着微小服务的增多，曾经在单个应用上发生的多余无用依赖、Java项目与生俱来的启动过程缓慢、JIT优化问题扩散到了每个微服务上面。而且传统的Java EE规范并没有微服务的模式解决方案，问题很迫切需要解决。幸运的事，随着Quarkus、Helidon等等一些新型Java开发框架的出现缓解了这个局面（以及目前Spring生态也开始了对GraalVM的大力支持），他们使Java变得更加本地化，不管是项目的体量方面还是资源消耗和运行效率方面都有显著提升。





