[返回上层](index)

# [译]HAL-超文本应用语言

## 精益超媒体类型

## 总结

HAL 是一种简单的格式，它提供了一种一致且简便的方法在 API 的资源之间进行超链接。

采用 HAL 将使您的 API 易于探索，并且其文档很容易从 API 本身中发现。简而言之，这将使您的 API 更易于使用，因此对客户端开发人员更具吸引力。

使用适用于大多数编程语言的开源库，可以轻松提供和使用采用HAL的API。它也很简单，您可以像处理其他JSON一样处理它。

## 一般描述

HAL提供了一组约定以JSON或XML表示超链接。（**HAL文档的其余部分只是普通的旧JSON或XML。**）

不要使用临时结构，也不要花费宝贵的时间来设计自己的格式；您可以采用HAL的约定，并专注于构建和记录构成API的数据和转换。

HAL 有点像计算机的 HTML，因为它是通用的，旨在通过超链接驱动许多不同类型的应用程序。不同的是，HTML 具有帮助"人工参与者"通过 Web 应用程序实现其目标的功能，而 HAL 旨在帮助"自动参与者"通过 Web API 实现其目标。

话虽如此**，HAL实际上也非常人性化**。其约定使 API 的文档可以从 API 消息本身发现。这使得开发人员能够直接进入基于 HAL 的 API 并探索其功能，而无需将一些外部文档映射到其旅程的认知开销。

## 例子

下面的示例是如何使用 hal_json 表示订单集合。需要查找的事项：

- 使用自链接(self)表示的主要资源的 URI（"/orders"）
- 指向下一页订单的"next"链接
- 名为"ea:find"的模板化链接，用于按 id 搜索订单
- 数组中包含多个“ ea:admin”链接对象
- 订单集合的两个属性； “currentlyProcessing（当前正在处理）”和“shippedToday（今天发货）”
- 具有自己的链接和属性的嵌入式订单资源
- 名为"ea"的紧凑型 URI （curie） 用于扩展指向其文档 URL 的链接的名称

### application/hal+json

```json
{
    "_links": {
        "self": { "href": "/orders" },
        "curies": [{ "name": "ea", "href": "http://example.com/docs/rels/{rel}", "templated": true }],
        "next": { "href": "/orders?page=2" },
        "ea:find": {
            "href": "/orders{?id}",
            "templated": true
        },
        "ea:admin": [{
            "href": "/admins/2",
            "title": "Fred"
        }, {
            "href": "/admins/5",
            "title": "Kate"
        }]
    },
    "currentlyProcessing": 14,
    "shippedToday": 20,
    "_embedded": {
        "ea:order": [{
            "_links": {
                "self": { "href": "/orders/123" },
                "ea:basket": { "href": "/baskets/98712" },
                "ea:customer": { "href": "/customers/7809" }
            },
            "total": 30.00,
            "currency": "USD",
            "status": "shipped"
        }, {
            "_links": {
                "self": { "href": "/orders/124" },
                "ea:basket": { "href": "/baskets/97213" },
                "ea:customer": { "href": "/customers/12369" }
            },
            "total": 20.00,
            "currency": "USD",
            "status": "processing"
        }]
    }
}
```

## HAL 型号

HAL约定围绕着代表两个简单的概念：资源和链接。

### 资源

资源具有：

- 链接（到 URI）
- 嵌入式资源（即其中包含的其他资源）
- 状态（沼点标准 JSON 或 XML 数据）

### 链接

链接有：

- 目标（URI）
- 关系，又名。"rel" （链接的名称）
- 其他一些可选属性，以帮助弃用、内容协商等。

下面的图像大致说明了HAL表示的结构：

