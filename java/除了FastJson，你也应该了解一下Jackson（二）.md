[返回上层](index)
# 除了FastJson，你也应该了解一下Jackson（二）

## 概览

上一篇文章介绍了Jackson中的映射器ObjectMapper，以及如何使用它来实现Json与Java对象之间的序列化和反序列化，最后介绍了Jackson中一些序列化/反序列化的高级特性。而本文将会介绍Jackson中的一些常用的（序列化/反序列化）注解，并且通过示例来演示如何使用这些注解，从而来提高我们在处理Json上的工作效率。

---

## 序列化注解

### @JsonAnyGetter

@JsonAnyGetter注解允许灵活地使用映射（键值对，如Map）字段作为标准属性。

我们声明如下Java类：

```java
@Data
@Accessors(chain = true)
public static class ExtendableBean {
    public String name;
    private Map<String, String> properties;

    @JsonAnyGetter
    public Map<String, String> getProperties() {
        return properties;
    }
}
```

编写测试代码，测试@JsonAnyGetter：

```java
@Test
public void testJsonAnyGetter() throws JsonProcessingException {
    ExtendableBean extendableBean = new ExtendableBean();
    Map<String, String> map = new HashMap<>();
    map.put("age", "13");
    extendableBean.setName("dxsn").setProperties(map);
    log.info(new ObjectMapper().writeValueAsString(extendableBean));
  	//打印：{"name":"dxsn","age":"13"}
    assertThat(new ObjectMapper().writeValueAsString(extendableBean)).contains("name");
    assertThat(new ObjectMapper().writeValueAsString(extendableBean)).contains("age");
}
```

如上，可以看properties属性中的键值对（Map）被扩展到了ExtendableBean的Json对象中。

### @JsonGetter

@JsonGetter注解是@JsonProperty注解的替代品，用来将一个方法标记为getter方法。

我们创建以下Java类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public static class MyBean {
    public int id;
    private String name;

    @JsonGetter("name")
    public String getTheName() {
        return name;
    }
}
```

如上，我们在类中声明了一个getTheName()方法，并且使用@JsonGetter("name")修饰，此时，该方法将会被Jackson认作是name属性的get方法。

编写测试代码：

```java
@Test
public void testJsonGetter() throws JsonProcessingException {
    MyBean myBean = new MyBean(1, "dxsn");
    String jsonStr = new ObjectMapper().writeValueAsString(myBean);
    log.info(jsonStr);
    assertThat(jsonStr).contains("id");
    assertThat(jsonStr).contains("name");
}
```

可以看到，jackson将私有属性name，也进行了序列化。

### @JsonPropertyOrder

我们可以使用@JsonPropertyOrder注解来指定Java对象的属性序列化顺序。

```java
@JsonPropertyOrder({"name", "id"})
//order by key's name
//@JsonPropertyOrder(alphabetic = true)
@Data
@Accessors(chain = true)
public static class MyOrderBean {
  public int id;
  public String name;
}
```

编写测试代码：

```java
@Test
public void testJsonPropertyOrder1() throws JsonProcessingException {
    MyOrderBean myOrderBean = new MyOrderBean().setId(1).setName("dxsn");
    String jsonStr = new ObjectMapper().writeValueAsString(myOrderBean);
    log.info(jsonStr);
    assertThat(jsonStr).isEqualTo("{\"name\":\"dxsn\",\"id\":1}");
}
```

如上，可以看到序列化得到的Json对象中属性的排列顺序正是我们在注解中指定的顺序。

### @JsonRawValue

@JsonRawValue注解可以指示Jackson按原样序列化属性。

在下面的例子中，我们使用@JsonRawValue嵌入一些定制的JSON作为一个实体的值:

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public static class RawBean {
    public String name;
    @JsonRawValue
    public String json;
}
```

编写测试代码：

