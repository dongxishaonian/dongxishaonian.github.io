[返回上层](index)
# 云原生时代高性能Java框架—Quarkus（二）

*构建Quarkus本地镜像、容器化部署Quarkus项目*

------

***Quarkus系列博文***

- *[Quarkus&GraalVM介绍、创建并启动第一个项目](云原生时代高性能Java框架—Quarkus（一）)*
- 构建Quarkus本地镜像、容器化部署Quarkus项目*
- *...*

------

## 概览

[上一篇文章](http://blog.dongxishaonian.tech/?p=824)主要介绍了Quarkus以及给Quarkus提供“神力”的Java虚拟机GraalVM，并演示了如何安装GraalVM以及Quarkus的初步用法。本文将主要指向Quarkus的“亮点”——本地化应用程序。

**以下是本文的两个目标：**

- 将Quarkus开发的Java应用程序编译成本地可执行文件。
- 将本地可执行文件打包到容器中。

**注：在本文中本地可执行文件又称本地镜像，二者意思相同。**

## 环境准备

以下为本文所演示时的环境配置

- Intellij IDEA
- Maven
- GraalVM 20.1.0
- Docker

接下来需要安装GraalVM的一个扩展——“native-image“，此扩展用于将Java程序编译成本地可执行文件，我们执行以下命令：

```bash
gu install native-image
```

运行以下命令，查看扩展是否已安装：

```bash
$ native-image --version
```

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112833.jpg)

## 生成本地可执行文件

生成本地可执行文件的步骤如下图：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112827.jpg)

IDEA打开[上一篇文章](http://blog.dongxishaonian.tech/?p=824)创建的项目，并打开控制台，执行maven命令：

```bash
./mvnw package -Pnative
```

控制台输出以下内容：

```bash
[INFO] Scanning for projects...
...
[INFO] Building untitled 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
... 
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.example.ExampleResourceTest
2020-07-19 22:24:08,962 INFO  [io.quarkus] (main) Quarkus 1.6.0.Final on JVM started in 1.085s. Listening on: http://0.0.0.0:8081
...
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
...
[INFO] [io.quarkus.deployment.pkg.steps.NativeImageBuildStep] Running Quarkus native-image plugin on GraalVM Version 20.1.0 (Java Version 11.0.7)
...
[INFO] [io.quarkus.deployment.QuarkusAugmentor] Quarkus augmentation completed in 93802ms
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  01:45 min
[INFO] Finished at: 2020-07-19T22:25:44+08:00
[INFO] ------------------------------------------------------------------------
```

打开项目中的target文件夹

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112821.jpg)

可以看到其中有个重要的文件:XXX-runner，**它是一个对JVM不依赖的本地可执行文件**，我们可以运行他来启动应用程序。

```bash
$ ./target/untitled-1.0-SNAPSHOT-runner
```

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112817.jpg)

成功启动应用程序，并且启动速度非常快🚀！

### 对比

在这里我们可以对比本地可执行文件与传统基于jvm启动速度的对比

运行如下命令，生成传统应用程序的jar文件：

```bash
./mvnw package
```

分别运行本地可执行文件和jar文件：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112812.jpg)

速度差异非常的悬殊！

### 相关配置

打开项目根目录的pom.xml，可以看到如下配置：

```xml
<profiles>
    <profile>
        <id>native</id>
        <properties>
            <quarkus.package.type>native</quarkus.package.type>
        </properties>
    </profile>
</profiles>
```

我们可以在id为native的profile中配置具体的配置项参数来自定义本地镜像（本地可执行文件）的生成。

如下为quarkus提供的具体配置列表：

Quarkus提供了许多生成本地镜像（native-image**即本地可执行文件**）的配置项，点击查看（可左右滑动）。

------

## 容器化本地可执行文件

我们可以很轻松的将Java应用程序的jar包进行容器化，当然我们也可以很轻松的将上一步生成的本地可执行文件进行容器化。

容器化本地可执行文件的步骤如下：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112806.jpg)容器化本地可执行文件

### 添加配置

我们要将生成的本地可执行文件进行容器化，所以需要考虑到本地可执行文件对环境的兼容问题，在这里所生成的本地可执行文件的格式应该和docker镜像中的环境兼容了，而不是我们的本机环境（MacOS，Linux，Windows等等）。***因为不同的操作系统支持的本地可执行文件的格式并不一样，quarkus在生成本地可执行文件的时候会根据不同的操作系统环境而选择不同的可执行文件格式。***

首先我们在项目的`src/main/resources/application.properties`文件中添加配置：

```properties
quarkus.native.container-runtime=docker
```

上面配置表明在容器化本地可执行文件时将基于docker环境，我们也可以基于其他的容器环境，比如podman。

执行以下命令生成兼容docker容器环境的本地可执行文件：

```bash
./mvnw package -Pnative -Dquarkus.native.container-build=true
```

执行以下命令，将本地可执行文件打包成docker镜像：

```bash
docker build -f src/main/docker/Dockerfile.native -t quarkus-quickstart/getting-started .
```

生成完毕，运行以下命令即可启动该容器：

```bash
docker run -i --rm -p 8080:8080 quarkus-quickstart/getting-started
```

可以看到通过容器方式启动应用程序速度也很快

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112800.jpg)

我们可以看一下这背后的Dockerfile，打开`src/main/docker/Dockerfile.native`：

```bash
FROM registry.access.redhat.com/ubi8/ubi-minimal:8.1
WORKDIR /work/
RUN chown 1001 /work \
    && chmod "g+rwX" /work \
    && chown 1001:root /work
COPY --chown=1001:root target/*-runner /work/application

EXPOSE 8080
USER 1001

CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
```

