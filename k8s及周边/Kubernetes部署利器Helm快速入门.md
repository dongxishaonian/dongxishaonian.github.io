[返回上层](index)

# Question❓

众所周知k8s是一个优秀的支持云原生应用的平台，其中拥有非常多的资源类型pod、deployment、service、ingress、namespace等等。并且k8s的部署方式是声明式的，这就造成了我们在使用k8s部署服务的时候就要去指定资源的规格了（spec）比如资源名称，期望的副本数，文件挂载等等，定义的这些规格、元信息等就要被写进部署文件里（通常是yml格式），这么多的资源类型加上这么多的规格约定，就导致了我们在部署k8s服务的时候就有可能经常涉及繁杂的yml文件（有时候既费体力也费脑力）。那么有没有一种工具能让我们尽可能在这种繁杂的工作中得到大程度的缓解呢🤔？

# 概览

## 什么是Helm？

Helm是一个Kubernetes包管理器，他通过一个叫“Helm charts”的概念来管理我们的应用程序，即使是最复杂的 Kubernetes 应用程序，都可以帮助您定义，安装和升级。**正如本文开头所描述的问题那样**，当我们的在K8s中的应用部署涉及到非常多的资源文件的时候，使用Helm就是一种很好的应用部署管理手段。

## Helm的特性

总结来看，Helm拥有四大特性来帮助我们更轻松的管理k8s应用程序，分别是：复杂性管理、易于升级、分发简单、回滚。

### 复杂性管理

即使是非常复杂或者涉及非常多资源文件的的k8s应用程序，都可以使用Helm来定义出对应的Charts（Helm charts），并且Helm将应用程序的安装/部署进行可重复化，从而保证无论什么时候部署应用程序都只需要执行相同的操作即可。我们甚至可以将Helm作为唯一的部署方式，从而到达权限的单一化，保证安全性，易于管理。

### 易于升级

Helm也提供了非常简单的应用程序升级操作，并且在升级的过程中Helm会自动帮我们维护应用程序的版本历史，从而也便于我们管理和查看应用程序的版本历史信息。

### 分发简单

Helm也提供了对应用程序的Charts的管理，我们可以将charts分发至专门的charts仓库（比如Harbor、远程文件系统等等）中进行版本化、统一化的管理。

### 回滚

Helm存储了我们应用程序的部署版本历史，在此基础上Helm也支持更便捷的应用程序回滚操作，使用`helm rollback`可以轻松回滚到该应用程序发行版的旧版本。

# Helm安装

helm目前有两个大版本，分别是Helm2和Helm3。Helm2的架构更为复杂（涉及到客户端和服务端以及二者交互组件的安装），而Helm3对此进行了简化，在使用Helm3的过程中只需要涉及到客户端即可，并且目前Helm3已渐渐成为主流，所以在本文中主要介绍的是对Helm3的安装和使用。

**Helm提供了多种安装方式，在本文中主要介绍如何使用脚本安装以及在Mac、Windows中的安装。**

## 使用脚本安装（适用于Linux操作系统）

Helm现在有个安装脚本可以自动拉取最新的Helm版本并在 [本地安装](https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3)。您可以获取这个脚本并在本地执行。它良好的文档会让您在执行之前知道脚本都做了什么。

