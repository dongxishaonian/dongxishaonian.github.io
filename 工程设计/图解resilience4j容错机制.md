[返回上层](index)
# 图解resilience4j容错机制

> Resilience4j是一个轻量级、易于使用的容错库，其灵感来自Netflix Hystrix，但专为Java 8和函数式编程设计。轻量级，因为库只使用Vavr，它没有任何其他外部库依赖项。相比之下，Netflix Hystrix对Archaius有一个编译依赖关系，Archaius有更多的外部库依赖关系，如Guava和Apache Commons。
>
> Resilience4j提供高阶函数（decorators）来增强任何功能接口、lambda表达式或方法引用，包括断路器、速率限制器、重试或舱壁。可以在任何函数接口、lambda表达式或方法引用上使用多个装饰器。优点是您可以选择所需的装饰器，而无需其他任何东西。
>
> 有了Resilience4j，你不必全力以赴，你可以选择你需要的。
>
> https://resilience4j.readme.io/docs/getting-started

## 概览

本文将介绍resilience4j中的四种容错机制，不过鉴于容错机制原理的通用性，后文所介绍的这几种容错机制也可以脱离resilience4j而独立存在（你完全可以自己编码实现它们或者采用其他类似的第三方库，如[Netflix Hystrix](https://github.com/Netflix/Hystrix)）。下面将会用图例来解释舱壁（[Bulkhead](https://resilience4j.readme.io/docs/bulkhead)）、断路器（[CircuitBreaker](https://resilience4j.readme.io/docs/circuitbreaker)）、限速器（[RateLimiter](https://resilience4j.readme.io/docs/ratelimiter)）、重试（[Retry](https://resilience4j.readme.io/docs/retry)）机制的概念和原理。

## 舱壁（Bulkhead）

Resilience4j提供了两种舱壁模式的实现，可用于限制并发执行的次数：

- SemaphoreBulkhead（信号量舱壁，默认），基于Java并发库中的Semaphore实现。
- FixedThreadPoolBulkhead（固定线程池舱壁），它使用一个有界队列和一个固定线程池。

SemaphoreBulkhead应该在各种线程和I / O模型上都能很好地工作。它基于信号量，与Hystrix不同，它不提供“影子”线程池选项。取决于客户端，以确保正确的线程池大小将与舱壁配置保持一致。

### 信号量舱壁（SemaphoreBulkhead）

![图解容错机制1](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-115012.jpg)

如上图，当信号量存在剩余时进入系统的请求会直接获取信号量并开始业务处理。当信号量全被占用时，接下来的请求将会进入阻塞状态，SemaphoreBulkhead提供了一个阻塞计时器，如果阻塞状态的请求在阻塞计时内无法获取到信号量则系统会拒绝这些请求。若请求在阻塞计时内获取到了信号量，那将直接获取信号量并执行相应的业务处理。

### 固定线程池舱壁（FixedThreadPoolBulkhead）

![图解容错机制0](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-115043.jpg)

FixedThreadPoolBulkhead的功能与SemaphoreBulkhead一样也是用于限制并发执行的次数的，但是二者的实现原理存在差别而且表现效果也存在细微的差别。FixedThreadPoolBulkhead使用一个固定线程池和一个等待队列来实现舱壁。当线程池中存在空闲时，则此时进入系统的请求将直接进入线程池开启新线程或使用空闲线程来处理请求。当线程池无空闲时接下来的请求将进入等待队列，若等待队列仍然无剩余空间时接下来的请求将直接被拒绝。在队列中的请求等待线程池出现空闲时，将进入线程池进行业务处理。

可以看到FixedThreadPoolBulkhead和SemaphoreBulkhead一个明显的差别是FixedThreadPoolBulkhead没有阻塞的概念，而SemaphoreBulkhead没有一个队列容量的限制。

## 限速器（RateLimiter）

![图解容错机制2](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-115053.jpg)

限速器（RateLimiter）的功能是防止突然的过量请求导致系统不堪重负，RateLimiter使用一个刷新周期的概念，限定在一个固定刷新周期内可处理的最大请求数量。若在某一个刷新周期内的请求数量已经达到最大，则本周期内接下来的请求将进入阻塞状态，如果在最大阻塞计时内新的刷新周期开启，则阻塞状态的请求将进入新的周期内进行处理。如最大的阻塞计时内新的刷新周期并未开启，则此时超出阻塞计时的那些请求将被直接拒绝。

## 断路器（CircuitBreaker）

![图解容错机制3](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-115103.jpg)

断路器（CircuitBreaker）相对于前面几个熔断机制更复杂，CircuitBreaker通常存在三种状态（CLOSE、OPEN、HALF_OPEN），并通过一个时间或数量窗口来记录当前的请求成功率或慢速率，从而根据这些指标来作出正确的容错响应。

当CircuitBreaker为CLOSE状态时客户端发起的请求将正常进入服务端系统，CircuitBreaker会计算出当前请求前的一个窗口里所有请求的异常率（失败率或慢速率），若异常率低于预期配置值，则系统将继续正常处理接下来的请求。当异常率不低于预期配置值时，此时服务端会进入OPEN状态，此时服务端将会暂时性的拒绝所有请求。在一段冷却时间（自定义配置）之后，服务端将自动进入HALF_OPEN状态，在半开状态服务端将尝试接受一定数量的请求（自定义配置），若这一定数量的请求的异常率低于预期，则此时服务端将再次恢复CLOSE状态，正常处理请求。而如果异常率还是高于预期则会继续退回到OPEN状态。

## 重试（Retry）

![图解容错机制4](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-115113.jpg)

重试机制比较简单，当服务端处理客户端请求异常时，服务端将会开启重试机制，重试期间内，服务端将每隔一段时间重试业务逻辑处理。 如果最大重试次数内成功处理业务，则停止重试，视为处理成功。如果在最大重试次数内处理业务逻辑依然异常，则此时系统将拒绝该请求。

## 总结

本文介绍了常用的几种容错机制，与其说是resilience4j中的容错机制不如直接把resilience4j去掉，因为可以看到这些机制原理并不只来源于某个库或只与某个特定库有关，它更是一种设计理念，他的通用性应该是跨语言的。此外虽然本文只介绍了这几种容错机制，但是如何使用他们完全取决于你的业务场景和架构设计。你甚至可以随意组合使用它们，并且**完全看不出**这些组合最后所展示的效果像哪一种机制，那也没有关系，怎么使用、怎么组合完全取决于你所处的技术/业务环境。

------

🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-115121.jpg)


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

