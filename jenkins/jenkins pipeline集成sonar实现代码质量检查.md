[返回上层](../index)
# jenkins pipeline集成sonar实现代码质量检查
## 1.sonarQube的简介

**SonarQube是一款自动化代码审查工具，用于检测代码中的错误、漏洞和代码异味。它可以与你现有的工作流集成，以支持跨项目分支和拉取请求的连续代码检查。**

其工作流程如下：

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162216.png" style="zoom:50%;" />

如图SonarQube由以下4个组件组成：

**1.SonarQube Server：**

- WebServer：供开发人员，管理人员浏览高质量的快照并配置SonarQube实例
- Search Server：基于Elasticsearch的Search Server从UI进行后退搜索(历史)
- computeEngine：负责处理代码分析报告并将其保存在SonarQube数据库中

**2.SonarQube Database：**

- SonarQube实例的配置（安全性、插件设置等）
- 项目、视图等的质量快照。

**3.SonarQube Plugins：**服务器上安装了多个SonarQube插件，可能包括语言，SCM，集成，身份验证和管理插件

**4.SonarScanners：**多种sonar扫描组件，在构建/持续集成服务器上运行以分析项目。

***关于sonar的具体介绍可参考其[官网](https://www.sonarqube.org)。***

---

## 2.SonarQube的安装

SonarQube提供了多种[安装方式](https://docs.sonarqube.org/latest/setup/install-server/)，本文将使用docker镜像的安装方式进行演示。

### 1.拉取sonarQube的docker容器

```shell
$> docker pull sonarqube:8.2-community
```

### 2.创建docker数据卷

```shell
#包含数据文件，例如嵌入式H2数据库和Elasticsearch索引
$> docker volume create --name sonarqube_data
#包含插件，例如语言分析器
$> docker volume create --name sonarqube_extensions
#包含有关访问，Web流程，CE流程和Elasticsearch的SonarQube日志
$> docker volume create --name sonarqube_logs
```

### 3.配置本地数据库（示例使用postgresql）

如果使用postgresql的默认schema "public"，则无需这一步。如果想自定义schema，则执行以下命令

```sql
ALTER USER mySonarUser SET search_path to <自定义的schema名称>
```

### 4.启动sonarQube

```shell
$> docker run -d --name sonarqube \
    -p 9000:9000 \
    #以下为给sonarQube的数据库配置，推荐postgresql
    -e SONAR_JDBC_URL=jdbc:postgresql://xxxx:5432/postgres \
    -e SONAR_JDBC_USERNAME=... \
    -e SONAR_JDBC_PASSWORD=... \
    -v sonarqube_data:/opt/sonarqube/data \
    -v sonarqube_extensions:/opt/sonarqube/extensions \
    -v sonarqube_logs:/opt/sonarqube/logs \
    <image_name>
```

本地浏览器访问localhost:9000即可访问。

---

## 3.sonarQube中创建项目

### 步骤1:新建项目

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162229.png" alt="企业微信截图_3a0966e9-ebb8-4878-8d1c-985bd65c0009" style="zoom:50%;" />

### 步骤2:填写项目信息

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162234.png" alt="image-20200409183243933" style="zoom:50%;" />

### 步骤3:创建令牌

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162238.png" alt="image-20200409183631966" style="zoom: 33%;" />

### 步骤4:记录令牌

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162243.png" alt="image-20200409183801845" style="zoom:33%;" />

**创建项目完成，并且记录下令牌，后续步骤会用到。**

---

## 4.jenkins配置sonarQube插件

### 1.安装sonarQube插件

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162248.png" alt="image-20200409181901457" style="zoom:33%;" />

### 2.添加sonarQube配置	

在jenkins>Manage Jenkins>global configuration中配置sonar的信息，如下图：

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162254.png" alt="image-20200409184010520" style="zoom: 50%;" />

name为自定义的名字，serverURL为sonarqube的访问地址

最后一项token需要添加。点击添加，如下图：

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162301.png" alt="image-20200409184332842" style="zoom: 50%;" />

类型选择 Secret text，并将我们在第三步中得到的token填入secret栏，其余栏目自定义。添加完后，回到上一步Server authentication token选择刚刚添加的token。

---

## 5.项目中引入sonarQube

以下为maven单模块项目示例：

### 1.引入sonarqube插件：

```xml
<!--添加参数，指定projectKey，即在sonar中创建项目时的名称-->
<properties>
	<sonar.projectKey>sonar-demo</sonar.projectKey>
</properties>
<!--添加sonarqube插件-->
<plugin>
  <groupId>org.sonarsource.scanner.maven</groupId>
  <artifactId>sonar-maven-plugin</artifactId>
  <version>3.6.0.1398</version>
</plugin>
```

### 2.手动代码扫描

执行以下命令

```shell
mvn sonar:sonar \
  -Dsonar.projectKey=sonar的项目名称 \
  -Dsonar.host.url=http://sonar的地址 \
  -Dsonar.login=第三步记录的令牌
```

执行完成回到sonarQube界面，可看到扫描记录及结果：

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162308.png" alt="image-20200409184114645" style="zoom:50%;" />

---

## 6.jenkins pipeline中集成sonarqube

在jenkinsfile中定义代码静态检查的stage，如下图：

<img src="https://images.cnblogs.com/cnblogs_com/dongxishaonian/1693598/o_200410031623image-20200409234426724.png" alt="image-20200409234426724" style="zoom:50%;" />

上图中定义了两个stage，1.代码静态检查  2.检查结果分析

⚠️**：检查结果分析阶段jenkins通过sonarqube的回调来得知扫描结果,需要在sonarqube中配置webhook：**

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162315.png" alt="image-20200409184522988" style="zoom: 50%;" />

**webhook域名为jenkins地址域名。**

至此，在jenkins pipeline（流水线）中成功集成sonarqube，如下图：

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162319.png" alt="image-20200409184044044" style="zoom: 50%;" />

---

---

## 7.sonarQube自定义质量阀

### 1.创建自定义质量阀

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162327.png" alt="image-20200409184142238" style="zoom: 50%;" />

### 2.配置质量阀

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162333.png" alt="image-20200409184212688" style="zoom: 50%;" />

### 3.测试

例如当阀值过高时，扫描结果不符合阀值要求，扫描结果则会失败。

示例中我们随便调整某个条件至当前项目无法到达的值

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162339.png" alt="image-20200410103807909" style="zoom: 50%;" />

此时当pipeline在运行的时候，代码质量检查就会失败，因为没有达到标准。

<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162346.png" alt="image-20200410104030284" style="zoom:50%;" />

⚠️：**所以，我们在自定义自己项目的阀值的时候要视不同的项目情况而定。**

---

## 8.总结

本文介绍了在jenkins多分支流水线中集成sonarQube,从而实现在持续集成中代码质量检查。文章涉及到的某些方面（jenkinsfile，sonarQube详细使用等等）没有详细介绍，只是快速带过了。sonarQube是业界知名度很高的代码检查工具，也是ci/cd中的工具生态成员。深入探索它，还会发现更多有用的，有趣的特性。代码质量检查时代码质量内建的一部分，在流水线中集成代码质量检查可以及时的发现代码中存在的问题和缺陷，从而及时修复问题，防止技术债务的堆积（还是那句话，解决问题最好的时机时问题出现的那一刻），否则当问题堆积到一定程度的时候，修复成本会越来越高。

---
**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**
<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162353.jpg" alt="image-20200410104030284" style="zoom:50%;" />
**java工程师一名，但不只是java工程师，涉略devops，自动化测试，ci/cd，敏捷开发等等。提供自动化测试，ci/cd等技术需求的解决方案，公众号，微信（cg8377），邮箱（837718548@qq.com）均可联系到我。也欢迎和各位做技术交流。**