```java
@Test
public void testJsonRawValue() throws JsonProcessingException {
    RawBean rawBean = new RawBean("dxsn", "{\"love\":\"true\"}");
    log.info(new ObjectMapper().writeValueAsString(rawBean));
  	//输出：{"name":"dxsn","json":{"love":"true"}}
    String result = new ObjectMapper().writeValueAsString(rawBean);
    assertThat(result).contains("dxsn");
    assertThat(result).contains("{\"love\":\"true\"}");
}
```

### @JsonValue

@JsonValue表示Jackson将使用一个方法来序列化整个实例。

下面我们创建一个枚举类：

```java
@AllArgsConstructor
public static enum TypeEnumWithValue {
    TYPE1(1, "Type A"), TYPE2(2, "Type 2");
    private Integer id;
    private String name;

    @JsonValue
    public String getName() {
        return name;
    }
}
```

如上，我们在getName()上使用@JsonValue进行修饰。

编写测试代码：

```java
@Test
public void testJsonValue() throws JsonProcessingException {
    String  jsonStr = new ObjectMapper().writeValueAsString(TypeEnumWithValue.TYPE2);
    log.info(jsonStr);
    assertThat(jsonStr).isEqualTo("Type 2");
}
```

可以看到，枚举类的对象序列化后的值即getName()方法的返回值。

### @JsonRootName

如果启用了包装（wrapping），则使用@JsonRootName注解可以指定要使用的根包装器的名称。

下面我们创建一个使用@JsonRootName修饰的Java类：

```java
@JsonRootName(value = "user")
@Data
@AllArgsConstructor
public static class UserWithRoot {
    public int id;
    public String name;
}
```

编写测试：

```java
@Test
public void testJsonRootName() throws JsonProcessingException {
    UserWithRoot userWithRoot = new UserWithRoot(1, "dxsn");
    ObjectMapper mapper = new ObjectMapper();
  	//⬇️重点！！！
    mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
  
    String result = mapper.writeValueAsString(userWithRoot);
    log.info(result);
  	//输出：{"user":{"id":1,"name":"dxsn"}}
    assertThat(result).contains("dxsn");
    assertThat(result).contains("user");
}
```

上面代码中，我们通过开启ObjectMapper的SerializationFeature.WRAP_ROOT_VALUE。可以看到UserWithRoot对象被序列化后的Json对象被包装在user中，而非单纯的`{"id":1,"name":"dxsn"}`。

### @JsonSerialize

@JsonSerialize注解表示序列化实体时要使用的自定义序列化器。

我们定义一个自定义的序列化器：

```java
public static class CustomDateSerializer extends StdSerializer<Date> {
    private static SimpleDateFormat formatter = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");

    public CustomDateSerializer() {
        this(null);
    }

    public CustomDateSerializer(Class<Date> t) {
        super(t);
    }

    @Override
    public void serialize(
        Date value, JsonGenerator gen, SerializerProvider arg2) throws IOException, JsonProcessingException {
        gen.writeString(formatter.format(value));
    }
}
```

使用自定义的序列化器，创建Java类：

```java
@Data
@AllArgsConstructor
public static class Event {
    public String name;
    @JsonSerialize(using = CustomDateSerializer.class)
    public Date eventDate;
}
```

编写测试代码：

```java
@Test
public void testJsonSerialize() throws ParseException, JsonProcessingException {
    SimpleDateFormat formatter = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    String toParse = "20-12-2014 02:30:00";
    Date date = formatter.parse(toParse);
    Event event = new Event("party", date);
    String result = new ObjectMapper().writeValueAsString(event);
    assertThat(result).contains(toParse);
}
```

可以看到，使用@JsonSerialize注解修饰指定属性后，将会使用指定的序列化器来序列化该属性。

---

## 反序列化注解

### @JsonCreator

我们可以使用@JsonCreator注解来优化/替换反序列化中使用的构造器/工厂。

当我们需要反序列化一些与我们需要获取的目标实体不完全匹配的JSON时，它非常有用。

现在，有如下一个Json对象：

