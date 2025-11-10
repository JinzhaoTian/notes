可用性（Availability）是指系统在特定时间段内能够正常提供服务的能力，通常用系统正常运行时间与总时间的百分比来表示。

```
可用性 = (系统正常运行时间) / (系统正常运行时间 + 故障时间)
```

## 可用性量化指标

1. **可用性级别**：通常用"几个9"来描述其可用性

| Availability | Downtime/Year | Downtime/Month | Downtime/Week   | Downtime/Day  |
| ------------ | ------------- | -------------- | --------------- | ------------- |
| 99.999%      | 5.256 Minutes | 0.438 Minutes  | 0.101 Minutes   | 0.014 Minutes |
| 99.995%      | 26.28 Minutes | 2.19 Minutes   | 0.505 Minutes   | 0.072 Minutes |
| 99.990%      | 52.56 Minutes | 4.38 Minutes   | 1.011 Minutes   | 0.144 Minutes |
| 99.950%      | 4.38 Hours    | 21.9 Minutes   | 5.054 Minutes   | 0.72 Minutes  |
| 99.900%      | 8.76 Hours    | 43.8 Minutes   | 10.108 Minutes  | 1.44 Minutes  |
| 99.500%      | 43.8 Hours    | 3.65 Hours     | 50.538 Minutes  | 7.2 Minutes   |
| 99.250%      | 65.7 Hours    | 5.475 Hours    | 75.808 Minutes  | 10.8 Minutes  |
| 99.000%      | 87.6 Hours    | 7.3 Hours      | 101.077 Minutes | 14.4 Minutes  |

2. **平均无故障时间**（MTBF - Mean Time Between Failures）
3. **平均修复时间**（MTTR - Mean Time To Repair）


## 提高分布式系统可用性的常见策略

1. **冗余设计**：多副本、多活部署
2. **故障隔离**：避免单点故障
3. **自动故障转移**：快速检测和恢复
4. **优雅降级**：核心功能保持可用
5. **混沌工程**：主动故障测试
6. **监控告警**：快速发现问题