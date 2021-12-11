[返回上层](index)
# 使用Reactor响应式编程

## 介绍

### 响应式编程

响应式编程不同于我们熟悉的命令式编程，我们熟悉的命令式编程即**代码就是一行接一行的指令，按照它们的顺序一次一条地出现。一个任务被执行，程序就需要等到它执行完了，才能执行下一个任务。每一步，数据都需要完全获取到了才能被处理，因此它需要作为一个整体来处理**。但是所谓的响应式编程**是函数式和声明式的。响应式流处理数据时只要数据是可用的就进行处理，而不是需要将数据作为一个整体进行提供。事实上，输入数据可以是无穷的（例如，一个地点的实时温度数据的恒定流）。**如下通过一个例子来描述响应式编程和命令式编程的差别：

🌰：某地发生火灾，附近有一个水池，我们需要利用水池中的水来灭火。

首先我们将这一系列步骤进行任务抽象：

1. 取到水池中的水。
2. 把水运送到火灾地进行灭火。

那么命令式编程，我们把一池水都看成一个整体，那个首先我们需要将一池子的水全部放入救火车中，全部放完后才能拉着这一池子水赶往火灾地进行灭火。这也符合上面对命令式编程的描述。**一个任务被执行，程序就需要等到它执行完了，才能执行下一个任务。每一步，数据都需要完全获取到了才能被处理，因此它需要作为一个整体来处理**。

但是响应式编程就不一样了，响应式编程并不要求我们把一池子水看成一个整体，而是一系列（无穷的水滴），我们的做法就像拉一根很长的水管，一端连着水池，一端在火灾地。我们使用抽水机把水源源不断的输送到火灾地进行灭火，而不需要命令式编程那样必须一个任务一个任务串行。即：**响应式流处理数据时只要数据是可用的就进行处理，而不是需要将数据作为一个整体进行提供。事实上，输入数据可以是无穷的**

通过上述的例子，可以清晰的分辨响应式编程和传统的命令式编程。

### Reactor

Reactor是基于响应式流的第四代响应式库规范，用于在JVM上构建非阻塞应用程序。Reactor 工程实现了响应式流的规范，它提供由响应式流组成的函数式 API。正如你将在后面看到的，Reactor 是 Spring 5 响应式编程模型的基础。关于响应式流在这里简要介绍下：

响应式流的规范可以通过四个接口定义来概括：Publisher，Subscriber，Subscription 和 Processor。

- Publisher：数据生产者
- Subscriber：数据订阅者
- Subscription：数据载体
- Processor：对生产者的数据进行特定处理，并传给Subscriber。

关于响应式流的具体规范可以看[这里](https://zhuanlan.zhihu.com/p/95966853)。

回头看Reactor中，存在两个核心概念:Mono和Flux。

Flux 表示零个、一个或多个（可能是无限个）数据项的管道。

Mono 特定用于已知的数据返回项不多于一个的响应式类型。

使用弹珠图来描述二者：

Flux：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162714.jpg)



Mono：

 ![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162727.jpg)

---

## Spring Boot中使用Reactor

### 添加依赖

```xml
<!--Reactor中的核心库-->
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
</dependency>
<!--Reactor测试库-->
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
    <scope>test</scope>
</dependency>
```

---

## Reactor使用示例

Flux和Mono的操作方法有很多，我们大致的将他们的所有操作分为四类：

- 创建操作
- 联合操作
- 传输操作
- 逻辑处理操作

### 创建操作

使用just()方法并传入元素来创建Flux：

```java
@Test
public void 创建一个Flux并且输出() {
  Flux<String> flux = Flux.just("1", "2", "3", "4", "5");
  flux.subscribe(f -> System.out.println("Here's some number: " + f));
}
```



我们可以传入数组，集合，Stream类来创建Flux：

```java
@Test
public void 从数组中创建一个集合() {
    String[] strs = {"1", "2", "3"};
    Flux<String> flux = Flux.fromArray(strs);
    StepVerifier.create(flux)
      .expectNext("1")
      .expectNext("2")
      .expectNext("3")
      .verifyComplete();

    List<String> strList = new ArrayList<>();
    strList.add("1");
    strList.add("2");
    strList.add("3");
    Flux.fromIterable(strList);

    Flux.fromStream(strList.stream());
}
```



