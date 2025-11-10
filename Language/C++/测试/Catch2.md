现代、轻量级的 header-only 测试框架。

**特点**

- 单头文件包含，易于集成
- 自然的 BDD 风格语法
- 低学习曲线
- 支持标签过滤测试
- 不需要夹具(fixtures)

**示例代码**

```cpp
#define CATCH_CONFIG_MAIN
#include <catch2/catch.hpp>

TEST_CASE("Vector can be sized and resized", "[vector]") {
    std::vector<int> v(5);
    
    REQUIRE(v.size() == 5);
    REQUIRE(v.capacity() >= 5);
    
    SECTION("resizing bigger changes size") {
        v.resize(10);
        REQUIRE(v.size() == 10);
    }
}
```