```bash
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

如果想直接执行安装，运行`curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash`。

## 在Mac中安装Helm

Helm社区成员贡献了一种在Homebrew构建Helm的方案，这个方案通常是最新的。

```bash
brew install helm
```

## 在Windows中安装Helm

在windows系统中安装Helm首先需要确保你已经安装了[Chocolatey](https://chocolatey.org/)（关于如何安装[Chocolatey](https://chocolatey.org/)可以参考[这里](https://sspai.com/post/55309)），Helm社区成员贡献了一个 [Helm包](https://chocolatey.org/packages/kubernetes-helm)在 [Chocolatey](https://chocolatey.org/)中构建， 包通常是最新的。

```bash
choco install kubernetes-helm
```

**更多安装方式可参考Helm的官方文档介绍：https://helm.sh/zh/docs/intro/install/**

# 初试Helm

## 创建Helm Charts

前面我们知道Helm管理了我们要部署服务的所有资源文件，所以如果仅仅靠手动来为所有资源文件创建模版肯定会非常麻烦，不过Helm提供了命令来帮我们快速生成Helm charts模版，运行以下命令创建出Helm Charts的模版：

[helm create NAME [flags]](https://helm.sh/zh/docs/helm/helm_create/)

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161445.jpg)

如上图，我们创建了一个Helm charts模版，其中releaseName名字为techflower。

## charts概览

创建完charts以后，我们可以看到运行命名的当前目录下新建了一个同releaseName的文件夹，这个就是Helm charts文件。我们来看下charts的内部结构：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161451.jpg)

在charts文件夹中主要分成了四个部分：

### Chart.yml:

该文件包含了该chart的描述和元信息:

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161455.jpg)

### charts目录

`charts/`目录可以包含其他的chart(称为子chart)。

### template目录

`templates/` 目录包括了模板文件。当Helm安装chart时，会通过模板渲染引擎将所有文件发送到`templates/`目录中。 然后收集模板的结果并发送给Kubernetes：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161501.jpg) templates目录中存在一系列的资源模版文件，如上图的service模版文件，其中关键的字段被一些占位文本所替换，这也是模版文件的一个关键，定义好资源的一个基本框架，然后在安装charts时指定参数将模版中的占位文本进行替换。

### values.yaml

`values.yaml`文件中定义了一系列默认的模版参数，当我们在安装Charts时如果没有指定模版参数，helm将使用该文件中的值作为默认值替换掉模版文件中声明的占位文本。

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161506.jpg)

## 安装helm charts

Helm提供了安装命令可以让我们快捷的安装charts到K8s集群中。通过一下命令即可将我们创建出来的charts进行安装：

[helm install [NAME\] [CHART] [flags]](https://helm.sh/zh/docs/helm/helm_install/)

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161516.jpg)

如上图，我们使用helm install命令安装了helm charts，在命令中有个`--wait`的参数，代表着命令的运行会一只阻塞知道所有的K8s资源都启动成功（更多的参数选择可以参考Helm命令行的官方文档）。

### 查看资源

安装完Helm charts资源以后，我们可以通过查看K8s集群中的资源来验证安装是否成功：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161522.jpg)

可以看到，我们charts文件中所定义的资源文件都已经安装到了K8s集群中。

Helm除了会帮助我们安装charts到K8s集群之外，自己也管理了被安装charts的一系列版本（releaseName），我们可以通过这个releaseName来对helm应用进行一些运维操作。通过一下命令可以查看k8s中被安装charts的版本记录：

[helm history RELEASE_NAME [flags]](https://helm.sh/zh/docs/helm/helm_history/)

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161528.jpg)

如上图，我们看到了我们安装的charts的版本历史，由于我们只进行了一次安装，所以版本历史中只有一条记录。

### 升级release

我们知道在安装Helm charts后，helm自身会维护一个Charts的release历史列表，当我们在修改完charts中的文件并想重新安装（升级）charts的时候，我们可以使用以下命令对已安装的charts进行升级：

[helm upgrade [RELEASE\] [CHART] [flags]](https://helm.sh/zh/docs/helm/helm_upgrade/)

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161534.jpg)

如上图，我们对charts进行了升级，在随后的输出中我们可以看到当前Release的Revision为2，再次使用helm history命令查看release的历史版本：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161538.jpg)

我们可以看到，当前release存在了两条历史记录。

### 回滚release

有了被安装charts的版本历史，helm也提供了方便的release回滚操作，通过以下命令即可实现回滚：

[helm rollback  [REVISION\] [flags]](https://helm.sh/zh/docs/helm/helm_rollback/)

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161544.jpg) 如上图，我们将当前的release版本回滚到了1。随后我们再次查看release的history：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161549.jpg)

我们可以看到此时release历史中又新增了一条历史，并且被描述为回滚操作。这样，Helm提供了较完善的回滚机制。

### 删除release

[helm uninstall RELEASE  [flags]](https://helm.sh/zh/docs/helm/helm_uninstall/)

helm也提供了删除release的操作，通过以下命令即可删除release，并且一起删除charts中所定义的所有K8s资源：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161554.jpg)

删除之后，我们再来验证下之前所安装的资源是否还存在：

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161559.jpg)

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161605.jpg)

可以看到，不管时release历史还K8s资源都已经被我们删除。

## 常用命令介绍

除了上面介绍的几个命令外，helm还提供了许多其他高效的命令。

### 查看helm模版的渲染结果

[helm template [NAME\] [CHART] [flags]](https://helm.sh/zh/docs/helm/helm_template/)

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161610.jpg)

可以看到此时输出了helm模版经过参数渲染之后的结果。

## 获取集群中Release的资源Yml

[helm get manifest RELEASE_NAME [flags]](https://helm.sh/zh/docs/helm/helm_get_manifest/))

![img](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-161619.jpg)

如上，展示了目前部署在集群中的release中资源yml。我们也可以通过`--revision int`来输出指定历史版本的release的Yml描述，如：`helm get manifest techflower --revision 1`

类似[`helm get manifest`](https://helm.sh/zh/docs/helm/helm_get_manifest/)的命令还有：[Helm 获取所有](https://helm.sh/zh/docs/helm/helm_get_all/)、[Helm 获取扩展](https://helm.sh/zh/docs/helm/helm_get/)、[Helm 获取注释](https://helm.sh/zh/docs/helm/helm_get_notes/)、[Helm 获取钩子](https://helm.sh/zh/docs/helm/helm_get_hooks/)。 

# 总结

Helm的出现是我们可以更加便捷的去管理我们的K8s资源文件，并且是我们的K8s资源部署有了版本化的效果，我们从中可以看到历史版本的资源详情，我们也可以通过Helm提供的版本化来快速进行线上资源的回滚操作。除此之外，Helm周边还有很多管理Charts文件的工具，比如Harbor，从而让我们可以集中式、版本化的对charts文件进行管理，详情请看[Helm 仓库](https://helm.sh/zh/docs/helm/helm_repo/)。
