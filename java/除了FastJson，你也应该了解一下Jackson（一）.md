[返回上层](index)
# 除了FastJson，你也应该了解一下Jackson（一）

在上月末的时候收到一条关于fastjson安全漏洞的消息，突然想到先前好像已经有好多次这样的事件了（在fastjson上面）。关于安全方面，虽然中枪的机率微小，但是在这个信息越来越复杂的时代，安全性也变得越来越重要，就像DevSecOps的诞生，在软件交付的整个价值流中我们也需要注重安全这方面。当然我们现在不谈关于FastJson的优劣，因为我们本文的目标是让大家了解和掌握Jackson。

---

## 概览

Jackson是一个非常流行和高效的基于Java的库，它可以序列化java对象或将java对象映射到JSON，反之亦然。当然除了Jackson，在Java中同类型的优秀的库也有很多，比如：

- [Gson](https://www.baeldung.com/java-json#gson)
- [json-io](https://www.baeldung.com/java-json#jsonio)
- [Genson](https://www.baeldung.com/java-json#genson)

关于哪一个最好或者哪一个最流行，没有明确的答案。技术的种类繁多，每个人对与不同技术的态度也不一样。言归正传，文章主要还是讨论Jackson的。本文主要讲解我们处理Json中最常见的两个操作：

- 将Java对象序列化为JSON
- JSON字符串反序列化为Java对象

---

## 引入依赖

由于在Spring/SpringBoot中很多组件已经自带了Jackson库，所以很多情况下不需要手动引入Jackson的依赖。

手动引入依赖：

```xml
<dependency> 
  <groupId>com.fasterxml.jackson.core</groupId> 
  <artifactId>jackson-databind</artifactId> 
  <version>2.9.8</version>
</dependency>
```

这个依赖关系还将传递地向类路径添加以下库:

> 1. jackson-annotations-2.9.8.jar 
> 2. jackson-core-2.9.8.jar
> 3. jackson-databind-2.9.8.jar

---

## JavaObject to Json

### ObjectMapper

ObjectMapper是一个映射器(或数据绑定器或编解码器)，提供了在Java对象(bean的实例)和JSON之间进行转换的功能。

#### 首先定义一个简单的Java类

```java
public class Car {
    private String color;
    private String type;
    // standard getters setters
}
```

#### 将Java对象转换成Json

我们使用ObjectMapper的*writeValue*相关Api来对Java对象进行序列化操作

```java
ObjectMapper objectMapper = new ObjectMapper();
Car car = new Car("blue","c1");
System.out.println(objectMapper.writeValueAsString(car));
```

此时输出

```json
{"color":"blue","type":"c1"}
```

#### 更多

ObjectMapper的*writeValue*相关Api还提供了很多便利的Json序列化操作方法，比如：将对象序列化成Json字节数组的`writeValueAsBytes()`方法、自定义输出源的`writeValue()`方法...

```java
ObjectMapper objectMapper = new ObjectMapper();
Car car = new Car("blue","c1");
objectMapper.writeValue(new File("./xxx.txt"),car);
```

运行上述代码，Java对象的序列化Json将被输出到xxx.txt文件。

![image-20200605232424859](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-161200.jpg)

---

## Json to JavaObject

### 将Json String转换成Java Object

```java
ObjectMapper objectMapper = new ObjectMapper();
String json = "{\"color\":\"blue\",\"type\":\"c1\"}";
Car car = objectMapper.readValue(json, Car.class);
```

readValue()方法也接受其他形式的输入，比如包含JSON字符串的文件:

```java
ObjectMapper objectMapper = new ObjectMapper();
Car car = objectMapper.readValue(new File("./xxx.txt"), Car.class);
System.out.println(car);
```

---

## JSON to Jackson JsonNode

### JsonNode

一个JSON可以被解析成一个JsonNode对象，用来从一个特定的节点检索数据.

使用readTree()方法，我们可以将Json字符串转换成JsonNode

```java
ObjectMapper objectMapper = new ObjectMapper();
String json = "{ \"color\" : \"Black\", \"type\" : \"FIAT\" }";
JsonNode jsonNode = objectMapper.readTree(json);
System.out.println(jsonNode.findValue("type").asText());
// 打印出“FAIT”
```

---

## JSONArrayString to JavaList

```java
ObjectMapper objectMapper = new ObjectMapper();
String jsonCarArray =
    "[{ \"color\" : \"Black\", \"type\" : \"BMW\" }, { \"color\" : 3. \"Red\", \"type\" : \"FIAT\" }]";
List<Car> listCar = objectMapper.readValue(jsonCarArray, new TypeReference<List<Car>>() {});
```

---

## JSONString to JavaMap

```java
ObjectMapper objectMapper = new ObjectMapper();
String json = "{ \"color\" : \"Black\", \"type\" : \"BMW\" }";
Map<String, Object> map = objectMapper.readValue(json, new TypeReference<Map<String, Object>>() {
});
```

---

💡：<u>*Jackson库最大的优点之一是高度可定制的序列化和反序列化过程。接下来将介绍一些**高级特性**，其中输入或输出JSON响应可以与生成或使用响应的对象不同。*</u>

## 配置序列化和反序列化特性

```java
String jsonString = "{ \"color\" : \"Black\", \"type\" : \"Fiat\", \"year\" :\"1970\" }";
```

假设使用如上json字符串来反序列化成Java对象，按照默认解析过程将导致UnrecognizedPropertyException异常，因为其中存在Car类中未包含的新字段year。

**通过配置序列化和反序列化特性来解决此问题：**

```java
ObjectMapper objectMapper = new ObjectMapper();
String jsonString = "{ \"color\" : \"Black\", \"type\" : \"Fiat\", \"year\" :\"1970\" }";
objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
Car car = objectMapper.readValue(jsonString, Car.class);
```

如上，我们在ObjectMapper中配置了`DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES=false`，从而实现忽略新的字段。

**类似：**另一个选项`FAIL_ON_NULL_FOR_PRIMITIVES`，它定义了是否允许原始值的空值；`FAIL_ON_NUMBERS_FOR_ENUM`控制是否允许enum值被序列化/反序列化为数字......

---

## 自定义序列化器或反序列化器

### 自定义序列化器

```java
public static class CustomCarSerializer extends StdSerializer<Car> {
    public CustomCarSerializer() {
        this(null);
    }

    public CustomCarSerializer(Class<Car> t) {
        super(t);
    }

    @Override
    public void serialize(Car car, JsonGenerator jsonGenerator, SerializerProvider serializer) throws IOException {
        jsonGenerator.writeStartObject();
        jsonGenerator.writeStringField("car_brand", car.getType());
        jsonGenerator.writeEndObject();
    }
}
```

如上，通过继承StdSerializer类，我们实现了一个自定义的序列化器。

使用自定义的序列化器：

```java
ObjectMapper mapper = new ObjectMapper();
SimpleModule module = new SimpleModule("CustomCarSerializer", new Version(1, 0, 0, null, null, null));
module.addSerializer(Car.class, new CustomCarSerializer());
mapper.registerModule(module);
Car car = new Car("yellow", "enault");
System.out.println(mapper.writeValueAsString(car));
//输出{"car_brand":"enault"}
```

### 自定义反序列化器

```java
public static class CustomCarDeserializer extends StdDeserializer<Car> {

        public CustomCarDeserializer() {
            this(null);
        }

        protected CustomCarDeserializer(Class<?> vc) {
            super(vc);
        }

        @Override
        public Car deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {
            Car car = new Car();
            ObjectCodec codec = p.getCodec();
            JsonNode node = codec.readTree(p);
            // try catch block
            JsonNode colorNode = node.get("color");
            String color = colorNode.asText();
            car.setColor(color);
            return car;
        }
    }
```

如上，通过继承StdDeserializer类，我们实现了一个自定义的序列化器。

使用自定义的反序列化器：

```java
String json = "{ \"color\" : \"Black\", \"type\" : \"BMW\"}";
ObjectMapper mapper = new ObjectMapper();
SimpleModule module = new SimpleModule("CustomCarDeserializer", new Version(1, 0, 0, null, null, null));
module.addDeserializer(Car.class, new CustomCarDeserializer());
mapper.registerModule(module);
Car car = mapper.readValue(json, Car.class);
//此时的car {color='Black', type='null'}
```

---

## 处理时间格式

⚠️：此处仅展示对于Java8的LocalDate&LocalDateTime的处理

首先创建一个带日期时间字段的Car类

```java
public class Car {
    private String color;
    private String type;
  	@JsonFormat(pattern = "yyyy-MM-dd")
    private LocalDateTime produceTime;
    // standard getters setters
}
```

自定义时间格式处理

```java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.findAndRegisterModules();
Car car = new Car().setColor("blue").setType("c1").setProduceTime(LocalDateTime.now());
String carAsString = objectMapper.writeValueAsString(car);
System.out.println(carAsString);
//此时输出：{"color":"blue","type":"c1","produceTime":"2020-06-06"}
```

---

## 处理集合

DeserializationFeature类提供的另一个小但有用的特性是能够从JSON数组响应生成我们想要的集合类型。

```java
String jsonCarArray = "[{ \"color\" : \"Black\", \"type\" : \"BMW\"}, { \"color\" : \"Red\", \"type\" : \"FIAT\"}]";
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.configure(DeserializationFeature.USE_JAVA_ARRAY_FOR_JSON_ARRAY, true);
Car[] cars = objectMapper.readValue(jsonCarArray, Car[].class);
```

如上，我们将一个JsonArray字符串转换成了对象数组。

我们也可以将其转换成集合：

```java
String jsonCarArray = "[{ \"color\" : \"Black\", \"type\" : \"BMW\"}, { \"color\" : \"Red\", \"type\" : \"FIAT\"}]";
ObjectMapper objectMapper = new ObjectMapper();
List<Car> listCar = objectMapper.readValue(jsonCarArray, new TypeReference<List<Car>>(){});
```

---

## 总结

Jackson是一个可靠而成熟的用于Java的JSON序列化/反序列化库。ObjectMapper API提供了一种简单的方法来解析和生成JSON响应对象，具有很大的灵活性。



---

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-161211.jpg)



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

