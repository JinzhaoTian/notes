---
project: JzRE
task: JzRE Profiling
status: To Do
---
在C++游戏引擎中设计性能分析工具链是一个系统工程，需要从数据采集、分析到可视化全链路考虑。以下是一个完整的设计方案：

## 1. 核心架构设计

### 分层架构

```cpp
// 性能分析系统核心接口
class IProfiler {
public:
    virtual ~IProfiler() = default;
    
    // 帧管理
    virtual void BeginFrame() = 0;
    virtual void EndFrame() = 0;
    
    // CPU性能分析
    virtual void BeginCPUProfile(const char* name) = 0;
    virtual void EndCPUProfile(const char* name) = 0;
    
    // GPU性能分析
    virtual void BeginGPUProfile(const char* name) = 0;
    virtual void EndGPUProfile(const char* name) = 0;
    
    // 内存跟踪
    virtual void TrackAllocation(void* ptr, size_t size, const char* category) = 0;
    virtual void TrackDeallocation(void* ptr) = 0;
};
```

## 2. CPU性能分析

### 高精度计时器

```cpp
class HighResolutionTimer {
public:
    HighResolutionTimer() {
        QueryPerformanceFrequency(&frequency_);
    }
    
    void Start() {
        QueryPerformanceCounter(&start_time_);
    }
    
    double Stop() {
        LARGE_INTEGER end_time;
        QueryPerformanceCounter(&end_time);
        return (end_time.QuadPart - start_time_.QuadPart) * 1000.0 / frequency_.QuadPart;
    }
    
private:
    LARGE_INTEGER frequency_;
    LARGE_INTEGER start_time_;
};
```

### 作用域式性能分析

```cpp
class ScopedCPUProfiler {
public:
    ScopedCPUProfiler(const char* name, IProfiler& profiler) 
        : name_(name), profiler_(profiler) {
        profiler_.BeginCPUProfile(name_);
    }
    
    ~ScopedCPUProfiler() {
        profiler_.EndCPUProfile(name_);
    }
    
private:
    const char* name_;
    IProfiler& profiler_;
};

// 使用宏简化调用
#define CPU_PROFILE_SCOPE(name) \
    ScopedCPUProfiler scoped_profiler##__LINE__(name, GetProfiler())
```
### 调用栈采样分析器

```cpp
class CallstackSampler {
public:
    void StartSampling(uint32_t sample_interval_ms = 1) {
        sampling_ = true;
        sampling_thread_ = std::thread(&CallstackSampler::SamplingThread, this, sample_interval_ms);
    }
    
    void StopSampling() {
        sampling_ = false;
        if (sampling_thread_.joinable()) {
            sampling_thread_.join();
        }
    }
    
private:
    void SamplingThread(uint32_t interval_ms) {
        while (sampling_) {
            CaptureCallstack();
            std::this_thread::sleep_for(std::chrono::milliseconds(interval_ms));
        }
    }
    
    void CaptureCallstack() {
        const int max_depth = 64;
        void* callstack[max_depth];
        int frames = CaptureStackBackTrace(0, max_depth, callstack, nullptr);
        
        std::lock_guard<std::mutex> lock(mutex_);
        // 统计调用栈频率
        CallstackHash hash = CalculateCallstackHash(callstack, frames);
        callstack_stats_[hash].count++;
    }
    
    std::atomic<bool> sampling_{false};
    std::thread sampling_thread_;
    std::mutex mutex_;
    std::unordered_map<CallstackHash, CallstackStats> callstack_stats_;
};
```

## 3. GPU性能分析

### GPU时间戳查询

