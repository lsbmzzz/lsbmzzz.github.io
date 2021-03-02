---
layout:     post
title:      SpringBoot2 单元测试笔记
subtitle:   SpringBoot2 单元测试笔记
date:       2021-03-02
author:     张三
header-img: img/spring.jpg
catalog: true
mathjax: true
tags:
    - SpringBoot

---

> JUnit5作为新版本的Junit框架，主要由 JUnit Platform、JUnit Jupiter、JUnit Vintage 三部分组成

- JUnit Platform ：在JVM上启动测试框架的基础，不仅支持JUnit自己的测试引擎，也可以接入其他测试引擎
- JUnit Jupiter ：提供了JUnit5的新的编程模型，是JUnit5的核心，包含一个用于在JUnit Platform上运行的测试引擎
- JUnit Vintage ：提供兼容JUnit4和JUnit3的测试引擎

SpringBoot2.4 默认移除了对Vintage的依赖，所以需要自行引入JUnit4才能使用

# 使用：

1. 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

2. 在测试类上添加 @SpringBootTest 注解

3. 测试
    - 编写测试方法，@Test标注
    - JUnit类具有Spring的功能（@Autowired、@Transactional等）

# 常用注解

- @Test :表示方法是测试方法。但是与JUnit4的@Test不同，他的职责非常单一不能声明任何属性，拓展的测试将会由Jupiter提供额外测试
- @ParameterizedTest :表示方法是参数化测试
- @RepeatedTest :表示方法可重复执行
- @DisplayName :为测试类或者测试方法设置展示名称
- @BeforeEach :表示在每个单元测试之前执行
- @AfterEach :表示在每个单元测试之后执行
- @BeforeAll :表示在所有单元测试之前执行
- @AfterAll :表示在所有单元测试之后执行
- @Tag :表示单元测试类别，类似于JUnit4中的@Categories
- @Disabled :表示测试类或测试方法不执行，类似于JUnit4中的@Ignore
- @Timeout :表示测试方法运行如果超过了指定时间将会返回错误
- @ExtendWith :为测试类或测试方法提供扩展类引用

```java
@SpringBootTest
public class JUnit5Test {

    @Autowired
    User user;
    
    @Tag("test")
    @DisplayName("测试DisplayName")
    @RepeatedTest(5) 
    public void test() {
        System.out.println(user);
    }
    
    @Test
    @Timeout(1)
    public void testTimeout() throws InterruptedException {
        Thread.sleep(1001);
        System.out.println("Test");
    }
    
    @Test
    @Disabled
    public void test3() {
        System.out.println("Test");
    }
    
    @BeforeEach
    public void beforeEach(){
        System.out.println("BeforeEach");
    }
    @AfterEach
    public void afterEach(){
        System.out.println("AfterEach");
    }
    @BeforeAll
    public static void beforeAll(){
        System.out.println("BeforeAll");
    }
    @AfterAll
    public static void afterAll(){
        System.out.println("AfterAll");
    }

}
```

# 断言机制

**断言** 用来对测试需要满足的条件进行验证。断言方法都是org.junit.jupiter.api.Assertions的静态方法。 

- 检查业务逻辑返回的数据是否合理。
- 所有的测试运行结束以后，会有一个详细的测试报告
- 断言失败 后面的代码都不会执行

JUnit5的内置断言分成几个类别：

1. 简单断言

| 方法 |  说明 |
| --- | --- |
| assertEquals | 判断两个对象或两个原始类型是否相等 |
| assertNotEquals | 判断两个对象或两个原始类型是否不相等 |
| assertSame | 判断两个对象引用是否指向同一个对象 |
| assertNotSame | 判断两个对象引用是否指向不同的对象 |
| assertTrue | 判断给定的布尔值是否为 true |
| assertFalse | 判断给定的布尔值是否为 false |
| assertNull | 判断给定的对象引用是否为 null |
| assertNotNull | 判断给定的对象引用是否不为 null |

```java
@SpringBootTest
public class AssertionsTest {
    @Test
    public void test() {
        Assertions.assertEquals(3, 1 + 1, "错误提示");
        Assertions.assertNotEquals(2, 1 + 1, "错误提示");
    }
}
```

2. 数组断言 assertArrayEquals 判断两个对象或原始类型的数组是否相等

```java
@SpringBootTest
public class AssertionsTest {
    @Test
    public void test() {
        Assertions.assertArrayEquals(arr1, arr2);
    }
}
```

