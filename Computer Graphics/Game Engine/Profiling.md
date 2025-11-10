在游戏引擎中，设计一个良好的性能分析工具链可以帮助开发者识别性能瓶颈，优化代码，并确保游戏运行流畅。

## 设计目标

首先，我们需要明确性能分析工具链的设计目标：
1. **低开销**：分析工具本身不应该对游戏性能造成显著影响。
2. **全面性**：能够监控引擎的各个方面，包括 CPU、GPU、内存、网络等。
3. **易用性**：提供清晰的界面和数据可视化，方便开发者理解性能数据。
4. **实时性**：能够实时显示性能数据，以便于即时调整。
5. **可扩展性**：允许根据需要添加新的性能分析模块。

## 架构设计

性能分析工具链可以分为以下几个模块：

1. **数据收集层**：数据收集层负责从引擎的各个子系统收集性能数据。以下是一些关键的数据收集点：
	- **CPU性能**：收集每一帧的时间、函数调用的时间、特定代码块的执行时间等。
	- **GPU性能**：收集 GPU 渲染每一帧的时间、渲染状态变化、Draw Call 数量等。
	- **内存使用**：跟踪内存分配和释放，检测内存泄漏。
	- **网络性能**：监控网络延迟、带宽使用等。
	- **资源管理**：跟踪资源加载和卸载的时间。

2. **数据存储层**：收集到的性能数据需要被存储起来以供分析，可以使用**循环缓冲区**来存储最近几帧的数据，这样既不会占用太多内存，又能保留足够的历史数据。
```cpp
struct ProfileSample {
    std::string name;
    uint64_t startTime;
    uint64_t endTime;
    // 其他元数据，如线程ID等
};

class ProfileDataBuffer {
public:
    void AddSample(const ProfileSample& sample);
    std::vector<ProfileSample> GetSamplesFromLastNFrames(int n) const;
private:
    std::vector<ProfileSample> m_samples;
    int m_currentIndex = 0;
    static const int BUFFER_SIZE = 10000; // 存储最近10000个样本
};
```

3. **数据分析与可视化层**：这一层负责将收集到的数据以可视化的方式呈现给开发者。常见的可视化形式包括：
	- **实时曲线图**：显示帧率、内存使用等随时间变化的曲线。
	- **火焰图**：显示函数调用栈及其耗时。
	- **统计表格**：显示各种性能指标的统计信息，如平均帧时间、最大帧时间等。

可以使用 ImGui 等库来快速构建一个实时性能分析界面。

4. **控制层**：控制层允许开发者动态控制性能分析工具的行为，例如开始/停止记录、设置采样频率、选择要监控的模块等。

## 简单示例

```cpp
// Profiler.h
#include <chrono>
#include <string>
#include <vector>
#include <unordered_map>
#include <thread>
#include <mutex>

struct ProfileSample {
    std::string name;
    uint64_t startTime; // 微秒
    uint64_t endTime;
    std::thread::id threadId;
};

class Profiler {
public:
    static Profiler& GetInstance();

    void BeginSample(const std::string& name);
    void EndSample(const std::string& name);

    // 每帧结束时调用，将当前帧的数据存储到历史数据中，并开始新的一帧
    void NewFrame();

    // 获取最近n帧的数据
    std::vector<ProfileSample> GetRecentSamples(int nFrames) const;

private:
    Profiler() = default;
    ~Profiler() = default;

    std::vector<ProfileSample> m_currentFrameSamples;
    std::vector<std::vector<ProfileSample>> m_history;
    mutable std::mutex m_mutex;
    static const int MAX_HISTORY_FRAMES = 600; // 保存最近600帧（10秒，假设60帧/秒）
};

// 自动作用域的性能采样宏
#define PROFILE_SCOPE(name) ProfilerScope __profile_scope__##__LINE__(name)

class ProfilerScope {
public:
    ProfilerScope(const std::string& name) : m_name(name) {
        Profiler::GetInstance().BeginSample(m_name);
    }
    ~ProfilerScope() {
        Profiler::GetInstance().EndSample(m_name);
    }
private:
    std::string m_name;
};
```

```cpp
// Profiler.cpp
#include "Profiler.h"

Profiler& Profiler::GetInstance() {
    static Profiler instance;
    return instance;
}

void Profiler::BeginSample(const std::string& name) {
    auto now = std::chrono::high_resolution_clock::now();
    auto microseconds = std::chrono::time_point_cast<std::chrono::microseconds>(now).time_since_epoch().count();
    std::lock_guard<std::mutex> lock(m_mutex);
    m_currentFrameSamples.push_back({name, static_cast<uint64_t>(microseconds), 0, std::this_thread::get_id()});
}

void Profiler::EndSample(const std::string& name) {
    auto now = std::chrono::high_resolution_clock::now();
    auto microseconds = std::chrono::time_point_cast<std::chrono::microseconds>(now).time_since_epoch().count();
    std::lock_guard<std::mutex> lock(m_mutex);
    // 查找最近一个具有相同名称且尚未结束的样本（注意：这假设同一线程中同一名称的样本不会嵌套）
    for (auto it = m_currentFrameSamples.rbegin(); it != m_currentFrameSamples.rend(); ++it) {
        if (it->name == name && it->endTime == 0 && it->threadId == std::this_thread::get_id()) {
            it->endTime = static_cast<uint64_t>(microseconds);
            break;
        }
    }
}

void Profiler::NewFrame() {
    std::lock_guard<std::mutex> lock(m_mutex);
    m_history.push_back(std::move(m_currentFrameSamples));
    m_currentFrameSamples.clear();
    if (m_history.size() > MAX_HISTORY_FRAMES) {
        m_history.erase(m_history.begin());
    }
}

std::vector<ProfileSample> Profiler::GetRecentSamples(int nFrames) const {
    std::lock_guard<std::mutex> lock(m_mutex);
    std::vector<ProfileSample> result;
    int startFrame = std::max(0, (int)m_history.size() - nFrames);
    for (int i = startFrame; i < m_history.size(); ++i) {
        result.insert(result.end(), m_history[i].begin(), m_history[i].end());
    }
    return result;
}
```

## 库

1. [gperftools/gperftools](https://github.com/gperftools/gperftools)：Google PerfTools 是一套由 Google 开发的多语言性能分析工具，主要用于 C++ 程序的性能分析和内存管理。
