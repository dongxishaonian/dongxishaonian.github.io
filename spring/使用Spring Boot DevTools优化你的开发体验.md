[返回上层](index)
# 使用Spring Boot DevTools优化你的开发体验

## 场景再现

某日少年收到前端同学发来的消息说联调的接口响应异常🙃，少年表现的很平静🙂，因为这种事情太平常了😑。于是询问详情之后开始打开自己的代码查找问题所在，没过五分钟就发现了问题。少年修改完代码之后将本地启动的项目停止然后再重新启动。由于当前的服务端项目是一个巨大的单体应用，启动需要花**三四分钟时间**，于是少年就拿出手机开始刷起朋友圈。刷着刷着（**由于注意力分散不知不觉花了十几分钟**）突然意识到项目已经重新启动，于是通知前端同学查看效果。

### 分析问题

上面的场景可能对很多开发者来说感同身受，在开发中修改项目是很平常且频繁的一件事情。当我们修改完代码或其他文件的时候，我们会重新启动项目来验证修改是否真的生效（这里忽略我们编写的测试代码），以供前端或者其他客户端来使用我们的修改。但是不知不觉这样的流程浪费了我们很多时间，**甚至被迫分散我们的注意力（打开社交软件、看新闻、和同事聊天）**，这些问题对我们的生产力是一个极大的威胁。

## spring-boot-devtools

能否有一种方案可以让我们对项目的修改快速生效，从而节省那些我们本该可以利用的时间呢？幸好有一种工具可以解决当前所存在的问题，这就是`**Spring Boot Dev Tools**`。

### 原理简介

您可能会说，了解Spring Boot Dev Tools的工作原理并不重要，但是由于开发过程中存在很多复杂的情况，所以了解Spring Boot Dev Tools的工作原理是对我们有帮助的。

Spring Boot Dev Tools钩接（hooks into）到Spring Boot的类加载器中，以提供一种方法来按需重新启动应用程序上下文或重新加载已更改的静态文件而无需重新启动整个应用程序。

为此，Spring Boot Dev Tools将划分应用程序的类路径并分配给两个不同的类加载器：

- **基本类加载器（base classloader）**：包含一些不可变类或者几乎不会被修改文件，例如Spring Boot JAR或第三方库。
- **重新启动类加载器（restart classloader）**：包含应用程序的文件，这些文件在项目开发过程中将频繁更改。

重新启动应用程序后，现有的**重新启动类加载器**将被丢弃，新的**重新启动类加载器**将被启动。这种方法意味着应用程序的重启通常比“冷启动”要快得多，因为**基本类加载器**没有受到影响并且一直存在着。

## 引入依赖

当我们使用**intellij IDEA**的**Spring Initializr**创建项目时，**Spring Initializr**提供了内置的**Spring Boot Dev Tools**依赖选项，我们只需选择它即可。

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-155750.jpg)**Spring Initializr**中引入**Spring Boot Dev Tools**

### Maven项目中引入**Spring Boot Dev Tools**

在项目的pom.xml文件中引入**Spring Boot Dev Tools**依赖即可

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

## 项目演示

### 单模块

在项目中添加一个简单的Controller

```java
@SpringBootApplication
public class DevToolApplication {

    public static void main(String[] args) {
        SpringApplication.run(DevToolApplication.class, args);
    }

    @RestController
    public static class HelloWorld {
        @GetMapping("test")
        public ResponseEntity<?> getTest() {
            return ResponseEntity.ok("hello world");
        }
    }
}
```

启动项目，访问http://localhost:8080/test，返回如下：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-155757.jpg)

我们简单修改代码

```java
@RestController
public static class HelloWorld {
    @GetMapping("test")
    public ResponseEntity<?> getTest() {
        return ResponseEntity.ok("hello world after change file");
    }
}
```

运行命令`mvn compile`，运行完毕重新访问http://localhost:8080/test

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-155803.jpg)

可以看到，我们的更改已经生效了。

### 多模块

假设现在我们的项目引用了其他项目作为子模块