3. 组合断言 assertAll 接受多个 org.junit.jupiter.api.Executable 函数式接口的实例作为要验证的断言，可以通过 lambda 表达式提供这些断言

```java
@SpringBootTest
public class AssertionsTest {
    @Test
    public void test() {
        Assertions.assertAll("Math",
            () -> Assertions.assertEquals(2, 1 + 1),
            () -> Assertions.assertTrue(1 > 0)
            );
    }
}
```

4. 异常断言 assertThrows 判断会发生异常

```java
@SpringBootTest
public class AssertionsTest {
    @Test
    public void test() {
        ArithmeticException exception = Assertions.assertThrows(
           //抛出断言异常
            ArithmeticException.class, () -> System.out.println(1 % 0));
    }
}
```

5. 超时断言 assertTimeout

```java
@SpringBootTest
public class AssertionsTest {
    @Test
    public void test() {
        Assertions.assertTimeout(Duration.ofMillis(1000), () -> Thread.sleep(500));

    }
}
```

6. 快速失败 通过 fail 方法直接使得测试失败

```java
@SpringBootTest
public class AssertionsTest {
    @Test
    public void test() {
        Assertions.fail("fail");
    }
}
```

# 前置条件 assumptions

> JUnit 5 中的前置条件类似于断言，不同之处在于不满足的断言会使得测试方法失败，而不满足的前置条件只会使得测试方法的执行终止。前置条件可以看成是测试方法执行的前提，当该前提不满足时，就没有继续执行的必要。

assumeTrue 和 assumFalse 确保给定的条件为 true 或 false，不满足条件会使得测试执行终止。assumingThat 的参数是表示条件的布尔值和对应的 Executable 接口的实现对象。只有条件满足时，Executable 对象才会被执行；当条件不满足时，测试执行并不会终止。

```java
@DisplayName("前置条件")
@SpringBootTest
public class AssumptionsTest {
    private final String environment = "DEV";

    @Test
    @DisplayName("simple")
    public void simpleAssume() {
        assumeTrue(Objects.equals(this.environment, "DEV"));
        assumeFalse(() -> Objects.equals(this.environment, "PROD"));
    }

    @Test
    @DisplayName("assume then do")
    public void assumeThenDo() {
        assumingThat(
                Objects.equals(this.environment, "DEV"),
                () -> System.out.println("In DEV")
        );
    }
}
```

# 嵌套测试

JUnit 5 可以通过 Java 中的内部类和@Nested 注解实现嵌套测试，从而可以更好的把相关的测试方法组织在一起。在内部类中可以使用@BeforeEach 和@AfterEach 注解，而且嵌套的层次没有限制。

**官网案例**

```java
@DisplayName("A stack")
class TestingAStackDemo {

    Stack<Object> stack;

    @Test
    @DisplayName("is instantiated with new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {

        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
```

# 参数化测试

可以使用不同参数进行多次测试

- @ValueSource 为参数化测试指定入参来源，支持八大基础类以及String类型,Class类型
- @NullSource 表示为参数化测试提供一个null的入参
- @EnumSource 表示为参数化测试提供一个枚举入参
- @CsvFileSource 表示读取指定CSV文件内容作为参数化测试入参
- @MethodSource 表示读取指定方法的返回值作为参数化测试入参(注意方法返回需要是一个流 Stream<String>)

只需要去实现ArgumentsProvider接口，任何外部文件都可以作为它的入参。

```java
@ParameterizedTest
@ValueSource(strings = {"one", "two", "three"})
@DisplayName("参数化测试1")
public void parameterizedTest1(String string) {
    System.out.println(string);
    Assertions.assertTrue(StringUtils.isNotBlank(string));
}
```

# JUnit4 版本迁移

在进行迁移的时候需要注意如下的变化：

- 注解在 org.junit.jupiter.api 包中，断言在 org.junit.jupiter.api.Assertions 类中，前置条件在 org.junit.jupiter.api.Assumptions 类中。
- 把@Before 和@After 替换成@BeforeEach 和@AfterEach。
- 把@BeforeClass 和@AfterClass 替换成@BeforeAll 和@AfterAll。
- 把@Ignore 替换成@Disabled。
- 把@Category 替换成@Tag。
- 把@RunWith、@Rule 和@ClassRule 替换成@ExtendWith。
