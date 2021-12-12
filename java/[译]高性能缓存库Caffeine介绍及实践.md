[返回上层](index)

# [译]高性能缓存库Caffeine介绍及实践

## 概览

本文我们将介绍[Caffeine](https://github.com/ben-manes/caffeine)-一个Java高性能缓存库。缓存和Map之间的一个根本区别是缓存会将储存的元素逐出。逐出策略决定了在什么时间应该删除哪些对象，逐出策略直接影响缓存的命中率，这是缓存库的关键特征。Caffeine使用**Window TinyLfu**逐出策略，该策略提供了接近最佳的命中率。

## 添加依赖

首先在pom.xml文件中添加Caffeine相关依赖：

```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>2.5.5</version>
</dependency>
```

您可以在Maven Central上找到最新版本的Caffeine。

## 缓存填充

让我们集中讨论Caffeine的三种缓存填充策略：手动，同步加载和异步加载。

首先，让我们创建一个用于存储到缓存中的DataObject类：

```java
class DataObject {
    private final String data;
 
    private static int objectCounter = 0;
    // standard constructors/getters
     
    public static DataObject get(String data) {
        objectCounter++;
        return new DataObject(data);
    }
}
```

### 手动填充

在这种策略中，我们手动将值插入缓存中，并在后面检索它们。

让我们初始化缓存：

```java
Cache<String, DataObject> cache = Caffeine.newBuilder()
  .expireAfterWrite(1, TimeUnit.MINUTES)
  .maximumSize(100)
  .build();
```

现在，我们可以使用getIfPresent方法从缓存中获取值。如果缓存中不存在该值，则此方法将返回null：

```java
String key = "A";
DataObject dataObject = cache.getIfPresent(key);
 
assertNull(dataObject);
```

我们可以使用put方法手动将值插入缓存：

```java
cache.put(key, dataObject);
dataObject = cache.getIfPresent(key);
 
assertNotNull(dataObject);
```

我们还可以使用get方法获取值，该方法将Lambda函数和键作为参数。如果缓存中不存在此键，则此Lambda函数将用于提供返回值，并且该返回值将在计算后插入缓存中：

```java
dataObject = cache
  .get(key, k -> DataObject.get("Data for A"));
 
assertNotNull(dataObject);
assertEquals("Data for A", dataObject.getData());
```

get方法以原子方式（atomically）执行计算。这意味着计算将只进行一次，即使多个线程同时请求该值。这就是为什么使用get比getIfPresent更好。

有时我们需要手动使某些缓存的值无效：

```java
cache.invalidate(key);
dataObject = cache.getIfPresent(key);
 
assertNull(dataObject);
```

### 同步加载

这种加载缓存的方法具有一个函数，该函数用于初始化值，类似于手动策略的get方法。让我们看看如何使用它。

首先，我们需要初始化缓存：

```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumSize(100)
  .expireAfterWrite(1, TimeUnit.MINUTES)
  .build(k -> DataObject.get("Data for " + k));
```

现在，我们可以使用get方法检索值：

```java
DataObject dataObject = cache.get(key);
 
assertNotNull(dataObject);
assertEquals("Data for " + key, dataObject.getData());
```

我们还可以使用getAll方法获得一组值：

```java
Map<String, DataObject> dataObjectMap 
  = cache.getAll(Arrays.asList("A", "B", "C"));
 
assertEquals(3, dataObjectMap.size());
```

从传递给build方法的初始化函数中检索值。这样就可以通过缓存在来装饰访问值。

### 异步加载

该策略与先前的策略相同，但是异步执行操作，并返回保存实际值的CompletableFuture：

```java
AsyncLoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumSize(100)
  .expireAfterWrite(1, TimeUnit.MINUTES)
  .buildAsync(k -> DataObject.get("Data for " + k));
```

考虑到它们返回CompletableFuture的事实，我们可以以相同的方式使用get和getAll方法：

```java
String key = "A";
 
cache.get(key).thenAccept(dataObject -> {
    assertNotNull(dataObject);
    assertEquals("Data for " + key, dataObject.getData());
});
 
cache.getAll(Arrays.asList("A", "B", "C"))
  .thenAccept(dataObjectMap -> assertEquals(3, dataObjectMap.size()));
```

CompletableFuture具有丰富而有用的API，您可以在[本文](https://www.baeldung.com/java-completablefuture)中了解更多信息。

## 逐出元素

Caffeine具有三种元素逐出策略：基于容量，基于时间和基于引用。

### 基于容量的逐出

这种逐出发生在超过配置的缓存容量大小限制时。有两种获取容量当前占用量的方法，计算缓存中的对象数量或获取它们的权重。

让我们看看如何处理缓存中的对象。初始化高速缓存时，其大小等于零：

```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumSize(1)
  .build(k -> DataObject.get("Data for " + k));
 
assertEquals(0, cache.estimatedSize());
```

当我们添加一个值时，大小显然会增加：

```java
cache.get("A");
 
assertEquals(1, cache.estimatedSize());
```

我们可以将第二个值添加到缓存中，从而导致删除第一个值：

```java
cache.get("B");
cache.cleanUp();
 
assertEquals(1, cache.estimatedSize());
```

值得一提的是，在获取缓存大小之前，我们先调用cleanUp方法。这是因为缓存逐出是异步执行的，并且此方法有助于等待逐出操作的完成。

我们还可以传递一个***weigher\***函数来指定缓存值的权重大小：

```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumWeight(10)
  .weigher((k,v) -> 5)
  .build(k -> DataObject.get("Data for " + k));
 
assertEquals(0, cache.estimatedSize());
 
cache.get("A");
assertEquals(1, cache.estimatedSize());
 
cache.get("B");
assertEquals(2, cache.estimatedSize());
```

当权重超过10时，将按照时间顺序从缓存中删除多余的值：

```java
cache.get("C");
cache.cleanUp();
 
assertEquals(2, cache.estimatedSize());
```

### 基于时间的逐出

此逐出策略基于元素的到期时间，并具有三种类型：

- **Expire after access** — 自上次读取或写入发生以来，经过过期时间之后该元素到期。
- **Expire after write** — 自上次写入以来，在经过过期时间之后该元素过期。
- **Custom policy** — 通过Expiry实现分别计算每个元素的到期时间。

让我们使用expireAfterAccess方法配置访问后过期策略：

```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .expireAfterAccess(5, TimeUnit.MINUTES)
  .build(k -> DataObject.get("Data for " + k));
```

要配置写后过期策略，我们使用expireAfterWrite方法：

```java
cache = Caffeine.newBuilder()
  .expireAfterWrite(10, TimeUnit.SECONDS)
  .weakKeys()
  .weakValues()
  .build(k -> DataObject.get("Data for " + k));
```

要初始化自定义策略，我们需要实现Expiry接口：

```java
cache = Caffeine.newBuilder().expireAfter(new Expiry<String, DataObject>() {
    @Override
    public long expireAfterCreate(
      String key, DataObject value, long currentTime) {
        return value.getData().length() * 1000;
    }
    @Override
    public long expireAfterUpdate(
      String key, DataObject value, long currentTime, long currentDuration) {
        return currentDuration;
    }
    @Override
    public long expireAfterRead(
      String key, DataObject value, long currentTime, long currentDuration) {
        return currentDuration;
    }
}).build(k -> DataObject.get("Data for " + k));
```

### 基于引用的逐出

我们可以将缓存配置为允许垃圾回收缓存的键或值。为此，我们将为键和值配置WeakRefence的用法，并且我们只能为值的垃圾收集配置为SoftReference。

当对象没有任何强引用时，WeakRefence用法允许对对象进行垃圾回收。 SoftReference允许根据JVM的全局“最近最少使用”策略对对象进行垃圾收集。有关Java引用的更多详细信息，请参见[此处](https://www.baeldung.com/java-weakhashmap)。

我们应该使用Caffeine.weakKeys()，Caffeine.weakValues()和Caffeine.softValues()来启用每个选项：

```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .expireAfterWrite(10, TimeUnit.SECONDS)
  .weakKeys()
  .weakValues()
  .build(k -> DataObject.get("Data for " + k));
 
cache = Caffeine.newBuilder()
  .expireAfterWrite(10, TimeUnit.SECONDS)
  .softValues()
  .build(k -> DataObject.get("Data for " + k));
```

## 刷新缓存

可以将缓存配置为在定义的时间段后自动刷新元素。让我们看看如何使用refreshAfterWrite方法执行此操作：

```java
Caffeine.newBuilder()
  .refreshAfterWrite(1, TimeUnit.MINUTES)
  .build(k -> DataObject.get("Data for " + k));
```

在这里，我们应该了解expireAfter和refreshAfter之间的区别。前者当请求过期元素时，执行将阻塞，直到build()计算出新值为止。

但是后者将返回旧值并异步计算出新值并插入缓存中，此时被刷新的元素的过期时间将重新开始计时计算。

## 统计

Caffeine可以记录有关缓存使用情况的统计信息：

```java
LoadingCache<String, DataObject> cache = Caffeine.newBuilder()
  .maximumSize(100)
  .recordStats()
  .build(k -> DataObject.get("Data for " + k));
cache.get("A");
cache.get("A");
 
assertEquals(1, cache.stats().hitCount());
assertEquals(1, cache.stats().missCount());
```

我们将recordStats传递给它，recordStats创建StatsCounter的实现。每次与统计相关的更改都将推送给此对象。

## 总结

在本文中，我们熟悉了Java的Caffeine缓存库。我们了解了如何配置和填充缓存，以及如何根据需要选择适当的过期或刷新策略。



原文地址：https://www.baeldung.com/java-caching-caffeine

------

🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟🌟

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-12-031744.jpg)
