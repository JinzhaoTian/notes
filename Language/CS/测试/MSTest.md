微软官方测试框架，集成在 Visual Studio 中。

**特点**

- 与 Visual Studio 深度集成
- 简单的学习曲线
- 支持数据驱动测试
- 提供代码覆盖率工具

**示例代码**

```c#
[TestClass]
public class CalculatorTests
{
    private Calculator _calculator;
    
    [TestInitialize]
    public void TestInitialize()
    {
        _calculator = new Calculator();
    }
    
    [TestMethod]
    public void Add_TwoNumbers_ReturnsSum()
    {
        // Arrange
        int a = 5, b = 3;
        
        // Act
        var result = _calculator.Add(a, b);
        
        // Assert
        Assert.AreEqual(8, result);
    }
    
    [DataTestMethod]
    [DataRow(1, 2, 3)]
    [DataRow(-1, -1, -2)]
    public void Add_MultipleCases_ReturnsCorrectSum(int a, int b, int expected)
    {
        var result = _calculator.Add(a, b);
        Assert.AreEqual(expected, result);
    }
}
```