Quarkus使用ubi-minimal镜像作为容器的基础镜像，它是一个通用基本镜像，Dockerfiles使用基本镜像的最小版本来减小生成的镜像的大小。

### 无GraalVM环境下的镜像生成

当我们处理一个CI/CD的环境或其他本地无GraalVM的环境时，此时就不能在本地生成本地可执行文件了。我们可以通过在docker中处理这些操作，在项目的`src/main/docker`中添加文件Dockerfile.multistage，并在文件中添加下面内容：

```bash
## Stage 1 : build with maven builder image with native capabilities
FROM quay.io/quarkus/centos-quarkus-maven:20.1.0-java11 AS build
COPY pom.xml /usr/src/app/
RUN mvn -f /usr/src/app/pom.xml -B de.qaware.maven:go-offline-maven-plugin:1.2.5:resolve-dependencies
COPY src /usr/src/app/src
USER root
RUN chown -R quarkus /usr/src/app
USER quarkus
RUN mvn -f /usr/src/app/pom.xml -Pnative clean package

## Stage 2 : create the docker final image
FROM registry.access.redhat.com/ubi8/ubi-minimal
WORKDIR /work/
COPY --from=build /usr/src/app/target/*-runner /work/application

# set up permissions for user `1001`
RUN chmod 775 /work /work/application \
  && chown -R 1001 /work \
  && chmod -R "g+rwX" /work \
  && chown -R 1001:root /work

EXPOSE 8080
USER 1001

CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
```

这是一个多阶段的镜像打包过程，第一阶段我们在docekr中构建本地可执行文件，第二阶段再将本地可执行文件打包成镜像。

运行如下命令：

```bash
docker build -f src/main/docker/Dockerfile.multistage -t quarkus-quickstart/getting-started .
```

如上操作将两个阶段的操作整合在一起，并生成了最终的镜像。

运行如下命令启动容器：

```bash
docker run -i --rm -p 8080:8080 quarkus-quickstart/getting-started
```

------

## 测试本地可执行文件

打开项目中的测试文件夹，可以看到有如下两个件

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112755.jpg)

其中`ExampleResourceTest`类为普通的Java测试类，他的运行基于JVM。

```java
@QuarkusTest
public class ExampleResourceTest {

    @Test
    public void testHelloEndpoint() {
        given()
            .when().get("/hello")
            .then()
            .statusCode(200)
            .body(is("hello"));
    }

}
```

上述测试类使用了`@QuarkusTest`注解，这个注解类似于Spring Boot中的`@SpringBootTest`，用来在测试前启动上下文。

而`NativeExampleResourceIT`则不同，该测试类的代码如下：

```java
package com.example;

import io.quarkus.test.junit.NativeImageTest;

@NativeImageTest 1️⃣
public class NativeExampleResourceIT extends ExampleResourceTest 2️⃣{

    // Execute the same tests but in native mode.
}
```

1️⃣：`@NativeImageTest` 注解表示此测试类是一个基于本地镜像的测试类，在测试之前，从本地可执行文件启动应用程序。可执行文件位置可在Maven的pom.xml中配置（`maven-failsafe-plugin`的`native.image.path`属性）。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>${surefire-plugin.version}</version>
    <executions>
        <execution>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
            <configuration>
                <systemPropertyVariables>
                    <native.image.path>${project.build.directory}/${project.build.finalName}-runner</native.image.path>
                    <java.util.logging.manager>org.jboss.logmanager.LogManager</java.util.logging.manager>
                    <maven.home>${maven.home}</maven.home>
                </systemPropertyVariables>
            </configuration>
        </execution>
    </executions>
</plugin>
```

2️⃣：这里的代码表示我们扩展了之前的测试，但是您也可以自定义实现您自己的测试。

运行本地镜像测试和普通测试的方式有差异，本地镜像测试需要使用Maven命令来启动，我们在IDEA控制台中运行`./mvnw verify -Pnative`即可启动本地镜像测试。**注意：由于我们上一步中在项目的配置文件中添加了`quarkus.native.container-runtime=docker`，现在我们需要去掉，否则生成的可执行文件格式可能和你本机的格式不兼容。**

```xml
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------------------< com.example:untitled >------------------------
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.example.NativeExampleResourceIT
Executing [/Users/chengang/IdeaProjects/quarkus-demo/target/untitled-1.0-SNAPSHOT-runner, -Dquarkus.http.port=8081, -Dquarkus.http.ssl-port=8444, -Dtest.url=http://localhost:8081, -Dquarkus.log.file.path=target/target/quarkus.log]
__  ____  __  _____   ___  __ ____  ______ 
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/ 
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \   
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/   
2020-07-23 22:21:09,626 INFO  [io.quarkus] (main) untitled 1.0-SNAPSHOT native (powered by Quarkus 1.6.0.Final) started in 0.019s. Listening on: http://0.0.0.0:8081
2020-07-23 22:21:09,626 INFO  [io.quarkus] (main) Profile prod activated. 
2020-07-23 22:21:09,626 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 2.3 s - in com.example.NativeExampleResourceIT
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] 
[INFO] --- maven-failsafe-plugin:2.22.1:verify (default) @ untitled ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  02:17 min
[INFO] Finished at: 2020-07-23T22:21:11+08:00
[INFO] ------------------------------------------------------------------------
chengang@chengangdeMacBook-Pro quarkus-demo % 
```

测试通过！

## 总结

本文主要介绍了Quarkus框架的本地化相关操作，我们具体介绍了如何将Quarkus项目编译成本地可执行文件，随后又演示了如何将生成的可执行文件打包成Docker镜像，最后我们演示了如何以本地可执行文件的形式测试业务代码。随着将Java应用程序编译成本地镜像，Java的性能优势有了极大的提升。

*本文参考：https://quarkus.io/guides/building-native-image*

------


🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-112732.jpg)