```json
{"id":1,"theName":"My bean"}
```

我们声名了一个Java类：

```java
@Data
public static class BeanWithCreator {
    private int id;
    private String name;
}
```

此时，在我们的目标实体中没有theName字段，只有name字段。现在，我们不想改变实体本身，此时可以通过使用@JsonCreator和@JsonProperty注解来修饰构造函数:

```java
@Data
public static class BeanWithCreator {
    private int id;
    private String name;

    @JsonCreator
    public BeanWithCreator(@JsonProperty("id") int id, @JsonProperty("theName") String name) {
        this.id = id;
        this.name = name;
    }
}
```

编写测试：

```java
@Test
public void beanWithCreatorTest() throws JsonProcessingException {
    String str = "{\"id\":1,\"theName\":\"My bean\"}";
    BeanWithCreator bean = new ObjectMapper()
        .readerFor(BeanWithCreator.class)
        .readValue(str);
 	 	assertThat(bean.getId()).isEqualTo(1);
    assertThat(bean.getName()).isEqualTo("My bean");
}
```

可以看到，即使Json对象中的字段名和实体类中不一样，但由于我们手动指定了映射字段的名字，从而反序列化成功。

### @JacksonInject

@JacksonInject表示java对象中的属性将通过注入来赋值，而不是从JSON数据中获得其值。

创建如下实体类，其中有字段被@JacksonInject修饰：

```java
public static class BeanWithInject {
    @JacksonInject
    public int id;
    public String name;
}
```

编写测试：

```java
@Test
public void jacksonInjectTest() throws JsonProcessingException {
    String json = "{\"name\":\"dxsn\"}";
    InjectableValues inject = new InjectableValues.Std()
        .addValue(int.class, 1);
    BeanWithInject bean = new ObjectMapper().reader(inject)
        .forType(BeanWithInject.class)
        .readValue(json);
    assertThat(bean.id).isEqualTo(1);
    assertThat(bean.name).isEqualTo("dxsn");
}
```

如上，我们在测试中将json字符串（仅存在name字段）进行反序列化，其中id通过注入的方式对属性进行赋值。

### @JsonAnySetter

@JsonAnySetter允许我们灵活地使用映射（键值对、Map）作为标准属性。在反序列化时，JSON的属性将被添加到映射中。

创建一个带有@JsonAnySetter的实体类：

```java
public static class ExtendableBean {
    public String name;
    public Map<String, String> properties;

    @JsonAnySetter
    public void add(String key, String value) {
        if (properties == null) {
            properties = new HashMap<>();
        }
        properties.put(key, value);
    }
}
```

编写测试：

```java
@Test
public void testJsonAnySetter() throws JsonProcessingException {
    String json = "{\"name\":\"dxsn\", \"attr2\":\"val2\", \"attr1\":\"val1\"}";
    ExtendableBean extendableBean = new ObjectMapper().readerFor(ExtendableBean.class).readValue(json);
    assertThat(extendableBean.name).isEqualTo("dxsn");
    assertThat(extendableBean.properties.size()).isEqualTo(2);
}
```

可以看到，json对象中的attr1，attr2属性在反序列化之后进入了properties。

### @JsonSetter

@JsonSetter是@JsonProperty的替代方法，它将方法标记为属性的setter方法。
当我们需要读取一些JSON数据，但目标实体类与该数据不完全匹配时，这非常有用，因此我们需要优化使其适合该数据。

创建如下实体类：

```java
@Data
public static class MyBean {
    public int id;
    private String name;

    @JsonSetter("name")
    public void setTheName(String name) {
        this.name = "hello " + name;
    }
}
```

编写测试：

```java
@Test
public void testJsonSetter() throws JsonProcessingException {
    String json = "{\"id\":1,\"name\":\"dxsn\"}";
    MyBean bean = new ObjectMapper().readerFor(MyBean.class).readValue(json);
    assertThat(bean.getName()).isEqualTo("hello dxsn");
}
```

