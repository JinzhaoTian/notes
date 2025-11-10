当前最流行的 C# 测试框架，被 .NET Core 团队选用。

**特点**

- 简洁的语法
- 社区活跃，持续更新
- 没有传统的 `[SetUp]` 和 `[TearDown]`，使用构造函数和 `IDisposable` 替代
- 支持并行测试执行

**示例**

```csharp
public class CalculatorTests
{
    private readonly Calculator _calculator;
    
    public CalculatorTests()
    {
        _calculator = new Calculator();
    }
    
    [Fact]
    public void Add_TwoNumbers_ReturnsSum()
    {
        // Arrange
        int a = 5, b = 3;
        
        // Act
        var result = _calculator.Add(a, b);
        
        // Assert
        Assert.Equal(8, result);
    }
    
    [Theory]
    [InlineData(1, 2, 3)]
    [InlineData(-1, -1, -2)]
    public void Add_MultipleCases_ReturnsCorrectSum(int a, int b, int expected)
    {
        var result = _calculator.Add(a, b);
        Assert.Equal(expected, result);
    }
}
```