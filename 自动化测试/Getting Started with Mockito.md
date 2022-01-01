[返回上层](index)
# Getting Started with Mockito

## 准备

在我们进一步讨论之前，让我们探索几种不同的方法来启用Mockito测试中注释的使用。 

**方式一** ***MockitoJUnitRunner***

我们拥有的第一个选择是使用MockitoJUnitRunner注释JUnit测试，如以下示例所示：

```java
@RunWith(MockitoJUnitRunner.class)
public class MockitoAnnotationTest {
    ...
}
```

**方式二 *MockitoAnnotations.initMocks()***

另外，我们也可以通过调用MockitoAnnotations.initMocks（）来以编程方式启用Mockito注释：

```java
@Before
public void init() {
    MockitoAnnotations.initMocks(this);
}
```

**方式三 *MockitoJUnit.rule()***

或者，我们可以使用MockitoJUnit.rule（），如下所示：

```java
public class MockitoInitWithMockitoJUnitRuleUnitTest {
 
    @Rule
    public MockitoRule initRule = MockitoJUnit.rule();
 
    ...
}
```

在这种情况下，我们必须记住将Rule成员设置为public。

---

## Mockito注解介绍

### @Mock  Annotation

Mockito中使用最广泛的注释是@Mock。我们可以使用@Mock来创建和注入模拟实例，而不必手动调用Mockito.mock。

在下面的示例中–我们将使用手动方式创建模拟ArrayList，而不使用@Mock注释：

```java
@Test
public void whenNotUseMockAnnotation_thenCorrect() {
    List mockList = Mockito.mock(ArrayList.class);
     
    mockList.add("one");
    Mockito.verify(mockList).add("one");
    assertEquals(0, mockList.size());
 
    Mockito.when(mockList.size()).thenReturn(100);
    assertEquals(100, mockList.size());
}
```

现在我们将做同样的事情，但是我们将使用@Mock注释注入模拟：

```java
@Mock
List<String> mockedList;
 
@Test
public void whenUseMockAnnotation_thenMockIsInjected() {
    mockedList.add("one");
    Mockito.verify(mockedList).add("one");
    assertEquals(0, mockedList.size());
 
    Mockito.when(mockedList.size()).thenReturn(100);
    assertEquals(100, mockedList.size());
}
```

请注意，在两个示例中，我们如何与模拟对象进行交互并验证其中的某些交互，只是为了确保模拟行为正确。



### @Spy Annotation

现在–让我们看看如何使用@Spy注释监视现有实例。

在下面的示例中，我们使用旧方法创建一个列表的spy，而不使用@spy注释：

```java
@Test
public void whenNotUseSpyAnnotation_thenCorrect() {
    List<String> spyList = Mockito.spy(new ArrayList<String>());
     
    spyList.add("one");
    spyList.add("two");
 
    Mockito.verify(spyList).add("one");
    Mockito.verify(spyList).add("two");
 
    assertEquals(2, spyList.size());
 
    Mockito.doReturn(100).when(spyList).size();
    assertEquals(100, spyList.size());
}
```

现在让我们做同样的事情–在列表上监视–但是使用@spy注释：

```java
@Spy
List<String> spiedList = new ArrayList<String>();
 
@Test
public void whenUseSpyAnnotation_thenSpyIsInjectedCorrectly() {
    spiedList.add("one");
    spiedList.add("two");
 
    Mockito.verify(spiedList).add("one");
    Mockito.verify(spiedList).add("two");
 
    assertEquals(2, spiedList.size());
 
    Mockito.doReturn(100).when(spiedList).size();
    assertEquals(100, spiedList.size());
}
```

请注意，像以前一样，我们在这里与间谍对象进行交互以确保其行为正确。在此示例中，我们：

- 使用实际方法spiedList.add（）将元素添加到spiedList。
- 使用Mockito.doReturn（）将spiedList.size（）方法存根以返回100而不是2。



### @Captor Annotation

接下来–让我们看看如何使用@Captor注释创建ArgumentCaptor实例。

在下面的示例中–我们使用旧方法创建ArgumentCaptor，而不使用@Captor注释：

```java
@Test
public void whenNotUseCaptorAnnotation_thenCorrect() {
    List mockList = Mockito.mock(List.class);
    ArgumentCaptor<String> arg = ArgumentCaptor.forClass(String.class);
 
    mockList.add("one");
    Mockito.verify(mockList).add(arg.capture());
 
    assertEquals("one", arg.getValue());
}
```

现在，出于相同的目的，使用@Captor –创建一个ArgumentCaptor实例：

```java
@Mock
List mockedList;
 
@Captor
ArgumentCaptor argCaptor;
 
@Test
public void whenUseCaptorAnnotation_thenTheSam() {
    mockedList.add("one");
    Mockito.verify(mockedList).add(argCaptor.capture());
 
    assertEquals("one", argCaptor.getValue());
}
```

请注意，当我们取出配置逻辑时，测试如何变得更简单、更可读。



### @InjectMocks Annotation

现在-让我们讨论如何使用@InjectMocks注释-将模拟字段自动注入到测试对象中。

在以下示例中，我们使用@InjectMocks将模拟wordMap注入MyDictionary dic：

```java
@Mock
Map<String, String> wordMap;
 
@InjectMocks
MyDictionary dic = new MyDictionary();
 
@Test
public void whenUseInjectMocksAnnotation_thenCorrect() {
    Mockito.when(wordMap.get("aWord")).thenReturn("aMeaning");
 
    assertEquals("aMeaning", dic.getMeaning("aWord"));
}
```

这是MyDictionary类：

```java
public class MyDictionary {
    Map<String, String> wordMap;
 
    public MyDictionary() {
        wordMap = new HashMap<String, String>();
    }
    public void add(final String word, final String meaning) {
        wordMap.put(word, meaning);
    }
    public String getMeaning(final String word) {
        return wordMap.get(word);
    }
}
```



## Notes

最后–这是有关Mockito注释的一些说明：

- Mockito的注释将重复的模拟创建代码减至最少
- 它们使测试更具可读性
- @InjectMocks支持同时注入@Spy和@Mock实例



**在本快速教程中，我们展示了Mockito库中注释的基础。**



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

