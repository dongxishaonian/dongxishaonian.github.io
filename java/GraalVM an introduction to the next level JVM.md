[返回上层](index)

# GraalVM – an introduction to the next level JVM

随着Red Hat宣布Quarkus作为…

> 为GraalVM和HotSpot量身定制的下一代Kubernetes原生Java框架，使用一流的Java库和标准构建
>
> https://quarkus.io

Red Hat展示的Quarkus[示例项目](https://quarkus.io/)的启动速度和内存消耗给我留下了深刻的印象。令人印象深刻的主要原因之一是，代码是用GraalVM**提前（ahead-of-time，AOT）**编译成本机映像（**native image**）的。为了帮助您更好地了解传统的HotSpot JVM和GraalVM之间的区别，我将在此博客文章中向您介绍GraalVM及其功能和历史。

**TL; DR：**GraalVM是Oracle开发的用纯Java编写的JVM扩展，支持多语言编程和提前编译。

## HotSpot Java虚拟机的历史

多年来，HotSpot是Oracle维护和分发的主要Java虚拟机，用于运行Java程序。 Java HotSpot Performance Engine于1999年发布，最初由Animorphic开发，该公司被Sun Microsystems收购，现在由Oracle拥有。该虚拟机主要用C / C ++编写，并且变得越来越复杂（2007年估计有250.000行代码）。

HotSpot JVM的主要目的是运行Java字节码（.class文件）并持续分析程序的性能，以查找程序中经常执行的所谓热点，并即时（JIT，全称**just-in-time**）将其编译为本机代码（机器代码）以提高性能。这是在运行时完成的，而不是在Java程序执行之前执行的，因此是即时（**just-in-time**）的。

在HotSpot JVM中运行Java代码的工作流程如下所示（简化）：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-105911.jpg)

HotSpot虚拟机主要解释程序提供的Java字节码，但在程序运行过程中发现有适合优化的部分时，也会及时将这部分字节码编译为机器代码。

当使用JIT编译器编译一个方法时，当该方法被调用时，jvm将直接执行编译出来的机器码，而不是解释它以此来提高性能。由于编译本机代码需要CPU时间和内存，JVM必须在运行时决定编译哪些方法，因为将所有方法直接编译为本机代码会影响性能。

## Java 9发行版中的更改

借助Java 9，特别是[JEP 295](https://openjdk.java.net/jeps/295)，JDK获得了**提前（ahead-of-time，AOT）**编译器jaotc。该编译器使用OpenJDK项目[Graal](https://openjdk.java.net/projects/graal/)进行后端代码生成，这样做的原因如下：

> JIT编译器速度很快，但是Java程序可能非常庞大，以至于JIT完全预热需要很长时间。很少使用的Java方法可能根本不会被编译，由于重复的解释调用可能会导致性能下降
>
> https://openjdk.java.net/jeps/295

**Graal OpenJDK项目**演示了用纯Java编写的编译器可以生成高度优化的代码。使用此AOT编译器和Java 9，您可以提前手动编译Java代码。这意味着在执行之前生成机器代码，而不是像JIT编译器那样在运行时生成代码，这是第一种实验性的方法。

```bash
# using the new AOT compiler (jaotc is bundeled within JDK 9 and above)
jaotc --output libHelloWorld.so HelloWorld.class
jaotc --output libjava.base.so --module java.base
 
# with Java 9 you have to manually specify the location of the native code
java -XX:AOTLibrary=./libHelloWorld.so,./libjava.base.so HelloWorld
```

这将改善启动时间，因为JIT编译器不必拦截程序的执行。这种方法的主要缺点是生成的机器代码依赖于程序所在的平台（Linux，MacOS，windows...）。这可能导致AOT编译代码与特定平台绑定。

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-105918.jpg)

## GraalVM的架构

基于Graal编译器，Oracle开始开发GraalVM，不仅与HotSpots JVM的复杂C/C++代码库一起工作，而且还可以通过用Java编写的虚拟机解决当前的多语言迁移问题。

GraalVM的架构如下所示：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-105923.jpg)

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-105928.jpg)

