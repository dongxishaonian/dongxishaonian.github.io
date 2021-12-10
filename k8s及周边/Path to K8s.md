[返回上层](../index)

<h2>概述</h2>
<p>如果您不熟悉 Kubernetes 和容器编排，并且想要开始学习，本学习路径涵盖从基本的预备知识到容器化所需的更高级技能的所有内容。</p>
<h3>目标</h3>
<p>完成本学习路径后，您将能够：</p>
<ul>
<li>了解容器的基本知识
<ul>
<li>构建镜像并在管理环境中运行这些镜像</li>
</ul>
</li>
<li>构建容器化应用程序并将其部署到 Kubernetes
<ul>
<li>开发多层应用程序</li>
<li>将应用程序部署到管理云（例如 IBM Cloud Kubernetes Service）</li>
<li>扩展应用程序</li>
<li>调试并推出针对应用程序的更新</li>
</ul>
</li>
<li>了解将 Helm 与 Kubernetes 结合使用的部署的优势
<ul>
<li>使用 Helm 图表来安装和管理应用程序</li>
</ul>
</li>
<li>了解服务目录如何允许轻松配置 Web 服务以及与应用程序的连接</li>
<li>使用 Kubernetes 部署各种微服务</li>
<li>了解 Kubernetes 中运行的应用程序的基本网络</li>
<li>通过使用 RBAC 了解应用程序安全的工作方式
<ul>
<li>创建角色和角色绑定</li>
<li>创建服务帐户以提供对 Kubernetes 资源的细粒度访问</li>
</ul>
</li>
<li>下载并安装 Istio 服务网格
<ul>
<li>设置 Istio Ingress Gateway</li>
<li>执行简单的流量管理</li>
<li>保护服务网格</li>
<li>执行微服务策略</li>
</ul>
</li>
</ul>
<h3>预备知识</h3>
<p>本学习路径适用于入门级 Kubernetes 开发者。但是，您需要对 Linux、YAML 和命令行有基本的了解。</p>
<h3>技能水平</h3>
<p>本学习路径的技能水平适用于初学者。</p>
<h3>预估完成时间</h3>
<p>完成整个学习路径大约需要 13 个小时。</p>
<h2>模块</h2>
<p>本学习路径由以下项目构成：</p>
<h3>Kubernetes 预备知识</h3>
<ul>
<li><a href="https://www.ibm.com/developerworks/cn/cloud/library/linux-basics-and-commands/index.html">Linux 基础：学习容器的预备知识</a>
<p>了解使用容器时常用的 Linux 基本命令。</p>
</li>
<li><a href="https://www.ibm.com/developerworks/cn/cloud/library/yaml-basics-and-usage-in-kubernetes/index.html">Kubernetes 中的 YAML 基础</a>
<p>探索有关如何在 Kubernetes 中使用 YAML 的示例。</p></li>
</ul>
<h3>容器：行动的开始</h3>
<ul>
<li><a href="https://github.com/IBM/intro-to-docker-lab">容器 101</a>
<p>向您介绍容器和 Docker 的实验。</p></li>
<li><a href="https://www.ibm.com/developerworks/cn/cloud/library/building-docker-images-locally-and-in-cloud/index.html">容器化：从 Docker 和 IBM Cloud 开始</a>
<p>Docker 命令、Dockerfiles 以及结合使用 Docker 与 IBM Cloud Container Registry 的简介。</p></li>
</ul>
<h3>Kubernetes：企业容器编排</h3>
<ul>
<li><a href="https://www.ibm.com/developerworks/cn/opensource/os-tutorials-kubernetes-101-labs/index.html">Kubernetes 101：旨在助您了解 Kubernetes 的实验</a>
<p>有关如何在 IBM Cloud Kubernetes Service 中的 Kubernetes 上使用 Docker 容器的介绍性实验</p></li>
<li><a href="https://developer.ibm.com/tutorials/developing-a-kubernetes-application-with-local-and-remote-clusters/">使用本地和远程集群开发 Kubernetes 应用程序</a>
<p>有关本地 Kubernetes 开发的教程。</p></li>
</ul>
<h3>实现应用程序容器化</h3>
<ul>
<li><a href="https://www.ibm.com/developerworks/cn/cloud/library/building-docker-images-locally-and-in-cloud/index.html">容器化：从 Docker 和 IBM Cloud 开始</a>
<p>Docker 命令、Dockerfile 以及结合使用 Docker 与 IBM Cloud Container Registry。</p></li>
</ul>
<h3>Kubernetes 中的应用程序管理</h3>
<ul>
<li><a href="https://www.ibm.com/developerworks/cn/cloud/library/helm-101-labs/index.html">Helm 101：旨在帮助您了解应用程序包管理器的实验</a>
<p>查看有关使用 Helm 的这些介绍性实验。</p></li>
</ul>
<h3>将应用程序部署至 Kubernetes</h3>
<ul>
<li><a href="https://developer.ibm.com/cn/patterns/scalable-wordpress-on-kubernetes/">在 Kubernetes 上部署可扩展的 WordPress 实现</a>
<p>利用 Kubernetes 的全部功能并通过 IBM Cloud Kubernetes Service 托管 WordPress</p></li>
<li><a href="https://developer.ibm.com/cn/patterns/deploy-java-microservices-on-kubernetes-with-polyglot-support/">将 Java 微服务部署在支持多语言的 Kubernetes 上</a>
<p>学习如何利用服务发现、注册和路由来部署一个与其他多语言微服务并列运行的应用程序。</p></li>
<li><a href="https://developer.ibm.com/cn/patterns/build-digital-bank-microservices-kubernetes/">创建基于微服务的数字银行 Web 应用程序</a>
<p>构建和部署能够管理用户账户、交易、转账和账单的数字银行。</p></li>
<li><a href="https://developer.ibm.com/cn/patterns/deploy-microprofile-java-microservices-on-kubernetes/">将基于 MicroProfile 的 Java 微服务部署到 Kubernetes 上</a>
<p>使用 MicroProfile 和 Kubernetes 创建并部署 Java 微服务。</p></li>
</ul>
<h3>对 Kubernetes 应用程序进行调试/记录</h3>
<ul>
<li><a href="https://www.ibm.com/developerworks/cn/cloud/library/debug-and-log-your-kubernetes-app/index.html">对 Kubernetes 应用程序进行调试和记录</a>
<p>了解 Kubernetes 集群。</p></li>
</ul>
<h3>Kubernetes 网络和服务</h3>
<ul>
<li><a href="https://www.ibm.com/developerworks/cn/cloud/library/kubernetes-networking-101-lab/index.html">Kubernetes 网络：关于基本网络概念的实验</a>
<p>了解 pod、网络策略等概念。</p></li>
<li><a href="https://github.com/IBM/svccat">服务目录：实现 Open Service Broker API 的 Kubernetes 扩展项目</a>
<p>这是旨在培训有关服务目录用途的资料合集。</p></li>
</ul>
<h3>高级网络：Istio</h3>
<ul>
<li><a href="https://github.com/IBM/istio101/tree/master/workshop">Istio 101</a>
<p>了解如何对名为 Guestbook 的简单模拟应用程序安装 Istio 和微服务。</p></li>
<li><a href="https://developer.ibm.com/cn/patterns/manage-microservices-traffic-using-istio/">使用 Istio 管理微服务流量</a>
<p>启用微服务和高级流量管理，并使用 Istio 请求跟踪功能。</p></li>
<li><a href="https://developer.ibm.com/patterns/make-java-microservices-resilient-with-istio/">借助 Istio 使 Java 微服务变得灵活</a>
<p>利用 Istio 启用具有高级连续性和容错能力的 Java 微服务。</p></li>
<li><a href="https://developer.ibm.com/cn/patterns/istio-for-multi-clusters-across-iks-and-icp/">在私有集群和公共集群之间使用 Istio</a>
<p>通过连接私有和公共 Kubernetes 集群之间的服务来构造混合云。</p></li>
</ul>
<h3>保护您的工作负载</h3>
<ul>
<li><a href="https://www.ibm.com/developerworks/cn/cloud/library/journey-to-kubernetes-security/index.html">Kubernetes 安全之旅</a>
<p>在实现应用程序现代化时优先考虑安全性。</p></li>
<li><a href="https://developer.ibm.com/tutorials/using-kubernetes-rbac-and-service-accounts/">使用 Kubernetes RBAC 和服务帐户</a>
<p>了解如何使用 RBAC 公开 Kubernetes API 的各部分。</p></li>
<li><a href="https://www.ibm.com/developerworks/cn/cloud/library/installing-and-using-sysdig-falco/index.html">使用 Sysdig Falco 和 Kubernetes 设置运行时容器安全监控</a>
<p>通过 IBM Cloud Kubernetes Service 安装和使用 Falco。</p></li>
</ul>
<h2>建议的后续步骤</h2>
<p>在完成此 Kubernetes 学习路径后，下一步做什么？采纳这些建议，将您的学习提高到新的水平：</p>
<ul>
<li><a href="https://developer.ibm.com/cn/blog/2020/extending-kubernetes-for-a-new-developer-experience/">扩展 Extending Kubernetes 以获得新的开发者体验</a>
<p>了解 Istio 和 Knative 可能会给 Kubernetes 应用程序开发者的生活带来重大转变。</p></li>
<li><a href="https://www.ibm.com/developerworks/cn/opensource/os-knative-what-is-it-why-you-should-care/index.html">Knative 是什么？为什么您需要关注它？</a>
<p>了解面向开发者的 Kubernetes 原生平台。</p></li>
<li><a href="https://www.ibm.com/developerworks/cn/opensource/os-tutorials-knative-101-labs/index.html">Knative 101：通过几个练习来了解 Knative</a>
<p>尝试介绍基于 Kubernetes 的平台的实验，该平台用于构建、部署和管理现代化的无服务器工作负载。</p></li>
<li><a href="https://developer.ibm.com/cn/collections/openshift-on-ibm/">Red Hat OpenShift on IBM Cloud</a>
<p>Red Hat OpenShift 现在是 IBM Cloud 上的开发选项。获取有关在企业 Kubernetes 环境中现代化和构建云原生应用程序所需的文章、教程、视频和 Code Pattern。</p></li>
</ul>


本文转载自IBM中国博客：https://developer.ibm.com/cn/blog/2020/kubernetes-learning-path/
