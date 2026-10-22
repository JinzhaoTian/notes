JSONL（JSON Lines，也常叫 NDJSON，Newline Delimited JSON）是一种按行存储 JSON 数据的文本格式：**每一行都是一个独立、完整的 JSON 值**，通常是一个 JSON 对象，行与行之间用**换行符**分隔。

JSONL：
```jsonl
{"id":1,"name":"Alice","age":30}
{"id":2,"name":"Bob","age":25}
{"id":3,"name":"Carol","age":28}
```

JSON：
```json
[
  {"id":1,"name":"Alice"},
  {"id":2,"name":"Bob"}
]
```

## 特点

1. **每行必须是合法 JSON**，不能一个 JSON 对象跨多行。
2. 通常使用 **UTF-8** 编码。
3. 文件扩展名常见为 `.jsonl` 或 `.ndjson`。
4. 适合**流式处理**：可以逐行读取、逐行写入，不必一次加载整个文件。
5. 常用于**日志、数据导出、机器学习训练数据、大数据处理、命令行工具**等场景。
6. 单行出错时，比较容易定位和跳过；而普通大 JSON 数组某处出错可能影响整体解析。