```cpp
class GPUProfiler {
public:
    struct GPUQuery {
        ID3D12QueryHeap* query_heap;
        ID3D12Resource* query_result;
        uint32_t start_index;
        uint32_t end_index;
    };
    
    void BeginFrame(ID3D12GraphicsCommandList* cmd_list) {
        current_frame_index_ = (current_frame_index_ + 1) % kFrameCount;
        
        // 读取上一帧的查询结果
        if (frames_[current_frame_index_].queries_ready) {
            ProcessQueryResults();
        }
        
        // 开始新的查询
        frames_[current_frame_index_].active_query_count = 0;
    }
    
    TimestampHandle BeginProfile(ID3D12GraphicsCommandList* cmd_list, const char* name) {
        auto& frame = frames_[current_frame_index_];
        uint32_t query_index = frame.active_query_count++;
        
        cmd_list->EndQuery(frame.query_heap.Get(), D3D12_QUERY_TYPE_TIMESTAMP, query_index * 2);
        return { current_frame_index_, query_index };
    }
    
    void EndProfile(ID3D12GraphicsCommandList* cmd_list, TimestampHandle handle) {
        auto& frame = frames_[handle.frame_index];
        cmd_list->EndQuery(frame.query_heap.Get(), D3D12_QUERY_TYPE_TIMESTAMP, handle.query_index * 2 + 1);
    }
    
private:
    static constexpr uint32_t kFrameCount = 3;
    static constexpr uint32_t kMaxQueriesPerFrame = 256;
    
    struct FrameData {
        ComPtr<ID3D12QueryHeap> query_heap;
        ComPtr<ID3D12Resource> query_result;
        uint32_t active_query_count = 0;
        bool queries_ready = false;
    };
    
    FrameData frames_[kFrameCount];
    uint32_t current_frame_index_ = 0;
};
```

## 4. 内存分析

### 内存跟踪器

```cpp
class MemoryTracker {
public:
    struct AllocationInfo {
        size_t size;
        const char* category;
        void* callstack[16];
        uint32_t callstack_depth;
        uint64_t timestamp;
    };
    
    void* TrackAllocation(size_t size, const char* category) {
        void* ptr = malloc(size + sizeof(AllocationHeader));
        
        auto& header = *static_cast<AllocationHeader*>(ptr);
        header.magic = kAllocationMagic;
        header.info.size = size;
        header.info.category = category;
        header.info.timestamp = GetHighResolutionTimestamp();
        CaptureCallstack(header.info.callstack, header.info.callstack_depth);
        
        std::lock_guard<std::mutex> lock(mutex_);
        allocations_[GetUserPointer(ptr)] = header.info;
        category_stats_[category].allocated += size;
        category_stats_[category].allocation_count++;
        
        return GetUserPointer(ptr);
    }
    
    void TrackDeallocation(void* ptr) {
        void* real_ptr = GetRealPointer(ptr);
        AllocationHeader* header = static_cast<AllocationHeader*>(real_ptr);
        
        if (header->magic == kAllocationMagic) {
            std::lock_guard<std::mutex> lock(mutex_);
            auto it = allocations_.find(ptr);
            if (it != allocations_.end()) {
                category_stats_[it->second.category].allocated -= it->second.size;
                category_stats_[it->second.category].deallocation_count++;
                allocations_.erase(it);
            }
        }
        
        free(real_ptr);
    }
    
private:
    struct AllocationHeader {
        uint32_t magic;
        AllocationInfo info;
    };
    
    static constexpr uint32_t kAllocationMagic = 0xDEADC0DE;
    
    std::mutex mutex_;
    std::unordered_map<void*, AllocationInfo> allocations_;
    std::unordered_map<const char*, CategoryStats> category_stats_;
};
```

## 5. 数据聚合与分析

### 性能数据分析器

