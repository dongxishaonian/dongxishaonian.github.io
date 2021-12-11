[返回上层](index)
# 使用Maven Archetype创建Java项目模板

## 1.over view

简而言之，Archetype是一个Maven项目模板工具包。原型被定义为一种原始的模式或模型，所有其他同类的东西都是从中产生的。当我们试图提供一个提供生成Maven项目的一致方法的系统时，这个名字就合适了。Archetype将帮助作者为用户创建Maven项目模板，并为用户提供生成这些项目模板的参数化版本的方法。

使用原型提供了一种很好的方法，可以与您的项目或组织所采用的最佳实践一致的方式快速地使开发人员受益。您可能希望在组织内部实现J2EE开发的标准化，因此您可能希望提供EJB，WAR或Web服务的原型。一旦创建了这些原型并将其部署在组织的存储库中，组织中的所有开发人员就可以使用它们。

---

## 2.do it

⚠️：**我们将使用springboot项目来演示如何生成一个maven archetype（原型），本文中（模板）（原型）交替使用，二者意思相同。**

示例，我们有一个现成的项目，其结构如下：

```properties
.
├── Dockerfile
├── README.md
├── last-demo.iml
├── mvnw
├── mvnw.cmd
├── pom.xml
├── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── demo
    │   │           └── data
    │   │               ├── Application.java
    │   │               └── your_business_package
    │   │                   ├── client
    │   │                   │   └── DemoClient.java
    │   │                   ├── constants
    │   │                   │   └── YourBusinessConstants.java
    │   │                   ├── enumerate
    │   │                   │   └── DemoStatus.java
    │   │                   ├── presistence
    │   │                   │   ├── DemoRepository.java
    │   │                   │   └── entity
    │   │                   │       └── DemoDO.java
    │   │                   ├── service
    │   │                   │   └── DemoService.java
    │   │                   └── web
    │   │                       ├── dto
    │   │                       │   └── DemoDTO.java
    │   │                       └── rest
    │   │                           └── DemoController.java
    │   └── resources
    │       ├── application.yml
    │       └── logback-spring.xml
    └── test
        ├── java
        │   └── com
        │       └── demo
        │           └── data
        │               └── ApplicationTests.java
        └── resources
            └── application.yml
```

我们将使用maven archetype来创建以该项目为基础的模板。

### 2.1 生成模板文件夹

执行以下maven命令：

```java
mvn archetype:create-from-project
```

此时项目中会生成**target/generated-sources/archetype**文件夹，其中存放的就是我们的模板相关文件。

### 2.2 自定义模板

探索**target/generated-sources/archetype**我们可以得知：

```properties
generated-sources
    └── archetype
        ├── pom.xml
        ├── src
        │   ├── main
        │   │   └── resources
        │   │       ├── META-INF
        │   │       │   └── maven
        │   │       │       └── archetype-metadata.xml ##⚠️原型描述符,描述了我们原型的结构
        │   │       └── archetype-resources ##⚠️经过maven转换后的项目文件包
        │   └── test
        │       └── resources
        │           └── projects
        │               └── basic
        └── target
            ├── classes
            │   └── archetype-resources
            ├── your_project_name.jar
            └── test-classes
                └── projects
                    └── basic
```

我们随机打开一个archetype-resources中的源文件，可以看到如下：

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-160018.jpg" alt="image-20200415142932902" style="zoom:50%;" />

上图中我们看到的${package}占位符，这个就是maven原型插件自动处理的结果，到时候我们根据原型生成项目的时候，这些占位符就会变成我们新生成项目的相关的值。类似，maven还提供了**groupId,artifactId, version**等关键字。如果我们项目中有其他地方也需要这种定制化，我们可以手动进行更改。

例如我们把项目配置文件改为如下（应用名用占位符代替），目的是实现项目的名称随新建的项目变动。

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-160023.jpg" alt="image-20200415145148757" style="zoom:33%;" />



接下来来分析archetype-metadata.xml，他是原型描述符号，我们可以指定那些文件进入原型里，那些文件需要排除，还能指定上面说的占位符需不需要被替换 等等。

如下为archetype-metadata.xml示例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<archetype-descriptor xsi:schemaLocation="https://maven.apache.org/plugins/maven-archetype-plugin/archetype-descriptor/1.1.0">
  <fileSets>
    <fileSet filtered="true" packaged="true" encoding="UTF-8">
      <directory>src/main/java</directory>
      <includes>
        <include>**/*.java</include>
      </includes>
    </fileSet>
    <fileSet filtered="true" encoding="UTF-8">
      <directory>src/main/resources</directory>
      <includes>
        <include>**/*.xml</include>
      </includes>
    </fileSet>
    <fileSet filtered="true" encoding="UTF-8">
      <directory>src/main/resources</directory>
      <includes>
        <include>**/*.yml</include>
      </includes>
    </fileSet>
    <!--下面还有更多项-->
```

⚠️：**fileSet**属性标签指定的那些文件需要纳入原型中，我们把不需要的删掉。

⚠️：**filtered**属性标签表示是否替换文件中的占位符，若为true则会替换，否则不会，**所以我们如果想要占位符最后会被替换为项目相关的信息，还需要通过这个标签指定**。

⚠️：**packaged**属性标签指定文件是否在项目的包里面，true或false。



### 2.3 生成模板（原型）

我们进入**target/generated-sources/archetype**目录，执行以下命令：

```java
mvn install
```

此时模板将在我们本地生成。

### 2.4 使用模板（原型）生成新项目

我们使用以下命令：

> ```java
> mvn archetype:generate  \
> -DarchetypeCatalog=local \
> -DgroupId=新建项目的groupId \
> -DartifactId=新建项目的artifactId \
> -DarchetypeGroupId=你的原型group \
> -DarchetypeArtifactId=你的原型项目名字-archetype \
> -DarchetypeVersion=你的原型版本 \
> -DinteractiveMode=false 
> ```

之后，我们会生成新项目。项目的结构符合我们的原型结构。查看我们手动指定的application.yml

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-160030.jpg" alt="image-20200415150954343" style="zoom:50%;" />

可以看到我们的占位符被我们项目的相关信息给替换了。

### 2.5 将模板上传至maven仓库

我们进入**target/generated-sources/archetype**目录，打开pom.xml

添加仓库信息：

```xml
<distributionManagement>
    <repository>
        <id>my-releases</id>
        <url>你的仓库地址</url>
    </repository>
    <snapshotRepository>
        <id>my-snapshots</id>
        <url>你的仓库地址</url>
    </snapshotRepository>
</distributionManagement>
```

```xml
<servers>
    <server>
        <id>my-snapshots</id>
        <username>对应仓库的username</username>
        <password>对应仓库的password</password>
    </server>
    <server>
        <id>my-releases</id>
        <username>对应仓库的username</username>
        <password>对应仓库的password</password>
    </server>
</servers>
```

随后指定如下命令：

```java
mvn deploy
```

随后，原型将被上传至你的mavne仓库。

---

## 3.summary

本文我们介绍的maven的原型及其特性带来的好处，并且我们演示了如何生成一个原型，并且利用原型来创建一个新项目。

---

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**
<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-160036.jpg" alt="image-20200410104030284" style="zoom:25%;" />