指定一个范围来创建Flux：

```java
@Test
public void 提供范围生成一个Flux() {
    Flux<Integer> flux = Flux.range(0, 3);
    StepVerifier.create(flux)
      .expectNext(0)
      .expectNext(1)
      .expectNext(2)
      .verifyComplete();

  	//🌞来个附加操作：interval方法设置Flux发送数据的频率，这里设置每一秒发送一次。
  	//🌛take方法表示限制条目数量，在这里我们设定Flux最多发送三条数据。
    Flux<Long> flux1 = Flux.interval(Duration.ofSeconds(1L)).take(3L);
    StepVerifier.create(flux1)
      .expectNext(0L)
      .expectNext(1L)
      .expectNext(2L)
      .verifyComplete();
}
```

### 联合操作

Flux提供了多种联合操作，来结合多个Flux流进行操作：

**merge操作：**

```java
@Test
public void merge多个Flux() {
    Flux<Integer> flux = Flux.range(0, 3).delayElements(Duration.ofMillis(500));
    Flux<Integer> flux1 = Flux.range(3,2).delaySubscription(Duration.ofMillis(250))
      .delayElements(Duration.ofMillis(500));
  	//👋使用mergeWith方法来结合两个Flux流，mergeWith方法不能保证合并后的流中元素的顺序
  	//👋所以上面操作我们使用delaySubscription和delayElements来保证元素的顺序
  	//delaySubscription：指定时间延迟发送  delayElements：发送元素的时间间隔 
    Flux<Integer> flux2 = flux.mergeWith(flux1);
    flux2.subscribe(f -> System.out.println("Here's some number: " + f));
    StepVerifier.create(flux2)
      .expectNext(0)
      .expectNext(3)
      .expectNext(1)
      .expectNext(4)
      .expectNext(2)
      .verifyComplete();
}
```

图解上述代码：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162734.jpg)

**zip操作：**

```java
@Test
public void 合并多个Flux() {
  Flux<Integer> flux = Flux.range(0, 3).delayElements(Duration.ofMillis(500));
  Flux<Integer> flux1 = Flux.range(3, 2).delaySubscription(Duration.ofMillis(250))
    .delayElements(Duration.ofMillis(500));
	//👋zip操作将合并两个Flux流，并且生成一个Tuple2对象，Tuple2中包含两个流中同顺序的元素各一个。
  Flux<Tuple2<Integer, Integer>> flux3 = Flux.zip(flux, flux1);
  flux3.take(3).subscribe(f -> System.out.println(f.toString()));
  StepVerifier.create(flux3)
    .expectNextMatches(t -> t.getT1() == 0 && t.getT2() == 3)
    .expectNextMatches(t -> t.getT1() == 1 && t.getT2() == 4)
    .verifyComplete();
}
```

图解上述代码：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162739.jpg)

**zip配合指定逻辑操作：**

```java
@Test
    public void 合并多个Flux() {
        Flux<Integer> flux = Flux.range(0, 3).delayElements(Duration.ofMillis(500));
        Flux<Integer> flux1 = Flux.range(3, 2).delaySubscription(Duration.ofMillis(250))
          .delayElements(Duration.ofMillis(500));
				//👋在zip操作中传入指定的逻辑操作，返回一个操作结果Flux
        Flux<Integer> flux4 = Flux.zip(flux, flux1, (x, y) -> x + y);
        flux4.take(3).subscribe(f -> System.out.println(f.toString()));
        StepVerifier.create(flux4)
                .expectNext(3)
                .expectNext(5)
                .verifyComplete();
    }
```

图解上述代码：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162744.jpg)

**first操作：**

```java
@Test
    public void 只获取最先发布的Flux() {
        Flux<Integer> flux = Flux.range(0, 3).delayElements(Duration.ofMillis(500));
        Flux<Integer> flux1 = Flux.range(3, 2).delaySubscription(Duration.ofMillis(250))
          .delayElements(Duration.ofMillis(500));
      	//first操作只会使用最先发布元素的那个流
        Flux<Integer> flux2 = Flux.first(flux, flux1);
        StepVerifier.create(flux2)
                .expectNext(0)
                .expectNext(1)
                .expectNext(2)
                .verifyComplete();
    }
```