首先，您可能会注意到一些非JVM语言的存在。现在可以在这个通用虚拟机中运行Ruby、R或JavaScript代码。只是因为GraalVM采用了Truffle框架。Truffle是一个开源库，用于构建编程语言实现，作为自修改（self-modifying）抽象语法树的解释器。有了这个特性，您现在可以在Java代码库中编写和执行例如JavaScript代码。

此外，GraalVM提供了以下功能，可以提前将程序编译成本机可执行文件：

> GraalVM允许您提前将程序编译为本地可执行文件。生成的程序不能在Java HotSpot VM上运行，而是使用必要的组件，例如内存管理，来自另一种虚拟机实现的线程调度（称为Substrate VM）。SubstrateVM用Java编写，然后编译进本地可执行文件。与Java VM相比，生成的程序具有更快的启动时间和更低的运行时内存开销。
>
> https://www.graalvm.org/docs/reference-manual/aot-compilation/

## 使用GraalVM编译并运行第一个Java程序

撰写本文时，GraalVM有两个版本：社区版（CE）和企业版（EE），仅适用于Mac OS X和Linux。要在开发过程中在Windows上使用GraalVM，您可以使用Oracle的官方Docker映像，以下示例中使用了该映像。

想象下面的简单HelloWorld类：

```java
public class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello World!");
  }
}
```

在GraalVM和Java 9的AOT编译器之前，您执行了如下代码：

```bash
$ javac HelloWorld.java
$ java HelloWorld
Hello World!
```

借助GraalVM，您现在可以选择使用现有方式（HotSpot JVM）运行应用程序，或者使用GraalVM AOT编译器创建本机映像并运行可执行文件：

```bash
$ javac HelloWorld
$ native-image HelloWorld
$ ./helloworld
HelloWorld!
```

在这个HelloWorld示例中，改进的性能是微不足道的，但是在更大和更现实的应用程序中，性能的改进是显著的。

在官方的GraalVM入门指南中可以找到一个简单的多语言应用程序示例：

```java
import java.io.*;
import java.util.stream.*;
import org.graalvm.polyglot.*;
 
public class PrettyPrintJSON {
  public static void main(String[] args) throws java.io.IOException {
      BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
      String input = reader.lines().collect(Collectors.joining(System.lineSeparator()));
      try (Context context = Context.create("js")) {
        Value parse = context.eval("js", "JSON.parse");
        Value stringify = context.eval("js", "JSON.stringify");
        Value result = stringify.execute(parse.execute(input));
        System.out.println(result.asString());
      }
    }
}
```

这个Java类负责漂亮地打印JSON，使用JavaScript方法JSON.parse() 和JSON.stringify() 。此类的本机镜像可通过如下方式构建：

```bash
$ javac PrettyPrintJSON.java
$ native-image --language:js PrettyPrintJSON
$ ./prettyprintjson < prettyMe.json
{
  "GraalVM": {
    "description": "Language Abstraction Platform",
    "supports": [
      "combining languages",
      "embedding languages",
      "creating native images"
    ],
    "languages": [
      "Java",
      "JavaScript",
      "Node.js",
      "Python",
      "Ruby",
      "R",
      "LLVM"
    ]
  }
}
```

现在可以测量运行本机映像和在HotSpot中运行应用程序之间的性能差异：

```bash
$ time bin/java PrettyPrintJSON < prettyMe.json > /dev/null
real    0m1.101s
user    0m2.471s
sys 0m0.237s
 
$ time ./prettyprintjson < prettyMe.json > /dev/null
real    0m0.037s
user    0m0.015s
sys 0m0.016s
```

在我看来，Oracle和GraalVM在Java作为编程语言的主导地位方面做得非常好。此外，这一举措提高了Java语言本身的可持续性和特性开发。有了多语言体系结构，这也增加了其他编程语言的采用。

您可以在我的[GitHub存储库](https://github.com/rieckpil/blog-tutorials/tree/master/graalvm-intro)中找到示例，然后直接在GraalVM上（Mac和Linux）或在Windows上的Docker上（确保为Docker提供至少6 GB的RAM和2-4个内核）进行尝试。

***原文地址：https://rieckpil.de/whatis-graalvm/***

------


🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-105939.jpg)