可以看到，json对象中的name属性为“dxsn”，我们通过在MyBean类中定义了使用@JsonSetter("name")注解修饰的方法，这表明该类的对象在反序列话的时候，name属性将来自此方法。最后MyBean对象中name的值变为了hello dxsn。

### @JsonDeserialize

@JsonDeserialize注解指定了在反序列化的时候使用的反序列化器。

如下，定义了一个自定义的反序列化器：

```java
public static class CustomDateDeserializer extends StdDeserializer<Date> {
    private static SimpleDateFormat formatter = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");

    public CustomDateDeserializer() {
        this(null);
    }

    public CustomDateDeserializer(Class<?> vc) {
        super(vc);
    }

    @Override
    public Date deserialize(JsonParser jsonparser, DeserializationContext context) throws IOException {
        String date = jsonparser.getText();
        try {
            return formatter.parse(date);
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }
}
```

创建一个使用@JsonDeserialize(using = CustomDateDeserializer.class)修饰的实体类：

```java
public static class Event {
    public String name;
    @JsonDeserialize(using = CustomDateDeserializer.class)
    public Date eventDate;
}
```

编写测试：

```java
@Test
public void whenDeserializingUsingJsonDeserialize_thenCorrect()
    throws IOException {
    String json = "{\"name\":\"party\",\"eventDate\":\"20-12-2014 02:30:00\"}";
    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    Event event = new ObjectMapper().readerFor(Event.class).readValue(json);
    assertThat(event.name).isEqualTo("party");
    assertThat(event.eventDate).isEqualTo(df.format(event.eventDate));

}
```

可以看到，在Event对象中，eventDate属性通过自定义的反序列化器，将“20-12-2014 02:30:00”反序列化成了Date对象。

### @JsonAlias

@JsonAlias在反序列化期间为属性定义一个或多个替代名称。让我们通过一个简单的例子来看看这个注解是如何工作的:

```java
@Data
public static class AliasBean {
    @JsonAlias({"fName", "f_name"})
    private String firstName;
    private String lastName;
}
```

如上，我们编写了一个使用@JsonAlias修饰的AliasBean实体类。

编写测试：

```java
@Test
public void whenDeserializingUsingJsonAlias_thenCorrect() throws IOException {
    String json = "{\"fName\": \"John\", \"lastName\": \"Green\"}";
    AliasBean aliasBean = new ObjectMapper().readerFor(AliasBean.class).readValue(json);
    assertThat(aliasBean.getFirstName()).isEqualTo("John");
}
```

可以看到，即使json对象中的字段名是fName，但是由于在AliasBean中使用@JsonAlias修饰了firstName属性，并且指定了两个别名。所以反序列化之后fName被映射到AliasBean对象的firstName属性上。

---

## 更多

除上述注解之外，Jackson还提供了很多额外的注解，这里不一一列举，接下来会例举几个常用的注解：

-  **@JsonProperty**：可以在类的指定属性上添加@JsonProperty注解来表示其对应在JSON中的属性名。
- **@JsonFormat**：此注解在序列化对象中的日期/时间类型属性时可以指定一种字符串格式输出，如：@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = “dd-MM-yyyy hh:mm:ss”)。
- **@JsonUnwrapped**：@JsonUnwrapped定义了在序列化/反序列化时应该被扁平化的值。
- **@JsonIgnore**:序列化/反序列化时忽略被修饰的属性。
- ......

---

## 总结

本文主要介绍了Jackson常用的序列化/反序列化注解，最后介绍了几个常用的通用注解。Jackson中提供的注解除了本文列举的还有很多很多，使用注解可以让我们的序列化/反序列化工作更加轻松。如果你想将某库换成Jackson，希望这篇文章可以帮到你。



本文涉及的代码地址：https://gitee.com/jeker8chen/jackson-annotation-demo



---

**关注笔者公众号，推送各类原创/优质技术文章 ⬇️**

![WechatIMG6](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-161240.jpg)



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