图解上述操作：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162749.jpg)

### 转换&过滤操作

**skip操作**

```java
@Test
public void 过滤Flux中的数据() {
  //👋skip操作，跳过指定数量的元素
  Flux<Integer> flux = Flux.range(0, 10).skip(8);
  StepVerifier.create(flux)
    .expectNext(8)
    .expectNext(9)
    .verifyComplete();
}
```

图解上述操作：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162754.jpg)

```java
@Test
public void 过滤Flux中的数据() {
  //👋在skip方法中传入是个时间段，表示跳过这个时间段内输出的元素
  //👋搭配delayElements方法，每个100毫秒输出一次
  //👋所以这个测试只会得到7，8，9
  Flux<Integer> flux1 = Flux.range(0, 10).delayElements(Duration.ofMillis(100))
  	.skip(Duration.ofMillis(800));
  StepVerifier.create(flux1)
    .expectNext(7)
    .expectNext(8)
    .expectNext(9)
    .verifyComplete();
}
```

图解上述方法：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162800.jpg)

**take操作**

```java
@Test
public void 过滤Flux中的数据() {
  //👋take操作与skip相反，表示获取指定数量的前几个元素
  Flux<Integer> flux2 = Flux.range(0, 10).delayElements(Duration.ofMillis(100))
    .take(Duration.ofMillis(350));
  StepVerifier.create(flux2)
    .expectNext(0)
    .expectNext(1)
    .expectNext(2)
    .verifyComplete();
}
```

图解上述方法：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162806.jpg)

```java
@Test
public void take() {
  	//👋take方法支持传入一个时间段，表示只取这个时间段内发布的元素
  	//👋下面操作中我们规定一秒发布一个元素，取3.5秒内的元素
  	//👋所以最后只能得到前三个元素
    Flux<String> nationalParkFlux = Flux.just(
        "Yellowstone", "Yosemite", "Grand Canyon","Zion", "Grand Teton")
        .delayElements(Duration.ofSeconds(1))
        .take(Duration.ofMillis(3500));
    
    StepVerifier.create(nationalParkFlux)
        .expectNext("Yellowstone", "Yosemite", "Grand Canyon")
        .verifyComplete();
}
```

图解上述方法：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162810.jpg)

**filter操作**

```java
@Test
public void 过滤Flux中的数据() {
  //👋filter方法规定一个条件，只拿取符合条件的元素
  //👋下面操作中，我们只拿取小于2的元素
  Flux<Integer> flux3 = Flux.range(0, 10).filter(n -> n < 2);
  StepVerifier.create(flux3)
    .expectNext(0)
    .expectNext(1)
    .verifyComplete();
}
```

图解上述方法：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162815.jpg)

**distinct操作**

```java
ja@Test
public void 过滤Flux中的数据() {
  //👋distinct方法用于元素去重
  Flux<Integer> flux4 = Flux.just(1, 2, 3, 3, 4, 5, 5).distinct();
  StepVerifier.create(flux4)
    .expectNext(1)
    .expectNext(2)
    .expectNext(3)
    .expectNext(4)
    .expectNext(5)
    .verifyComplete();
}

```

图解上述方法：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162821.jpg)

**map操作**

```java
@Test
public void 映射Flux() {
	//👋map方法，将元素转换成指定的另一种数据
  //👋下面操作中我们传入一个匿名的转换类，指定了我们将字符串转换为数字
  Flux<Integer> flux = Flux.just("1", "2", "3")
  	.map(Integer::valueOf);
  StepVerifier.create(flux)
    .expectNext(1)
    .expectNext(2)
    .expectNext(3)
    .verifyComplete();
}
```

图解如上方法：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162826.jpg)

**flatMap操作**

flatMap() 将每个对象映射到一个新的 Mono 或 Flux，最后这些新的Mono或者Flux会被压成（合成）一个新的Flux。

