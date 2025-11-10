「日志先行 + 异步处理」的设计模式常见于在金融、电商等**对数据一致性要求高**的场景（如：订单系统、库存系统、支付系统等）

- **可靠性**：MySQL 的持久化能力比 Redis 强，先落盘日志可防止宕机丢失关键操作
	- 写日志虽然也是写磁盘，但是它是**顺序写**，相比随机写开销更小，能提升语句执行的性能。
- **可追溯**：通过日志表可以重建现场，排查问题
- **最终一致性**：即使异步处理失败，也有日志可以触发补偿机制


#### 操作流程

1. 先插入操作日志（MySQL）
```sql
INSERT INTO operation_log 
(job_id, operation_type, biz_id, status, params, create_time)
VALUES 
('123e4567', 'deduct_balance', 1001, 'processing', '{"amount":500}', NOW());
```

2. 扣减 Redis 中的余额（原子操作）
```
DECRBY user:1001:balance 500
```

3. 异步服务消费时：
	- 处理成功：更新日志状态为 success 
	- 处理失败：根据日志记录进行补偿/重试


#### 典型日志表结构

```sql
CREATE TABLE `operation_log` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `job_id` varchar(64) NOT NULL COMMENT '唯一任务ID',
  `operation_type` varchar(32) NOT NULL COMMENT '操作类型',
  `biz_id` bigint NOT NULL COMMENT '业务ID',
  `status` varchar(16) NOT NULL COMMENT 'processing/success/failed',
  `params` json DEFAULT NULL COMMENT '操作参数',
  `retry_count` int DEFAULT '0',
  `create_time` datetime NOT NULL,
  `update_time` datetime NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_job_id` (`job_id`),
  KEY `idx_biz` (`operation_type`,`biz_id`)
) ENGINE=InnoDB;
```

#### **异常处理建议**

- 定时任务扫描 `status='processing'` 且超时的记录
- 对于失败操作，可以根据日志中的原始参数进行重试
- 重要操作建议实现[幂等性](../Backend/API%20Design/幂等性.md)处理