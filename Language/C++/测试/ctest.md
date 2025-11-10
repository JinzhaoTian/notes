CTest 是 CMake 家族中的一个测试工具，它用于管理和运行项目中的测试。

## 工作流程

CTest 的工作流程与 CMake 的构建流程紧密集成：

1. **配置（Configure）**：在你的 `CMakeLists.txt` 文件中，你使用 `add_test()` 命令来定义每一个测试。CMake 不会立即运行它们，只是记录下来。
2. **生成（Generate）**：CMake 会为你的构建系统（如 Makefiles 或 Visual Studio 项目文件）生成包含这些测试定义的脚本或项目。
3. **构建（Build）**：你像往常一样编译你的项目和测试用例。
4. **测试（Test）**：这是 CTest 登场的时候。你不在直接运行每个测试可执行文件，而是运行 `ctest` 命令，CTest 会：
    - 读取 CMake 生成的 `Testfile`。
    - 找到所有通过 `add_test()` 定义的测试。
    - 按照你的指令（例如，运行所有测试、只运行失败的测试、运行匹配某个名称的测试等）来执行它们。
    - 收集每个测试的结果（通过、失败、跳过等）。
    - 生成一个清晰的摘要报告。

```bash
# 1. 常规的 CMake 配置和构建
mkdir build && cd build
cmake ..
cmake --build . 

# 2. 使用 CTest 运行测试
ctest

# 你也可以使用 make test（如果你用的是 Makefile 生成器），
# 但直接使用 ctest 命令能获得更多控制和更详细的输出。
```

运行 `ctest` 后的输出通常类似于：

```
Test project /path/to/build/dir
    Start 1: MyAwesomeTest
1/1 Test #1: MyAwesomeTest ...................   Passed    0.01 sec

100% tests passed, 0 tests failed out of 1
```

## 高级特性

CTest 的强大之处在于它的各种选项和功能：

1. **测试属性**：你可以使用 `set_tests_properties()` 为测试设置各种属性。
    - `WILL_FAIL`：预期测试失败，如果它通过了反而算失败。
    - `PASS_REGULAR_EXPRESSION`：检查测试输出是否匹配某个正则表达式才算通过。
    - `WORKING_DIRECTORY`：在特定目录下运行测试。
    - `ENVIRONMENT`：设置环境变量。
```cmake

add_test(NAME MyTest COMMAND my_test)
set_tests_properties(MyTest PROPERTIES
	PASS_REGULAR_EXPRESSION "All tests passed"
	ENVIRONMENT "TEST_DATA_DIR=/path/to/data"
)
```

2. **测试分组（Fixtures）**：你可以设置测试依赖关系，例如定义 `SETUP`、`TEST` 和 `CLEANUP` 测试，CTest 可以确保它们按顺序运行。

3. **强大的命令行控制**：
    - `-R <regex>`：运行名称匹配正则表达式的测试。
    - `-E <regex>`：排除名称匹配正则表达式的测试。
    - `-L <regex>`：运行标签（Label）匹配正则表达式的测试。
    - `-j <num>`：并行运行测试（`ctest -j 4` 使用4个线程）。
    - `--output-on-failure`：如果测试失败，则打印其输出内容，非常有用！
    - `-T memcheck`：与 Valgrind 等内存检查工具集成，运行内存检测。

4. **与 CDash 集成**：CDash 是一个用于聚合测试结果的web仪表板，可以使用 `ctest -D Experimental` 将本地测试结果提交到 CDash 服务器，方便团队查看历史记录和趋势，这是**持续集成**的核心。

5. **测试标签（Labels）**：你可以给测试打上标签（如“单元测试”、“集成测试”、“长时间运行”），然后选择性地运行某一类测试。
```cmake

set_tests_properties(MyTest PROPERTIES LABELS "unit;fast")
```
然后使用 `ctest -L fast` 来运行所有带有 “fast” 标签的测试。



## 使用

