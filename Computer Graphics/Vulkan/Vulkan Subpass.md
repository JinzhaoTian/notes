Vulkan Subpass 是一种为 Tile-Based GPU（TBDR 架构）设计的优化技术，其核心目标是显著降低内存带宽，尤其是在移动设备上节省功耗和散热。然而，它并非银弹，其优化效果具有不确定性，甚至可能导致时钟周期性能（clock-for-clock performance）下降。现代 Vulkan (1.4+) 正在用更明确的 Dynamic Rendering Local Read（DRLR）方案取代它。



