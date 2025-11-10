Google PerfTools 是一套由 Google 开发的一套高性能性能分析工具集，主要用于 C++ 程序的性能分析和内存管理，项目地址：[gperftools/gperftools](https://github.com/gperftools/gperftools)。

它包含多个组件，其中最著名的是 CPU 分析器（CPU Profiler）和堆分析器（Heap Profiler）。此外，还包括堆检查器（Heap Checker）和内存检查器（Memory Checker）等。

## 核心组件

1. **CPU Profiler**：基于采样的性能分析
```cpp
#include <gperftools/profiler.h>

int main() {
    ProfilerStart("my_profile.prof");
    
    // 你的业务代码
    RunApplication();
    
    ProfilerStop();
    return 0;
}

// 或者使用环境变量控制
void EnableProfiling() {
    if (getenv("CPUPROFILE")) {
        ProfilerStart(getenv("CPUPROFILE"));
    }
}
```

2. **Heap Profiler**：内存分析功能
```cpp
#include <gperftools/heap-profiler.h>

int main() {
    HeapProfilerStart("heap_profile");
    
    // 内存操作
    char* buffer = new char[1024 * 1024];  // 1MB 分配
    
    // 生成堆快照
    HeapProfilerDump("initial_allocation");
    
    delete[] buffer;
    HeapProfilerStop();
}

// 环境变量方式
// HEAPPROFILE=heap_output HEAP_PROFILE_ALLOCATION_INTERVAL=104857600 ./program
```

3. **Heap Checker**：内存泄漏检测
```cpp
#include <gperftools/heap-checker.h>

int main() {
    HeapLeakChecker checker("main_function_check");
    
    {
        // 检查这个作用域内的内存泄漏
        HeapLeakChecker::Scope scope(checker);
        
        char* leaky_memory = new char[100];  // 这将被检测为泄漏
        
        // 如果正确释放，则不会报泄漏
        // delete[] leaky_memory;
    }
    
    if (!checker.NoLeaks()) {
        printf("内存泄漏 detected!\n");
    }
}
```