```java
@Test
public void 映射Flux() {
  //👋如下的flatMap方法将传入的每个元素都转成一个Mono
  //👋随后在Mono里面传入一个map转换逻辑（String->Integer）
  //👋使用subscribeOn来做了一个异步处理
  //👋最终会形成一个新的Flux，包含来转换后的元素，但是由于异步，不能保证顺序
  Flux<Integer> flux1 = Flux.just("1", "2", "3", "4")
    .flatMap(m -> Mono.just(m).map(c -> Integer.valueOf(c))
             .subscribeOn(Schedulers.parallel()));
  List<Integer> list = Stream.of(1, 2, 3, 4).collect(Collectors.toList());
  StepVerifier.create(flux1)
    .expectNextMatches(list::contains)
    .expectNextMatches(list::contains)
    .expectNextMatches(list::contains)
    .expectNextMatches(list::contains)
    .verifyComplete();
}
```

图解上述代码：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162833.jpg)

**buffer操作**

```java
@Test
public void 缓冲Flux() {
  Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5, 6);
  //👋buffer方法起到一个缓冲的作用
  //👋我们在buffer中指定一个数字，只有buffer被充满时或者没有剩余元素时，才会发布出去
  //👋因为你有了缓存，所以发布出去的是一个元素集合
  Flux<List<Integer>> listFlux = flux.buffer(3);
  StepVerifier.create(listFlux)
    .expectNext(Arrays.asList(1, 2, 3))
    .expectNext(Arrays.asList(4, 5, 6))
    .verifyComplete();

  //👋运行下面的代码，查看buffer是如何工作的
  Flux.just("apple", "orange", "banana", "kiwi", "strawberry")
    .buffer(3)
    .flatMap(x ->
             Flux.fromIterable(x)
             .map(y -> y.toUpperCase())
             .subscribeOn(Schedulers.parallel())
             .log()
            ).subscribe();
}
```

图解上述方法：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162839.jpg)

**collectList操作**

```java
@Test
public void 缓冲Flux() {
  Flux<Integer> flux1 = Flux.range(1, 6);
  //👋collectList方法用于将含有多个元素的Flux转换为含有一个元素列表的Mono
  Mono<List<Integer>> mono2 = flux1.collectList();
  StepVerifier.create(mono2)
    .expectNext(Arrays.asList(1, 2, 3, 4, 5, 6))
    .verifyComplete();
}
```

图解上述方法：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162845.jpg)

**collectMap操作**

```java
@Test
public void 缓冲Flux() {
  	//👋collectMap方法用于将含有多个元素的Flux转换为含有一个Map的Mono
  	//👋collectMap方法中传入的是生成键的逻辑
    Flux<Integer> flux2 = Flux.range(1, 6);
    Mono<Map<Object, Integer>> mapMono = flux2.collectMap(f -> String.valueOf(f + "i"));
    StepVerifier.create(mapMono)
      .expectNextMatches(m -> m.size() == 6
                         && m.get("1i").equals(1)
                         && m.get("2i").equals(2)
                         && m.get("3i").equals(3))
      .verifyComplete();
}
```

图解上述方法：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162851.jpg)

### 逻辑操作

```java
@Test
public void Flux的逻辑操作() {
  	//👋有时你只需要知道 Mono 或 Flux 发布的条目是否符合某些条件。all() 和 any() 操作将执行这样的逻辑。
    Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5, 6);
  	//👋any方法，只要任何一个元素符合要求，即返回true
    Mono<Boolean> mono = flux.any(f -> f < 0);
    StepVerifier.create(mono)
            .expectNext(false)
            .verifyComplete();
		//👋all方法，所有元素符合要求，即返回true
    Mono<Boolean> mono1 = flux.all(f -> f > 0);
    StepVerifier.create(mono1)
            .expectNext(true)
            .verifyComplete();
}
```

图解上述方法：

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162859.jpg)

![](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162909.jpg)

---

## 总结

本文主要介绍了响应式编程的基本概念，并用一个例子来说明响应式编程和命令式编程的差别。介绍了响应式流模型的实现库Reactor，并且解释了Reactor中的一些响应式流概念。使用SpringBoot引入Reactor库来进行Reactor开发，最后演示了Reactor的一些常见操作。



本文示例代码地址：[https://gitee.com/jeker8chen/react-demo.git](https://gitee.com/jeker8chen/react-demo.git)

---

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-162920.jpg)