![https://tva1.sinaimg.cn/large/007S8ZIlly1gdwwz2klywj30m80go0t4.jpg](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-12-032619.jpg)
## HAL 在 API 中的使用方式

HAL 旨在构建 API，其中客户端通过以下链接围绕资源进行导航。

链接通过链接关系标识。链接关系是超媒体 API 的命脉：它们是告诉客户端开发人员哪些可用资源以及如何与其交互的方式，它们就是它们编写的代码将如何选择要遍历的链接。

但是，链接关系不仅仅是HAL中的标识字符串。 它们实际上是URL，开发人员可以遵循这些 URL 来读取给定链接的文档。 这就是所谓的“可发现性”。 这样的想法是，开发人员可以输入您的API，通读可用链接的文档，然后通过API进行操作。

HAL鼓励将链接关系（rel）用于：

1. 识别表示中的链接和嵌入资源
2. 推断目标资源的预期结构和意义
3. 向目标资源发出哪些请求和表示信号

## 如何为 HAL 服务

HAL 具有 JSON 和 XML 变体的介质类型，其名称是和分别。`application/hal+json和application/hal+xml`

在 HTTP 上提供 HAL 时，响应应包含相关的媒体类型名称。`Content-Type`

## HAL 文档的结构

### 最低有效文件

HAL 文档必须至少包含空资源。

空的 JSON 对象：

```json
{}
```

### 资源

在大多数情况下，资源应具有自己的URI

通过"self"链接表示：

```json
{
    "_links": {
        "self": { "href": "/example_resource" }
    }
}
```

### 链接

链接必须直接包含在资源中：

链接表示为包含在哈希中的 JSON 对象，该哈希必须是资源对象的直接属性：`_links`

```java
{
    "_links": {
        "next": { "href": "/page=2" }
    }
}
```

#### 链接关系

链接有关系（又名）。"rel"）。这表示特定链接的语义 - 含义。

链接 rels 是区分资源链接的主要方法。

它基本上只是哈希中的一个键，将链接含义（"rel"）与包含数据（如实际"href"值）的链接对象相关联：`_links`

```json
{
    "_links": {
        "next": { "href": "/page=2" }
    }
}
```

#### API 可发现性

链接关系rels 应该是显示有关给定链接的文档的 URL，使它们"可发现"。URL 通常相当长，并且有点讨厌用作密钥。为了绕过这一点，HAL 提供了"CURIEs"，它们基本上是名为令牌，您可以在文档中定义，并用于以更友好、更紧凑的方式表达链接关系 URI,例如`ex:widget` 而不是`http://example.com/rels/widget`。详细信息可在稍下一点的"CURIEs"部分中提供。

### 表示具有相同关系的多个链接

资源可能有多个共享同一链接关系的链接。

对于可能具有多个链接的链接关系，我们使用链接数组。

```json
{
    "_links": {
      "items": [{
          "href": "/first_item"
      },{
          "href": "/second_item"
      }]
    }
}
```

**注：**如果您不确定链接是否应是单数，则假定该链接是多个链接。如果选择单数并发现需要更改它，则需要创建新的链接关系或面对断开现有客户端。

### CURIEs

"CURIEs"帮助提供指向资源文档的链接。

HAL 为您提供了一个保留的链接关系"curies"，您可以使用它来提示资源文档的位置。

```json
"_links": {
  "curies": [
    {
      "name": "doc",
      "href": "http://haltalk.herokuapp.com/docs/{rel}",
      "templated": true
    }
  ],

  "doc:latest-posts": {
    "href": "/posts/latest"
  }
}
```

"curies"部分中可以有多个链接。它们带有一个"name"和模板化的"href"，其中必须包含占位符。`{rel}`

然后，链接可以在其“ rel”之前加上curies的名称。将latest-posts链接与doc文档curies关联，将导致链接“ rel”设置为doc:latest-posts。

若要检索有关资源的文档，客户端将扩展关联的 curies 链接与实际链接的"rel"。这将导致一个 URL，该 URL 应返回有关此资源的文档latest-posts ： `http://haltalk.herokuapp.com/docs/latest-posts`。



 ***原文地址：http://stateless.co/hal_specification.html***

---

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**
<img src="http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-12-031547.jpg" alt="image-20200410104030284" style="zoom:25%;" />





---
---
---


## 🤔  💭 👇👇👇

<script src="https://utteranc.es/client.js"
        repo="dongxishaonian/issue-posted"
        issue-term="pathname"
        label="🙂🙃😡🥶😬🤣😄"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>

