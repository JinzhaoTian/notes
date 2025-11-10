成熟稳定的测试框架，源自 Java 的 JUnit。


**特点**

- 丰富的断言 API
- 支持参数化测试
- 有 SetUp 和 TearDown 特性
- 广泛的插件生态系统

**示例代码**

```c#
[TestFixture]
public class CalculatorTests
{
    private Calculator _calculator;
    
    [SetUp]
    public void Setup()
    {
        _calculator = new Calculator();
    }
    
    [Test]
    public void Add_TwoNumbers_ReturnsSum()
    {
        // Arrange
        int a = 5, b = 3;
        
        // Act
        var result = _calculator.Add(a, b);
        
        // Assert
        Assert.That(result, Is.EqualTo(8));
    }
    
    [TestCase(1, 2, 3)]
    [TestCase(-1, -1, -2)]
    public void Add_MultipleCases_ReturnsCorrectSum(int a, int b, int expected)
    {
        var result = _calculator.Add(a, b);
        Assert.AreEqual(expected, result);
    }
}
```