```cpp
class PerformanceAnalyzer {
public:
    struct FrameStats {
        double frame_time;
        double cpu_time;
        double gpu_time;
        std::unordered_map<std::string, double> scope_times;
        size_t memory_usage;
        uint32_t draw_calls;
        uint32_t triangle_count;
    };
    
    void AddFrameData(const FrameStats& stats) {
        std::lock_guard<std::mutex> lock(mutex_);
        
        frame_history_.push_back(stats);
        if (frame_history_.size() > kMaxFrameHistory) {
            frame_history_.pop_front();
        }
        
        // 更新统计信息
        UpdateStatistics(stats);
    }
    
    void DetectBottlenecks() {
        // 分析性能瓶颈
        AnalyzeFrameTimeSpikes();
        AnalyzeMemoryLeaks();
        AnalyzeHotPaths();
    }
    
private:
    static constexpr size_t kMaxFrameHistory = 600; // 10秒@60fps
    
    std::mutex mutex_;
    std::deque<FrameStats> frame_history_;
    StatisticalData frame_time_stats_;
    
    void AnalyzeFrameTimeSpikes() {
        if (frame_history_.size() < 2) return;
        
        const auto& current = frame_history_.back();
        const auto& previous = frame_history_[frame_history_.size() - 2];
        
        double frame_time_increase = current.frame_time - previous.frame_time;
        if (frame_time_increase > frame_time_stats_.average * 2.0) {
            ReportPerformanceSpike(current, frame_time_increase);
        }
    }
};
```

## 6. 可视化系统

### 实时性能HUD

```cpp
class PerformanceHUD {
public:
    void Render() {
        if (!visible_) return;
        
        ImGui::SetNextWindowPos(ImVec2(10, 10), ImGuiCond_FirstUseEver);
        ImGui::SetNextWindowSize(ImVec2(300, 200), ImGuiCond_FirstUseEver);
        
        if (ImGui::Begin("Performance HUD", &visible_)) {
            RenderFrameTimeGraph();
            RenderCPUProfiler();
            RenderMemoryInfo();
            RenderGPUInfo();
        }
        ImGui::End();
    }
    
private:
    void RenderFrameTimeGraph() {
        auto& analyzer = GetPerformanceAnalyzer();
        const auto& history = analyzer.GetFrameHistory();
        
        ImGui::PlotLines("Frame Time (ms)", 
            [](void* data, int idx) { 
                return static_cast<float>(history[idx].frame_time); 
            }, 
            nullptr, history.size(), 0, nullptr, 0.0f, 50.0f, ImVec2(0, 80));
        
        float current_fps = 1000.0f / history.back().frame_time;
        ImGui::Text("FPS: %.1f", current_fps);
        ImGui::Text("Frame: %.2f ms", history.back().frame_time);
    }
    
    void RenderCPUProfiler() {
        if (ImGui::CollapsingHeader("CPU Profiler")) {
            auto& profiler = GetCPUProfiler();
            const auto& scopes = profiler.GetScopeTimings();
            
            for (const auto& [name, timing] : scopes) {
                ImGui::Text("%s: %.3f ms", name.c_str(), timing.average_time);
            }
        }
    }
    
    bool visible_ = true;
};
```

## 7. 工具链集成

### 构建时配置

```cmake
# CMake配置选项
option(ENABLE_PROFILING "Enable performance profiling" ON)
option(ENABLE_MEMORY_TRACKING "Enable memory tracking" ON)
option(ENABLE_GPU_PROFILING "Enable GPU profiling" ON)

if(ENABLE_PROFILING)
    add_definitions(-DPROFILING_ENABLED)
    target_sources(Engine PRIVATE src/Profiling/*.cpp)
endif()
```
### 运行时控制

```cpp
class ProfilingManager {
public:
    void SetProfilingEnabled(bool enabled) {
        cpu_profiler_.SetEnabled(enabled);
        gpu_profiler_.SetEnabled(enabled);
        memory_tracker_.SetEnabled(enabled);
    }
    
    void DumpProfileData(const std::string& filename) {
        // 导出为Chrome Tracing格式
        ExportChromeTracingFormat(filename);
        
        // 导出为自定义分析格式
        ExportCustomFormat(filename + ".bin");
    }
    
    void StartCapture() { /* 开始性能捕获 */ }
    void StopCapture() { /* 停止性能捕获 */ }
    
private:
    CPUProfiler cpu_profiler_;
    GPUProfiler gpu_profiler_;
    MemoryTracker memory_tracker_;
    PerformanceAnalyzer analyzer_;
};
```

这个性能分析工具链提供了从底层数据采集到高层可视化的完整解决方案，可以帮助开发者快速定位和解决性能瓶颈问题。