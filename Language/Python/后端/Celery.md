Celery 是一个分布式任务队列系统，主要用于处理异步任务或定时任务。它通过消息队列来分发任务，并允许多个工作节点并行处理任务，适合需要后台处理耗时操作或需要扩展任务处理能力的应用程序。Celery 通常与消息代理（如 RabbitMQ 或 Redis）配合使用来管理任务队列。

Celery 的主要特点：

1. **异步任务执行**：可以在后台异步执行长时间运行的任务（如发送邮件、处理文件）。
2. **分布式处理**：支持多个工作节点分布式处理任务，便于扩展。
3. **定时任务调度**：可以使用 Celery Beat 来定时执行任务，类似于 cron。
4. **支持多种消息队列**：支持 RabbitMQ、Redis、Amazon SQS 等作为消息队列的后端。
5. **任务重试机制**：如果任务执行失败，可以配置自动重试机制。
6. **任务结果存储**：可以将任务的结果存储在数据库或其他存储后端中，以便稍后获取。

Celery 常用在 Web 框架中，用于处理诸如发送邮件、数据处理、缓存更新等耗时操作。

使用 Celery 的基本流程：
1. 定义一个任务（Python 函数）。
2. 通过 Celery 调用该任务，将其放入消息队列。
3. 工作节点从队列中取出任务并执行。
4. 如果配置了结果后端，执行结果可以被存储并随后获取。


## 使用


#### task

`@celery_app.task()` 是 Celery 中用来定义任务的装饰器，允许你将一个普通的函数转换为一个可以由 Celery worker 异步执行的任务。通过这个装饰器，函数被注册为 Celery 任务，并且可以通过 Celery 的调度、队列、异步调用等机制进行管理和执行。

```python
@celery_app.task(bind=True, max_retries=3, default_retry_delay=60, autoretry_for=(ValueError,), acks_late=True, queue='critical_tasks')
def process_data(self, data):
    try:
        # 执行数据处理
        result = complex_data_processing(data)
    except ValueError as exc:
        # 如果抛出 ValueError，任务将自动重试
        raise self.retry(exc=exc)
    return result
```

**参数详解**：
- **`name`** ：允许你为任务指定一个自定义名称，如果没有提供，Celery 默认使用函数的全名（包含模块路径）。
- **`bind`** ：`bind=True` 会将任务实例（`self`）作为第一个参数传递给任务函数，这样你可以访问任务的上下文信息，包括任务的 ID、请求信息、任务状态等。
- **`max_retries`** ：设置任务的最大重试次数。
- **`default_retry_delay`** ：设置任务重试的默认延迟时间，单位是秒。
- **`ignore_result`** ：`ignore_result=True` 使得任务的执行结果不会存储在后端数据库中。
- **`queue`** ：指定任务要发送到的队列名称。