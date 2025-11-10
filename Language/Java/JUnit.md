JUnit 是 Java 生态中最流行的单元测试框架，也是 xUnit 家族的重要成员，对现代软件开发中的测试驱动开发（TDD）实践产生了深远影响。


### 基本注解

| 注解             | 说明            |
| -------------- | ------------- |
| `@Test`        | 标记测试方法        |
| `@BeforeEach`  | 每个测试方法前执行     |
| `@AfterEach`   | 每个测试方法后执行     |
| `@BeforeAll`   | 所有测试前执行(静态方法) |
| `@AfterAll`    | 所有测试后执行(静态方法) |
| `@DisplayName` | 自定义测试显示名称     |
| `@Disabled`    | 禁用测试          |
| `@Nested`      | 嵌套测试类         |

### 测试示例

```java
import org.junit.jupiter.api.*;

class CalculatorTest {
    
    Calculator calculator;
    
    @BeforeEach
    void setUp() {
        calculator = new Calculator();
    }
    
    @Test
    @DisplayName("加法测试")
    void testAddition() {
        assertEquals(4, calculator.add(2, 2));
    }
    
    @Test
    @Disabled("暂未实现")
    void testSubtraction() {
        // TODO
    }
}
```


### 与 Mock 框架集成

JUnit 通常与 Mock 框架配合使用：
```java
@ExtendWith(MockitoExtension.class)
class ServiceTest {
    
    @Mock
    private Repository repository;
    
    @InjectMocks
    private Service service;
    
    @Test
    void testFindById() {
        when(repository.findById(1L)).thenReturn(new Entity());
        assertNotNull(service.findById(1L));
    }
}
```