```xml
<dependency>
    <groupId>org.example</groupId>
    <artifactId>untitled</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

我们需要在程序运行时，对上述子模块的修改也即时生效。

在多模块项目中使用**Spring Boot Dev Tools**比单模块项目略复杂，由于在多模块项目中主模块对子模块是引用关系，并且在运行时主模块通过引用子模块的jar文件的形式来启动应用程序，根据前面**Spring Boot Dev Tools**的原理，jar文件的加载将归属于**基本类加载器**，因此按照现在的做法无法做到子模块的修改即时生效。

不过**Spring Boot Dev Tools**提供了对多模块项目的支持，我们只需要添加简单的配置即可实现多模块项目的修改即时生效。

在项目的`/resources`中创建`META-INF/spring-devtools.properties`文件，并添加配置

```properties
restart.include.projectcommon=/untitled-1.0-SNAPSHOT.jar
```

上述配置表明**重新启动类加载器**在重新启动的时候，会加载最新的子模块依赖，从而做到子模块的修改即时生效。

现在子模块中存在如下类

```java
public class DemoA {
    private String name;

    public String getName() {
        return name;
    }

    public DemoA setName(String name) {
        this.name = name+"cgsj111";
        return this;
    }
}
```

主模块中引用了上面的类

```java
@RestController
public static class HelloWorld {
    @GetMapping("test")
    public ResponseEntity<?> getTest() {
        DemoA demo = new DemoA();
        demo.setName("demo name");
        return ResponseEntity.ok(demo);
    }
}
```

此时启动应用程序，访问`http://localhost:8080/test`

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-155810.jpg)

可以看到响应正常返回，此时我们修改子模块的代码

```java
public class DemoA {
    private String name;

    public String getName() {
        return name;
    }

    public DemoA setName(String name) {
        this.name = name+"cgsj111 After Change";
        return this;
    }
}
```

然后在主模块中运行命令` mvn compile`，此时再次访问接口

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-155816.jpg)

可以看到子模块的修改已经在主模块中即时生效了。

## 远程调试

**Spring Boot Dev Tools**所展现的高效便捷之处不仅仅局限于本地调试，对于远程调试也有很好的支持。选择性地启用远程支持是因为启用它可能会带来安全风险。仅当在受信任的网络上运行或使用SSL保护时，才应启用它。如果这两个选项都不满足，则不应使用DevTools的远程支持。您永远不应该在生产环境中启用他。

启用它需要确保构建物中包含devtools，修改至如下配置：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <!--确保项目打包是将Devtools包含进去-->
                <excludeDevtools>false</excludeDevtools>
            </configuration>
        </plugin>
    </plugins>
</build>
```

然后，您需要设置spring.devtools.remote.secret属性。像任何重要的密码或机密一样，该值应唯一且强壮，以免被猜测或强行使用，例如，在`application.properties`中设置：

```properties
spring.devtools.remote.secret=cgsj8377
```

远程devtools支持分为两部分：接受连接的服务器端端点和在IDE中运行的客户端应用程序。设置`spring.devtools.remote.secret`属性后，将自动启用服务器组件，客户端组件必须手动启动。

### 调试演示

在项目文件夹中运行命令 `mvn package`生成jar文件，将jar文件部署到服务器（在这里我们以本地运行jar包的方式来模拟远程部署）。然后在IDE进行如下配置（以Intellij IDEA为例）

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-155824.jpg)

如上图我们添加了一个启动器，启动类为`org.springframework.boot.devtools.RemoteSpringApplication`，并且传递了一个程序参数来指定远程应用程序的地址，此处笔者在本机上试验所以是一个本机的地址。

接下来我们启动我们刚刚创建的启动器![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-155831.jpg)

启动日志如下

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-155835.jpg)

修改代码至如下

```java
@RestController
public static class HelloWorld {
    @GetMapping("test")
    public ResponseEntity<?> getTest() {
        DemoA demo = new DemoA();
        demo.setName("remote test");
        return ResponseEntity.ok(demo);
    }
}
```

然后运行命令`mvn compile`，可以看到我们的更改在运行的程序中即时生效了

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-155840.jpg)

## 总结

在我们的日常的开发过程中总会存在各种各样的“等待”，这些时刻很大程度上会影响开发者的效率和注意力。而Developer Tools的出现缓解了这个问题，他使应用程序的调试更加的便捷高效。有一点要注意的是在让我们的更改生效之前需要执行`mvn compile`命令，从而使本地代码能被编译成程序可以理解的字节码文件。

**本文示例代码：https://gitee.com/jeker8chen/dev-tool**

------

**本文参考资料：**

https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-devtools

https://reflectoring.io/spring-boot-dev-tools/
